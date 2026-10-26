#1번 답안

...
#include "raylib.h"
#include <math.h>
#include <stdbool.h>
#include <stdio.h>

#define SCREEN_WIDTH  960
#define SCREEN_HEIGHT 720

#define BLOCK_ROWS    6
#define BLOCK_COLS    10

#define BLOCK_WIDTH   78.0f
#define BLOCK_HEIGHT  25.0f
#define BLOCK_GAP     7.0f
#define BLOCK_TOP     115.0f

#define PADDLE_WIDTH  120.0f
#define PADDLE_HEIGHT 16.0f
#define PADDLE_Y      650.0f

#define BALL_RADIUS   8.0f

typedef enum GameState {
    STATE_TITLE,
    STATE_PLAYING,
    STATE_PAUSED,
    STATE_GAME_OVER,
    STATE_CLEAR
} GameState;

typedef struct Block {
    Rectangle rect;
    Color color;
    bool active;
} Block;

typedef struct Ball {
    Vector2 position;
    Vector2 velocity;
    float radius;
} Ball;

static Block blocks[BLOCK_ROWS][BLOCK_COLS];
static Ball ball;

static Rectangle paddle;
static GameState gameState = STATE_TITLE;

static int score = 0;
static int lives = 3;
static int stage = 1;
static int remainingBlocks = 0;

static float paddleSpeed = 560.0f;
static float baseBallSpeed = 390.0f;

static const Color rainbowColors[BLOCK_ROWS] = {
    {255,  70,  90, 255},  // Red
    {255, 145,  55, 255},  // Orange
    {255, 220,  65, 255},  // Yellow
    { 75, 225, 130, 255},  // Green
    { 65, 165, 255, 255},  // Blue
    {180, 105, 255, 255}   // Purple
};

static Color BackgroundColor(void)
{
    return (Color){13, 17, 38, 255};
}

static void CenterText(const char *text, int y, int fontSize, Color color)
{
    int width = MeasureText(text, fontSize);
    DrawText(text, (SCREEN_WIDTH - width) / 2, y, fontSize, color);
}

static void ResetBall(void)
{
    ball.position = (Vector2){
        paddle.x + paddle.width / 2.0f,
        paddle.y - BALL_RADIUS - 2.0f
    };

    float direction = (GetRandomValue(0, 1) == 0) ? -1.0f : 1.0f;

    ball.velocity = (Vector2){
        direction * baseBallSpeed * 0.55f,
        -baseBallSpeed * 0.84f
    };

    ball.radius = BALL_RADIUS;
}

static void CreateBlocks(void)
{
    remainingBlocks = 0;

    float totalWidth =
        BLOCK_COLS * BLOCK_WIDTH +
        (BLOCK_COLS - 1) * BLOCK_GAP;

    float startX = (SCREEN_WIDTH - totalWidth) / 2.0f;

    for (int row = 0; row < BLOCK_ROWS; row++) {
        for (int col = 0; col < BLOCK_COLS; col++) {
            blocks[row][col].rect = (Rectangle){
                startX + col * (BLOCK_WIDTH + BLOCK_GAP),
                BLOCK_TOP + row * (BLOCK_HEIGHT + BLOCK_GAP),
                BLOCK_WIDTH,
                BLOCK_HEIGHT
            };

            blocks[row][col].color = rainbowColors[row];
            blocks[row][col].active = true;
            remainingBlocks++;
        }
    }
}

static void ResetGame(void)
{
    score = 0;
    lives = 3;
    stage = 1;
    baseBallSpeed = 390.0f;

    paddle = (Rectangle){
        (SCREEN_WIDTH - PADDLE_WIDTH) / 2.0f,
        PADDLE_Y,
        PADDLE_WIDTH,
        PADDLE_HEIGHT
    };

    CreateBlocks();
    ResetBall();
    gameState = STATE_TITLE;
}

static void NextStage(void)
{
    stage++;
    baseBallSpeed = 390.0f + (stage - 1) * 35.0f;

    if (baseBallSpeed > 650.0f) {
        baseBallSpeed = 650.0f;
    }

    CreateBlocks();
    ResetBall();
    gameState = STATE_PLAYING;
}

static void DrawBackground(void)
{
    ClearBackground(BackgroundColor());

    // Decorative stars
    for (int i = 0; i < 55; i++) {
        int x = (i * 173 + 37) % SCREEN_WIDTH;
        int y = (i * 97 + 19) % SCREEN_HEIGHT;

        DrawCircle(x, y, 1.5f, (Color){100, 120, 180, 100});
    }

    DrawRectangle(0, 0, SCREEN_WIDTH, 76,
                  (Color){23, 29, 57, 255});

    DrawLine(0, 76, SCREEN_WIDTH, 76,
             (Color){75, 85, 145, 180});
}

static void DrawHUD(void)
{
    char text[128];

    snprintf(text, sizeof(text), "SCORE  %06d", score);
    DrawText(text, 24, 25, 22, RAYWHITE);

    snprintf(text, sizeof(text), "STAGE  %d", stage);
    DrawText(text, 410, 25, 22, (Color){180, 210, 255, 255});

    DrawText("LIVES", 765, 28, 18, RAYWHITE);

    for (int i = 0; i < lives; i++) {
        DrawCircle(845 + i * 32, 37, 9,
                   (Color){255, 95, 125, 255});
    }

    DrawText("P: PAUSE", 24, 682, 15,
             (Color){145, 155, 190, 255});
}

static void DrawGame(void)
{
    DrawBackground();
    DrawHUD();

    // Rainbow blocks
    for (int row = 0; row < BLOCK_ROWS; row++) {
        for (int col = 0; col < BLOCK_COLS; col++) {
            Block *block = &blocks[row][col];

            if (!block->active) {
                continue;
            }

            DrawRectangleRounded(block->rect, 0.22f, 5,
                                 block->color);

            DrawRectangleRounded(
                (Rectangle){
                    block->rect.x + 3,
                    block->rect.y + 3,
                    block->rect.width - 6,
                    4
                },
                0.5f, 4, (Color){255, 255, 255, 85}
            );
        }
    }

    // Paddle glow and body
    DrawRectangleRounded(
        (Rectangle){
            paddle.x - 3,
            paddle.y - 3,
            paddle.width + 6,
            paddle.height + 6
        },
        0.5f, 8, (Color){80, 150, 255, 65}
    );

    DrawRectangleRounded(paddle, 0.5f, 8,
                         (Color){120, 190, 255, 255});

    DrawRectangleRounded(
        (Rectangle){
            paddle.x + 8,
            paddle.y + 3,
            paddle.width - 16,
            4
        },
        0.5f, 5, RAYWHITE
    );

    // Ball
    DrawCircleV(ball.position, ball.radius + 4,
                (Color){100, 180, 255, 55});

    DrawCircleV(ball.position, ball.radius, RAYWHITE);
    DrawCircleV(
        (Vector2){
            ball.position.x - 2,
            ball.position.y - 2
        },
        2.5f,
        (Color){180, 220, 255, 255}
    );
}

static void UpdatePaddle(float dt)
{
    float mouseX = (float)GetMouseX();

    if (IsMouseButtonDown(MOUSE_BUTTON_LEFT) ||
        IsMouseButtonDown(MOUSE_BUTTON_RIGHT)) {
        paddle.x = mouseX - paddle.width / 2.0f;
    } else {
        if (IsKeyDown(KEY_LEFT) || IsKeyDown(KEY_A)) {
            paddle.x -= paddleSpeed * dt;
        }

        if (IsKeyDown(KEY_RIGHT) || IsKeyDown(KEY_D)) {
            paddle.x += paddleSpeed * dt;
        }
    }

    if (paddle.x < 0) {
        paddle.x = 0;
    }

    if (paddle.x + paddle.width > SCREEN_WIDTH) {
        paddle.x = SCREEN_WIDTH - paddle.width;
    }
}

static void UpdateBall(float dt)
{
    Vector2 previousPosition = ball.position;

    ball.position.x += ball.velocity.x * dt;
    ball.position.y += ball.velocity.y * dt;

    // Left and right walls
    if (ball.position.x - ball.radius <= 0) {
        ball.position.x = ball.radius;
        ball.velocity.x = fabsf(ball.velocity.x);
    }

    if (ball.position.x + ball.radius >= SCREEN_WIDTH) {
        ball.position.x = SCREEN_WIDTH - ball.radius;
        ball.velocity.x = -fabsf(ball.velocity.x);
    }

    // Top wall
    if (ball.position.y - ball.radius <= 0) {
        ball.position.y = ball.radius;
        ball.velocity.y = fabsf(ball.velocity.y);
    }

    // Paddle collision: only when moving down
    if (ball.velocity.y > 0 &&
        CheckCollisionCircleRec(
            ball.position, ball.radius, paddle)) {

        ball.position.y = paddle.y - ball.radius;

        float paddleCenter = paddle.x + paddle.width / 2.0f;

        float hitOffset =
            (ball.position.x - paddleCenter) /
            (paddle.width / 2.0f);

        if (hitOffset < -1.0f) hitOffset = -1.0f;
        if (hitOffset >  1.0f) hitOffset =  1.0f;

        // Reflect according to where the ball hits the paddle
        float maxAngle = 1.12f;
        float angle = hitOffset * maxAngle;

        float speed = sqrtf(
            ball.velocity.x * ball.velocity.x +
            ball.velocity.y * ball.velocity.y
        );

        ball.velocity.x = speed * sinf(angle);
        ball.velocity.y = -speed * cosf(angle);
    }

    // Block collision
    for (int row = 0; row < BLOCK_ROWS; row++) {
        for (int col = 0; col < BLOCK_COLS; col++) {
            Block *block = &blocks[row][col];

            if (!block->active) {
                continue;
            }

            if (!CheckCollisionCircleRec(
                    ball.position, ball.radius, block->rect)) {
                continue;
            }

            block->active = false;
            remainingBlocks--;
            score += 10 * stage;

            // Determine the collision axis using the previous frame.
            if (previousPosition.y + ball.radius <= block->rect.y ||
                previousPosition.y - ball.radius >=
                    block->rect.y + block->rect.height) {
                ball.velocity.y *= -1.0f;
            } else {
                ball.velocity.x *= -1.0f;
            }

            // Only one block is removed per frame.
            goto block_collision_done;
        }
    }

block_collision_done:

    // Ball lost
    if (ball.position.y - ball.radius > SCREEN_HEIGHT) {
        lives--;

        if (lives <= 0) {
            gameState = STATE_GAME_OVER;
        } else {
            ResetBall();
        }
    }

    // All blocks cleared
    if (remainingBlocks <= 0) {
        gameState = STATE_CLEAR;
    }
}

static void DrawTitle(void)
{
    DrawBackground();

    CenterText("RAINBOW BREAKER", 165, 48,
               (Color){255, 220, 100, 255});

    CenterText("BREAK THE BLOCKS!", 225, 24,
               (Color){170, 205, 255, 255});

    // Rainbow decoration
    for (int i = 0; i < BLOCK_ROWS; i++) {
        DrawRectangle(
            250 + i * 77, 300, 70, 20,
            rainbowColors[i]
        );
    }

    CenterText("ARROW KEYS / A D : MOVE", 390, 20, RAYWHITE);
    CenterText("MOUSE : MOVE PADDLE", 425, 20, RAYWHITE);
    CenterText("P : PAUSE", 460, 20, RAYWHITE);

    CenterText("PRESS ENTER TO START", 540, 28,
               (Color){100, 255, 190, 255});

    CenterText("A colorful arcade adventure", 620, 17,
               (Color){130, 145, 185, 255});
}

static void DrawOverlay(const char *title,
                        const char *subtitle,
                        Color titleColor)
{
    DrawRectangle(0, 0, SCREEN_WIDTH, SCREEN_HEIGHT,
                  (Color){5, 8, 24, 190});

    CenterText(title, 270, 46, titleColor);
    CenterText(subtitle, 350, 22, RAYWHITE);
}

int main(void)
{
    SetConfigFlags(FLAG_VSYNC_HINT);
    InitWindow(SCREEN_WIDTH, SCREEN_HEIGHT,
               "Rainbow Breaker - raylib");
    SetTargetFPS(60);

    ResetGame();

    while (!WindowShouldClose()) {
        float dt = GetFrameTime();

        // Avoid large physics jumps after window stalls.
        if (dt > 0.033f) {
            dt = 0.033f;
        }

        if (gameState == STATE_TITLE) {
            if (IsKeyPressed(KEY_ENTER) ||
                IsKeyPressed(KEY_SPACE)) {
                gameState = STATE_PLAYING;
            }
        } else if (gameState == STATE_PLAYING) {
            if (IsKeyPressed(KEY_P) ||
                IsKeyPressed(KEY_ESCAPE)) {
                gameState = STATE_PAUSED;
            } else {
                UpdatePaddle(dt);
                UpdateBall(dt);
            }
        } else if (gameState == STATE_PAUSED) {
            if (IsKeyPressed(KEY_P) ||
                IsKeyPressed(KEY_ESCAPE)) {
                gameState = STATE_PLAYING;
            }
        } else if (gameState == STATE_GAME_OVER ||
                   gameState == STATE_CLEAR) {
            if (IsKeyPressed(KEY_R)) {
                ResetGame();
            } else if (IsKeyPressed(KEY_ENTER)) {
                if (gameState == STATE_CLEAR) {
                    NextStage();
                } else {
                    ResetGame();
                    gameState = STATE_PLAYING;
                }
            }
        }

        BeginDrawing();

        if (gameState == STATE_TITLE) {
            DrawTitle();
        } else {
            DrawGame();

            if (gameState == STATE_PAUSED) {
                DrawOverlay("PAUSED",
                            "Press P or ESC to resume",
                            (Color){255, 220, 100, 255});
            } else if (gameState == STATE_GAME_OVER) {
                char result[100];

                snprintf(result, sizeof(result),
                         "Final score: %d", score);

                DrawOverlay("GAME OVER", result,
                            (Color){255, 95, 120, 255});

                CenterText("ENTER: PLAY AGAIN   R: RESET",
                           420, 20, RAYWHITE);
            } else if (gameState == STATE_CLEAR) {
                char result[100];

                snprintf(result, sizeof(result),
                         "Stage %d complete! Score: %d",
                         stage, score);

                DrawOverlay("RAINBOW CLEAR!", result,
                            (Color){100, 255, 190, 255});

                CenterText("ENTER: NEXT STAGE   R: RESET",
                           420, 20, RAYWHITE);
            }
        }

        EndDrawing();
    }

    CloseWindow();
    return 0;
}





<img width="979" height="759" alt="image" src="https://github.com/user-attachments/assets/eee448a4-64c5-4edc-90e9-1925b064c8de" />
