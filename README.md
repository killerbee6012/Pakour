<Willkommen>
<html>
<head>
    <title>Ultimativer Parkour Runner</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            font-family: 'Arial', sans-serif;
            text-align: center;
            user-select: none;
            background: linear-gradient(45deg, #FF4136, #0074D9, #2ECC40, #FFDC00);
            background-size: 400% 400%;
            animation: rgbBackground 15s ease infinite;
        }
        @keyframes rgbBackground {
            0% { background-position: 0% 50% }
            50% { background-position: 100% 50% }
            100% { background-position: 0% 50% }
        }
        #game-container {
            position: relative;
            width: 700px;
            height: 300px;
            margin: 20px auto;
            background-color: rgba(0,0,0,0.7);
            overflow: hidden;
            border-radius: 15px;
            box-shadow: 0 0 30px rgba(0,0,0,0.8);
            border: 2px solid white;
        }
        #ground {
            position: absolute;
            bottom: 0;
            width: 100%;
            height: 30px;
            background: linear-gradient(to right, #2E8B57, #3CB371);
        }
        #player {
            position: absolute;
            width: 35px;
            height: 60px;
            background-color: #FF6347;
            bottom: 30px;
            left: 80px;
            border-radius: 8px 8px 0 0;
            transition: all 0.2s ease;
            z-index: 10;
        }
        .player-slide {
            height: 30px !important;
            bottom: 45px !important;
            background-color: #FF851B !important;
        }
        .obstacle {
            position: absolute;
            bottom: 30px;
            right: -50px;
            border-radius: 5px;
        }
        .low-obstacle {
            width: 25px;
            height: 45px;
            background-color: #4682B4;
        }
        .high-obstacle {
            width: 25px;
            height: 90px;
            background-color: #6A5ACD;
        }
        .flying-obstacle {
            width: 40px;
            height: 25px;
            bottom: 150px;
            background-color: #FF69B4;
        }
        .coin {
            width: 20px;
            height: 20px;
            background-color: gold;
            border-radius: 50%;
            bottom: 100px;
            animation: spin 2s linear infinite;
            z-index: 5;
        }
        @keyframes spin {
            100% { transform: rotate(360deg); }
        }
        #score-display {
            font-size: 28px;
            margin: 15px;
            color: white;
            text-shadow: 2px 2px 4px black;
            font-weight: bold;
        }
        #controls {
            margin: 20px;
        }
        button {
            padding: 12px 25px;
            font-size: 18px;
            background: linear-gradient(to bottom, #4CAF50, #45a049);
            color: white;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            margin: 0 15px;
            box-shadow: 0 4px 8px rgba(0,0,0,0.3);
            transition: all 0.3s;
        }
        button:hover {
            transform: translateY(-2px);
            box-shadow: 0 6px 12px rgba(0,0,0,0.4);
        }
        #game-over {
            display: none;
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: rgba(0, 0, 0, 0.9);
            color: white;
            padding: 30px;
            border-radius: 15px;
            text-align: center;
            width: 80%;
            z-index: 20;
            border: 2px solid #FF6347;
        }
        #instructions {
            margin: 15px;
            color: white;
            text-shadow: 1px 1px 2px black;
            font-size: 18px;
        }
        #coin-count {
            position: absolute;
            top: 10px;
            right: 20px;
            color: gold;
            font-size: 24px;
            text-shadow: 1px 1px 2px black;
        }
    </style>
</head>
<body>
    <h1 style="color: white; text-shadow: 2px 2px 4px black; font-size: 36px;">Ultimativer Parkour Runner</h1>
    <div id="instructions">
        ↑/Space = Springen | Shift = Slide | Gesammelte Münzen: <span id="coin-display">0</span>
    </div>
    <div id="score-display">Punkte: 0</div>
    <div id="controls">
        <button id="start-btn">Spiel starten</button>
    </div>
    <div id="game-container">
        <div id="ground"></div>
        <div id="player"></div>
        <div id="coin-count">Münzen: 0</div>
        <div id="game-over">
            <h2 style="color: #FF6347; margin-top: 0;">Game Over!</h2>
            <p>Dein Score: <span id="final-score">0</span></p>
            <p>Gesammelte Münzen: <span id="final-coins">0</span></p>
            <button id="restart-btn">Neustart</button>
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
        const coinDisplay = document.getElementById('coin-display');
        const coinCount = document.getElementById('coin-count');
        const finalCoins = document.getElementById('final-coins');

        // Game Variables
        let score = 0;
        let coins = 0;
        let isGameRunning = false;
        let gameSpeed = 6;
        let obstacleInterval;
        let gameLoop;
        let obstacles = [];
        let isSliding = false;
        let slideTimeout;
        let isJumping = false;
        let jumpPower = 0;
        const gravity = 0.6;

        // Player Properties
        const playerProps = {
            x: 80,
            y: gameContainer.clientHeight - 90,
            width: 35,
            normalHeight: 60,
            slidingHeight: 30,
            speed: 0
        };

        // Initialize game
        function initGame() {
            player.style.height = playerProps.normalHeight + 'px';
            player.style.bottom = '30px';
            player.style.left = playerProps.x + 'px';
            player.classList.remove('player-slide');
        }

        // Game Functions
        function startGame() {
            if (isGameRunning) return;
            
            // Reset game state
            isGameRunning = true;
            score = 0;
            coins = 0;
            gameSpeed = 6;
            scoreDisplay.textContent = 'Punkte: ' + score;
            coinDisplay.textContent = coins;
            coinCount.textContent = 'Münzen: ' + coins;
            gameOverScreen.style.display = 'none';
            startBtn.style.display = 'none';
            
            // Clear existing objects
            document.querySelectorAll('.obstacle, .coin').forEach(obj => obj.remove());
            obstacles = [];
            
            // Reset player
            playerProps.y = gameContainer.clientHeight - 90;
            isJumping = false;
            jumpPower = 0;
            isSliding = false;
            clearTimeout(slideTimeout);
            initGame();
            
            // Start game loops
            obstacleInterval = setInterval(createObstacle, 1200);
            gameLoop = requestAnimationFrame(updateGame);
        }

        function createObstacle() {
            if (!isGameRunning) return;
            
            // Create different types of obstacles
            const obstacleTypes = ['low', 'high', 'flying', 'coin'];
            const weights = [0.35, 0.25, 0.25, 0.15]; // Probability weights
            const type = weightedRandom(obstacleTypes, weights);
            
            const obstacle = document.createElement('div');
            obstacle.className = 'obstacle ' + type + '-obstacle';
            
            gameContainer.appendChild(obstacle);
            
            obstacles.push({
                element: obstacle,
                x: gameContainer.clientWidth,
                width: parseInt(obstacle.style.width) || 25,
                height: parseInt(obstacle.style.height) || 45,
                speed: gameSpeed,
                type: type,
                isCoin: type === 'coin',
                y: type === 'flying' ? 150 : (type === 'coin' ? 100 : gameContainer.clientHeight - (type === 'high' ? 120 : 75))
            });
        }

        function weightedRandom(items, weights) {
            let random = Math.random();
            let weightSum = 0;
            
            for (let i = 0; i < items.length; i++) {
                weightSum += weights[i];
                if (random <= weightSum) return items[i];
            }
            
            return items[0];
        }

        function jump() {
            if (isJumping || isSliding || !isGameRunning) return;
            
            isJumping = true;
            jumpPower = 14;
            endSlide();
        }

        function startSlide() {
            if (isJumping || isSliding || !isGameRunning) return;
            
            isSliding = true;
            player.classList.add('player-slide');
            
            // Automatically stand up after 800ms
            slideTimeout = setTimeout(endSlide, 800);
        }

        function endSlide() {
            if (!isSliding) return;
            
            clearTimeout(slideTimeout);
            isSliding = false;
            player.classList.remove('player-slide');
        }

        function updatePlayer() {
            // Apply gravity if jumping
            if (isJumping) {
                playerProps.y -= jumpPower;
                jumpPower -= gravity;
                
                // Check if landed
                if (playerProps.y >= gameContainer.clientHeight - 90) {
                    playerProps.y = gameContainer.clientHeight - 90;
                    isJumping = false;
                    jumpPower = 0;
                }
            }
            
            // Update player position
            player.style.bottom = (gameContainer.clientHeight - playerProps.y - 
                                (isSliding ? playerProps.slidingHeight : playerProps.normalHeight)) + 'px';
        }

        function updateObstacles() {
            for (let i = obstacles.length - 1; i >= 0; i--) {
                const obj = obstacles[i];
                obj.x -= obj.speed;
                obj.element.style.right = (gameContainer.clientWidth - obj.x) + 'px';
                
                // Remove if off screen
                if (obj.x + obj.width < 0) {
                    gameContainer.removeChild(obj.element);
                    obstacles.splice(i, 1);
                    if (!obj.isCoin) {
                        score++;
                        scoreDisplay.textContent = 'Punkte: ' + score;
                        
                        // Increase difficulty
                        if (score % 10 === 0) gameSpeed += 0.5;
                    }
                }
                
                // Check collisions
                if (checkCollision(obj)) {
                    if (obj.isCoin) {
                        collectCoin(obj, i);
                    } else {
                        endGame();
                        return;
                    }
                }
            }
        }

        function checkCollision(obj) {
            const playerLeft = playerProps.x;
            const playerRight = playerProps.x + playerProps.width;
            const playerTop = playerProps.y;
            const playerBottom = playerProps.y + (isSliding ? playerProps.slidingHeight : playerProps.normalHeight);
            
            const objLeft = obj.x;
            const objRight = obj.x + obj.width;
            const objTop = obj.y;
            const objBottom = obj.y + obj.height;
            
            // Coin collection
            if (obj.isCoin) {
                return (playerRight > objLeft &&
                        playerLeft < objRight &&
                        playerBottom > objTop &&
                        playerTop < objBottom);
            }
            
            // Obstacle collision
            return (playerRight > objLeft &&
                    playerLeft < objRight &&
                    playerBottom > objTop &&
                    playerTop < objBottom &&
                    !(isSliding && obj.type === 'low'));
        }

        function collectCoin(obj, index) {
            coins++;
            coinDisplay.textContent = coins;
            coinCount.textContent = 'Münzen: ' + coins;
            gameContainer.removeChild(obj.element);
            obstacles.splice(index, 1);
            score += 2; // Bonus points for coins
            scoreDisplay.textContent = 'Punkte: ' + score;
            
            // Play coin sound (would need audio file)
            // new Audio('coin.mp3').play();
        }

        function updateGame() {
            if (!isGameRunning) return;
            
            updatePlayer();
            updateObstacles();
            
            gameLoop = requestAnimationFrame(updateGame);
        }

        function endGame() {
            isGameRunning = false;
            cancelAnimationFrame(gameLoop);
            clearInterval(obstacleInterval);
            
            finalScore.textContent = score;
            finalCoins.textContent = coins;
            gameOverScreen.style.display = 'block';
            startBtn.style.display = 'inline-block';
        }

        // Event Listeners
        document.addEventListener('keydown', (e) => {
            if (!isGameRunning && (e.code === 'Space' || e.key === 'ArrowUp' || e.key === 'Shift')) {
                startGame();
                return;
            }
            
            if (e.code === 'Space' || e.key === 'ArrowUp') {
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
