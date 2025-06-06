<Willkommen>
<html>
<head>
    <title>Parkour Runner - Fix</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            background-color: #87CEEB;
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
        let gravity;
        let obstacleInterval;
        let gameLoop;
        let obstacles = [];

        // Player Properties
        const playerProps = {
            x: 50,
            y: gameContainer.clientHeight - 70, // 70 = player height (50) + ground height (20)
            width: 30,
            height: 50,
            isJumping: false,
            jumpPower: 0,
            gravity: 0.5
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
            
            // Reset player position
            playerProps.y = gameContainer.clientHeight - 70;
            playerProps.isJumping = false;
            playerProps.jumpPower = 0;
            player.style.bottom = '20px';
            
            // Start game loops
            obstacleInterval = setInterval(createObstacle, 1500);
            gameLoop = setInterval(updateGame, 20);
        }

        function createObstacle() {
            if (!isGameRunning) return;
            
            const obstacle = document.createElement('div');
            obstacle.className = 'obstacle';
            obstacle.style.right = '-20px';
            gameContainer.appendChild(obstacle);
            
            obstacles.push({
                element: obstacle,
                x: gameContainer.clientWidth,
                width: 20,
                height: 40,
                speed: gameSpeed
            });
        }

        function jump() {
            if (playerProps.isJumping || !isGameRunning) return;
            
            playerProps.isJumping = true;
            playerProps.jumpPower = 12;
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
            player.style.bottom = (gameContainer.clientHeight - playerProps.y - playerProps.height) + 'px';
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
            const playerBottom = playerProps.y + playerProps.height;
            
            const obstacleLeft = obstacle.x;
            const obstacleRight = obstacle.x + obstacle.width;
            const obstacleTop = gameContainer.clientHeight - obstacle.height - 20;
            const obstacleBottom = gameContainer.clientHeight - 20;
            
            return (
                playerRight > obstacleLeft &&
                playerLeft < obstacleRight &&
                playerBottom > obstacleTop &&
                playerTop < obstacleBottom
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
            if ((e.code === 'Space' || e.key === 'ArrowUp') && isGameRunning) {
                e.preventDefault();
                jump();
            }
        });

        startBtn.addEventListener('click', startGame);
        restartBtn.addEventListener('click', startGame);
    </script>
</body>
</html>
