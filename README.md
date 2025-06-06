<Willkommen Zum pakour spiel>
<html>
<head>
    <title>Parkour Runner</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            background-color: #f0f0f0;
            font-family: Arial, sans-serif;
            text-align: center;
        }
        #game {
            position: relative;
            width: 600px;
            height: 200px;
            margin: 50px auto;
            background-color: white;
            border: 2px solid #333;
            overflow: hidden;
        }
        #player {
            position: absolute;
            width: 30px;
            height: 50px;
            background-color: #4CAF50;
            bottom: 0;
            left: 50px;
        }
        .obstacle {
            position: absolute;
            width: 20px;
            height: 40px;
            background-color: #FF5722;
            bottom: 0;
            right: 0;
        }
        #score {
            font-size: 24px;
            margin-top: 20px;
        }
        #start {
            padding: 10px 20px;
            font-size: 18px;
            background-color: #2196F3;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
        #game-over {
            display: none;
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background-color: rgba(0, 0, 0, 0.8);
            color: white;
            padding: 20px;
            border-radius: 10px;
        }
    </style>
</head>
<body>
    <h1>Parkour Runner</h1>
    <p>Drücke <strong>Leertaste</strong>, um zu springen und Hindernisse zu vermeiden!</p>
    <button id="start">Spiel starten</button>
    <div id="score">Punkte: 0</div>
    <div id="game">
        <div id="player"></div>
        <div id="game-over">
            <h2>Game Over!</h2>
            <p id="final-score">Punkte: 0</p>
            <button id="restart">Nochmal spielen</button>
        </div>
    </div>

    <script>
        const player = document.getElementById("player");
        const game = document.getElementById("game");
        const scoreDisplay = document.getElementById("score");
        const startButton = document.getElementById("start");
        const gameOverScreen = document.getElementById("game-over");
        const finalScore = document.getElementById("final-score");
        const restartButton = document.getElementById("restart");

        let score = 0;
        let isJumping = false;
        let isGameRunning = false;
        let gameSpeed = 5;
        let obstacleInterval;
        let gravity;
        let gameLoop;

        function startGame() {
            if (isGameRunning) return;
            
            isGameRunning = true;
            score = 0;
            scoreDisplay.textContent = "Punkte: 0";
            gameOverScreen.style.display = "none";
            game.innerHTML = '<div id="player"></div><div id="game-over">...</div>';
            player.style.bottom = "0px";
            
            // Hindernis-Spawn
            obstacleInterval = setInterval(createObstacle, 2000);
            
            // Schwerkraft
            gravity = setInterval(() => {
                if (isJumping) return;
                const currentBottom = parseInt(player.style.bottom || "0");
                if (currentBottom <= 0) {
                    player.style.bottom = "0px";
                    clearInterval(gravity);
                } else {
                    player.style.bottom = (currentBottom - 5) + "px";
                }
            }, 50);
            
            // Spiel-Loop (Kollisionserkennung & Bewegung)
            gameLoop = setInterval(updateGame, 20);
        }

        function createObstacle() {
            if (!isGameRunning) return;
            
            const obstacle = document.createElement("div");
            obstacle.className = "obstacle";
            obstacle.style.right = "0px";
            game.appendChild(obstacle);
            
            let obstaclePos = 0;
            const moveObstacle = setInterval(() => {
                obstaclePos += gameSpeed;
                obstacle.style.right = obstaclePos + "px";
                
                // Hindernis entfernen, wenn es außerhalb des Bildschirms ist
                if (obstaclePos > game.clientWidth) {
                    clearInterval(moveObstacle);
                    game.removeChild(obstacle);
                    score++;
                    scoreDisplay.textContent = "Punkte: " + score;
                    
                    // Schwierigkeit erhöhen
                    if (score % 5 === 0) {
                        gameSpeed += 1;
                    }
                }
            }, 20);
        }

        function jump() {
            if (isJumping || !isGameRunning) return;
            
            isJumping = true;
            let jumpHeight = 0;
            const jumpUp = setInterval(() => {
                jumpHeight += 5;
                player.style.bottom = jumpHeight + "px";
                
                if (jumpHeight >= 100) {
                    clearInterval(jumpUp);
                    const fallDown = setInterval(() => {
                        jumpHeight -= 5;
                        player.style.bottom = jumpHeight + "px";
                        
                        if (jumpHeight <= 0) {
                            clearInterval(fallDown);
                            player.style.bottom = "0px";
                            isJumping = false;
                        }
                    }, 20);
                }
            }, 20);
        }

        function updateGame() {
            const playerRect = player.getBoundingClientRect();
            const obstacles = document.querySelectorAll(".obstacle");
            
            obstacles.forEach(obstacle => {
                const obstacleRect = obstacle.getBoundingClientRect();
                
                // Kollisionserkennung
                if (
                    playerRect.right > obstacleRect.left &&
                    playerRect.left < obstacleRect.right &&
                    playerRect.bottom > obstacleRect.top
                ) {
                    endGame();
                }
            });
        }

        function endGame() {
            isGameRunning = false;
            clearInterval(obstacleInterval);
            clearInterval(gravity);
            clearInterval(gameLoop);
            
            finalScore.textContent = "Punkte: " + score;
            gameOverScreen.style.display = "block";
        }

        // Steuerung
        document.addEventListener("keydown", (e) => {
            if (e.code === "Space") {
                e.preventDefault();
                jump();
            }
        });

        startButton.addEventListener("click", startGame);
        restartButton.addEventListener("click", startGame);
    </script>
</body>
</html># Pakour
