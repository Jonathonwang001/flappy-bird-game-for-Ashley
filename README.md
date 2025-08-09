# flappy-bird-game-for-Ashley
Creating an interesting game only for my love, Ashley. Hope her happy everyday!
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <title>Flappy Bird - 移动版</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            -webkit-tap-highlight-color: transparent;
            -webkit-touch-callout: none;
            -webkit-user-select: none;
            user-select: none;
        }
        
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background: linear-gradient(to bottom, #87CEEB 0%, #98FB98 100%);
            font-family: 'Arial', sans-serif;
            overflow: hidden;
        }
        
        #gameContainer {
            position: relative;
            width: 100vw;
            height: 100vh;
            max-width: 400px;
            max-height: 600px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
        }
        
        #gameCanvas {
            border: 2px solid #333;
            box-shadow: 0 0 20px rgba(0,0,0,0.3);
            background: linear-gradient(to bottom, #87CEEB 0%, #98FB98 80%, #90EE90 100%);
            touch-action: manipulation;
        }
        
        #score {
            position: absolute;
            top: 20px;
            left: 20px;
            font-size: 24px;
            color: white;
            font-weight: bold;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.5);
            z-index: 10;
        }
        
        #bestScore {
            position: absolute;
            top: 20px;
            right: 20px;
            font-size: 16px;
            color: white;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.5);
            z-index: 10;
        }
        
        #instructions {
            position: absolute;
            bottom: 20px;
            color: rgba(255,255,255,0.8);
            font-size: 14px;
            text-align: center;
            text-shadow: 1px 1px 2px rgba(0,0,0,0.5);
            z-index: 10;
        }
        
        #gameOver {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: rgba(0,0,0,0.9);
            color: white;
            padding: 30px;
            border-radius: 15px;
            text-align: center;
            display: none;
            z-index: 100;
            min-width: 250px;
        }
        
        #gameOver button {
            background: #4CAF50;
            color: white;
            border: none;
            padding: 15px 25px;
            font-size: 16px;
            border-radius: 8px;
            cursor: pointer;
            margin: 10px 5px;
            transition: background 0.3s;
        }
        
        #gameOver button:hover,
        #gameOver button:active {
            background: #45a049;
        }
        
        #startScreen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.7);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            color: white;
            z-index: 50;
        }
        
        #startScreen h1 {
            font-size: 36px;
            margin-bottom: 20px;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.5);
        }
        
        #startScreen button {
            background: #FF6347;
            color: white;
            border: none;
            padding: 20px 40px;
            font-size: 20px;
            border-radius: 10px;
            cursor: pointer;
            margin-top: 20px;
            transition: background 0.3s;
        }
        
        #startScreen button:hover,
        #startScreen button:active {
            background: #FF4500;
        }
        
        /* 响应式设计 */
        @media (max-width: 480px) {
            #gameCanvas {
                width: 95vw;
                height: calc(95vw * 1.5);
                max-height: 85vh;
            }
        }
        
        @media (orientation: landscape) and (max-height: 500px) {
            #gameCanvas {
                height: 90vh;
                width: calc(90vh / 1.5);
            }
        }
    </style>
</head>
<body>
    <div id="gameContainer">
        <div id="score">得分: 0</div>
        <div id="bestScore">最佳: 0</div>
        <canvas id="gameCanvas" width="300" height="500"></canvas>
        <div id="instructions">点击屏幕让小鸟飞翔！</div>
        
        <div id="startScreen">
            <h1>🐦 Flappy Bird</h1>
            <p>点击屏幕控制小鸟飞行</p>
            <p>避开管道获得高分！</p>
            <button onclick="startGame()">开始游戏</button>
        </div>
        
        <div id="gameOver">
            <h2>💥 游戏结束!</h2>
            <p>本次得分: <span id="finalScore">0</span></p>
            <p>最佳分数: <span id="bestScoreDisplay">0</span></p>
            <div id="newRecord" style="display: none; color: #FFD700; margin: 10px 0;">
                🎉 新纪录！
            </div>
            <button onclick="restartGame()">重新开始</button>
            <button onclick="showStartScreen()">返回主页</button>
        </div>
    </div>

    <script>
        // 获取画布和上下文
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreElement = document.getElementById('score');
        const bestScoreElement = document.getElementById('bestScore');
        const gameOverElement = document.getElementById('gameOver');
        const startScreenElement = document.getElementById('startScreen');
        const finalScoreElement = document.getElementById('finalScore');
        const bestScoreDisplayElement = document.getElementById('bestScoreDisplay');
        const newRecordElement = document.getElementById('newRecord');

        // 动态调整画布大小
        function resizeCanvas() {
            const container = document.getElementById('gameContainer');
            const maxWidth = Math.min(400, window.innerWidth * 0.95);
            const maxHeight = Math.min(600, window.innerHeight * 0.85);
            
            if (window.innerWidth < window.innerHeight) {
                // 竖屏
                canvas.width = maxWidth;
                canvas.height = maxWidth * 1.5;
            } else {
                // 横屏
                canvas.height = maxHeight;
                canvas.width = maxHeight / 1.5;
            }
            
            // 重新初始化游戏元素位置
            if (gameState !== 'start') {
                bird.x = canvas.width * 0.15;
                bird.y = canvas.height / 2;
            }
        }

        // 游戏状态
        let gameState = 'start'; // start, playing, gameOver
        let score = 0;
        let bestScore = localStorage.getItem('flappyBirdMobileBest') || 0;
        bestScoreElement.textContent = '最佳: ' + bestScore;

        // 小鸟对象
        const bird = {
            x: 0,
            y: 0,
            width: 25,
            height: 25,
            velocity: 0,
            gravity: 0.5,
            jump: -10,
            color: '#FFD700'
        };

        // 管道数组
        let pipes = [];
        const pipeWidth = 60;
        let pipeGap = 140;
        const pipeSpeed = 2;

        // 背景云朵
        let clouds = [];
        
        // 初始化云朵
        function initClouds() {
            clouds = [];
            const cloudCount = Math.floor(canvas.width / 80);
            for (let i = 0; i < cloudCount; i++) {
                clouds.push({
                    x: Math.random() * canvas.width,
                    y: Math.random() * (canvas.height / 2),
                    size: Math.random() * 25 + 15,
                    speed: Math.random() * 0.3 + 0.1
                });
            }
        }

        // 绘制云朵
        function drawClouds() {
            ctx.fillStyle = 'rgba(255, 255, 255, 0.6)';
            clouds.forEach(cloud => {
                ctx.beginPath();
                ctx.arc(cloud.x, cloud.y, cloud.size, 0, Math.PI * 2);
                ctx.fill();
                ctx.beginPath();
                ctx.arc(cloud.x + cloud.size * 0.4, cloud.y, cloud.size * 0.6, 0, Math.PI * 2);
                ctx.fill();
                ctx.beginPath();
                ctx.arc(cloud.x - cloud.size * 0.4, cloud.y, cloud.size * 0.6, 0, Math.PI * 2);
                ctx.fill();
            });
        }

        // 更新云朵位置
        function updateClouds() {
            clouds.forEach(cloud => {
                cloud.x -= cloud.speed;
                if (cloud.x + cloud.size < 0) {
                    cloud.x = canvas.width + cloud.size;
                    cloud.y = Math.random() * (canvas.height / 2);
                }
            });
        }

        // 创建管道
        function createPipe() {
            const minHeight = 50;
            const maxHeight = canvas.height - pipeGap - 100;
            const pipeHeight = Math.random() * (maxHeight - minHeight) + minHeight;
            
            pipes.push({
                x: canvas.width,
                topHeight: pipeHeight,
                bottomY: pipeHeight + pipeGap,
                bottomHeight: canvas.height - (pipeHeight + pipeGap) - 50,
                passed: false
            });
        }

        // 绘制小鸟
        function drawBird() {
            const centerX = bird.x + bird.width/2;
            const centerY = bird.y + bird.height/2;
            
            // 鸟身体
            ctx.fillStyle = bird.color;
            ctx.beginPath();
            ctx.arc(centerX, centerY, bird.width/2, 0, Math.PI * 2);
            ctx.fill();
            
            // 鸟嘴
            ctx.fillStyle = '#FF6347';
            ctx.beginPath();
            ctx.moveTo(bird.x + bird.width, centerY);
            ctx.lineTo(bird.x + bird.width + 10, centerY - 4);
            ctx.lineTo(bird.x + bird.width + 10, centerY + 4);
            ctx.closePath();
            ctx.fill();
            
            // 眼睛
            ctx.fillStyle = 'white';
            ctx.beginPath();
            ctx.arc(centerX + 4, centerY - 4, 4, 0, Math.PI * 2);
            ctx.fill();
            
            ctx.fillStyle = 'black';
            ctx.beginPath();
            ctx.arc(centerX + 5, centerY - 4, 2, 0, Math.PI * 2);
            ctx.fill();
        }

        // 绘制管道
        function drawPipes() {
            pipes.forEach(pipe => {
                // 上管道
                ctx.fillStyle = '#32CD32';
                ctx.fillRect(pipe.x, 0, pipeWidth, pipe.topHeight);
                
                // 下管道  
                ctx.fillRect(pipe.x, pipe.bottomY, pipeWidth, pipe.bottomHeight);
                
                // 管道顶部装饰
                ctx.fillStyle = '#228B22';
                ctx.fillRect(pipe.x - 5, pipe.topHeight - 15, pipeWidth + 10, 15);
                ctx.fillRect(pipe.x - 5, pipe.bottomY, pipeWidth + 10, 15);
            });
        }

        // 绘制地面
        function drawGround() {
            const groundHeight = 50;
            const groundY = canvas.height - groundHeight;
            
            // 地面
            ctx.fillStyle = '#8B4513';
            ctx.fillRect(0, groundY, canvas.width, groundHeight);
            
            // 草地效果
            ctx.fillStyle = '#228B22';
            for (let x = 0; x < canvas.width; x += 8) {
                const grassHeight = Math.random() * 10 + 3;
                ctx.fillRect(x, groundY - grassHeight, 6, grassHeight);
            }
        }

        // 更新游戏逻辑
        function update() {
            if (gameState !== 'playing') return;

            // 更新小鸟
            bird.velocity += bird.gravity;
            bird.y += bird.velocity;

            // 更新云朵
            updateClouds();

            // 更新管道
            pipes.forEach((pipe, index) => {
                pipe.x -= pipeSpeed;

                // 检查得分
                if (!pipe.passed && pipe.x + pipeWidth < bird.x) {
                    pipe.passed = true;
                    score++;
                    scoreElement.textContent = '得分: ' + score;
                    
                    // 触觉反馈
                    if (navigator.vibrate) {
                        navigator.vibrate(50);
                    }
                }

                // 移除离屏管道
                if (pipe.x + pipeWidth < 0) {
                    pipes.splice(index, 1);
                }
            });

            // 创建新管道
            if (pipes.length === 0 || pipes[pipes.length - 1].x < canvas.width - 150) {
                createPipe();
            }

            // 碰撞检测
            checkCollision();
        }

        // 碰撞检测
        function checkCollision() {
            // 检查地面碰撞
            if (bird.y + bird.height >= canvas.height - 50) {
                gameOver();
                return;
            }

            // 检查天花板碰撞
            if (bird.y <= 0) {
                gameOver();
                return;
            }

            // 检查管道碰撞
            pipes.forEach(pipe => {
                if (bird.x + bird.width > pipe.x && bird.x < pipe.x + pipeWidth) {
                    if (bird.y < pipe.topHeight || bird.y + bird.height > pipe.bottomY) {
                        gameOver();
                        return;
                    }
                }
            });
        }

        // 开始游戏
        function startGame() {
            gameState = 'playing';
            bird.x = canvas.width * 0.15;
            bird.y = canvas.height / 2;
            bird.velocity = 0;
            pipes = [];
            score = 0;
            scoreElement.textContent = '得分: 0';
            startScreenElement.style.display = 'none';
            initClouds();
        }

        // 游戏结束
        function gameOver() {
            gameState = 'gameOver';
            finalScoreElement.textContent = score;
            bestScoreDisplayElement.textContent = bestScore;
            
            // 检查是否是新纪录
            if (score > bestScore) {
                bestScore = score;
                localStorage.setItem('flappyBirdMobileBest', bestScore);
                bestScoreElement.textContent = '最佳: ' + bestScore;
                newRecordElement.style.display = 'block';
                
                // 庆祝振动
                if (navigator.vibrate) {
                    navigator.vibrate([100, 50, 100, 50, 100]);
                }
            } else {
                newRecordElement.style.display = 'none';
            }
            
            gameOverElement.style.display = 'block';
        }

        // 重新开始游戏
        function restartGame() {
            gameOverElement.style.display = 'none';
            startGame();
        }

        // 显示开始界面
        function showStartScreen() {
            gameState = 'start';
            gameOverElement.style.display = 'none';
            startScreenElement.style.display = 'flex';
        }

        // 小鸟跳跃
        function jump() {
            if (gameState === 'playing') {
                bird.velocity = bird.jump;
                
                // 触觉反馈
                if (navigator.vibrate) {
                    navigator.vibrate(30);
                }
            }
        }

        // 绘制游戏
        function draw() {
            // 清空画布
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // 绘制背景渐变
            const gradient = ctx.createLinearGradient(0, 0, 0, canvas.height);
            gradient.addColorStop(0, '#87CEEB');
            gradient.addColorStop(0.8, '#98FB98');
            gradient.addColorStop(1, '#90EE90');
            ctx.fillStyle = gradient;
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // 绘制云朵
            drawClouds();

            // 绘制管道
            drawPipes();

            // 绘制地面
            drawGround();

            // 绘制小鸟
            if (gameState !== 'start') {
                drawBird();
            }
        }

        // 游戏循环
        function gameLoop() {
            update();
            draw();
            requestAnimationFrame(gameLoop);
        }

        // 防止页面滚动和缩放
        document.addEventListener('touchmove', (e) => {
            e.preventDefault();
        }, { passive: false });

        document.addEventListener('gesturestart', (e) => {
            e.preventDefault();
        });

        // 触摸事件监听
        canvas.addEventListener('touchstart', (e) => {
            e.preventDefault();
            jump();
        }, { passive: false });

        // 点击事件监听
        canvas.addEventListener('click', (e) => {
            e.preventDefault();
            jump();
        });

        // 键盘事件监听（用于调试）
        document.addEventListener('keydown', (e) => {
            if (e.code === 'Space') {
                e.preventDefault();
                jump();
            }
        });

        // 窗口大小改变时重新调整
        window.addEventListener('resize', () => {
            resizeCanvas();
            initClouds();
        });

        // 初始化游戏
        function initGame() {
            resizeCanvas();
            initClouds();
            gameLoop();
        }

        // 启动游戏
        initGame();
    </script>
</body>
</html>
