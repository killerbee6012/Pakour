<!DOCTYPE html>
<html>
<head>
    <title>Mario-ähnliches Laufspiel</title>
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
        .cloud {
            position: absolute;
            background: white;
            border-radius: 50%;
            opacity: 0.8;
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
        let groundHeight = 40;
        let playerWidth = 30;
        let playerHeight = 50;

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
                // Main cloud circle
                ctx.arc(cloud.x, cloud.y, cloud.width/2, 0, Math.PI * 2);
                // Smaller circles for fluffiness
                ctx.arc(cloud.x - cloud.width/3, cloud.y - cloud.width/6, cloud.width/3, 0, Math.PI * 2);
                ctx.arc(cloud.x + cloud.width/3, cloud.y - cloud.width/6, cloud.width/3, 0, Math.PI * 2);
                ctx.arc(cloud.x - cloud.width/4, cloud.y + cloud.width/6, cloud.width/3, 0, Math.PI * 2);
                ctx.arc(cloud.x + cloud.width/4, cloud.y + cloud.width/6, cloud.width/3, 0, Math.PI * 2);
                ctx.fillStyle = 'rgba(255, 255, 255, 0.8)';
                ctx.fill();
                
                // Move clouds
                cloud.x -= cloud.speed;
                if (cloud.x < -cloud.width) {
                    cloud.x = 800 + cloud.width;
                    cloud.y = Math.random() * 100 + 20;
                }
            });
            ctx.restore();
        }

        // Create Player (Mario-ähnlich)
        function createPlayer() {
            player = Bodies.rectangle(100, 400 - groundHeight - playerHeight/2, 
                                    playerWidth, playerHeight, {
                restitution: 0.2,
                friction: 0.3,
                density: 0.05,
                label: 'player'
            });
            Composite.add(world, player);
        }

        // Draw Player
        function drawPlayer() {
            ctx.save();
            ctx.translate(player.position.x, player.position.y);
            ctx.rotate(player.angle);
            
            // Player body
            ctx.fillStyle = '#FF0000';
            ctx.fillRect(-playerWidth/2, -playerHeight/2, playerWidth, playerHeight);
            
            // Player overalls
            ctx.fillStyle = '#0000FF';
            ctx.fillRect(-playerWidth/2, -playerHeight/6, playerWidth, playerHeight/3);
            
            // Player face
            ctx.fillStyle = '#FFC0CB';
            ctx.beginPath();
            ctx.arc(0, -playerHeight/4, playerWidth/3, 0, Math.PI * 2);
            ctx.fill();
            
            ctx.restore();
        }

        // Create Ground
        function createGround() {
            ground = Bodies.rectangle(400, 400 - groundHeight/2, 800, groundHeight, {
                isStatic: true,
                label: 'ground',
                render: { fillStyle: '#4CAF50' }
            });
            Composite.add(world, ground);
        }

        // Create Obstacles
        function createObstacle() {
            const types = ['pipe', 'block', 'gap'];
            const type = types[Math.floor(Math.random() * types.length)];
            
            let obstacle;
            const x = 850; // Start rechts außerhalb
            
            switch (type) {
                case 'pipe':
                    const pipeHeight = Math.random() * 100 + 80;
                    obstacle = Bodies.rectangle(x, 400 - groundHeight - pipeHeight/2, 
                                              60, pipeHeight, {
                        isStatic: true,
                        label: 'obstacle',
                        render: { fillStyle: '#00AA00' }
                    });
                    break;
                    
                case 'block':
                    obstacle = Bodies.rectangle(x, 400 - groundHeight - 30, 
                                              40, 40, {
                        isStatic: true,
                        label: 'obstacle',
                        render: { fillStyle: '#B87333' }
                    });
                    break;
                    
                case 'gap':
                    // Ein unsichtbares Hindernis, das den Spieler fallen lässt
                    obstacle = Bodies.rectangle(x + 100, 400 - groundHeight/2, 
                                              200, groundHeight, {
                        isStatic: true,
                        isSensor: true,
                        label: 'gap',
                        render: { fillStyle: 'transparent' }
                    });
                    break;
            }
            
            obstacles.push(obstacle);
            Composite.add(world, obstacle);
        }

        // Game Functions
        function startGame() {
            gameActive = true;
            score = 0;
            scrollSpeed = 3;
            document.getElementById('startButton').style.display = 'none';
            document.getElementById('gameOver').style.display = 'none';
            
            // Clear existing bodies
            Composite.clear(world);
            
            // Create new elements
            createPlayer();
            createGround();
            createClouds();
            
            // Start obstacle spawner
            obstacleInterval = setInterval(() => {
                if (gameActive) {
                    createObstacle();
                }
            }, 2000);
            
            // Start game loop
            if (!runner) {
                runner = Runner.create();
                Runner.run(runner, engine);
            }
            
            // Start render loop
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
            
            // Clear canvas
            ctx.clearRect(0, 0, 800, 400);
            
            // Draw scrolling background
            drawBackground();
            
            // Draw clouds
            drawClouds();
            
            // Update player position
            if (player.position.y > 500) {
                gameOver();
            }
            
            // Move obstacles and ground
            obstacles.forEach(obstacle => {
                Matter.Body.setPosition(obstacle, {
                    x: obstacle.position.x - scrollSpeed,
                    y: obstacle.position.y
                });
                
                // Remove obstacles that are off screen
                if (obstacle.position.x < -100) {
                    Composite.remove(world, obstacle);
                    obstacles = obstacles.filter(o => o.id !== obstacle.id);
                }
            });
            
            // Move ground (for infinite scrolling)
            backgroundOffset += scrollSpeed;
            if (backgroundOffset >= 800) backgroundOffset = 0;
            
            // Increase difficulty
            score++;
            document.getElementById('score').textContent = `Score: ${score}`;
            
            if (score % 500 === 0) {
                scrollSpeed += 0.5;
            }
            
            // Draw player
            drawPlayer();
            
            // Continue loop
            requestAnimationFrame(gameLoop);
        }

        // Draw repeating background
        function drawBackground() {
            // Sky
            ctx.fillStyle = '#87CEEB';
            ctx.fillRect(0, 0, 800, 400 - groundHeight);
            
            // Ground
            ctx.fillStyle = '#4CAF50';
            ctx.fillRect(0, 400 - groundHeight, 800, groundHeight);
            
            // Ground details (for scrolling effect)
            ctx.fillStyle = '#388E3C';
            for (let x = -backgroundOffset % 40; x < 800; x += 40) {
                ctx.fillRect(x, 400 - groundHeight, 20, 5);
            }
        }

        // Event Listeners
        document.getElementById('startButton').addEventListener('click', startGame);
        
        document.addEventListener('keydown', (e) => {
            if (!gameActive) return;
            
            // Jump (Space or Up Arrow)
            if ((e.code === 'Space' || e.key === 'ArrowUp') && player.position.y > 400 - groundHeight - playerHeight - 5) {
                Matter.Body.applyForce(player, player.position, { x: 0, y: -0.1 });
            }
            
            // Slide (Down Arrow)
            if (e.key === 'ArrowDown') {
                playerHeight = 30; // Squat
                Matter.Body.setPosition(player, { 
                    x: player.position.x, 
                    y: 400 - groundHeight - playerHeight/2 
                });
            }
        });
        
        document.addEventListener('keyup', (e) => {
            if (e.key === 'ArrowDown') {
                playerHeight = 50; // Stand up
            }
        });

        // Kollisionserkennung
        Events.on(engine, 'collisionStart', (event) => {
            const pairs = event.pairs;
            for (let i = 0; i < pairs.length; i++) {
                const pair = pairs[i];
                if ((pair.bodyA.label === 'player' && pair.bodyB.label === 'obstacle') || 
                    (pair.bodyB.label === 'player' && pair.bodyA.label === 'obstacle')) {
                    gameOver();
                }
            }
        });

        // Initial setup
        Render.run(render);
        createClouds();
    </script>
</body>
</html>
