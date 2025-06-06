<WIllkommen>
<html>
<head>
    <title>Parkour Runner - Fix</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            background-color: #87CEEB; /* Himmelblau */
            font-family: Arial, sans-serif;
            text-align: center;
            user-select: none;
        }
        #game-container {
            position: relative;
            width: 600px;
            height: 200px;
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
            background-color: #2E8B57; /* Grün */
        }
        #player {
            position: absolute;
            width: 30px;
            height: 50px;
            background-color: #FF6347; /* Tomatenrot */
            bottom: 20px; /* Auf dem Boden */
            left: 50px;
            border-radius: 5px 5px 0 0;
            transition: bottom 0.1s linear;
        }
        .obstacle {
            position: absolute;
            width: 20px;
            height: 40px;
            background-color: #4682B4; /* Stahlblau */
            bottom: 20px;
            right: -20px;
            border-radius: 5px 5px 0 0;
        }
        #score-display {
            font-size: 24px;
            margin: 10px;
            color: #333;
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
        button:hover {
            background-color: #45a049;
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
        }
    </style>
</head>
<body>
    <h1>Parkour Runner</h1>
    <div id="score-display">Punkte: 0</div>
    <div id="controls">
        <button id="start-btn">Start</button>
        <button id="restart-btn" style="display:none;">Neustart</button>
    </div>
    <div id="game-container">
        <div id="ground"></div>
        <div id="player"></div>
        <div id="game-over">
            <h2>Game Over!</h2>
            <p id="final-score">Punkte: 0</p>
            <button id="restart-game-btn">Nochmal spielen</button>
        </div>
    </div>

    <script>
        // Game Elements
        const gameContainer = document.getElementById('game-container');
        const player = document.getElementById('player');
        const scoreDisplay = document.getElementById('score-display');
        const startBtn = document.getElementById('start-btn');
        const restartBtn = document.getElementById('restart-btn');
        const gameOverScreen = document.getElementById('game-over');
        const finalScore = document.getElementById('final-score');
        const restartGameBtn = document.getElementById('restart-game-btn');

        // Game Variables
        let score = 0;
        let isJumping = false;
        let isGameRunning = false;
        let gameSpeed = 5;
        let gravity;
        let obstacleInterval;
        let animationFrame;
        let obstacles = [];

        // Player Properties
        const playerProps = {
            width: 30,
            height: 50,
            x: 50,
            y: gameContainer.clientHeight - 70, // 70 = player height (50) + ground height (20)
            jumpHeight: 100,
            jumpSpeed: 5,
            isJumping: false
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
            restartBtn.style.display = 'inline-block';
            
            // Clear existing obstacles
            document.querySelectorAll('.obstacle').forEach(obs => obs.remove());
            obstacles = [];
            
            // Start game loops
            obstacleInterval = setInterval(createObstacle, 1500);
            gravity = setInterval(applyGravity, 20);
            animationFrame = requestAnimationFrame(updateGame);
        }

        function createObstacle() {
            if (!isGameRunning) return;
            
            const obstacle = document.createElement('div');
            obstacle.className = 'obstacle';
            obstacle.style.right = '-20px';
            gameContainer.appendChild(obstacle);
            
            const obstacleObj = {
                element: obstacle,
                x: gameContainer.clientWidth,
                width: 20,
                height: 40,
                speed: gameSpeed
            };
            
            obstacles.push(obstacleObj);
        }

        function applyGravity() {
            if (!isGameRunning) return;
            
            const currentBottom = parseInt(player.style.bottom || '20');
            const groundLevel = 20;
            
            if (!playerProps.isJumping && currentBottom > groundLevel) {
                player.style.bottom = (currentBottom - 5) + 'px';
            } else if (currentBottom < groundLevel) {
                player.style.bottom = groundLevel + 'px';
            }
        }

        function jump() {
            if (playerProps.isJumping || !isGameRunning) return;
            
            playerProps.isJumping = true;
            let jumpCount = 0;
            const maxJump = playerProps.jumpHeight;
            
            const jumpInterval = setInterval(() => {
                jumpCount += playerProps.jumpSpeed;
                player.style.bottom = (20 + jumpCount) + 'px';
                
                if (jumpCount >= maxJump) {
                    clearInterval(jumpInterval);
                    playerProps.isJumping = false;
                }
            }, 20);
        }

        function updateGame() {
            if (!isGameRunning) return;
            
            // Move obstacles
            obstacles.forEach((obstacle, index) => {
                obstacle.x -= obstacle.speed;
                obstacle.element.style.right = (gameContainer.clientWidth - obstacle.x) + 'px';
                
                // Remove obstacles that are off screen
                if (obstacle.x + obstacle.width < 0) {
                    gameContainer.removeChild(obstacle.element);
                    obstacles.splice(index, 1);
                    score++;
                    scoreDisplay.textContent = 'Punkte: ' + score;
                    
                    // Increase difficulty every 5 points
                    if (score % 5 === 0) {
                        gameSpeed += 0.5;
                    }
                }
                
                // Check collision
                if (checkCollision(player, obstacle)) {
                    endGame();
                }
            });
            
            animationFrame = requestAnimationFrame(updateGame);
        }

        function checkCollision(player, obstacle) {
            const playerRect = {
                x: playerProps.x,
                y: gameContainer.clientHeight - (parseInt(player.style.bottom) + playerProps.height),
                width: playerProps.width,
                height: playerProps.height
            };
            
            const obstacleRect = {
                x: obstacle.x,
                y: gameContainer.clientHeight - (20 + obstacle.height),
                width: obstacle.width,
                height: obstacle.height
            };
            
            return (
                playerRect.x < obstacleRect.x + obstacleRect.width &&
                playerRect.x + playerRect.width > obstacleRect.x &&
                playerRect.y < obstacleRect.y + obstacleRect.height &&
                playerRect.y + playerRect.height > obstacleRect.y
            );
        }

        function endGame() {
            isGameRunning = false;
            
            clearInterval(obstacleInterval);
            clearInterval(gravity);
            cancelAnimationFrame(animationFrame);
            
            finalScore.textContent = 'Punkte: ' + score;
            gameOverScreen.style.display = 'block';
        }

        // Event Listeners
        document.addEventListener('keydown', (e) => {
            if (e.code === 'Space' || e.key === 'ArrowUp') {
                e.preventDefault();
                jump();
            }
        });

        startBtn.addEventListener('click', startGame);
        restartBtn.addEventListener('click', startGame);
        restartGameBtn.addEventListener('click', startGame);
    </script>
</body>
</html>
