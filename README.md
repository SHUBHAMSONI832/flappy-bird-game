<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Flappy Bird Game</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            background: linear-gradient(135deg, #87CEEB, #98FB98);
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            font-family: Arial, sans-serif;
        }
        
        #gameContainer {
            text-align: center;
            background: rgba(255, 255, 255, 0.1);
            padding: 20px;
            border-radius: 15px;
            box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
        }
        
        canvas {
            border: 3px solid #2E8B57;
            border-radius: 10px;
            background: linear-gradient(to bottom, #87CEEB 0%, #98FB98 100%);
            display: block;
            margin: 0 auto;
        }
        
        #instructions {
            color: white;
            margin-top: 15px;
            font-size: 18px;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
        }
        
        #title {
            color: #FFD700;
            font-size: 32px;
            font-weight: bold;
            margin-bottom: 15px;
            text-shadow: 3px 3px 6px rgba(0, 0, 0, 0.7);
        }
    </style>
</head>
<body>
    <div id="gameContainer">
        <div id="title">🐦 Flappy Bird 🐦</div>
        <canvas id="gameCanvas" width="400" height="600"></canvas>
        <div id="instructions">Press SPACEBAR to jump! 🚀</div>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        let gameState = 'start';
        let bird = {
            x: 80, y: 200, width: 30, height: 25,
            velocity: 0, gravity: 0.4, jumpPower: -8, color: '#FFD700'
        };

        let pipes = [];
        let score = 0;
        let bestScore = localStorage.getItem('bestScore') || 0;
        let frameCount = 0;

        const pipeWidth = 60;
        const pipeGap = 160;
        const pipeSpeed = 2;

        function drawBird() {
            ctx.fillStyle = bird.color;
            ctx.fillRect(bird.x, bird.y, bird.width, bird.height);
            
            ctx.fillStyle = 'white';
            ctx.fillRect(bird.x + 20, bird.y + 5, 8, 8);
            ctx.fillStyle = 'black';
            ctx.fillRect(bird.x + 22, bird.y + 7, 4, 4);
            
            ctx.fillStyle = '#FFA500';
            ctx.fillRect(bird.x + 30, bird.y + 12, 8, 4);
        }

        function drawPipe(x, topHeight) {
            ctx.fillStyle = '#228B22';
            ctx.fillRect(x, 0, pipeWidth, topHeight);
            ctx.fillRect(x, topHeight + pipeGap, pipeWidth, canvas.height - topHeight - pipeGap);
            
            ctx.fillStyle = '#32CD32';
            ctx.fillRect(x - 5, topHeight - 20, pipeWidth + 10, 20);
            ctx.fillRect(x - 5, topHeight + pipeGap, pipeWidth + 10, 20);
        }

        function createPipe() {
            const minHeight = 50;
            const maxHeight = canvas.height - pipeGap - 50;
            const topHeight = Math.random() * (maxHeight - minHeight) + minHeight;
            pipes.push({ x: canvas.width, topHeight: topHeight, passed: false });
        }

        function updatePipes() {
            if (frameCount % 120 === 0) createPipe();
            
            for (let i = pipes.length - 1; i >= 0; i--) {
                pipes[i].x -= pipeSpeed;
                
                if (!pipes[i].passed && pipes[i].x + pipeWidth < bird.x) {
                    pipes[i].passed = true;
                    score++;
                }
                
                if (pipes[i].x + pipeWidth < 0) pipes.splice(i, 1);
            }
        }

        function checkCollision() {
            if (bird.y + bird.height > canvas.height || bird.y < 0) return true;
            
            for (let pipe of pipes) {
                if (bird.x < pipe.x + pipeWidth && bird.x + bird.width > pipe.x &&
                    (bird.y < pipe.topHeight || bird.y + bird.height > pipe.topHeight + pipeGap)) {
                    return true;
                }
            }
            return false;
        }

        function drawScore() {
            ctx.fillStyle = 'white';
            ctx.font = 'bold 24px Arial';
            ctx.textAlign = 'center';
            ctx.fillText(`Score: ${score}`, canvas.width / 2, 40);
            ctx.font = '16px Arial';
            ctx.fillText(`Best: ${bestScore}`, canvas.width / 2, 65);
        }

        function drawStartScreen() {
            ctx.fillStyle = 'rgba(0, 0, 0, 0.7)';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            ctx.fillStyle = 'white';
            ctx.font = 'bold 36px Arial';
            ctx.textAlign = 'center';
            ctx.fillText('Flappy Bird', canvas.width / 2, 200);
            
            ctx.font = '20px Arial';
            ctx.fillText('Press SPACE to Start!', canvas.width / 2, 250);
            
            ctx.font = '16px Arial';
            ctx.fillText('Avoid the pipes and fly as far as you can!', canvas.width / 2, 300);
        }

        function drawGameOverScreen() {
            ctx.fillStyle = 'rgba(0, 0, 0, 0.8)';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            ctx.fillStyle = '#FF6B6B';
            ctx.font = 'bold 32px Arial';
            ctx.textAlign = 'center';
            ctx.fillText('Game Over!', canvas.width / 2, 250);
            
            ctx.fillStyle = 'white';
            ctx.font = '20px Arial';
            ctx.fillText(`Final Score: ${score}`, canvas.width / 2, 290);
            
            if (score > bestScore) {
                ctx.fillStyle = '#FFD700';
                ctx.fillText('🎉 New Best Score! 🎉', canvas.width / 2, 320);
            }
            
            ctx.fillStyle = 'white';
            ctx.font = '16px Arial';
            ctx.fillText('Press SPACE to Play Again', canvas.width / 2, 360);
        }

        function resetGame() {
            bird.y = 200;
            bird.velocity = 0;
            pipes = [];
            score = 0;
            frameCount = 0;
            gameState = 'playing';
        }

        function gameLoop() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            if (gameState === 'start') {
                drawBird();
                drawStartScreen();
            } else if (gameState === 'playing') {
                bird.velocity += bird.gravity;
                bird.y += bird.velocity;
                updatePipes();
                
                if (checkCollision()) {
                    gameState = 'gameOver';
                    if (score > bestScore) {
                        bestScore = score;
                        localStorage.setItem('bestScore', bestScore);
                    }
                }
                
                pipes.forEach(pipe => drawPipe(pipe.x, pipe.topHeight));
                drawBird();
                drawScore();
                frameCount++;
            } else if (gameState === 'gameOver') {
                pipes.forEach(pipe => drawPipe(pipe.x, pipe.topHeight));
                drawBird();
                drawScore();
                drawGameOverScreen();
            }
            
            requestAnimationFrame(gameLoop);
        }

        document.addEventListener('keydown', function(e) {
            if (e.code === 'Space') {
                e.preventDefault();
                if (gameState === 'start') resetGame();
                else if (gameState === 'playing') bird.velocity = bird.jumpPower;
                else if (gameState === 'gameOver') resetGame();
            }
        });

        canvas.addEventListener('touchstart', function(e) {
            e.preventDefault();
            if (gameState === 'start') resetGame();
            else if (gameState === 'playing') bird.velocity = bird.jumpPower;
            else if (gameState === 'gameOver') resetGame();
        });

        gameLoop();
    </script>
</body>
</html>
