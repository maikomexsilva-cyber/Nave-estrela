<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>Nave Estelar</title>

<style>
* {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
}

body {
    overflow: hidden;
    background: #050816;
    font-family: Arial, sans-serif;
    color: white;
    touch-action: none;
}

canvas {
    display: block;
    width: 100vw;
    height: 100vh;
}

#hud {
    position: fixed;
    top: 15px;
    left: 15px;
    right: 15px;
    display: flex;
    justify-content: space-between;
    font-size: 18px;
    font-weight: bold;
    z-index: 5;
    text-shadow: 0 2px 5px black;
}

#menu {
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 20, 0.82);
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    text-align: center;
    z-index: 10;
}

#menu h1 {
    font-size: 42px;
    margin-bottom: 10px;
    color: #55d9ff;
    text-shadow: 0 0 20px #55d9ff;
}

#menu p {
    font-size: 17px;
    margin-bottom: 25px;
}

button {
    border: none;
    border-radius: 15px;
    padding: 15px 35px;
    font-size: 20px;
    font-weight: bold;
    background: #55d9ff;
    color: #03101a;
    cursor: pointer;
    box-shadow: 0 0 25px #55d9ff;
}

button:active {
    transform: scale(0.95);
}

#help {
    margin-top: 20px;
    font-size: 13px;
    opacity: 0.7;
}
</style>
</head>

<body>

<div id="hud">
    <div id="score">Pontos: 0</div>
    <div id="lives">❤️❤️❤️</div>
</div>

<div id="menu">
    <h1 id="title">🚀 NAVE ESTELAR</h1>

    <p id="message">
        Destrua os inimigos e sobreviva o máximo possível!
    </p>

    <button id="start">JOGAR</button>

    <div id="help">
        📱 Arraste a nave com o dedo<br>
        💻 Use ← → no computador<br>
        🔫 A nave dispara automaticamente
    </div>
</div>

<canvas id="game"></canvas>

<script>

const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

let width;
let height;

function resize() {
    width = canvas.width = window.innerWidth;
    height = canvas.height = window.innerHeight;
}

resize();
window.addEventListener("resize", resize);


// ==========================
// ESTADO DO JOGO
// ==========================

let playing = false;

let score = 0;
let lives = 3;

let enemies = [];
let bullets = [];
let particles = [];
let stars = [];

let enemyTimer = 0;
let bulletTimer = 0;

let lastTime = 0;


// ==========================
// NAVE
// ==========================

const player = {
    x: width / 2,
    y: height - 100,
    width: 42,
    height: 55,
    speed: 7
};


// ==========================
// ESTRELAS
// ==========================

function createStars() {

    stars = [];

    for (let i = 0; i < 100; i++) {

        stars.push({
            x: Math.random() * width,
            y: Math.random() * height,
            size: Math.random() * 2 + 1,
            speed: Math.random() * 2 + 1
        });

    }
}

createStars();


// ==========================
// MENU
// ==========================

const menu = document.getElementById("menu");
const startButton = document.getElementById("start");

startButton.addEventListener("click", startGame);


// ==========================
// COMEÇAR
// ==========================

function startGame() {

    playing = true;

    score = 0;
    lives = 3;

    enemies = [];
    bullets = [];
    particles = [];

    player.x = width / 2;
    player.y = height - 100;

    enemyTimer = 0;
    bulletTimer = 0;

    menu.style.display = "none";

    updateHUD();
}


// ==========================
// GAME OVER
// ==========================

function gameOver() {

    playing = false;

    document.getElementById("title").textContent = "💥 FIM DE JOGO";

    document.getElementById("message").textContent =
        "Você fez " + score + " pontos!";

    startButton.textContent = "JOGAR NOVAMENTE";

    menu.style.display = "flex";
}


// ==========================
// HUD
// ==========================

function updateHUD() {

    document.getElementById("score").textContent =
        "Pontos: " + score;

    document.getElementById("lives").textContent =
        "❤️".repeat(lives);
}


// ==========================
// CRIAR INIMIGO
// ==========================

function createEnemy() {

    const size = 28 + Math.random() * 20;

    enemies.push({

        x: size + Math.random() * (width - size * 2),

        y: -size,

        size: size,

        speed: 2 + Math.random() * 2 + score / 500

    });
}


// ==========================
// ATIRAR
// ==========================

function shoot() {

    bullets.push({

        x: player.x,

        y: player.y - 30,

        speed: 10

    });
}


// ==========================
// EXPLOSÃO
// ==========================

function explosion(x, y) {

    for (let i = 0; i < 20; i++) {

        particles.push({

            x: x,

            y: y,

            vx: (Math.random() - 0.5) * 7,

            vy: (Math.random() - 0.5) * 7,

            life: 1

        });

    }
}


// ==========================
// COLISÃO
// ==========================

function collision(a, b) {

    return (

        Math.abs(a.x - b.x) <
        (a.width + b.size) / 2

        &&

        Math.abs(a.y - b.y) <
        (a.height + b.size) / 2

    );

}


// ==========================
// ATUALIZAÇÃO
// ==========================

function update(delta) {

    if (!playing) return;


    // ESTRELAS

    for (const star of stars) {

        star.y += star.speed;

        if (star.y > height) {

            star.y = 0;
            star.x = Math.random() * width;

        }

    }


    // INIMIGOS

    enemyTimer -= delta;

    if (enemyTimer <= 0) {

        createEnemy();

        enemyTimer =
            Math.max(300, 900 - score * 2);

    }


    // TIROS

    bulletTimer -= delta;

    if (bulletTimer <= 0) {

        shoot();

        bulletTimer = 280;

    }


    for (const bullet of bullets) {

        bullet.y -= bullet.speed;

    }

    bullets =
        bullets.filter(bullet => bullet.y > -20);


    // INIMIGOS DESCENDO

    for (const enemy of enemies) {

        enemy.y += enemy.speed;

    }


    // COLISÕES

    for (let i = enemies.length - 1; i >= 0; i--) {

        const enemy = enemies[i];

        let destroyed = false;


        // TIRO CONTRA INIMIGO

        for (let j = bullets.length - 1; j >= 0; j--) {

            const bullet = bullets[j];

            const distance = Math.hypot(
                bullet.x - enemy.x,
                bullet.y - enemy.y
            );

            if (distance < enemy.size / 2 + 8) {

                bullets.splice(j, 1);
                enemies.splice(i, 1);

                score += 10;

                explosion(enemy.x, enemy.y);

                destroyed = true;

                break;
            }

        }


        if (destroyed) continue;


        // INIMIGO ATINGIU A NAVE

        if (collision(player, enemy)) {

            enemies.splice(i, 1);

            lives--;

            explosion(player.x, player.y);

            updateHUD();

            if (lives <= 0) {

                gameOver();
                return;

            }

            continue;
        }


        // INIMIGO PASSOU

        if (enemy.y > height + 50) {

            enemies.splice(i, 1);

            lives--;

            updateHUD();

            if (lives <= 0) {

                gameOver();
                return;

            }

        }

    }


    // PARTÍCULAS

    for (const particle of particles) {

        particle.x += particle.vx;
        particle.y += particle.vy;

        particle.life -= 0.025;

    }

    particles =
        particles.filter(p => p.life > 0);


    updateHUD();
}


// ==========================
// DESENHAR
// ==========================

function draw() {

    ctx.clearRect(0, 0, width, height);


    // FUNDO

    const gradient =
        ctx.createLinearGradient(
            0,
            0,
            0,
            height
        );

    gradient.addColorStop(0, "#03051a");
    gradient.addColorStop(1, "#111b45");

    ctx.fillStyle = gradient;

    ctx.fillRect(
        0,
        0,
        width,
        height
    );


    // ESTRELAS

    ctx.fillStyle = "white";

    for (const star of stars) {

        ctx.globalAlpha = 0.4 + star.size / 3;

        ctx.fillRect(
            star.x,
            star.y,
            star.size,
            star.size
        );

    }

    ctx.globalAlpha = 1;


    // NAVE

    drawPlayer();


    // TIROS

    for (const bullet of bullets) {

        ctx.fillStyle = "#fff45c";

        ctx.shadowBlur = 15;
        ctx.shadowColor = "#fff45c";

        ctx.fillRect(
            bullet.x - 3,
            bullet.y - 12,
            6,
            18
        );

        ctx.shadowBlur = 0;

    }


    // INIMIGOS

    for (const enemy of enemies) {

        drawEnemy(enemy);

    }


    // EXPLOSÕES

    for (const particle of particles) {

        ctx.globalAlpha =
            particle.life;

        ctx.fillStyle =
            "#ffb52e";

        ctx.fillRect(
            particle.x,
            particle.y,
            5,
            5
        );

    }

    ctx.globalAlpha = 1;

}


// ==========================
// DESENHAR NAVE
// ==========================

function drawPlayer() {

    ctx.save();

    ctx.translate(
        player.x,
        player.y
    );

    ctx.shadowBlur = 20;
    ctx.shadowColor = "#50d9ff";


    // CORPO DA NAVE

    ctx.fillStyle = "#45d9ff";

    ctx.beginPath();

    ctx.moveTo(0, -30);

    ctx.lineTo(22, 25);

    ctx.lineTo(0, 15);

    ctx.lineTo(-22, 25);

    ctx.closePath();

    ctx.fill();


    // JANELA

    ctx.fillStyle = "white";

    ctx.beginPath();

    ctx.arc(
        0,
        -8,
        7,
        0,
        Math.PI * 2
    );

    ctx.fill();


    // MOTOR

    ctx.fillStyle = "#ffcf3f";

    ctx.beginPath();

    ctx.moveTo(-7, 18);
    ctx.lineTo(0, 34);
    ctx.lineTo(7, 18);

    ctx.closePath();

    ctx.fill();


    ctx.restore();

}


// ==========================
// DESENHAR INIMIGO
// ==========================

function drawEnemy(enemy) {

    ctx.save();

    ctx.translate(
        enemy.x,
        enemy.y
    );

    ctx.shadowBlur = 15;
    ctx.shadowColor = "#ff315d";

    ctx.fillStyle = "#ff4568";

    ctx.beginPath();

    ctx.arc(
        0,
        0,
        enemy.size / 2,
        0,
        Math.PI * 2
    );

    ctx.fill();


    // OLHOS

    ctx.fillStyle = "#180915";

    ctx.beginPath();

    ctx.arc(
        -enemy.size * 0.18,
        -3,
        4,
        0,
        Math.PI * 2
    );

    ctx.arc(
        enemy.size * 0.18,
        -3,
        4,
        0,
        Math.PI * 2
    );

    ctx.fill();


    ctx.restore();

}


// ==========================
// CONTROLE TOUCH
// ==========================

let touching = false;

canvas.addEventListener(
    "pointerdown",
    function(event) {

        touching = true;

        movePlayer(
            event.clientX,
            event.clientY
        );

    }
);


canvas.addEventListener(
    "pointermove",
    function(event) {

        if (!touching) return;

        movePlayer(
            event.clientX,
            event.clientY
        );

    }
);


canvas.addEventListener(
    "pointerup",
    function() {

        touching = false;

    }
);


canvas.addEventListener(
    "pointercancel",
    function() {

        touching = false;

    }
);


function movePlayer(x, y) {

    player.x = Math.max(
        25,
        Math.min(
            width - 25,
            x
        )
    );

    player.y = Math.max(
        height * 0.45,
        Math.min(
            height - 50,
            y
        )
    );

}


// ==========================
// TECLADO
// ==========================

window.addEventListener(
    "keydown",
    function(event) {

        if (!playing) return;

        if (event.key === "ArrowLeft") {

            player.x -= player.speed * 5;

        }

        if (event.key === "ArrowRight") {

            player.x += player.speed * 5;

        }

        player.x = Math.max(
            25,
            Math.min(
                width - 25,
                player.x
            )
        );

    }
);


// ==========================
// LOOP
// ==========================

function gameLoop(time) {

    const delta =
        time - lastTime;

    lastTime = time;

    update(delta);

    draw();

    requestAnimationFrame(gameLoop);

}

requestAnimationFrame(gameLoop);

</script>

</body>
</html>
