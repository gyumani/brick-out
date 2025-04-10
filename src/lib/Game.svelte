<script>
    import { onMount } from 'svelte';
    import Ball from './Ball.svelte';
    import Paddle from './Paddle.svelte';
    import Brick from './Brick.svelte';

    let ball = {
        x: 400,
        y: 570,
        dx: 4,
        dy: -4,
        radius: 10,
        color: '#FF0000'
    };

    let paddle = {
        width: 75,
        height: 10,
        x: 362.5,
        color: '#FFA500'
    };

    let bricks = [];
    let score = 0;
    let gameOver = false;
    let gameWin = false;
    let visibleBricks = [];

    const BRICK_ROWS = 5;
    const BRICK_COLS = 8;
    const BRICK_WIDTH = 75;
    const BRICK_HEIGHT = 20;
    const BRICK_PADDING = 10;
    const BRICK_OFFSET_TOP = 30;
    const BRICK_OFFSET_LEFT = 30;

    onMount(() => {
        // 벽돌 초기화
        for (let c = 0; c < BRICK_COLS; c++) {
            bricks[c] = [];
            for (let r = 0; r < BRICK_ROWS; r++) {
                bricks[c][r] = { 
                    x: c * (BRICK_WIDTH + BRICK_PADDING) + BRICK_OFFSET_LEFT,
                    y: r * (BRICK_HEIGHT + BRICK_PADDING) + BRICK_OFFSET_TOP,
                    width: BRICK_WIDTH,
                    height: BRICK_HEIGHT,
                    status: 1 
                };
            }
        }
        visibleBricks = bricks.flat().filter(brick => brick.status === 1);

        // 게임 루프 시작
        requestAnimationFrame(gameLoop);
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
                    // 공의 중심이 벽돌 내부에 있는지 확인
                    if (ball.x > b.x && ball.x < b.x + BRICK_WIDTH && 
                        ball.y > b.y && ball.y < b.y + BRICK_HEIGHT) {
                        // 충돌 방향에 따라 공의 방향을 변경
                        if (ball.dy > 0) {
                            // 아래에서 충돌
                            ball.dy = -ball.dy;
                        } else {
                            // 위에서 충돌
                            ball.dy = -ball.dy;
                        }
                        b.status = 0;
                        score++;
                        // visibleBricks 업데이트
                        visibleBricks = bricks.flat().filter(brick => brick.status === 1);
                        
                        // 모든 벽돌이 깨졌는지 확인
                        if (visibleBricks.length === 0) {
                            gameWin = true;
                        }
                    }
                }
            }
        }

        requestAnimationFrame(gameLoop);
    }

    function handleMouseMove(e) {
        const relativeX = e.clientX - e.currentTarget.getBoundingClientRect().left;
        if (relativeX > 0 && relativeX < 800) {
            paddle.x = relativeX - paddle.width / 2;
        }
    }
</script>

<div class="game-container">
    <svg width="800" height="600" on:mousemove={handleMouseMove}>
        <rect width="800" height="600" fill="#eee" />
        
        <!-- 벽돌 그리기 -->
        {#each visibleBricks as brick}
            <Brick {...brick} color="#800080" />
        {/each}

        <!-- 패들 그리기 -->
        <Paddle x={paddle.x} y={590} width={paddle.width} height={paddle.height} color={paddle.color} />

        <!-- 공 그리기 -->
        <Ball x={ball.x} y={ball.y} radius={ball.radius} color={ball.color} />
    </svg>
    
    <div class="score">점수: {score}</div>
    {#if gameOver}
        <div class="game-over">게임 오버!</div>
    {/if}
    {#if gameWin}
        <div class="game-win">승리!</div>
    {/if}
</div>

<style>
    .game-container {
        position: relative;
        width: 800px;
        margin: 0 auto;
    }

    svg {
        display: block;
    }

    .score {
        position: absolute;
        top: 10px;
        left: 10px;
        font-size: 20px;
    }

    .game-over {
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        font-size: 40px;
        color: red;
    }

    .game-win {
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        font-size: 40px;
        color: green;
    }
</style> 