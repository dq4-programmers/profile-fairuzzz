<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Game Dinosaurus</title>

    <style>
        * {
            box-sizing: border-box;
        }

        body {
            margin: 0;
            font-family: Arial, sans-serif;
            background: #f7f7f7;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }

        .game-container {
            width: 900px;
            max-width: 95%;
            text-align: center;
        }

        h1 {
            margin-bottom: 10px;
        }

        #game {
            position: relative;
            width: 100%;
            height: 300px;
            background: white;
            border: 3px solid #333;
            overflow: hidden;
        }

        #score {
            position: absolute;
            top: 10px;
            right: 15px;
            font-size: 22px;
            font-weight: bold;
            z-index: 10;
        }

        #dino {
            position: absolute;
            left: 80px;
            bottom: 30px;
            width: 45px;
            height: 55px;
            background: #333;
            border-radius: 8px 8px 3px 3px;
        }

        #dino::before {
            content: "";
            position: absolute;
            width: 28px;
            height: 25px;
            background: #333;
            right: -15px;
            top: 0;
            border-radius: 5px;
        }

        #dino::after {
            content: "";
            position: absolute;
            width: 7px;
            height: 7px;
            background: white;
            right: -5px;
            top: 7px;
            border-radius: 50%;
        }

        .cactus {
            position: absolute;
            bottom: 30px;
            width: 25px;
            height: 55px;
            background: #228b22;
            border-radius: 5px;
        }

        .cactus::before,
        .cactus::after {
            content: "";
            position: absolute;
            background: #228b22;
            width: 15px;
            height: 25px;
            border-radius: 5px;
        }

        .cactus::before {
            left: -10px;
            top: 20px;
        }

        .cactus::after {
            right: -10px;
            top: 10px;
        }

        .ground {
            position: absolute;
            bottom: 0;
            left: 0;
            width: 100%;
            height: 30px;
            background: #333;
        }

        #gameOver {
            display: none;
            position: absolute;
            inset: 0;
            background: rgba(255,255,255,0.9);
            justify-content: center;
            align-items: center;
            flex-direction: column;
            z-index: 20;
        }

        #gameOver h2 {
            font-size: 40px;
            margin: 5px;
        }

        button {
            border: none;
            padding: 12px 25px;
            font-size: 18px;
            cursor: pointer;
            border-radius: 8px;
            background: #333;
            color: white;
        }

        button:hover {
            background: #555;
        }

        .info {
            margin-top: 15px;
            color: #555;
        }
    </style>
</head>

<body>

<div class="game-container">

    <h1>🦖 Dino Game</h1>

    <div id="game">

        <div id="score">
            Skor: 0
        </div>

        <div id="dino"></div>

        <div class="ground"></div>

        <div id="gameOver">
            <h2>GAME OVER</h2>
            <p id="finalScore">Skor: 0</p>
            <button onclick="restartGame()">Main Lagi</button>
        </div>

    </div>

    <div class="info">
        Tekan <b>SPACE</b> untuk melompat
    </div>

</div>

<script>

    const dino = document.getElementById("dino");
    const game = document.getElementById("game");
    const scoreText = document.getElementById("score");
    const gameOverScreen = document.getElementById("gameOver");
    const finalScore = document.getElementById("finalScore");

    let isJumping = false;
    let gameRunning = true;
    let score = 0;
    let cactusList = [];

    /*
        =========================
        LOMPAT
        =========================
    */

    function jump() {

        if (isJumping || !gameRunning) return;

        isJumping = true;

        let position = 30;
        let velocity = 14;

        const jumpInterval = setInterval(() => {

            velocity -= 0.7;
            position += velocity;

            if (position <= 30) {

                position = 30;
                isJumping = false;

                clearInterval(jumpInterval);
            }

            dino.style.bottom = position + "px";

        }, 20);
    }


    /*
        =========================
        BUAT KAKTUS
        =========================
    */

    function createCactus() {

        if (!gameRunning) return;

        const cactus = document.createElement("div");

        cactus.classList.add("cactus");

        cactus.style.right = "-30px";

        game.appendChild(cactus);

        cactusList.push(cactus);

        moveCactus(cactus);
    }


    /*
        =========================
        GERAKKAN KAKTUS
        =========================
    */

    function moveCactus(cactus) {

        let position = -30;

        const interval = setInterval(() => {

            if (!gameRunning) {
                clearInterval(interval);
                return;
            }

            position += 7;

            cactus.style.right = position + "px";

            checkCollision(cactus);

            if (position > game.offsetWidth + 50) {

                clearInterval(interval);

                cactus.remove();

                cactusList = cactusList.filter(c => c !== cactus);

            }

        }, 20);
    }


    /*
        =========================
        CEK TABRAKAN
        =========================
    */

    function checkCollision(cactus) {

        const dinoRect = dino.getBoundingClientRect();
        const cactusRect = cactus.getBoundingClientRect();

        if (
            dinoRect.left < cactusRect.right &&
            dinoRect.right > cactusRect.left &&
            dinoRect.top < cactusRect.bottom &&
            dinoRect.bottom > cactusRect.top
        ) {

            endGame();

        }
    }


    /*
        =========================
        GAME OVER
        =========================
    */

    function endGame() {

        gameRunning = false;

        finalScore.textContent = "Skor: " + score;

        gameOverScreen.style.display = "flex";

    }


    /*
        =========================
        SKOR
        =========================
    */

    setInterval(() => {

        if (!gameRunning) return;

        score++;

        scoreText.textContent = "Skor: " + score;

    }, 100);


    /*
        =========================
        SPAWN KAKTUS
        =========================
    */

    function spawnLoop() {

        if (!gameRunning) return;

        createCactus();

        const randomTime =
            Math.floor(Math.random() * 1200) + 800;

        setTimeout(spawnLoop, randomTime);
    }


    /*
        =========================
        RESTART
        =========================
    */

    function restartGame() {

        cactusList.forEach(cactus => cactus.remove());

        cactusList = [];

        score = 0;

        scoreText.textContent = "Skor: 0";

        gameRunning = true;

        gameOverScreen.style.display = "none";

        dino.style.bottom = "30px";

        spawnLoop();

    }


    /*
        =========================
        KEYBOARD
        =========================
    */

    document.addEventListener("keydown", function(event) {

        if (event.code === "Space") {

            event.preventDefault();

            if (!gameRunning) {
                restartGame();
            } else {
                jump();
            }

        }

    });


    /*
        =========================
        MOUSE / HP
        =========================
    */

    game.addEventListener("click", function() {

        if (gameRunning) {
            jump();
        }

    });


    /*
        =========================
        MULAI GAME
        =========================
    */

    spawnLoop();

</script>

</body>
</html>
