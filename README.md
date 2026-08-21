
<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Yılan Oyunu</title>

<style>
    * {
        box-sizing: border-box;
    }

    body {
        margin: 0;
        background: #111;
        color: white;
        font-family: Arial, sans-serif;
        text-align: center;
    }

    h1 {
        margin: 20px 0 5px;
    }

    #score {
        font-size: 22px;
        margin-bottom: 15px;
    }

    canvas {
        background: #1b1b1b;
        border: 3px solid #00ff66;
        max-width: 95vw;
        height: auto;
    }

    #controls {
        margin: 20px auto;
        width: 220px;
        display: grid;
        grid-template-columns: 70px 70px 70px;
        gap: 8px;
        justify-content: center;
    }

    button {
        height: 60px;
        font-size: 25px;
        border: none;
        border-radius: 12px;
        background: #222;
        color: white;
    }

    button:active {
        background: #00aa44;
    }

    #up {
        grid-column: 2;
    }

    #left {
        grid-column: 1;
        grid-row: 2;
    }

    #down {
        grid-column: 2;
        grid-row: 2;
    }

    #right {
        grid-column: 3;
        grid-row: 2;
    }

    #restart {
        margin-top: 10px;
        padding: 12px 25px;
        font-size: 18px;
        background: #00cc55;
        color: white;
    }
</style>
</head>

<body>

<h1>🐍 YILAN OYUNU</h1>
<div id="score">Skor: 0</div>

<canvas id="game" width="400" height="400"></canvas>

<div id="controls">
    <button id="up">⬆️</button>
    <button id="left">⬅️</button>
    <button id="down">⬇️</button>
    <button id="right">➡️</button>
</div>

<button id="restart">🔄 Yeniden Başlat</button>

<script>

const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

const box = 20;

let snake;
let food;
let direction;
let nextDirection;
let score;
let game;

function startGame() {

    snake = [
        {x: 200, y: 200},
        {x: 180, y: 200},
        {x: 160, y: 200}
    ];

    direction = "RIGHT";
    nextDirection = "RIGHT";
    score = 0;

    document.getElementById("score").innerText = "Skor: 0";

    createFood();

    clearInterval(game);
    game = setInterval(draw, 100);
}

function createFood() {

    food = {
        x: Math.floor(Math.random() * (canvas.width / box)) * box,
        y: Math.floor(Math.random() * (canvas.height / box)) * box
    };

    // Yemeğin yılanın üstüne gelmesini engelle
    for (let part of snake) {
        if (part.x === food.x && part.y === food.y) {
            createFood();
            return;
        }
    }
}

function draw() {

    ctx.fillStyle = "#1b1b1b";
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    direction = nextDirection;

    let head = {
        x: snake[0].x,
        y: snake[0].y
    };

    if (direction === "UP") head.y -= box;
    if (direction === "DOWN") head.y += box;
    if (direction === "LEFT") head.x -= box;
    if (direction === "RIGHT") head.x += box;

    // Duvara çarpma
    if (
        head.x < 0 ||
        head.x >= canvas.width ||
        head.y < 0 ||
        head.y >= canvas.height
    ) {
        gameOver();
        return;
    }

    // Kendine çarpma
    for (let part of snake) {
        if (head.x === part.x && head.y === part.y) {
            gameOver();
            return;
        }
    }

    snake.unshift(head);

    // Yemek
    if (head.x === food.x && head.y === food.y) {

        score++;

        document.getElementById("score").innerText =
            "Skor: " + score;

        createFood();

    } else {
        snake.pop();
    }

    // Yılanı çiz
    snake.forEach((part, index) => {

        ctx.fillStyle = index === 0
            ? "#00ff66"
            : "#00bb55";

        ctx.fillRect(
            part.x + 1,
            part.y + 1,
            box - 2,
            box - 2
        );
    });

    // Yemek çiz
    ctx.fillStyle = "red";
    ctx.beginPath();

    ctx.arc(
        food.x + box / 2,
        food.y + box / 2,
        box / 2 - 2,
        0,
        Math.PI * 2
    );

    ctx.fill();
}

function gameOver() {

    clearInterval(game);

    setTimeout(() => {
        alert("Oyun Bitti! 🐍\nSkorun: " + score);
    }, 100);
}

function changeDirection(newDirection) {

    if (newDirection === "UP" && direction !== "DOWN")
        nextDirection = "UP";

    if (newDirection === "DOWN" && direction !== "UP")
        nextDirection = "DOWN";

    if (newDirection === "LEFT" && direction !== "RIGHT")
        nextDirection = "LEFT";

    if (newDirection === "RIGHT" && direction !== "LEFT")
        nextDirection = "RIGHT";
}

// Klavye kontrolleri
document.addEventListener("keydown", function(event) {

    if (event.key === "ArrowUp")
        changeDirection("UP");

    if (event.key === "ArrowDown")
        changeDirection("DOWN");

    if (event.key === "ArrowLeft")
        changeDirection("LEFT");

    if (event.key === "ArrowRight")
        changeDirection("RIGHT");
});

// Telefon kontrolleri
document.getElementById("up").onclick =
    () => changeDirection("UP");

document.getElementById("down").onclick =
    () => changeDirection("DOWN");

document.getElementById("left").onclick =
    () => changeDirection("LEFT");

document.getElementById("right").onclick =
    () => changeDirection("RIGHT");

document.getElementById("restart").onclick =
    () => startGame();

startGame();

</script>

</body>
</html>
