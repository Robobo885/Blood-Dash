# Blood-Dash
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Blood Dash</title>
    <style>
        body { margin: 0; overflow: hidden; background: #050000; font-family: 'Courier New', monospace; }
        canvas { display: block; }
        #ui {
            position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
            text-align: center; color: #8a0303; pointer-events: none;
        }
        .score { position: absolute; top: 20px; left: 20px; color: #fff; font-size: 24px; }
    </style>
</head>
<body>

    <div id="ui">
        <h1 id="msg">BLOOD DASH</h1>
        <p id="submsg">Кликни, чтобы прыгнуть</p>
    </div>
    <div class="score">Счет: <span id="scoreVal">0</span></div>

    <script>
        const canvas = document.createElement('canvas');
        const ctx = canvas.getContext('2d');
        document.body.appendChild(canvas);

        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;

        // Игровые переменные
        let gravity = 0.6;
        let isGameOver = false;
        let score = 0;
        let frames = 0;
        let gameStarted = false;

        const player = {
            x: 150,
            y: canvas.height / 2,
            radius: 15,
            velocity: 0,
            jumpForce: -10,
            draw() {
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = '#fff';
                ctx.shadowBlur = 20;
                ctx.shadowColor = '#fff';
                ctx.fill();
                ctx.closePath();
                ctx.shadowBlur = 0;
            },
            update() {
                if (!gameStarted) return;
                this.velocity += gravity;
                this.y += this.velocity;

                // Границы экрана
                if (this.y + this.radius > canvas.height || this.y - this.radius < 0) {
                    gameOver();
                }
            },
            jump() {
                this.velocity = this.jumpForce;
            }
        };

        const obstacles = [];
        class Obstacle {
            constructor() {
                this.width = 60;
                this.gap = 200;
                this.x = canvas.width;
                this.topHeight = Math.random() * (canvas.height - this.gap - 100) + 50;
                this.passed = false;
            }
            draw() {
                ctx.fillStyle = '#8a0303';
                // Верхняя колонна
                ctx.fillRect(this.x, 0, this.width, this.topHeight);
                // Нижняя колонна
                ctx.fillRect(this.x, this.topHeight + this.gap, this.width, canvas.height);
                
                // Рисуем "зубы" (шипы)
                ctx.beginPath();
                ctx.moveTo(this.x, this.topHeight);
                ctx.lineTo(this.x + this.width/2, this.topHeight + 20);
                ctx.lineTo(this.x + this.width, this.topHeight);
                ctx.fill();
            }
            update() {
                this.x -= 5 + (score * 0.1); // Ускорение со временем
            }
        }

        function spawnObstacle() {
            if (frames % 100 === 0 && gameStarted) {
                obstacles.push(new Obstacle());
            }
        }

        function checkCollision() {
            obstacles.forEach(obs => {
                if (
                    player.x + player.radius > obs.x &&
                    player.x - player.radius < obs.x + obs.width
                ) {
                    if (player.y - player.radius < obs.topHeight || 
                        player.y + player.radius > obs.topHeight + obs.gap) {
                        gameOver();
                    }
                }
                if (!obs.passed && obs.x + obs.width < player.x) {
                    score++;
                    obs.passed = true;
                    document.getElementById('scoreVal').innerText = score;
                }
            });
        }

        function gameOver() {
            isGameOver = true;
            gameStarted = false;
            document.getElementById('msg').innerText = "ТЫ ПОГИБ";
            document.getElementById('submsg').innerText = "Кликни, чтобы восстать";
            document.getElementById('ui').style.display = "block";
        }

        function resetGame() {
            player.y = canvas.height / 2;
            player.velocity = 0;
            obstacles.length = 0;
            score = 0;
            frames = 0;
            isGameOver = false;
            gameStarted = true;
            document.getElementById('scoreVal').innerText = "0";
            document.getElementById('ui').style.display = "none";
        }

        function loop() {
            ctx.fillStyle = 'rgba(5, 0, 0, 0.3)'; // Эффект шлейфа
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            player.update();
            player.draw();

            spawnObstacle();
            obstacles.forEach((obs, index) => {
                obs.update();
                obs.draw();
                if (obs.x + obs.width < 0) obstacles.splice(index, 1);
            });

            if (!isGameOver) {
                checkCollision();
                frames++;
                requestAnimationFrame(loop);
            }
        }

        window.addEventListener('mousedown', () => {
            if (isGameOver) {
                resetGame();
                loop();
            } else if (!gameStarted) {
                resetGame();
                loop();
            } else {
                player.jump();
            }
        });

        window.addEventListener('keydown', (e) => {
            if (e.code === 'Space') {
                if (isGameOver || !gameStarted) { resetGame(); loop(); }
                else player.jump();
            }
        });

        loop();
    </script>
</body>
</html>
