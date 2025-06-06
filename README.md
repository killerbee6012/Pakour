<Willkommen>
<html>
<head>
    <title>Parkour Runner - Slide & RGB</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            font-family: Arial, sans-serif;
            text-align: center;
            user-select: none;
            animation: rgbBackground 10s infinite alternate;
        }
        @keyframes rgbBackground {
            0% { background-color: #FF4136; }
            33% { background-color: #0074D9; }
            66% { background-color: #2ECC40; }
            100% { background-color: #FFDC00; }
        }
        #game-container {
            position: relative;
            width: 600px;
            height: 250px; /* Höher für Slide-Mechanik */
            margin: 20px auto;
            background-color: #333;
            overflow: hidden;
            border-radius: 10px;
            box-shadow: 0 0 20px rgba(0,0,0,0.5);
        }
        #ground {
            position: absolute;
            bottom: 0;
            width: 100%;
            height: 20px;
            background-color: #2E8B57;
        }
        #player {
            position: absolute;
            width: 30px;
            height: 50px;
            background-color: #FF6347;
            bottom: 20px;
            left: 50px;
            border-radius: 5px 5px 0 0;
            transition: height 0.2s;
        }
        .obstacle {
            position: absolute;
            width: 20px;
            height: 40px;
            background-color: #4682B4;
            bottom: 20px;
            right: -20px;
            border-radius: 5px 5px 0 0;
        }
        .high-obstacle {
            height: 80px; /* Höhere Hindernisse für Slide-Mechanik */
        }
        #score-display {
            font-size: 24px;
            margin: 10px;
            color: white;
            text-shadow: 1px 1px 2px black;
        }
        #controls {
            margin: 20px;
        }
        button {
            padding: 10px 20px;
            font-size: 16px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            margin: 0 10px;
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
            text-align: center;
            width: 80%;
            z-index: 10;
        }
        #instructions {
            margin: 10px;
            color: white;
            text-shadow: 1px 1px 2px black;
        }
    </style>
</head>
<body>
    <h1 style="color: white; text-shadow: 1px 1px 2px black;">Parkour Runner</h1>
    <div id="instructions">
        Leertaste = Springen | Shift = Slide unter Hindernissen
    </div>
    <div id="score-display">Punkte: 0</div>
    <div id="controls">
        <button id="start-btn">Start</button>
    </div>
    <div id="game-container">
        <div id="ground"></div>
        <div id="player"></div>
        <div id="game-over">
            <h2>Game Over!</h2>
            <p id="final-score">Punkte: 0</p>
            <button id="restart-btn">Nochmal spielen</button>
        </div>
    </div>

    <script>
        // Game Elements
        const gameContainer = document.getElementById('game-container');
        const player = document.getElementById('player');
        const scoreDisplay = document.getElementById('score-display');
        const startBtn = document.getElementById('start-btn');
        const gameOverScreen = document.getElementById('game-over');
        const finalScore = document.getElementById('final-score');
        const restartBtn = document.getElementById('restart-btn');

        // Game Variables
        let score = 0;
        let isGameRunning = false;
        let gameSpeed = 5;
        let obstacleInterval;
        let gameLoop;
        let obstacles = [];
        let isSliding = false;
        let slideTimeout;

        // Player Properties
        const playerProps = {
            x: 50,
            y: gameContainer.clientHeight - 70,
            width: 30,
            height: 50,
            isJumping: false,
            jumpPower: 0,
            gravity: 0.5,
            normalHeight: 50,
            slidingHeight: 25
        };

        // Initialize player position
        player.style.left = playerProps.x + 'px';
        player.style.bottom = '20px';

        // Game Functions
        function startGame() {
            if (isGameRunning) return;
            
            // Reset game state
            isGameRunning = true;
            score = 0;
            gameSpeed = 5;
            scoreDisplay.textContent = 'Punkte: 0';
            gameOverScreen.style.display = 'none';
            startBtn.style.display = 'none';
            
            // Clear existing obstacles
            document.querySelectorAll('.obstacle').forEach(obs => obs.remove());
            obstacles = [];
            
            // Reset player position and state
            playerProps.y = gameContainer.clientHeight - 70;
            playerProps.isJumping = false;
            playerProps.jumpPower = 0;
            player.style.bottom = '20px';
            player.style.height = playerProps.normalHeight + 'px';
            isSliding = false;
            
            // Start game loops
            obstacleInterval = setInterval(createObstacle, 1500);
            gameLoop = setInterval(updateGame, 20);
        }

        function createObstacle() {
            if (!isGameRunning) return;
            
            const obstacle = document.createElement('div');
            obstacle.className = 'obstacle';
            
            // 30% Chance für ein hohes Hindernis (nur durch Sliden vermeidbar)
            if (Math.random() < 0.3) {
                obstacle.classList.add('high-obstacle');
            }
            
            obstacle.style.right = '-20px';
            gameContainer.appendChild(obstacle);
            
            obstacles.push({
                element: obstacle,
                x: gameContainer.clientWidth,
                width: 20,
                height: obstacle.classList.contains('high-obstacle') ? 80 : 40,
                speed: gameSpeed,
                isHigh: obstacle.classList.contains('high-obstacle')
            });
        }

        function jump() {
            if (playerProps.isJumping || !isGameRunning || isSliding) return;
            
            playerProps.isJumping = true;
            playerProps.jumpPower = 12;
            endSlide(); // Falls während des Sprungs geslidet wird
        }

        function startSlide() {
            if (playerProps.isJumping || !isGameRunning || isSliding) return;
            
            isSliding = true;
            player.style.height = playerProps.slidingHeight + 'px';
            player.style.bottom = (20 - (playerProps.normalHeight - playerProps.slidingHeight)) + 'px';
            
            // Automatisches Aufstehen nach 1 Sekunde
            slideTimeout = setTimeout(endSlide, 1000);
        }

        function endSlide() {
            if (!isSliding) return;
            
            clearTimeout(slideTimeout);
            isSliding = false;
            player.style.height = playerProps.normalHeight + 'px';
            player.style.bottom = '20px';
        }

        function updatePlayer() {
            // Apply gravity
            playerProps.y -= playerProps.jumpPower;
            playerProps.jumpPower -= playerProps.gravity;
            
            // Ground collision
            if (playerProps.y >= gameContainer.clientHeight - 70) {
                playerProps.y = gameContainer.clientHeight - 70;
                playerProps.isJumping = false;
                playerProps.jumpPower = 0;
            }
            
            // Update player position
            player.style.bottom = (gameContainer.clientHeight - playerProps.y - (isSliding ? playerProps.slidingHeight : playerProps.normalHeight)) + 'px';
        }

        function updateObstacles() {
            for (let i = obstacles.length - 1; i >= 0; i--) {
                const obstacle = obstacles[i];
                obstacle.x -= obstacle.speed;
                obstacle.element.style.right = (gameContainer.clientWidth - obstacle.x) + 'px';
                
                // Remove obstacles that are off screen
                if (obstacle.x + obstacle.width < 0) {
                    gameContainer.removeChild(obstacle.element);
                    obstacles.splice(i, 1);
                    score++;
                    scoreDisplay.textContent = 'Punkte: ' + score;
                    
                    // Increase difficulty every 5 points
                    if (score % 5 === 0) {
                        gameSpeed += 0.5;
                    }
                }
                
                // Check collision
                if (checkCollision(obstacle)) {
                    endGame();
                    return;
                }
            }
        }

        function checkCollision(obstacle) {
            const playerLeft = playerProps.x;
            const playerRight = playerProps.x + playerProps.width;
            const playerTop = playerProps.y;
            const playerBottom = playerProps.y + (isSliding ? playerProps.slidingHeight : playerProps.normalHeight);
            
            const obstacleLeft = obstacle.x;
            const obstacleRight = obstacle.x + obstacle.width;
            const obstacleTop = gameContainer.clientHeight - obstacle.height - 20;
            const obstacleBottom = gameContainer.clientHeight - 20;
            
            // Kollision nur wenn nicht geslidet wird oder Hindernis zu hoch ist
            return (
                playerRight > obstacleLeft &&
                playerLeft < obstacleRight &&
                playerBottom > obstacleTop &&
                playerTop < obstacleBottom &&
                !(isSliding && !obstacle.isHigh) // Keine Kollision wenn geslidet wird und Hindernis nicht hoch ist
            );
        }

        function updateGame() {
            if (!isGameRunning) return;
            
            updatePlayer();
            updateObstacles();
        }

        function endGame() {
            isGameRunning = false;
            clearInterval(obstacleInterval);
            clearInterval(gameLoop);
            
            finalScore.textContent = 'Punkte: ' + score;
            gameOverScreen.style.display = 'block';
            startBtn.style.display = 'inline-block';
        }

        // Event Listeners
        document.addEventListener('keydown', (e) => {
            if (!isGameRunning) return;
            
            if ((e.code === 'Space' || e.key === 'ArrowUp') && !isSliding) {
                e.preventDefault();
                jump();
            } else if (e.key === 'Shift') {
                e.preventDefault();
                startSlide();
            }
        });

        document.addEventListener('keyup', (e) => {
            if (e.key === 'Shift' && isSliding) {
                endSlide();
            }
        });

        startBtn.addEventListener('click', startGame);
        restartBtn.addEventListener('click', startGame);
    </script>
</body>
</html>
