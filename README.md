<!DOCTYPE html>
<html>
<head>
    <title>Mario-ähnliches Laufspiel (Final)</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background: #333;
            font-family: Arial, sans-serif;
            overflow: hidden;
        }
        #gameContainer {
            position: relative;
            width: 800px;
            height: 400px;
            border: 4px solid #555;
            box-shadow: 0 0 20px rgba(0, 0, 0, 0.7);
            overflow: hidden;
        }
        #gameCanvas {
            position: absolute;
            background: linear-gradient(to bottom, #87CEEB 60%, #4CAF50 60%);
        }
        #score {
            position: absolute;
            top: 10px;
            left: 10px;
            font-size: 24px;
            color: white;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.8);
            z-index: 10;
        }
        #gameOver {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            font-size: 48px;
            color: red;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.8);
            display: none;
            z-index: 10;
        }
        #startButton {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            padding: 15px 30px;
            font-size: 24px;
            background: #FF5722;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            z-index: 10;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
        }
        #startButton:hover {
            background: #E64A19;
        }
    </style>
</head>
<body>
    <div id="gameContainer">
        <div id="score">Score: 0</div>
        <div id="gameOver">Game Over!</div>
        <button id="startButton">Spiel Starten</button>
        <canvas id="gameCanvas" width="800" height="400"></canvas>
    </div>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/matter-js/0.18.0/matter.min.js"></script>
    <script>
        // Game Variables
        let gameActive = false;
        let score = 0;
        let scrollSpeed = 3;
        const groundHeight = 40;
        let playerWidth = 30;
        let playerHeight = 50;
        let isJumping = false;
        let isSliding = false;
        let slideTimer = 0;

        // Matter.js Setup
        const Engine = Matter.Engine,
              Render = Matter.Render,
              Runner = Matter.Runner,
              Bodies = Matter.Bodies,
              Composite = Matter.Composite,
              Events = Matter.Events;

        // Engine erstellen
        const engine = Engine.create();
        const world = engine.world;
        engine.gravity.y = 0.8;

        // Canvas & Renderer
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const render = Render.create({
            canvas: canvas,
            engine: engine,
            options: {
                width: 800,
                height: 400,
                wireframes: false,
                background: 'transparent'
            }
        });

        // Game Elements
        let player;
        let ground;
        let obstacles = [];
        let clouds = [];
        let backgroundOffset = 0;

        // Create Clouds
        function createClouds() {
            for (let i = 0; i < 5; i++) {
                clouds.push({
                    x: Math.random() * 800,
                    y: Math.random() * 100 + 20,
                    width: Math.random() * 60 + 40,
                    speed: Math.random() * 0.5 + 0.2
                });
            }
        }

        // Draw Clouds
        function drawClouds() {
            ctx.save();
            clouds.forEach(cloud => {
                ctx.beginPath();
                ctx.arc(cloud.x, cloud.y, cloud.width/2, 0, Math.PI * 2);
                ctx.arc(cloud.x - cloud.width/3, cloud.y - cloud.width/6, cloud.width/3, 0, Math.PI * 2);
                ctx.arc(cloud.x + cloud.width/3, cloud.y - cloud.width/6, cloud.width/3, 0, Math.PI * 2);
                ctx.arc(cloud.x - cloud.width/4, cloud.y + cloud.width/6, cloud.width/3, 0, Math.PI * 2);
                ctx.arc(cloud.x + cloud.width/4, cloud.y + cloud.width/6, cloud.width/3, 0, Math.PI * 2);
                ctx.fillStyle = 'rgba(255, 255, 255, 0.8)';
                ctx.fill();
                
                cloud.x -= cloud.speed;
                if (cloud.x < -cloud.width) {
                    cloud.x = 800 + cloud.width;
                    cloud.y = Math.random() * 100 + 20;
                }
            });
            ctx.restore();
        }

        // Create Player
        function createPlayer() {
            player = Bodies.rectangle(100, 400 - groundHeight - playerHeight/2, 
                                    playerWidth, playerHeight, {
                restitution: 0.2,
                friction: 0.3,
                density: 0.05,
                label: 'player',
                chamfer: { radius: 5 }
            });
            Composite.add(world, player);
        }

        // Draw Player
        function drawPlayer() {
            ctx.save();
            ctx.translate(player.position.x, player.position.y);
            ctx.rotate(player.angle);
            
            if (isSliding) {
                // Slide position (flach)
                ctx.fillStyle = '#FF0000';
                ctx.fillRect(-playerWidth/2, -15, playerWidth, 30);
                
                ctx.fillStyle = '#0000FF';
                ctx.fillRect(-playerWidth/2, -5, playerWidth, 20);
            } else {
                // Normal position
                ctx.fillStyle = '#FF0000';
                ctx.fillRect(-playerWidth/2, -playerHeight/2, playerWidth, playerHeight);
                
                ctx.fillStyle = '#0000FF';
                ctx.fillRect(-playerWidth/2, -playerHeight/6, playerWidth, playerHeight/3);
            }
            
            // Kopf in beiden Positionen
            ctx.fillStyle = '#FFC0CB';
            ctx.beginPath();
            ctx.arc(0, isSliding ? -10 : -playerHeight/4, playerWidth/3, 0, Math.PI * 2);
            ctx.fill();
            
            ctx.restore();
        }

        // Create Ground
        function createGround() {
            ground = Bodies.rectangle(400, 400 - groundHeight/2, 800, groundHeight, {
                isStatic: true,
                label: 'ground',
                render: { visible: false }
            });
            Composite.add(world, ground);
        }

        // Draw Ground
        function drawGround() {
            ctx.fillStyle = '#4CAF50';
            ctx.fillRect(0, 400 - groundHeight, 800, groundHeight);
            
            // Ground details
            ctx.fillStyle = '#388E3C';
            for (let x = -backgroundOffset % 40; x < 800; x += 40) {
                ctx.fillRect(x, 400 - groundHeight, 20, 5);
            }
        }

        // Create Obstacles
        function createObstacle() {
            const types = ['pipe', 'block'];
            const type = types[Math.floor(Math.random() * types.length)];
            
            let obstacle;
            const x = 850;
            
            if (type === 'pipe') {
                // Kleinere Röhren (50-100px hoch statt 80-180px)
                const pipeHeight = Math.random() * 50 + 50;
                obstacle = Bodies.rectangle(x, 400 - groundHeight - pipeHeight/2, 
                                          50, pipeHeight, { // Schmalere Röhren (50px breit)
                    isStatic: true,
                    label: 'obstacle'
                });
            } else {
                obstacle = Bodies.rectangle(x, 400 - groundHeight - 30, 
                                          40, 40, {
                    isStatic: true,
                    label: 'obstacle'
                });
            }
            
            obstacles.push({
                body: obstacle,
                type: type
            });
            Composite.add(world, obstacle);
        }

        // Draw Obstacles
        function drawObstacles() {
            obstacles.forEach(obstacle => {
                ctx.save();
                ctx.translate(obstacle.body.position.x, obstacle.body.position.y);
                ctx.rotate(obstacle.body.angle);
                
                if (obstacle.type === 'pipe') {
                    ctx.fillStyle = '#00AA00';
                    ctx.fillRect(-25, -obstacle.body.bounds.max.y + obstacle.body.position.y, 50, obstacle.body.bounds.max.y - obstacle.body.bounds.min.y);
                    // Pipe details
                    ctx.fillStyle = '#007700';
                    ctx.fillRect(-20, -obstacle.body.bounds.max.y + obstacle.body.position.y + 5, 40, 8);
                } else {
                    ctx.fillStyle = '#B87333';
                    ctx.fillRect(-20, -20, 40, 40);
                    // Block details
                    ctx.fillStyle = '#A05A2C';
                    ctx.fillRect(-15, -15, 30, 5);
                }
                
                ctx.restore();
            });
        }

        // Game Functions
        function startGame() {
            gameActive = true;
            score = 0;
            scrollSpeed = 3;
            isJumping = false;
            isSliding = false;
            document.getElementById('startButton').style.display = 'none';
            document.getElementById('gameOver').style.display = 'none';
            
            Composite.clear(world);
            obstacles = [];
            clouds = [];
            
            createPlayer();
            createGround();
            createClouds();
            
            obstacleInterval = setInterval(() => {
                if (gameActive) {
                    createObstacle();
                }
            }, 2000);
            
            if (!runner) {
                runner = Runner.create();
                Runner.run(runner, engine);
            }
            
            requestAnimationFrame(gameLoop);
        }

        function gameOver() {
            gameActive = false;
            clearInterval(obstacleInterval);
            document.getElementById('gameOver').style.display = 'block';
            document.getElementById('startButton').style.display = 'block';
        }

        // Game Loop
        let runner;
        let obstacleInterval;
        function gameLoop() {
            if (!gameActive) return;
            
            ctx.clearRect(0, 0, 800, 400);
            
            // Draw background
            ctx.fillStyle = '#87CEEB';
            ctx.fillRect(0, 0, 800, 400 - groundHeight);
            
            drawClouds();
            drawGround();
            drawObstacles();
            drawPlayer();
            
            // Move obstacles
            obstacles.forEach(obstacle => {
                Matter.Body.setPosition(obstacle.body, {
                    x: obstacle.body.position.x - scrollSpeed,
                    y: obstacle.body.position.y
                });
                
                if (obstacle.body.position.x < -100) {
                    Composite.remove(world, obstacle.body);
                    obstacles = obstacles.filter(o => o.body.id !== obstacle.body.id);
                }
            });
            
            backgroundOffset += scrollSpeed;
            if (backgroundOffset >= 800) backgroundOffset = 0;
            
            // Slide timer
            if (isSliding) {
                slideTimer++;
                if (slideTimer > 60) { // Nach 1 Sekunde aufstehen
                    isSliding = false;
                    playerHeight = 50;
                    Matter.Body.setPosition(player, { 
                        x: player.position.x, 
                        y: 400 - groundHeight - playerHeight/2 
                    });
                }
            }
            
            // Check if player is on ground
            const playerBottom = player.position.y + (isSliding ? 15 : playerHeight/2);
            isJumping = playerBottom < 400 - groundHeight - 5;
            
            score++;
            document.getElementById('score').textContent = `Score: ${score}`;
            
            if (score % 500 === 0) {
                scrollSpeed += 0.5;
            }
            
            requestAnimationFrame(gameLoop);
        }

        // Event Listeners
        document.getElementById('startButton').addEventListener('click', startGame);
        
        document.addEventListener('keydown', (e) => {
            if (!gameActive) return;
            
            // Jump (Space or Up Arrow)
            if ((e.code === 'Space' || e.key === 'ArrowUp') && !isJumping && !isSliding) {
                Matter.Body.setVelocity(player, { x: 0, y: -10 });
                isJumping = true;
            }
            
            // Slide (Down Arrow)
            if ((e.key === 'ArrowDown') && !isSliding && !isJumping) {
                isSliding = true;
                slideTimer = 0;
                playerHeight = 30;
                Matter.Body.setPosition(player, { 
                    x: player.position.x, 
                    y: 400 - groundHeight - playerHeight/2 
                });
            }
        });
        
        document.addEventListener('keyup', (e) => {
            if (e.key === 'ArrowDown' && isSliding) {
                isSliding = false;
                playerHeight = 50;
                Matter.Body.setPosition(player, { 
                    x: player.position.x, 
                    y: 400 - groundHeight - playerHeight/2 
                });
            }
        });

        // Collision detection
        Events.on(engine, 'collisionStart', (event) => {
            const pairs = event.pairs;
            for (let i = 0; i < pairs.length; i++) {
                const pair = pairs[i];
                
                // Kollision mit Hindernissen
                if ((pair.bodyA.label === 'player' && pair.bodyB.label === 'obstacle') || 
                    (pair.bodyB.label === 'player' && pair.bodyA.label === 'obstacle')) {
                    
                    // Nur Game Over wenn nicht am Sliden unter einer Röhre
                    const obstacle = pair.bodyA.label === 'obstacle' ? pair.bodyA : pair.bodyB;
                    const obstacleHeight = obstacle.bounds.max.y - obstacle.bounds.min.y;
                    
                    if (!isSliding || obstacleHeight < 60) { // Kleine Hindernisse (Blöcke) immer tödlich
                        gameOver();
                    }
                }
                
                // Check if player landed on ground
                if ((pair.bodyA.label === 'player' && pair.bodyB.label === 'ground') || 
                    (pair.bodyB.label === 'player' && pair.bodyA.label === 'ground')) {
                    isJumping = false;
                }
            }
        });

        // Initial setup
        Render.run(render);
        createClouds();
    </script>
</body>
</html>
