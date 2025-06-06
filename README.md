<!DOCTYPE html>
<html>
<head>
    <title>Jump & Slide Game</title>
    <style>
        canvas {
            background: white;
            border: 1px solid black;
            display: block;
            margin: 0 auto;
        }
        body {
            text-align: center;
            font-family: Arial, sans-serif;
        }
    </style>
</head>
<body>
    <h1>Jump & Slide Game</h1>
    <canvas id="gameCanvas" width="800" height="400"></canvas>
    <p>Pfeiltasten: ← → bewegen | ↑ springen | ↓ slideln</p>
    <p id="score">Score: 0</p>
    <button id="restart" style="display:none;">Neustart</button>

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
        const obstacleFrequency = 1500; // ms

        // Spieler
        const player = {
            width: 40,
            originalHeight: 60,
            height: 60,
            x: 100,
            y: canvas.height - 60 - 50,
            speed: 5,
            jumping: false,
            jumpVelocity: 16, // Höherer Sprung
            jumpCount: 16,
            gravity: 0.8,
            sliding: false,
            slideHeight: 30,
            slideCooldown: 0,
            color: 'red'
        };

        // Hindernisse
        let obstacles = [];

        // Tastenzustände
        const keys = {
            ArrowLeft: false,
            ArrowRight: false,
            ArrowUp: false,
            ArrowDown: false
        };

        // Tasten-Events
        window.addEventListener('keydown', (e) => {
            if (['ArrowLeft', 'ArrowRight', 'ArrowUp', 'ArrowDown'].includes(e.key)) {
                keys[e.key] = true;
                e.preventDefault();
            }
            if (e.key === 'r' && gameOver) restartGame();
        });

        window.addEventListener('keyup', (e) => {
            if (['ArrowLeft', 'ArrowRight', 'ArrowUp', 'ArrowDown'].includes(e.key)) {
                keys[e.key] = false;
                e.preventDefault();
            }
        });

        // Neustart-Button
        restartButton.addEventListener('click', restartGame);

        function restartGame() {
            player.x = 100;
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
            if (now - lastObstacleTime > obstacleFrequency) {
                lastObstacleTime = now;
                const heights = [60, 80, 100];
                const height = heights[Math.floor(Math.random() * heights.length)];
                obstacles.push({
                    width: 50,
                    height: height,
                    x: canvas.width,
                    y: canvas.height - height - 50,
                    color: 'green',
                    passed: false
                });
            }
        }

        function updatePlayer() {
            // Bewegung
            if (keys.ArrowLeft) player.x -= player.speed;
            if (keys.ArrowRight) player.x += player.speed;

            // Sprung (höher)
            if (!player.jumping && keys.ArrowUp) {
                player.jumping = true;
                player.jumpCount = player.jumpVelocity;
            }

            if (player.jumping) {
                if (player.jumpCount >= -player.jumpVelocity) {
                    const neg = player.jumpCount < 0 ? -1 : 1;
                    player.y -= (player.jumpCount ** 2) * 0.5 * neg;
                    player.jumpCount -= 1;
                } else {
                    player.jumping = false;
                }
            }

            // Slide (unter Hindernisse)
            if (keys.ArrowDown && !player.jumping && player.slideCooldown === 0) {
                player.sliding = true;
                player.height = player.slideHeight;
            } else if (!keys.ArrowDown) {
                player.sliding = false;
                player.height = player.originalHeight;
            }

            if (player.slideCooldown > 0) player.slideCooldown--;

            // Gravitation
            if (player.y < canvas.height - player.height - 50 && !player.jumping) {
                player.y += player.gravity;
            } else {
                player.y = canvas.height - player.height - 50;
            }

            // Bildschirmbegrenzung
            player.x = Math.max(0, Math.min(player.x, canvas.width - player.width));
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
                    if (player.sliding && (player.y + player.height) >= (obstacle.y + obstacle.height - 5)) {
                        // Erfolgreich unter Hindernis durchgerutscht
                        continue;
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
                
                // Hindernis entfernen
                if (obstacles[i].x < -obstacles[i].width) {
                    obstacles.splice(i, 1);
                }
            }
        }

        function draw() {
            // Clear canvas
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            // Boden zeichnen
            ctx.fillStyle = 'black';
            ctx.fillRect(0, canvas.height - 50, canvas.width, 50);
            
            // Spieler zeichnen
            ctx.fillStyle = player.color;
            ctx.fillRect(player.x, player.y, player.width, player.height);
            
            // Hindernisse zeichnen
            ctx.fillStyle = 'green';
            obstacles.forEach(obstacle => {
                ctx.fillRect(obstacle.x, obstacle.y, obstacle.width, obstacle.height);
            });
            
            // Score anzeigen
            scoreElement.textContent = `Score: ${score}`;
            
            if (gameOver) {
                ctx.fillStyle = 'black';
                ctx.font = '36px Arial';
                ctx.fillText('GAME OVER', canvas.width/2 - 100, canvas.height/2);
            }
        }

        function gameLoop() {
            if (!gameOver) {
                updatePlayer();
                spawnObstacle();
                updateObstacles();
                checkCollisions();
                
                // Schwierigkeit erhöhen
                gameSpeed = 5 + Math.floor(score / 5);
            }
            
            draw();
            requestAnimationFrame(gameLoop);
        }

        // Spiel starten
        gameLoop();
    </script>
</body>
</html>
