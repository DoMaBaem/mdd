# graphics.h 및 graphics.c 상세 설명

## graphics.h - 헤더 파일

```c
#ifndef GRAPHICS_H
#define GRAPHICS_H
```
**설명**: 헤더 가드 시작.

```c
#include <stdio.h>
#include <windows.h>
#include "game.h"
```
**설명**: 필요한 라이브러리와 game.h를 포함합니다.

---

### 더블 버퍼링용 전역 변수

```c
extern char screenBuffer[HEIGHT + 4][WIDTH + 1];
extern char prevBuffer[HEIGHT + 4][WIDTH + 1];
extern int colorBuffer[HEIGHT + 4][WIDTH + 1];
extern int prevColorBuffer[HEIGHT + 4][WIDTH + 1];
```
**설명**: 화면 버퍼와 색상 버퍼를 extern으로 선언합니다.
- screenBuffer: 현재 프레임의 문자
- prevBuffer: 이전 프레임의 문자
- colorBuffer: 현재 프레임의 색상
- prevColorBuffer: 이전 프레임의 색상

---

### 함수 선언

```c
void gotoxy(int x, int y);
```
**설명**: 커서를 특정 위치로 이동합니다.

```c
void hideCursor();
```
**설명**: 커서를 숨깁니다.

```c
void setColor(int color);
```
**설명**: 텍스트 색상을 설정합니다.

```c
void clearBuffer();
```
**설명**: 화면 버퍼를 초기화합니다.

```c
void drawToBuffer(int x, int y, char c, int color);
```
**설명**: 버퍼에 문자를 그립니다.

```c
void renderBuffer();
```
**설명**: 버퍼를 화면에 렌더링합니다 (더블 버퍼링).

```c
void drawGame(GameState* game);
```
**설명**: 게임 전체를 그립니다.

```c
void drawBorder();
void drawPlatforms(GameState* game);
void drawEnemies(GameState* game);
void drawCoins(GameState* game);
void drawParticles(GameState* game);
void drawPlayer(GameState* game);
void drawHUD(GameState* game);
```
**설명**: 각 요소를 그리는 함수들입니다.

---

## graphics.c - 구현 파일

### 전역 변수 정의

```c
char screenBuffer[HEIGHT + 4][WIDTH + 1];
char prevBuffer[HEIGHT + 4][WIDTH + 1];
int colorBuffer[HEIGHT + 4][WIDTH + 1];
int prevColorBuffer[HEIGHT + 4][WIDTH + 1];
```
**설명**: 더블 버퍼링을 위한 버퍼 배열을 정의합니다. HEIGHT + 4는 HUD 공간을 포함합니다.

---

### gotoxy 함수

```c
void gotoxy(int x, int y) {
    COORD coord = { x, y };
    SetConsoleCursorPosition(GetStdHandle(STD_OUTPUT_HANDLE), coord);
}
```
**설명**: Windows API를 사용하여 커서를 (x, y) 위치로 이동합니다.

---

### hideCursor 함수

```c
void hideCursor() {
    CONSOLE_CURSOR_INFO info = { 100, FALSE };
    SetConsoleCursorInfo(GetStdHandle(STD_OUTPUT_HANDLE), &info);
}
```
**설명**: 커서를 숨겨서 깨끗한 화면을 만듭니다.

---

### setColor 함수

```c
void setColor(int color) {
    SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), color);
}
```
**설명**: 콘솔 텍스트 색상을 설정합니다.
- 7: 회색, 8: 어두운 회색, 10: 초록, 11: 하늘색, 12: 빨강, 13: 보라, 14: 노랑, 15: 흰색

---

### clearBuffer 함수

```c
void clearBuffer() {
    for (int y = 0; y < HEIGHT + 4; y++) {
        for (int x = 0; x < WIDTH; x++) {
            screenBuffer[y][x] = ' ';
            colorBuffer[y][x] = 7;
        }
        screenBuffer[y][WIDTH] = '\0';
    }
}
```
**설명**: 모든 버퍼를 공백(' ')과 기본 색상(7)으로 초기화합니다.

---

### drawToBuffer 함수

```c
void drawToBuffer(int x, int y, char c, int color) {
    if (x >= 0 && x < WIDTH && y >= 0 && y < HEIGHT + 4) {
        screenBuffer[y][x] = c;
        colorBuffer[y][x] = color;
    }
}
```
**설명**: 경계 체크 후 버퍼에 문자와 색상을 저장합니다.

---

### renderBuffer 함수 (더블 버퍼링 핵심)

```c
void renderBuffer() {
    for (int y = 0; y < HEIGHT + 4; y++) {
        for (int x = 0; x < WIDTH; x++) {
```
**설명**: 모든 픽셀을 순회합니다.

```c
            if (screenBuffer[y][x] != prevBuffer[y][x] || 
                colorBuffer[y][x] != prevColorBuffer[y][x]) {
```
**설명**: 이전 프레임과 비교하여 변경된 부분만 처리합니다.

```c
                setColor(colorBuffer[y][x]);
                gotoxy(x, y);
                printf("%c", screenBuffer[y][x]);
```
**설명**: 색상을 설정하고, 커서를 이동한 후, 문자를 출력합니다.

```c
                prevBuffer[y][x] = screenBuffer[y][x];
                prevColorBuffer[y][x] = colorBuffer[y][x];
            }
        }
    }
}
```
**설명**: 이전 버퍼를 현재 버퍼로 업데이트합니다.

**더블 버퍼링의 장점**:
- 화면 깜박임 방지
- 변경된 부분만 렌더링하여 성능 향상
- 부드러운 애니메이션

---

### drawBorder 함수

```c
void drawBorder() {
    for (int x = 0; x < WIDTH; x++) {
        drawToBuffer(x, 0, '#', 8);
        drawToBuffer(x, HEIGHT, '#', 8);
    }
    for (int y = 0; y <= HEIGHT; y++) {
        drawToBuffer(0, y, '#', 8);
        drawToBuffer(WIDTH - 1, y, '#', 8);
    }
}
```
**설명**: 화면 테두리를 '#' 문자(어두운 회색)로 그립니다.

---

### drawPlatforms 함수

```c
void drawPlatforms(GameState* game) {
    for (int i = 0; i < game->platformCount; i++) {
        if (game->platforms[i].active) {
            for (int x = 0; x < game->platforms[i].width; x++) {
                drawToBuffer(game->platforms[i].x + x, 
                           game->platforms[i].y, '=', 6);
            }
        }
    }
}
```
**설명**: 활성화된 플랫폼을 '=' 문자(갈색, 6)로 그립니다.

---

### drawEnemies 함수

```c
void drawEnemies(GameState* game) {
    for (int i = 0; i < game->enemyCount; i++) {
        if (game->enemies[i].active) {
            if (game->enemies[i].type == 0) {
                drawToBuffer(game->enemies[i].x, game->enemies[i].y, 'M', 12);
            } else if (game->enemies[i].type == 1) {
                drawToBuffer(game->enemies[i].x, game->enemies[i].y, 'W', 13);
            } else if (game->enemies[i].type == 2) {
                drawToBuffer(game->enemies[i].x, game->enemies[i].y, 'X', 5);
            }
        }
    }
}
```
**설명**: 몬스터를 타입에 따라 다른 문자와 색상으로 그립니다.
- Type 0: 'M' (빨강, 12)
- Type 1: 'W' (보라, 13)
- Type 2: 'X' (자주색, 5)

---

### drawCoins 함수

```c
void drawCoins(GameState* game) {
    for (int i = 0; i < game->coinCount; i++) {
        if (!game->coins[i].collected) {
            drawToBuffer(game->coins[i].x, game->coins[i].y, '$', 14);
        }
    }
    
    if (game->portal.active && !game->inHiddenStage) {
        drawToBuffer(game->portal.x, game->portal.y, '@', 10);
    }
}
```
**설명**: 수집되지 않은 코인을 '$' (노랑, 14)로 그리고, 포탈을 '@' (초록, 10)로 그립니다.

---

### drawParticles 함수

```c
void drawParticles(GameState* game) {
    for (int i = 0; i < MAX_PARTICLES; i++) {
        if (game->particles[i].life > 0) {
            char sym = game->particles[i].symbol;
            if ((sym == '*' || sym == '+' || sym == '!' || sym == '.') &&
                game->particles[i].x > 0 && game->particles[i].x < WIDTH - 1 &&
                game->particles[i].y > 0 && game->particles[i].y < HEIGHT) {
                drawToBuffer(game->particles[i].x, game->particles[i].y, sym, 7);
            }
        }
    }
}
```
**설명**: 생명이 남은 파티클을 회색(7)으로 그립니다.

---

### drawPlayer 함수

```c
void drawPlayer(GameState* game) {
    int color = (game->player.invincible && (game->player.invincibleTimer % 4 < 2)) 
                ? 15 : 11;
    if (game->player.direction == 'R') {
        drawToBuffer(game->player.x, game->player.y, '>', color);
    }
    else {
        drawToBuffer(game->player.x, game->player.y, '<', color);
    }
}
```
**설명**: 플레이어를 방향에 따라 '>' 또는 '<'로 그립니다. 무적 상태일 때 깜박입니다.

---

### drawHUD 함수

```c
void drawHUD(GameState* game) {
    char hud1[WIDTH + 20];
    char hud2[WIDTH + 20];

    int healthColor = game->player.health > 50 ? 10 : 
                     (game->player.health > 25 ? 14 : 12);

    if (game->inHiddenStage) {
        sprintf(hud1, "HIDDEN STAGE! Time: %d  Score: %d  Coins: %d",
            game->hiddenStageTimer / 20, game->player.score, game->player.coins);
    } else {
        sprintf(hud1, "Score: %d  Coins: %d  Health: %d  Lives: %d  Level: %d",
            game->player.score, game->player.coins, game->player.health,
            game->player.lives, game->level);
    }
    sprintf(hud2, "[A/D] Move [W/SPC] Jump [R] Restart [ESC] Quit | M:-15HP W:-25HP X:-30HP");

    for (int i = 0; hud1[i] != '\0' && i < WIDTH - 2; i++) {
        drawToBuffer(2 + i, HEIGHT + 1, hud1[i], healthColor);
    }
    for (int i = 0; hud2[i] != '\0' && i < WIDTH - 2; i++) {
        drawToBuffer(2 + i, HEIGHT + 2, hud2[i], 8);
    }
}
```
**설명**: 상단 정보창을 그립니다. 체력에 따라 색상이 변합니다 (초록 > 노랑 > 빨강).

---

### drawGame 함수 (메인 렌더링)

```c
void drawGame(GameState* game) {
    clearBuffer();
    drawBorder();
    drawPlatforms(game);
    drawCoins(game);
    drawEnemies(game);
    drawParticles(game);
    drawPlayer(game);
    drawHUD(game);
    
    if (game->stageCleared) {
        char msg1[] = "STAGE CLEAR!";
        char msg2[] = "Press > to next stage";
        int x1 = (WIDTH - strlen(msg1)) / 2;
        int x2 = (WIDTH - strlen(msg2)) / 2;
        int y = HEIGHT / 2;
        for (int i = 0; msg1[i] != '\0'; i++) {
            drawToBuffer(x1 + i, y, msg1[i], 14);
        }
        for (int i = 0; msg2[i] != '\0'; i++) {
            drawToBuffer(x2 + i, y + 2, msg2[i], 11);
        }
    }
    renderBuffer();
}
```
**설명**: 모든 요소를 순서대로 그리고, 스테이지 클리어 메시지를 표시한 후, 더블 버퍼링으로 화면에 출력합니다.
