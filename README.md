<!DOCTYPE html>
<html>
<head>
    <title>Jump & Slide RGB</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            font-family: Arial, sans-serif;
        }
        canvas {
            display: block;
            border-radius: 10px;
            box-shadow: 0 0 20px rgba(0,0,0,0.5);
        }
        #ui {
            position: absolute;
            top: 10px;
            left: 10px;
            color: white;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.5);
        }
        #restart {
            position: absolute;
            bottom: 20px;
            padding: 10px 20px;
            font-size: 18px;
            background: #4CAF50;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            display: none;
        }
    </style>
</head>
<body>
    <div id="ui">
        <h1>Jump & Slide RGB</h1>
        <p id="score">Score: 0</p>
    </div>
    <canvas id="gameCanvas" width="800" height="400"></canvas>
    <button id="restart">Neustart (R)</button>

    <script>
        // Canvas setup
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreElement = document.getElementById('score');
        const restartButton = document.getElementById('restart');

        // Spielvariablen
        let score = 0;
        let gameSpeed = 5;
        let gameOver = false;
        let lastObstacleTime = 0;
        let hue = 0;
        const obstacleTypes = [
            { width: 50, height: 80, gap: 0 },
            { width: 30, height: 120, gap: 0 },
            { width: 70, height: 60, gap: 0 },
            { width: 50, height: 100, gap: 30 }
        ];

        // Spieler
        const player = {
            width: 40,
            originalHeight: 60,
            height: 60,
            x: 100,
            y: canvas.height - 60 - 50,
            jumping: false,
            jumpVelocity: 15,
            jumpCount: 15,
            gravity: 0.8,
            sliding: false,
            slideHeight: 30,
            color: '#2196F3'
        };

        // Hindernisse
        let obstacles = [];

        // Tastenzustände
        const keys = {
            ArrowUp: false,
            ArrowDown: false,
            ' ': false
        };

        // Tasten-Events
        window.addEventListener('keydown', (e) => {
            if (['ArrowUp', 'ArrowDown', ' '].includes(e.key)) {
                keys[e.key] = true;
                e.preventDefault();
            }
            if (e.key === 'r' && gameOver) restartGame();
        });

        window.addEventListener('keyup', (e) => {
            if (['ArrowUp', 'ArrowDown', ' '].includes(e.key)) {
                keys[e.key] = false;
                e.preventDefault();
            }
        });

        // Neustart-Button
        restartButton.addEventListener('click', restartGame);

        function restartGame() {
            player.y = canvas.height - player.originalHeight - 50;
            player.height = player.originalHeight;
            player.jumping = false;
            player.sliding = false;
            obstacles = [];
            score = 0;
            gameSpeed = 5;
            gameOver = false;
            lastObstacleTime = 0;
            restartButton.style.display = 'none';
        }

        function spawnObstacle() {
            const now = Date.now();
            if (now - lastObstacleTime > 1000 + Math.random() * 1000) {
                lastObstacleTime = now;
                const type = obstacleTypes[Math.floor(Math.random() * obstacleTypes.length)];
                
                obstacles.push({
                    width: type.width,
                    height: type.height,
                    x: canvas.width,
                    y: canvas.height - type.height - 50 - type.gap,
                    color: `hsl(${Math.random() * 360}, 70%, 50%)`,
                    passed: false,
                    hasGap: type.gap > 0
                });
            }
        }

        function updatePlayer() {
            // Sprung
            if (!player.jumping && (keys.ArrowUp || keys[' '])) {
                player.jumping = true;
                player.jumpCount = player.jumpVelocity;
            }

            if (player.jumping) {
                if (player.jumpCount >= -player.jumpVelocity) {
                    const neg = player.jumpCount < 0 ? -1 : 1;
                    player.y -= (player.jumpCount ** 2) * 0.4 * neg;
                    player.jumpCount -= 1;
                } else {
                    player.jumping = false;
                }
            }

            // Slide
            if ((keys.ArrowDown || keys[' ']) && !player.jumping) {
                player.sliding = true;
                player.height = player.slideHeight;
            } else {
                player.sliding = false;
                player.height = player.originalHeight;
            }

            // Gravitation
            if (player.y < canvas.height - player.height - 50 && !player.jumping) {
                player.y += player.gravity;
            } else {
                player.y = canvas.height - player.height - 50;
            }
        }

        function checkCollisions() {
            const playerRect = {
                x: player.x,
                y: player.y,
                width: player.width,
                height: player.height
            };

            for (let i = 0; i < obstacles.length; i++) {
                const obstacle = obstacles[i];
                const obstacleRect = {
                    x: obstacle.x,
                    y: obstacle.y,
                    width: obstacle.width,
                    height: obstacle.height
                };

                // Kollisionserkennung
                if (rectCollision(playerRect, obstacleRect)) {
                    if (player.sliding && (player.y + player.height) >= (obstacle.y + obstacle.height - 5) && !obstacle.hasGap) {
                        continue; // Durchgerutscht
                    } else {
                        gameOver = true;
                        restartButton.style.display = 'block';
                        return;
                    }
                }

                // Punkte zählen
                if (obstacle.x + obstacle.width < player.x && !obstacle.passed) {
                    obstacle.passed = true;
                    score++;
                    scoreElement.textContent = `Score: ${score}`;
                }
            }
        }

        function rectCollision(rect1, rect2) {
            return rect1.x < rect2.x + rect2.width &&
                   rect1.x + rect1.width > rect2.x &&
                   rect1.y < rect2.y + rect2.height &&
                   rect1.y + rect1.height > rect2.y;
        }

        function updateObstacles() {
            for (let i = obstacles.length - 1; i >= 0; i--) {
                obstacles[i].x -= gameSpeed;
                
                if (obstacles[i].x < -obstacles[i].width) {
                    obstacles.splice(i, 1);
                }
            }
        }

        function drawRGBBackground() {
            // RGB Farbverlauf
            hue = (hue + 0.5) % 360;
            const gradient = ctx.createLinearGradient(0, 0, canvas.width, canvas.height);
            gradient.addColorStop(0, `hsl(${hue}, 100%, 50%)`);
            gradient.addColorStop(0.5, `hsl(${(hue + 120) % 360}, 100%, 50%)`);
            gradient.addColorStop(1, `hsl(${(hue + 240) % 360}, 100%, 50%)`);
            
            ctx.fillStyle = gradient;
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            // Boden
            ctx.fillStyle = '#333';
            ctx.fillRect(0, canvas.height - 50, canvas.width, 50);
        }

        function drawPlayer() {
            ctx.fillStyle = player.color;
            
            if (player.sliding) {
                // Slide-Position
                ctx.fillRect(player.x, player.y + 15, player.width, player.height - 15);
            } else if (player.jumping) {
                // Sprung-Position
                ctx.beginPath();
                ctx.arc(player.x + player.width/2, player.y + player.height/2, 
                        player.width/2, 0, Math.PI * 2);
                ctx.fill();
            } else {
                // Normale Position
                ctx.fillRect(player.x, player.y, player.width, player.height);
            }
        }

        function draw() {
            // Hintergrund
            drawRGBBackground();
            
            // Hindernisse
            obstacles.forEach(obstacle => {
                ctx.fillStyle = obstacle.color;
                ctx.fillRect(obstacle.x, obstacle.y, obstacle.width, obstacle.height);
                
                if (obstacle.hasGap) {
                    ctx.fillStyle = '#333';
                    ctx.fillRect(obstacle.x, obstacle.y + obstacle.height, obstacle.width, 30);
                }
            });
            
            // Spieler
            drawPlayer();
            
            // Game Over
            if (gameOver) {
                ctx.fillStyle = 'rgba(0, 0, 0, 0.7)';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                ctx.fillStyle = 'white';
                ctx.font = '36px Arial';
                ctx.textAlign = 'center';
                ctx.fillText('GAME OVER', canvas.width/2, canvas.height/2 - 20);
                ctx.font = '24px Arial';
                ctx.fillText(`Score: ${score}`, canvas.width/2, canvas.height/2 + 20);
                ctx.textAlign = 'left';
            }
        }

        function gameLoop() {
            if (!gameOver) {
                updatePlayer();
                spawnObstacle();
                updateObstacles();
                checkCollisions();
                gameSpeed = 5 + Math.floor(score / 10);
            }
            
            draw();
            requestAnimationFrame(gameLoop);
        }

        // Spiel starten
        gameLoop();
    </script>
</body>
</html>
