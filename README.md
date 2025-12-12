<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Cookie Clicker Online</title>
    <style>
        body { text-align: center; font-family: Arial, sans-serif; background-color: #f0f0f0; margin: 0; padding: 0; }
        h1 { margin-top: 30px; }
        #cookie { font-size: 100px; cursor: pointer; margin-top: 20px; user-select: none; }
        #count { font-size: 36px; margin-top: 20px; }
        .upgrade { margin: 10px auto; padding: 10px 20px; font-size: 20px; cursor: pointer; display: block; width: 220px; }
        #upgrades { margin-top: 30px; }
        #leaderboard { margin-top: 30px; font-size: 18px; }
        #footer { margin-top: 30px; font-size: 14px; color: #555; }
    </style>
    <!-- Firebase scripts -->
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>
</head>
<body>
    <h1>🍪 Cookie Clicker Online 🍪</h1>
    <div id="count">0 Cookies</div>
    <button id="cookie">🍪</button>

    <div id="upgrades">
        <button class="upgrade" id="autoClicker">Buy Auto-Clicker (50 Cookies)</button>
        <button class="upgrade" id="doubleClick">Double Cookies Per Click (100 Cookies)</button>
    </div>

    <div id="leaderboard">
        <h2>Leaderboard (Top 5)</h2>
        <ol id="leaderboardList"></ol>
    </div>

    <div id="footer">Progressie wordt automatisch opgeslagen!</div>

    <script>
        // Vraag spelernaam
        let playerName = prompt("Wat is je naam?");
        if(!playerName) playerName = "Speler";

        // Firebase config (vul hier je eigen gegevens in)
        const firebaseConfig = {
          apiKey: "JOUW_API_KEY",
          authDomain: "JOUW_PROJECT_ID.firebaseapp.com",
          databaseURL: "https://JOUW_PROJECT_ID-default-rtdb.firebaseio.com",
          projectId: "JOUW_PROJECT_ID",
          storageBucket: "JOUW_PROJECT_ID.appspot.com",
          messagingSenderId: "JOUW_SENDER_ID",
          appId: "JOUW_APP_ID"
        };
        firebase.initializeApp(firebaseConfig);
        const database = firebase.database();

        // Basiswaarden
        let cookies = 0;
        let cookiesPerClick = 1;
        let autoClickers = 0;

        const countDisplay = document.getElementById("count");
        const cookieBtn = document.getElementById("cookie");
        const autoClickerBtn = document.getElementById("autoClicker");
        const doubleClickBtn = document.getElementById("doubleClick");
        const leaderboardList = document.getElementById("leaderboardList");

        // Klik event
        cookieBtn.addEventListener("click", () => {
            cookies += cookiesPerClick;
            updateDisplay();
            saveProgress();
        });

        // Auto-clicker kopen
        autoClickerBtn.addEventListener("click", () => {
            let price = 50 + autoClickers*25;
            if(cookies >= price){
                cookies -= price;
                autoClickers++;
                updateDisplay();
                saveProgress();
            }
        });

        // Double click kopen
        doubleClickBtn.addEventListener("click", () => {
            let price = 100 + cookiesPerClick*50;
            if(cookies >= price){
                cookies -= price;
                cookiesPerClick *= 2;
                updateDisplay();
                saveProgress();
            }
        });

        // Auto-clicker interval
        setInterval(() => {
            cookies += autoClickers;
            updateDisplay();
            saveProgress();
        }, 1000);

        // Update display
        function updateDisplay(){
            countDisplay.textContent = cookies + " Cookies";
            autoClickerBtn.textContent = `Buy Auto-Clicker (${50 + autoClickers*25} Cookies)`;
            doubleClickBtn.textContent = `Double Cookies Per Click (${100 + cookiesPerClick*50} Cookies)`;
            updateLeaderboard();
        }

        // Opslaan lokaal en naar Firebase
        function saveProgress(){
            localStorage.setItem("cookies", cookies);
            localStorage.setItem("cpc", cookiesPerClick);
            localStorage.setItem("autoClickers", autoClickers);
            saveScoreFirebase();
        }

        // Firebase leaderboard update
        function saveScoreFirebase(){
            database.ref('leaderboard/' + playerName).set({
                name: playerName,
                score: cookies
            });
        }

        // Firebase leaderboard display
        function updateLeaderboard(){
            database.ref('leaderboard').orderByChild('score').limitToLast(5).on('value', snapshot => {
                leaderboardList.innerHTML = "";
                const data = snapshot.val();
                if(data){
                    const sorted = Object.values(data).sort((a,b) => b.score - a.score);
                    sorted.forEach(player => {
                        const li = document.createElement("li");
                        li.textContent = player.name + ": " + player.score + " Cookies";
                        leaderboardList.appendChild(li);
                    });
                }
            });
        }

        // Laad lokaal opgeslagen waarden
        if(localStorage.getItem("cookies")){
            cookies = parseInt(localStorage.getItem("cookies"));
        }
        if(localStorage.getItem("cpc")){
            cookiesPerClick = parseInt(localStorage.getItem("cpc"));
        }
        if(localStorage.getItem("autoClickers")){
            autoClickers = parseInt(localStorage.getItem("autoClickers"));
        }

        // Initial display
        updateDisplay();
    </script>
</body>
</html>
