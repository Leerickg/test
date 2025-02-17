
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Plateforme de Jeux</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Firebase SDK -->
    <script src="https://www.gstatic.com/firebasejs/9.22.1/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.22.1/firebase-auth-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.22.1/firebase-database-compat.js"></script>
</head>
<body class="bg-blue-50 min-h-screen">
    <div id="app" class="container mx-auto"></div>

    <script>
        // Configuration Firebase 
        const firebaseConfig = {
            apiKey: "VOTRE_API_KEY",
            authDomain: "VOTRE_DOMAINE_AUTH",
            databaseURL: "VOTRE_URL_BASE_DE_DONNEES",
            projectId: "VOTRE_ID_PROJET"
        };

        // Initialisation Firebase
        firebase.initializeApp(firebaseConfig);
        const auth = firebase.auth();
        const database = firebase.database();

        // État global des jeux
        let gameState = {
            currentGame: null,
            currentView: 'login',
            platform: {
                games: [
                    { name: 'Bowling', icon: '🎳', type: 'bowling' },
                    { name: 'Chess', icon: '♟️', type: 'chess' },
                    { name: 'Poker', icon: '🃏', type: 'poker' }
                ],
                onlineMode: true
            }
        };

        // Gestionnaire d'utilisateurs
        const UserManager = {
            register(email, password) {
                return auth.createUserWithEmailAndPassword(email, password)
                    .then((userCredential) => {
                        const user = userCredential.user;
                        return database.ref('users/' + user.uid).set({
                            email: email,
                            points: 0,
                            games: {}
                        });
                    })
                    .catch((error) => {
                        throw error;
                    });
            },

            login(email, password) {
                return auth.signInWithEmailAndPassword(email, password);
            },

            logout() {
                return auth.signOut();
            },

            getCurrentUser() {
                return auth.currentUser;
            },

            saveGameProgress(gameType, gameData) {
                const user = this.getCurrentUser();
                if (user) {
                    return database.ref(`users/${user.uid}/games/${gameType}`).set(gameData);
                }
            },

            loadGameProgress(gameType) {
                const user = this.getCurrentUser();
                if (user) {
                    return database.ref(`users/${user.uid}/games/${gameType}`).once('value');
                }
            },

            updatePoints(points) {
                const user = this.getCurrentUser();
                if (user) {
                    return database.ref(`users/${user.uid}/points`).transaction((currentPoints) => {
                        return (currentPoints || 0) + points;
                    });
                }
            },

            convertPointsToEuros() {
                const user = this.getCurrentUser();
                if (user) {
                    return database.ref(`users/${user.uid}/points`).transaction((currentPoints) => {
                        const POINTS_PER_EURO = 500;
                        if (currentPoints >= POINTS_PER_EURO) {
                            const euros = Math.floor(currentPoints / POINTS_PER_EURO);
                            const remainingPoints = currentPoints % POINTS_PER_EURO;
                            alert(`Conversion : ${euros} euros transférés. Points restants : ${remainingPoints}`);
                            return remainingPoints;
                        }
                        return currentPoints;
                    });
                }
            }
        };

        // Stratégies du robot pour le bowling
        const BowlingRobotStrategies = {
            moyen: (remainingBalls) => {
                return Math.min(Math.floor(Math.random() * 3) + 1, remainingBalls);
            }
        };

        // Fonctions de navigation et de rendu [Le reste du code précédent reste identique]
        // ... [Conserver les fonctions renderLoginForm, renderRegisterForm, etc.]

        // Modification des fonctions de gestion de connexion
        function handleLogin() {
            const email = document.getElementById('loginEmail').value;
            const password = document.getElementById('loginPassword').value;
            
            UserManager.login(email, password)
                .then(() => {
                    navigateTo('menu');
                })
                .catch((error) => {
                    alert(error.message);
                });
        }

        function handleRegister() {
            const email = document.getElementById('registerEmail').value;
            const password = document.getElementById('registerPassword').value;
            
            UserManager.register(email, password)
                .then(() => {
                    alert("Inscription réussie");
                    navigateTo('login');
                })
                .catch((error) => {
                    alert(error.message);
                });
        }

        // Écouteur d'authentification
        auth.onAuthStateChanged((user) => {
            if (user) {
                navigateTo('menu');
            } else {
                navigateTo('login');
            }
        });

        // Initialisation
        renderApp();
    </script>
</body>
</html>
