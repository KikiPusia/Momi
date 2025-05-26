<!DOCTYPE html>
<html lang="pl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pszczela Przygoda: Królowa Ula</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background-color: #87CEEB; /* Jasnoniebieskie tło nieba */
            font-family: 'Arial', sans-serif;
            color: #333;
        }

        #game-container {
            display: flex;
            border: 5px solid #5C4033;
            border-radius: 10px;
            box-shadow: 0 0 20px rgba(0, 0, 0, 0.5);
            background-color: #fff;
            overflow: hidden;
            position: relative; /* Dla pozycjonowania elementów wewnątrz */
        }

        #gameCanvas {
            background-color: #A3D97F; /* Jasnozielone tło trawy */
            display: block;
            border-right: 2px solid #5C4033;
        }

        #game-info {
            padding: 20px;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            background-color: #F0E68C; /* Kolor miodu */
            width: 200px; /* Szerokość panelu info */
        }

        #game-info p {
            margin: 10px 0;
            font-size: 1.2em;
            font-weight: bold;
        }

        #startButton, #restartButton {
            background-color: #FFD700; /* Złoty kolor */
            color: #8B4513; /* Brązowy tekst */
            border: 2px solid #8B4513;
            padding: 15px 25px;
            font-size: 1.5em;
            font-weight: bold;
            cursor: pointer;
            border-radius: 8px;
            transition: background-color 0.3s ease;
            margin-top: 20px;
        }

        #startButton:hover, #restartButton:hover {
            background-color: #FFA500; /* Pomarańczowy po najechaniu */
        }

        #instructions {
            margin-top: 20px;
            padding: 10px;
            border: 1px dashed #8B4513;
            border-radius: 5px;
            background-color: #FFFACD;
        }

        #instructions h2 {
            color: #8B4513;
            text-align: center;
            margin-bottom: 10px;
        }

        #game-over-screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.7);
            color: white;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            text-align: center;
            z-index: 10;
        }

        .hidden {
            display: none !important;
        }

        #touch-controls {
            position: absolute;
            bottom: 10px;
            left: 10px;
            display: grid;
            grid-template-columns: repeat(3, 60px);
            grid-template-rows: repeat(2, 60px);
            gap: 5px;
            z-index: 5; /* Nad canvasem, ale pod game-over-screen */
            display: none; /* Domyślnie ukryte, pokazane po starcie gry */
        }

        #touch-controls button {
            background-color: rgba(255, 215, 0, 0.7); /* Półprzezroczysty złoty */
            border: 2px solid #8B4513;
            color: #8B4513;
            font-size: 1.8em;
            font-weight: bold;
            border-radius: 5px;
            cursor: pointer;
            display: flex;
            justify-content: center;
            align-items: center;
            user-select: none; /* Zapobiega zaznaczaniu tekstu */
            -webkit-tap-highlight-color: rgba(0,0,0,0); /* Usuwa niebieskie podświetlenie na dotyku */
        }

        #touch-controls button:active {
            background-color: rgba(255, 165, 0, 0.9); /* Pomarańczowy po naciśnięciu */
        }

        #touch-up { grid-column: 2; grid-row: 1; }
        #touch-left { grid-column: 1; grid-row: 2; }
        #touch-right { grid-column: 3; grid-row: 2; }
        #touch-down { grid-column: 2; grid-row: 2; }

    </style>
</head>
<body>
    <div id="game-container">
        <canvas id="gameCanvas"></canvas>
        <div id="game-info">
            <p>Pyłek: <span id="pollenCount">0</span></p>
            <p>Wynik: <span id="score">0</span></p>
            <p>Życia: <span id="lives">3</span></p>
            <button id="startButton">Start Gry</button>
            <div id="instructions">
                <h2>Instrukcje</h2>
                <p>Użyj klawiszy strzałek (lub WSAD), aby sterować pszczołą.</p>
                <p>Zbieraj kwiaty, aby zdobyć pyłek i dostarcz go do ula.</p>
                <p>Unikaj pająków!</p>
                <p>Na urządzeniach dotykowych użyj przycisków na ekranie.</p>
            </div>
        </div>
        <div id="game-over-screen" class="hidden">
            <h2>Koniec Gry!</h2>
            <p>Twój wynik: <span id="finalScore">0</span></p>
            <button id="restartButton">Zagraj ponownie</button>
        </div>

        <div id="touch-controls">
            <button id="touch-up">↑</button>
            <button id="touch-left">←</button>
            <button id="touch-right">→</button>
            <button id="touch-down">↓</button>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const pollenCountSpan = document.getElementById('pollenCount');
        const scoreSpan = document.getElementById('score');
        const livesSpan = document.getElementById('lives');
        const startButton = document.getElementById('startButton');
        const gameOverScreen = document.getElementById('game-over-screen');
        const finalScoreSpan = document.getElementById('finalScore');
        const restartButton = document.getElementById('restartButton');
        const instructionsDiv = document.getElementById('instructions');
        const touchControls = document.getElementById('touch-controls');

        const touchUpBtn = document.getElementById('touch-up');
        const touchLeftBtn = document.getElementById('touch-left');
        const touchRightBtn = document.getElementById('touch-right');
        const touchDownBtn = document.getElementById('touch-down');


        canvas.width = 800;
        canvas.height = 600;

        let gameStarted = false;
        let score = 0;
        let pollen = 0;
        let lives = 3;
        let gameOver = false;
        let animationFrameId;

        // Pszczoła
        const bee = {
            x: 50,
            y: canvas.height / 2,
            width: 40,
            height: 30,
            speed: 5,
            dx: 0,
            dy: 0,
            image: new Image()
        };
        bee.image.src = 'https://raw.githubusercontent.com/mdabrowski101/Pszczela-Przygoda/main/bee.png';

        // Ul
        const hive = {
            x: 0,
            y: canvas.height / 2 - 50,
            width: 60,
            height: 100,
            image: new Image()
        };
        hive.image.src = 'https://raw.githubusercontent.com/mdabrowski101/Pszczela-Przygoda/main/hive.png';

        // Kwiaty
        let flowers = [];
        const flowerImage = new Image();
        flowerImage.src = 'https://raw.githubusercontent.com/mdabrowski101/Pszczela-Przygoda/main/flower.png';

        // Pająki
        let spiders = [];
        const spiderImage = new Image();
        spiderImage.src = 'https://raw.githubusercontent.com/mdabrowski101/Pszczela-Przygoda/main/spider.png';

        // Dźwięki
        const hitSound = new Audio('https://raw.githubusercontent.com/mdabrowski101/Pszczela-Przygoda/main/hit.mp3');
        const deliverPollenSound = new Audio('https://raw.githubusercontent.com/mdabrowski101/Pszczela-Przygoda/main/deliver.mp3');
        const gameOverSound = new Audio('https://raw.githubusercontent.com/mdabrowski101/Pszczela-Przygoda/main/gameover.mp3');


        function drawBee() {
            ctx.drawImage(bee.image, bee.x, bee.y, bee.width, bee.height);
        }

        function drawHive() {
            ctx.drawImage(hive.image, hive.x, hive.y, hive.width, hive.height);
        }

        function drawFlowers() {
            flowers.forEach(flower => {
                ctx.drawImage(flowerImage, flower.x, flower.y, flower.width, flower.height);
            });
        }

        function drawSpiders() {
            spiders.forEach(spider => {
                ctx.drawImage(spiderImage, spider.x, spider.y, spider.width, spider.height);
            });
        }

        function update() {
            if (gameOver || !gameStarted) return;

            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Ruch pszczoły
            bee.x += bee.dx;
            bee.y += bee.dy;

            // Ograniczenie ruchu pszczoły
            if (bee.x < 0) bee.x = 0;
            if (bee.x + bee.width > canvas.width) bee.x = canvas.width - bee.width;
            if (bee.y < 0) bee.y = 0;
            if (bee.y + bee.height > canvas.height) bee.y = canvas.height - bee.height;

            drawHive();
            drawBee();
            drawFlowers();
            drawSpiders();

            // Aktualizacja i ruch pająków
            spiders.forEach(spider => {
                spider.x += spider.dx;
                spider.y += spider.dy;

                // Odbijanie od krawędzi
                if (spider.x + spider.width > canvas.width || spider.x < 0) {
                    spider.dx *= -1;
                }
                if (spider.y + spider.height > canvas.height || spider.y < 0) {
                    spider.dy *= -1;
                }

                // Kolizja z pszczołą
                if (
                    bee.x < spider.x + spider.width &&
                    bee.x + bee.width > spider.x &&
                    bee.y < spider.y + spider.height &&
                    bee.y + bee.height > spider.y
                ) {
                    hitSound.play();
                    lives--;
                    livesSpan.textContent = lives;
                    resetBeePosition(); // Przenieś pszczołę w bezpieczne miejsce
                    if (lives <= 0) {
                        endGame();
                    }
                }
            });

            // Kolizje z kwiatami
            flowers.forEach((flower, index) => {
                if (
                    bee.x < flower.x + flower.width &&
                    bee.x + bee.width > flower.x &&
                    bee.y < flower.y + flower.height &&
                    bee.y + bee.height > flower.y
                ) {
                    pollen++;
                    pollenCountSpan.textContent = pollen;
                    flowers.splice(index, 1); // Usuń zebrany kwiat
                    spawnFlower(); // Dodaj nowy kwiat
                    score += 10; // Dodaj punkty za zebranie pyłku
                    scoreSpan.textContent = score;
                }
            });

            // Kolizja z ulem
            if (
                bee.x < hive.x + hive.width &&
                bee.x + bee.width > hive.x &&
                bee.y < hive.y + hive.height &&
                bee.y + bee.height > hive.y
            ) {
                if (pollen > 0) {
                    deliverPollenSound.play();
                    score += pollen * 20; // Więcej punktów za dostarczenie
                    pollen = 0;
                    pollenCountSpan.textContent = pollen;
                    scoreSpan.textContent = score;
                }
            }

            animationFrameId = requestAnimationFrame(update);
        }

        function resetBeePosition() {
            bee.x = 50;
            bee.y = canvas.height / 2;
            bee.dx = 0;
            bee.dy = 0;
        }

        function spawnFlower() {
            const x = Math.random() * (canvas.width - 50 - hive.width) + hive.width;
            const y = Math.random() * (canvas.height - 50);
            flowers.push({ x, y, width: 30, height: 30 });
        }

        function spawnSpider() {
            const x = Math.random() * (canvas.width - 50 - hive.width) + hive.width;
            const y = Math.random() * (canvas.height - 50);
            const dx = (Math.random() - 0.5) * 3;
            const dy = (Math.random() - 0.5) * 3;
            spiders.push({ x, y, width: 50, height: 50, dx, dy });
        }

        function initGame() {
            score = 0;
            pollen = 0;
            lives = 3;
            gameOver = false;
            gameStarted = true;

            pollenCountSpan.textContent = pollen;
            scoreSpan.textContent = score;
            livesSpan.textContent = lives;
            gameOverScreen.classList.add('hidden');
            startButton.classList.add('hidden');
            instructionsDiv.classList.add('hidden');
            touchControls.style.display = 'grid'; // Pokaż przyciski dotykowe

            flowers = [];
            spiders = [];

            resetBeePosition();

            for (let i = 0; i < 5; i++) {
                spawnFlower();
            }
            for (let i = 0; i < 2; i++) {
                spawnSpider();
            }

            setInterval(spawnSpider, 15000);
            setInterval(spawnFlower, 5000);

            if (animationFrameId) {
                cancelAnimationFrame(animationFrameId);
            }
            update();
        }

        function endGame() {
            gameOver = true;
            gameStarted = false;
            cancelAnimationFrame(animationFrameId);
            finalScoreSpan.textContent = score;
            gameOverScreen.classList.remove('hidden');
            gameOverSound.play();
            touchControls.style.display = 'none'; // Ukryj przyciski dotykowe
        }

        // --- Kontrola klawiatury ---
        document.addEventListener('keydown', e => {
            if (!gameStarted) return;
            if (e.key === 'ArrowRight' || e.key === 'd') bee.dx = bee.speed;
            if (e.key === 'ArrowLeft' || e.key === 'a') bee.dx = -bee.speed;
            if (e.key === 'ArrowUp' || e.key === 'w') bee.dy = -bee.speed;
            if (e.key === 'ArrowDown' || e.key === 's') bee.dy = bee.speed;
        });

        document.addEventListener('keyup', e => {
            if (!gameStarted) return;
            if (
                e.key === 'ArrowRight' ||
                e.key === 'd' ||
                e.key === 'ArrowLeft' ||
                e.key === 'a'
            ) {
                bee.dx = 0;
            }
            if (
                e.key === 'ArrowUp' ||
                e.key === 'w' ||
                e.key === 'ArrowDown' ||
                e.key === 's'
            ) {
                bee.dy = 0;
            }
        });

        // --- Kontrola dotykowa ---
        touchUpBtn.addEventListener('touchstart', () => { if (gameStarted) bee.dy = -bee.speed; });
        touchUpBtn.addEventListener('touchend', () => { if (gameStarted) bee.dy = 0; });

        touchLeftBtn.addEventListener('touchstart', () => { if (gameStarted) bee.dx = -bee.speed; });
        touchLeftBtn.addEventListener('touchend', () => { if (gameStarted) bee.dx = 0; });

        touchRightBtn.addEventListener('touchstart', () => { if (gameStarted) bee.dx = bee.speed; });
        touchRightBtn.addEventListener('touchend', () => { if (gameStarted) bee.dx = 0; });

        touchDownBtn.addEventListener('touchstart', () => { if (gameStarted) bee.dy = bee.speed; });
        touchDownBtn.addEventListener('touchend', () => { if (gameStarted) bee.dy = 0; });

        // Dodatkowe zdarzenia touchstart/touchend dla myszki na desktopie
        // Choć głównie dla dotyku, 'mousedown' i 'mouseup' są często używane do testowania na desktopie
        touchUpBtn.addEventListener('mousedown', () => { if (gameStarted) bee.dy = -bee.speed; });
        touchUpBtn.addEventListener('mouseup', () => { if (gameStarted) bee.dy = 0; });

        touchLeftBtn.addEventListener('mousedown', () => { if (gameStarted) bee.dx = -bee.speed; });
        touchLeftBtn.addEventListener('mouseup', () => { if (gameStarted) bee.dx = 0; });

        touchRightBtn.addEventListener('mousedown', () => { if (gameStarted) bee.dx = bee.speed; });
        touchRightBtn.addEventListener('mouseup', () => { if (gameStarted) bee.dx = 0; });

        touchDownBtn.addEventListener('mousedown', () => { if (gameStarted) bee.dy = bee.speed; });
        touchDownBtn.addEventListener('mouseup', () => { if (gameStarted) bee.dy = 0; });


        startButton.addEventListener('click', initGame);
        restartButton.addEventListener('click', initGame);

        // Początkowe rysowanie ula i pszczoły, zanim gra się rozpocznie
        bee.image.onload = () => {
            ctx.drawImage(bee.image, bee.x, bee.y, bee.width, bee.height);
        };
        hive.image.onload = () => {
            ctx.drawImage(hive.image, hive.x, hive.y, hive.width, hive.height);
        };

        // Rysuj ul i pszczołę na początku (przed startem gry)
        drawHive();
        drawBee();
    </script>
</body>
</html>
