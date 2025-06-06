<!DOCTYPE html>
<html>
<head>
    <title>Mini-Physik-Spiel</title>
    <style>
        body {
            margin: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background: #222;
            font-family: Arial, sans-serif;
        }
        #gameContainer {
            position: relative;
            width: 800px;
            height: 500px;
            border: 4px solid #444;
            box-shadow: 0 0 20px rgba(0, 0, 0, 0.5);
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
    <div id="gameContainer">
        <div id="score">Score: 0</div>
        <div id="gameOver">Game Over!</div>
        <canvas id="gameCanvas" width="800" height="500"></canvas>
    </div>
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
        const render = Render.create({
            canvas: canvas,
            engine: engine,
            options: {
                width: 800,
                height: 500,
                wireframes: false,
                background: '#87CEEB'
            }
        });

        // Spieler-Objekt (Ball)
        const player = Bodies.circle(100, 250, 20, {
            restitution: 0.5,
            friction: 0.1,
            density: 0.04,
            render: {
                fillStyle: '#FF5722'
            },
            label: 'player'
        });

        // Fester Boden
        const ground = Bodies.rectangle(400, 490, 800, 20, {
            isStatic: true,
            render: { fillStyle: '#4CAF50' },
            label: 'ground'
        });

        // Hindernisse (abwechselnde Typen)
        const obstacles = [];
        let lastObstacleType = null;

        function createObstacle() {
            const types = ['rectangle', 'circle', 'triangle', 'trampoline'];
            let type;
            
            // Wähle einen anderen Typ als den zuletzt verwendeten
            do {
                type = types[Math.floor(Math.random() * types.length)];
            } while (type === lastObstacleType && types.length > 1);
            
            lastObstacleType = type;

            const x = 850; // Rechts außerhalb des Canvas
            const y = Math.random() * 300 + 150; // Zufällige Höhe

            let obstacle;
            switch (type) {
                case 'rectangle':
                    obstacle = Bodies.rectangle(x, y, 60, 40, { 
                        isStatic: false, 
                        restitution: 0.4,
                        render: { fillStyle: '#8D6E63' },
                        label: 'obstacle'
                    });
                    break;
                case 'circle':
                    obstacle = Bodies.circle(x, y, 25, { 
                        isStatic: false, 
                        restitution: 0.7,
                        render: { fillStyle: '#3949AB' },
                        label: 'obstacle'
                    });
                    break;
                case 'triangle':
                    obstacle = Bodies.polygon(x, y, 3, 25, { 
                        isStatic: false, 
                        restitution: 0.6,
                        render: { fillStyle: '#FFA000' },
                        label: 'obstacle'
                    });
                    break;
                case 'trampoline':
                    obstacle = Bodies.rectangle(x, y, 80, 10, { 
                        isStatic: false, 
                        restitution: 1.3, // Hoher Rückprall
                        render: { fillStyle: '#00E676' },
                        label: 'trampoline'
                    });
                    break;
            }

            // Hindernis nach links bewegen
            Matter.Body.setVelocity(obstacle, { x: -5, y: 0 });
            obstacles.push(obstacle);
            Composite.add(world, obstacle);
        }

        // Spiel-Logik
        let score = 0;
        let gameActive = true;
        const scoreElement = document.getElementById('score');
        const gameOverElement = document.getElementById('gameOver');

        // Hindernis-Spawner (alle 1.5 Sekunden)
        setInterval(() => {
            if (gameActive) {
                createObstacle();
                score++;
                scoreElement.textContent = `Score: ${score}`;
            }
        }, 1500);

        // Spieler-Steuerung (Springen mit Leertaste)
        document.addEventListener('keydown', (e) => {
            if (e.code === 'Space' && gameActive) {
                Matter.Body.applyForce(player, player.position, { x: 0, y: -0.03 });
            }
        });

        // Kollisionserkennung (Game Over bei Bodenberührung)
        Events.on(engine, 'collisionStart', (event) => {
            const pairs = event.pairs;
            for (let i = 0; i < pairs.length; i++) {
                const pair = pairs[i];
                if ((pair.bodyA.label === 'player' && pair.bodyB.label === 'ground') || 
                    (pair.bodyB.label === 'player' && pair.bodyA.label === 'ground')) {
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

        // Alte Hindernisse regelmäßig entfernen
        setInterval(() => {
            if (!gameActive) return;
            for (let i = obstacles.length - 1; i >= 0; i--) {
                if (obstacles[i].position.x < -50) {
                    Composite.remove(world, obstacles[i]);
                    obstacles.splice(i, 1);
                }
            }
        }, 1000);
    </script>
</body>
</html>
