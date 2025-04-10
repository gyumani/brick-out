<script>
    import { onMount } from 'svelte';
    import Ball from './Ball.svelte';
    import Paddle from './Paddle.svelte';
    import Brick from './Brick.svelte';

    const BALL_SPEED = 4; // 공 속도 상수로 고정

    let ball = {
        x: 400,
        y: 570,
        dx: BALL_SPEED,
        dy: -BALL_SPEED,
        radius: 10,
        color: '#000000'
    };

    let paddle = {
        width: 75,
        height: 10,
        x: 362.5,
        y: 580,
        color: '#000000'
    };

    let bricks = [];
    let score = 0;
    let gameOver = false;
    let gameWin = false;
    let visibleBricks = [];
    let level = 1;
    let maxLevel = 50;
    let paddleWidth = 75;
    let gameLoopId = null; // 게임 루프 ID를 저장할 변수 추가

    const BRICK_ROWS = 6;
    const BRICK_COLS = 10;
    const BRICK_WIDTH = 70;
    const BRICK_HEIGHT = 20;
    const BRICK_PADDING = 5;
    const BRICK_OFFSET_TOP = 30;
    const BRICK_OFFSET_LEFT = 15;

    const BRICK_COLORS = [
        '#FF0000', // 빨강
        '#FF7F00', // 주황
        '#FFFF00', // 노랑
        '#00FF00', // 초록
        '#0000FF', // 파랑
        '#4B0082'  // 남색
    ];

    function initializeLevel() {
        // 레벨에 따른 난이도 조절
        paddleWidth = Math.max(50, 75 - (level - 1) * 2);
        
        // 공 속도 초기화
        ball.dx = BALL_SPEED;
        ball.dy = -BALL_SPEED;
        
        paddle.width = paddleWidth;

        // 벽돌 초기화
        bricks = [];
        for (let c = 0; c < BRICK_COLS; c++) {
            bricks[c] = [];
            for (let r = 0; r < BRICK_ROWS; r++) {
                let status = 1;
                if (level >= 5 && Math.random() < 0.1 + (level - 5) * 0.02) {
                    status = 2;
                }
                if (level >= 10 && Math.random() < 0.05 + (level - 10) * 0.01) {
                    status = 3;
                }
                
                bricks[c][r] = { 
                    x: c * (BRICK_WIDTH + BRICK_PADDING) + BRICK_OFFSET_LEFT,
                    y: r * (BRICK_HEIGHT + BRICK_PADDING) + BRICK_OFFSET_TOP,
                    width: BRICK_WIDTH,
                    height: BRICK_HEIGHT,
                    status: status,
                    color: status === 1 ? BRICK_COLORS[r % BRICK_COLORS.length] : 
                           status === 2 ? '#FF0000' : '#000000'
                };
            }
        }
        visibleBricks = bricks.flat().filter(brick => brick.status === 1);
    }

    onMount(() => {
        initializeLevel();
        gameLoopId = requestAnimationFrame(gameLoop);
        return () => {
            if (gameLoopId) {
                cancelAnimationFrame(gameLoopId);
            }
        };
    });

    function gameLoop() {
        if (gameOver || gameWin) return;

        // 공 이동
        ball.x += ball.dx;
        ball.y += ball.dy;

        // 벽 충돌 감지
        if (ball.x + ball.dx > 800 - ball.radius || ball.x + ball.dx < ball.radius) {
            ball.dx = -ball.dx;
        }
        if (ball.y + ball.dy < ball.radius) {
            ball.dy = -ball.dy;
        } else if (ball.y + ball.dy > 600 - ball.radius) {
            if (ball.x > paddle.x && ball.x < paddle.x + paddle.width) {
                ball.dy = -ball.dy;
            } else {
                gameOver = true;
            }
        }

        // 벽돌 충돌 감지
        for (let c = 0; c < BRICK_COLS; c++) {
            for (let r = 0; r < BRICK_ROWS; r++) {
                const b = bricks[c][r];
                if (b.status === 1) {
                    if (ball.x > b.x && ball.x < b.x + BRICK_WIDTH && 
                        ball.y > b.y && ball.y < b.y + BRICK_HEIGHT) {
                        ball.dy = -ball.dy;
                        b.status = 0;
                        score++;
                        visibleBricks = bricks.flat().filter(brick => brick.status === 1);
                        
                        if (visibleBricks.length === 0) {
                            if (level < maxLevel) {
                                level++;
                                gameWin = true;
                                cancelAnimationFrame(gameLoopId); // 현재 게임 루프 취소
                                setTimeout(() => {
                                    gameWin = false;
                                    initializeLevel();
                                    ball.x = 400;
                                    ball.y = 570;
                                    ball.dx = BALL_SPEED;
                                    ball.dy = -BALL_SPEED;
                                    gameLoopId = requestAnimationFrame(gameLoop);
                                }, 2000);
                            } else {
                                gameWin = true;
                            }
                        }
                    }
                }
            }
        }

        gameLoopId = requestAnimationFrame(gameLoop);
    }

    function handleMouseMove(e) {
        const relativeX = e.clientX - e.currentTarget.getBoundingClientRect().left;
        if (relativeX > 0 && relativeX < 800) {
            paddle.x = relativeX - paddle.width / 2;
        }
    }

    function restartGame() {
        if (gameLoopId) {
            cancelAnimationFrame(gameLoopId);
        }
        level = 1;
        score = 0;
        gameOver = false;
        gameWin = false;
        initializeLevel();
        ball.x = 400;
        ball.y = 570;
        ball.dx = BALL_SPEED;
        ball.dy = -BALL_SPEED;
        gameLoopId = requestAnimationFrame(gameLoop);
    }
</script>

<div class="game-container">
    <div class="game-header">
        <div class="game-title">벽돌깨기</div>
        <div class="game-controls">
            <button class="control-button" on:click={restartGame}>새 게임</button>
            <div class="game-info">
                <div class="score">점수: {score}</div>
                <div class="level">레벨: {level}/{maxLevel}</div>
            </div>
        </div>
    </div>
    
    <div class="game-board">
        <svg width="800" height="600" on:mousemove={handleMouseMove}>
            <rect width="800" height="600" fill="#C0C0C0" />
            
            <!-- 벽돌 그리기 -->
            {#each visibleBricks as brick}
                <Brick {...brick} color={brick.color} />
            {/each}

            <!-- 패들 그리기 -->
            <Paddle x={paddle.x} y={paddle.y} width={paddle.width} height={paddle.height} color={paddle.color} />

            <!-- 공 그리기 -->
            <Ball x={ball.x} y={ball.y} radius={ball.radius} color={ball.color} />
        </svg>
    </div>
    
    {#if gameOver}
        <div class="game-over">
            <div class="message">Game Over!</div>
            <button class="restart-button" on:click={restartGame}>Retry?</button>
        </div>
    {/if}
    {#if gameWin}
        <div class="game-win">
            {#if level === maxLevel}
                <div class="message">Congratulations! You've cleared all levels!💃</div>
            {:else}
                <div class="message">Level {level-1} Clear!</div>
                <div class="sub-message">Going to next level...</div>
            {/if}
        </div>
    {/if}
</div>

<style>
    .game-container {
        position: relative;
        width: 800px;
        margin: 0 auto;
        background: #C0C0C0;
        border: 2px solid #000;
        box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
    }

    .game-header {
        background: #C0C0C0;
        padding: 10px;
        border-bottom: 2px solid #000;
        display: flex;
        justify-content: space-between;
        align-items: center;
    }

    .game-title {
        font-size: 24px;
        font-weight: bold;
        color: #000;
    }

    .game-controls {
        display: flex;
        align-items: center;
        gap: 20px;
    }

    .control-button {
        padding: 5px 15px;
        background: #C0C0C0;
        border: 2px solid #000;
        font-size: 14px;
        cursor: pointer;
    }

    .control-button:hover {
        background: #D0D0D0;
    }

    .game-board {
        padding: 10px;
    }

    .score, .level {
        font-size: 16px;
        font-weight: bold;
        color: #000;
    }

    .game-over, .game-win {
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        text-align: center;
        background: rgba(192, 192, 192, 0.9);
        padding: 20px;
        border: 2px solid #000;
        border-radius: 5px;
    }

    .message {
        font-size: 24px;
        font-weight: bold;
        margin-bottom: 10px;
    }

    .sub-message {
        font-size: 16px;
        margin-top: 10px;
    }

    .game-over .message {
        color: #FF0000;
    }

    .game-win .message {
        color: #0000FF;
    }

    .restart-button {
        margin-top: 20px;
        padding: 8px 16px;
        background: #C0C0C0;
        border: 2px solid #000;
        font-size: 14px;
        cursor: pointer;
    }

    .restart-button:hover {
        background: #D0D0D0;
    }
</style> 