# game.h 상세 설명

## 개요
게임의 모든 구조체, 상수, 함수 선언을 포함하는 헤더 파일입니다.

---

## 코드 및 설명

### 헤더 가드

```c
#ifndef GAME_H
#define GAME_H
```
**설명**: 중복 포함 방지를 위한 헤더 가드 시작.

```c
#include <stdio.h>
#include <stdlib.h>
#include <conio.h>
#include <windows.h>
#include <time.h>
#include <string.h>
```
**설명**: 필요한 표준 라이브러리를 포함합니다.

---

### 게임 상수 정의

```c
#define WIDTH 80
```
**설명**: 게임 화면의 가로 크기 (80칸).

```c
#define HEIGHT 25
```
**설명**: 게임 화면의 세로 크기 (25칸).

```c
#define GRAVITY 1
```
**설명**: 중력 가속도. 매 프레임마다 velocityY에 1씩 더해집니다.

```c
#define MAX_ENEMIES 15
```
**설명**: 최대 몬스터 개수 (배열 크기).

```c
#define MAX_COINS 60
```
**설명**: 최대 코인 개수 (히든 스테이지 50개 포함).

```c
#define MAX_PLATFORMS 25
```
**설명**: 최대 플랫폼 개수.

```c
#define MAX_PARTICLES 50
```
**설명**: 최대 파티클 개수 (동시 표시 가능한 효과).

```c
#define JUMP_POWER -3
```
**설명**: 점프 시 초기 수직 속도. 음수는 위쪽 방향.

---

### 플레이어 구조체

```c
typedef struct {
    int x, y;
```
**설명**: 플레이어의 현재 위치 (x: 가로, y: 세로).

```c
    int velocityY;
```
**설명**: 수직 속도. 양수는 아래, 음수는 위로 이동.

```c
    int velocityX;
```
**설명**: 수평 속도. 양수는 오른쪽, 음수는 왼쪽.

```c
    int onGround;
```
**설명**: 지상 상태 (1: 플랫폼 위, 0: 공중).

```c
    int health;
```
**설명**: 현재 체력 (최대 100).

```c
    int score;
```
**설명**: 총 점수 (코인 +50, 몬스터 +100/150/200, 클리어 +500).

```c
    int coins;
```
**설명**: 수집한 코인 개수.

```c
    int lives;
```
**설명**: 남은 생명 (시작 시 3개).

```c
    int invincible;
```
**설명**: 무적 상태 (1: 무적, 0: 일반).

```c
    int invincibleTimer;
```
**설명**: 무적 시간 카운터 (30프레임 = 1.5초).

```c
    char direction;
```
**설명**: 바라보는 방향 ('R': 오른쪽, 'L': 왼쪽).

```c
    char name[50];
} Player;
```
**설명**: 플레이어 이름 (최대 50자).

---

### 플랫폼 구조체

```c
typedef struct {
    int x, y;
```
**설명**: 플랫폼의 시작 위치.

```c
    int width;
```
**설명**: 플랫폼의 너비 (칸 수).

```c
    int active;
} Platform;
```
**설명**: 활성화 여부 (1: 활성, 0: 비활성).

---

### 몬스터 구조체

```c
typedef struct {
    int x, y;
```
**설명**: 몬스터의 현재 위치.

```c
    int velocityX;
    int velocityY;
```
**설명**: 이동 속도 (X: 수평, Y: 수직).

```c
    int active;
```
**설명**: 활성화 여부 (0: 죽음, 1: 살아있음).

```c
    int type;
```
**설명**: 몬스터 타입 (0: 일반 'M', 1: 추적 'W', 2: 비행 'X').

```c
    int homingTimer;
```
**설명**: 추적 타이머 (Type 1 전용, 120프레임마다 추적).

```c
    int homingDuration;
```
**설명**: 추적 지속 시간 (Type 1 전용, 80프레임).

```c
    int respawnTimer;
```
**설명**: 리스폰 타이머 (처치 후 100프레임 후 부활).

```c
    int spawnX;
    int spawnY;
} Enemy;
```
**설명**: 리스폰 위치 (원래 위치).

---

### 코인 구조체

```c
typedef struct {
    int x, y;
```
**설명**: 코인의 위치.

```c
    int collected;
} Coin;
```
**설명**: 수집 여부 (0: 미수집, 1: 수집됨).

---

### 포탈 구조체

```c
typedef struct {
    int x, y;
```
**설명**: 포탈의 위치.

```c
    int active;
} Portal;
```
**설명**: 활성화 여부 (히든 스테이지 진입 가능).

---

### 파티클 구조체

```c
typedef struct {
    int x, y;
```
**설명**: 파티클의 현재 위치.

```c
    int velocityX, velocityY;
```
**설명**: 이동 속도 (중력 영향 받음).

```c
    int life;
```
**설명**: 남은 생명 (0이 되면 사라짐).

```c
    char symbol;
} Particle;
```
**설명**: 파티클 문자 ('*': 처치, '+': 코인, '!': 피격, '.': 점프).

---

### 게임 상태 구조체

```c
typedef struct {
    Player player;
    Platform platforms[MAX_PLATFORMS];
    Enemy enemies[MAX_ENEMIES];
    Coin coins[MAX_COINS];
    Particle particles[MAX_PARTICLES];
    Portal portal;
```
**설명**: 게임의 모든 객체를 포함하는 구조체.

```c
    int platformCount;
    int enemyCount;
    int coinCount;
    int particleCount;
```
**설명**: 각 요소의 실제 개수.

```c
    int gameOver;
```
**설명**: 게임 오버 여부 (1: 종료, 0: 진행).

```c
    int level;
```
**설명**: 현재 스테이지 번호 (1 또는 2).

```c
    int timer;
```
**설명**: 게임 타이머 (프레임 카운터).

```c
    int stageCleared;
```
**설명**: 스테이지 클리어 여부.

```c
    int inHiddenStage;
```
**설명**: 히든 스테이지 진입 여부.

```c
    int hiddenStageTimer;
```
**설명**: 히든 스테이지 제한 시간 (200프레임 = 10초).

```c
    int returnLevel;
} GameState;
```
**설명**: 히든 스테이지에서 돌아갈 원래 스테이지.

---

### 함수 선언

```c
void initGame(GameState* game);
```
**설명**: 게임 전체 초기화.

```c
void initPlayer(Player* player);
```
**설명**: 플레이어 초기화.

```c
void initPlatforms(GameState* game);
```
**설명**: Stage 1 플랫폼 배치.

```c
void initEnemies(GameState* game);
```
**설명**: Stage 1 몬스터 배치.

```c
void initCoins(GameState* game);
```
**설명**: Stage 1 코인 배치.

```c
void updatePlayer(GameState* game);
```
**설명**: 플레이어 물리 및 충돌 업데이트.

```c
void updateEnemies(GameState* game);
```
**설명**: 몬스터 AI 및 충돌 업데이트.

```c
void updateCoins(GameState* game);
```
**설명**: 코인 수집 및 포탈 확인.

```c
void updateParticles(GameState* game);
```
**설명**: 파티클 효과 업데이트.

```c
void handleInput(GameState* game);
```
**설명**: 키보드 입력 처리.

```c
void checkGameOver(GameState* game);
```
**설명**: 게임 오버 조건 확인.

```c
void addParticle(GameState* game, int x, int y, char symbol);
```
**설명**: 새 파티클 생성.

```c
void initStage2(GameState* game);
```
**설명**: Stage 2 초기화.

```c
void nextStage(GameState* game);
```
**설명**: 다음 스테이지 진행.

```c
void initHiddenStage(GameState* game);
```
**설명**: 히든 스테이지 초기화.

```c
void returnFromHiddenStage(GameState* game);
```
**설명**: 히든 스테이지에서 복귀.

```c
#endif
```
**설명**: 헤더 가드 종료.
