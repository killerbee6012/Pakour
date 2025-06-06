<!DOCTYPE html>
<html>
<head>
    <title>Physik-Basiertes Hindernis-Spiel</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            background: #f0f0f0;
            font-family: Arial, sans-serif;
        }
        #gameCanvas {
            display: block;
            background: #87CEEB; /* Himmelblau */
        }
        #score {
            position: absolute;
            top: 10px;
            left: 10px;
            font-size: 24px;
            color: white;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
        }
        #gameOver {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            font-size: 48px;
            color: red;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
            display: none;
        }
    </style>
</head>
<body>
    <div id="score">Score: 0</div>
    <div id="gameOver">Game Over!</div>
    <canvas id="gameCanvas"></canvas>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/matter-js/0.18.0/matter.min.js"></script>
    <script>
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

        // Canvas & Renderer
        const canvas = document.getElementById('gameCanvas');
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;
        const render = Render.create({
            canvas: canvas,
            engine: engine,
            options: {
                width: window.innerWidth,
                height: window.innerHeight,
                wireframes: false,
                background: '#87CEEB'
            }
        });

        // Spieler-Objekt (Ball)
        const player = Bodies.circle(100, 200, 20, {
            restitution: 0.8,
            friction: 0.1,
            density: 0.04,
            render: {
                fillStyle: '#FF5722'
            },
            label: 'player'
        });

        // Boden
        const ground = Bodies.rectangle(
            window.innerWidth / 2,
            window.innerHeight - 10,
            window.innerWidth,
            20,
            { isStatic: true, render: { fillStyle: '#4CAF50' }, label: 'ground' }
        );

        // Hindernisse (zufällig generiert)
        const obstacles = [];
        function createObstacle() {
            const types = ['rectangle', 'circle', 'triangle', 'trampoline', 'lava'];
            const type = types[Math.floor(Math.random() * types.length)];
            let obstacle;

            const x = window.innerWidth + 100;
            const y = Math.random() * (window.innerHeight - 200) + 100;

            switch (type) {
                case 'rectangle':
                    obstacle = Bodies.rectangle(x, y, 50, 50, { 
                        isStatic: false, 
                        restitution: 0.6,
                        render: { fillStyle: '#8D6E63' },
                        label: 'obstacle'
                    });
                    break;
                case 'circle':
                    obstacle = Bodies.circle(x, y, 30, { 
                        isStatic: false, 
                        restitution: 0.8,
                        render: { fillStyle: '#3949AB' },
                        label: 'obstacle'
                    });
                    break;
                case 'triangle':
                    obstacle = Bodies.polygon(x, y, 3, 30, { 
                        isStatic: false, 
                        restitution: 0.7,
                        render: { fillStyle: '#FFA000' },
                        label: 'obstacle'
                    });
                    break;
                case 'trampoline':
                    obstacle = Bodies.rectangle(x, y, 80, 10, { 
                        isStatic: false, 
                        restitution: 1.5, // Hoher Rückprall
                        render: { fillStyle: '#00E676' },
                        label: 'trampoline'
                    });
                    break;
                case 'lava':
                    obstacle = Bodies.rectangle(x, y, 70, 15, { 
                        isStatic: true, 
                        render: { fillStyle: '#FF3D00' },
                        label: 'lava'
                    });
                    break;
            }

            obstacles.push(obstacle);
            Composite.add(world, obstacle);
        }

        // Spiel-Logik
        let score = 0;
        let gameActive = true;
        const scoreElement = document.getElementById('score');
        const gameOverElement = document.getElementById('gameOver');

        // Hindernis-Spawner
        setInterval(() => {
            if (gameActive) {
                createObstacle();
                score++;
                scoreElement.textContent = `Score: ${score}`;
            }
        }, 2000);

        // Spieler-Steuerung (Springen mit Leertaste)
        document.addEventListener('keydown', (e) => {
            if (e.code === 'Space' && gameActive) {
                Matter.Body.applyForce(player, player.position, { x: 0, y: -0.05 });
            }
        });

        // Kollisionserkennung (Game Over bei Lava)
        Events.on(engine, 'collisionStart', (event) => {
            const pairs = event.pairs;
            for (let i = 0; i < pairs.length; i++) {
                const pair = pairs[i];
                if ((pair.bodyA.label === 'player' && pair.bodyB.label === 'lava') || 
                    (pair.bodyB.label === 'player' && pair.bodyA.label === 'lava')) {
                    gameOver();
                }
            }
        });

        function gameOver() {
            gameActive = false;
            gameOverElement.style.display = 'block';
            setTimeout(() => {
                document.location.reload();
            }, 2000);
        }

        // Alles zur Welt hinzufügen
        Composite.add(world, [player, ground]);

        // Render & Run
        Render.run(render);
        const runner = Runner.create();
        Runner.run(runner, engine);

        // Fensteranpassung
        window.addEventListener('resize', () => {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
            render.options.width = window.innerWidth;
            render.options.height = window.innerHeight;
            Matter.Body.setPosition(ground, { x: window.innerWidth / 2, y: window.innerHeight - 10 });
        });
    </script>
</body>
</html>
