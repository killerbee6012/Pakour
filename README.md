<Willkommen>
<html>
<head>
    <title>Jump & Slide Adventure</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            background: #222;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            font-family: Arial, sans-serif;
        }
        canvas {
            background: linear-gradient(to bottom, #87CEEB, #E0F7FA);
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
        <h1>Jump & Slide Adventure</h1>
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
        let backgroundOffset = 0;
        const obstacleTypes = [
            { width: 50, height: 80, color: '#4CAF50', gap: 0 }, // Normales Hindernis
            { width: 30, height: 120, color: '#F44336', gap: 0 }, // Hohes Hindernis
            { width: 70, height: 60, color: '#FFC107', gap: 0 },  // Breites Hindernis
            { width: 50, height: 100, color: '#9C27B0', gap: 30 } // Lücke darunter
        ];

        // Spieler
        const player = {
            width: 40,
            originalHeight: 60,
            height: 60,
            x: 100,
            y: canvas.height - 60 - 50,
            speed: 0, // Keine seitliche Bewegung
            jumping: false,
            jumpVelocity: 18, // Höherer Sprung
            jumpCount: 18,
            gravity: 0.8,
            sliding: false,
            slideHeight: 30,
            slideCooldown: 0,
            color: '#2196F3'
        };

        // Hindernisse
        let obstacles = [];
        let clouds = [];
        
        // Wolken erstellen
        for (let i = 0; i < 10; i++) {
            clouds.push({
                x: Math.random() * canvas.width,
                y: Math.random() * 150,
                width: 60 + Math.random() * 60,
                speed: 0.5 + Math.random() * 1
            });
        }

        // Tastenzustände
        const keys = {
            ArrowUp: false,
            ArrowDown: false
        };

        // Tasten-Events
        window.addEventListener('keydown', (e) => {
            if (['ArrowUp', 'ArrowDown', ' '].includes(e.key)) {
                if (e.key === ' ') keys.ArrowUp = true; // Leertaste für Sprung
                else keys[e.key] = true;
                e.preventDefault();
            }
            if (e.key === 'r' && gameOver) restartGame();
        });

        window.addEventListener('keyup', (e) => {
            if (['ArrowUp', 'ArrowDown', ' '].includes(e.key)) {
                if (e.key === ' ') keys.ArrowUp = false;
                else keys[e.key] = false;
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
            if (now - lastObstacleTime > 1000 + Math.random() * 1000) {
                lastObstacleTime = now;
                const type = obstacleTypes[Math.floor(Math.random() * obstacleTypes.length)];
                
                obstacles.push({
                    width: type.width,
                    height: type.height,
                    x: canvas.width,
                    y: canvas.height - type.height - 50 - type.gap,
                    color: type.color,
                    passed: false,
                    hasGap: type.gap > 0
                });
            }
        }

        function updatePlayer() {
            // Sprung (höher)
            if (!player.jumping && (keys.ArrowUp || keys[' '])) {
                player.jumping = true;
                player.jumpCount = player.jumpVelocity;
            }

            if (player.jumping) {
                if (player.jumpCount >= -player.jumpVelocity) {
                    const neg = player.jumpCount < 0 ? -1 : 1;
                    player.y -= (player.jumpCount ** 2) * 0.4 * neg; // Weichere Sprungkurve
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
                
                // Hindernis entfernen
                if (obstacles[i].x < -obstacles[i].width) {
                    obstacles.splice(i, 1);
                }
            }
        }

        function drawBackground() {
            // Himmel
            const gradient = ctx.createLinearGradient(0, 0, 0, canvas.height);
            gradient.addColorStop(0, '#87CEEB');
            gradient.addColorStop(1, '#E0F7FA');
            ctx.fillStyle = gradient;
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            // Wolken
            ctx.fillStyle = 'rgba(255, 255, 255, 0.7)';
            clouds.forEach(cloud => {
                ctx.beginPath();
                ctx.arc(cloud.x, cloud.y, cloud.width/3, 0, Math.PI * 2);
                ctx.arc(cloud.x + cloud.width/4, cloud.y - cloud.width/6, cloud.width/4, 0, Math.PI * 2);
                ctx.arc(cloud.x + cloud.width/3, cloud.y, cloud.width/3, 0, Math.PI * 2);
                ctx.arc(cloud.x - cloud.width/4, cloud.y + cloud.width/6, cloud.width/4, 0, Math.PI * 2);
                ctx.fill();
                
                // Wolken bewegen
                cloud.x -= cloud.speed;
                if (cloud.x < -cloud.width) {
                    cloud.x = canvas.width + cloud.width;
                    cloud.y = Math.random() * 150;
                }
            });
            
            // Boden
            ctx.fillStyle = '#795548';
            ctx.fillRect(0, canvas.height - 50, canvas.width, 50);
            
            // Gras
            ctx.fillStyle = '#4CAF50';
            ctx.fillRect(0, canvas.height - 55, canvas.width, 5);
        }

        function drawPlayer() {
            ctx.fillStyle = player.color;
            
            // Spieler mit Animation
            if (player.sliding) {
                // Slide-Position
                ctx.fillRect(player.x, player.y + 15, player.width, player.height - 15);
            } else if (player.jumping) {
                // Sprung-Position
                ctx.beginPath();
                ctx.ellipse(player.x + player.width/2, player.y + player.height/2, 
                            player.width/2, player.height/2, 0, 0, Math.PI * 2);
                ctx.fill();
            } else {
                // Normale Position
                ctx.fillRect(player.x, player.y, player.width, player.height);
            }
        }

        function draw() {
            // Hintergrund
            drawBackground();
            
            // Hindernisse
            obstacles.forEach(obstacle => {
                ctx.fillStyle = obstacle.color;
                ctx.fillRect(obstacle.x, obstacle.y, obstacle.width, obstacle.height);
                
                // Lücke zeichnen wenn vorhanden
                if (obstacle.hasGap) {
                    ctx.fillStyle = '#795548';
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
                ctx.fillText(`Final Score: ${score}`, canvas.width/2, canvas.height/2 + 20);
                ctx.textAlign = 'left';
            }
        }

        function gameLoop() {
            if (!gameOver) {
                updatePlayer();
                spawnObstacle();
                updateObstacles();
                checkCollisions();
                
                // Schwierigkeit erhöhen
                gameSpeed = 5 + Math.floor(score / 10);
                
                // Wolken aktualisieren
                clouds.forEach(cloud => {
                    cloud.x -= cloud.speed * 0.5;
                    if (cloud.x < -cloud.width) {
                        cloud.x = canvas.width + cloud.width;
                        cloud.y = Math.random() * 150;
                    }
                });
            }
            
            draw();
            requestAnimationFrame(gameLoop);
        }

        // Spiel starten
        gameLoop();
    </script>
</body>
</html>
