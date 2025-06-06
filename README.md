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
            bottom: 30px !important;
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
    <!-- HTML bleibt unverändert -->
    <!-- ... -->

    <script>
        // Unveränderte Spiel-Logik, außer zwei Stellen:

        const gravity = 0.6; // Kann optional auf 0.7 erhöht werden

        function jump() {
            if (isJumping || isSliding || !isGameRunning) return;

            isJumping = true;
            jumpPower = 20; // ← Erhöhte Sprungkraft
            endSlide();
        }

        // Restlicher JavaScript-Code bleibt gleich
    </script>
</body>
</html>
