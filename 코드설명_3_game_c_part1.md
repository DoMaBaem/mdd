# game.c 상세 설명 (Part 1)

## 개요
게임 로직의 핵심 구현 파일입니다. 플레이어, 몬스터, 코인, 플랫폼 등의 초기화 및 업데이트를 담당합니다.

---

## initPlayer 함수

```c
void initPlayer(Player* player) {
```
**설명**: 플레이어를 초기 상태로 설정하는 함수입니다.

```c
    player->x = 5;
```
**설명**: 시작 X 좌표를 5로 설정 (화면 왼쪽).

```c
    player->y = HEIGHT - 3;
```
**설명**: 시작 Y 좌표를 바닥에서 3칸 위로 설정.

```c
    player->velocityY = 0;
```
**설명**: 수직 속도를 0으로 초기화 (정지 상태).

```c
    player->velocityX = 0;
```
**설명**: 수평 속도를 0으로 초기화.

```c
    player->onGround = 0;
```
**설명**: 공중 상태로 시작 (곧 중력으로 떨어짐).

```c
    player->health = 100;
```
**설명**: 최대 체력 100으로 설정.

```c
    player->score = 0;
```
**설명**: 점수를 0으로 초기화.

```c
    player->coins = 0;
```
**설명**: 코인 개수를 0으로 초기화.

```c
    player->lives = 3;
```
**설명**: 생명을 3개로 설정.

```c
    player->invincible = 0;
```
**설명**: 무적 상태 아님.

```c
    player->invincibleTimer = 0;
```
**설명**: 무적 타이머를 0으로 초기화.

```c
    player->direction = 'R';
}
```
**설명**: 기본 방향을 오른쪽('R')으로 설정.

---

## initPlatforms 함수 (Stage 1)

```c
void initPlatforms(GameState* game) {
```
**설명**: Stage 1의 플랫폼 24개를 배치하는 함수입니다.

```c
    game->platformCount = 24;
```
**설명**: 총 플랫폼 개수를 24로 설정.

```c
    game->portal.x = 47;
    game->portal.y = HEIGHT - 19;
    game->portal.active = 1;
```
**설명**: 포탈 위치를 설정 (플랫폼 22번 위, 히든 스테이지 진입용).

```c
    game->platforms[0].x = 1;
    game->platforms[0].y = HEIGHT - 2;
    game->platforms[0].width = WIDTH - 2;
    game->platforms[0].active = 1;
```
**설명**: 플랫폼 0번 - 바닥 플랫폼 (거의 전체 너비).

```c
    game->platforms[1].x = 5;
    game->platforms[1].y = HEIGHT - 4;
    game->platforms[1].width = 10;
    game->platforms[1].active = 1;
```
**설명**: 플랫폼 1번 - 바닥에서 4칸 위, 너비 10.

```c
    game->platforms[2].x = 12;
    game->platforms[2].y = HEIGHT - 4;
    game->platforms[2].width = 10;
    game->platforms[2].active = 1;
```
**설명**: 플랫폼 2번 - 같은 높이, 오른쪽으로 배치.

**나머지 플랫폼들 (3~23번)**:
- 각 플랫폼은 x, y 좌표와 width(너비)를 가집니다.
- 다양한 높이에 배치되어 점프 난이도를 조절합니다.
- 높이가 높을수록 Y 값이 작아집니다 (화면 위쪽).
- 플랫폼 22번은 가장 높은 위치에 있으며 포탈이 그 위에 있습니다.

---

## initEnemies 함수 (Stage 1)

```c
void initEnemies(GameState* game) {
```
**설명**: Stage 1의 몬스터 14개를 배치하는 함수입니다.

```c
    game->enemyCount = 14;
```
**설명**: 총 몬스터 개수를 14로 설정.

```c
    game->enemies[0].x = 11;
    game->enemies[0].y = HEIGHT - 5;
    game->enemies[0].velocityX = 1;
    game->enemies[0].velocityY = 0;
    game->enemies[0].active = 1;
    game->enemies[0].type = 0;
    game->enemies[0].homingTimer = 0;
    game->enemies[0].homingDuration = 0;
    game->enemies[0].respawnTimer = 0;
    game->enemies[0].spawnX = 11;
    game->enemies[0].spawnY = HEIGHT - 5;
```
**설명**: 첫 번째 몬스터 설정.
- 위치: (11, HEIGHT-5)
- 속도: 오른쪽으로 1칸씩 이동
- 타입: 0 (일반 몬스터 'M')
- 리스폰 위치 저장

```c
    int positions[][2] = {{33, HEIGHT-7}, {18, HEIGHT-9}, ...};
    int velocities[] = {-1, 2, -1, 2, 1, -2, 1, -2, 2, 2, -2, 2, -2};
    int types[] = {0, 1, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1, 1};
```
**설명**: 나머지 13개 몬스터의 데이터를 배열로 정의.
- positions: 각 몬스터의 (x, y) 위치
- velocities: 각 몬스터의 이동 속도
- types: 몬스터 타입 (0: 일반, 1: 추적)

```c
    for (int i = 1; i < 14; i++) {
        game->enemies[i].x = positions[i-1][0];
        game->enemies[i].y = positions[i-1][1];
        game->enemies[i].velocityX = velocities[i-1];
        game->enemies[i].velocityY = 0;
        game->enemies[i].active = 1;
        game->enemies[i].type = types[i-1];
        game->enemies[i].homingTimer = 0;
        game->enemies[i].homingDuration = 0;
        game->enemies[i].respawnTimer = 0;
        game->enemies[i].spawnX = positions[i-1][0];
        game->enemies[i].spawnY = positions[i-1][1];
    }
}
```
**설명**: 반복문으로 나머지 몬스터들을 초기화합니다.

---

## initCoins 함수 (Stage 1)

```c
void initCoins(GameState* game) {
```
**설명**: Stage 1의 코인 38개를 배치하는 함수입니다.

```c
    game->coinCount = 38;
```
**설명**: 총 코인 개수를 38로 설정.

```c
    game->coins[0].x = 10;
    game->coins[0].y = HEIGHT - 5;
    game->coins[0].collected = 0;
```
**설명**: 첫 번째 코인 설정.
- 위치: (10, HEIGHT-5)
- collected: 0 (아직 수집 안 됨)

**나머지 코인들 (1~37번)**:
- 각 코인은 x, y 좌표를 가집니다.
- 플랫폼 근처에 배치되어 플레이어가 수집할 수 있습니다.
- 일부는 점프해야만 닿을 수 있는 높이에 배치됩니다.
- 모든 코인의 collected는 0으로 초기화됩니다.

---

## addParticle 함수

```c
void addParticle(GameState* game, int x, int y, char symbol) {
```
**설명**: 새로운 파티클을 생성하는 함수입니다.

```c
    for (int i = 0; i < MAX_PARTICLES; i++) {
```
**설명**: 파티클 배열을 순회하며 빈 슬롯을 찾습니다.

```c
        if (game->particles[i].life <= 0) {
```
**설명**: 생명이 0 이하인 파티클은 사용 가능합니다.

```c
            game->particles[i].x = x;
            game->particles[i].y = y;
```
**설명**: 파티클의 시작 위치를 설정합니다.

```c
            game->particles[i].velocityX = (rand() % 3) - 1;
```
**설명**: 수평 속도를 랜덤으로 설정 (-1, 0, 1).

```c
            game->particles[i].velocityY = -(rand() % 3 + 1);
```
**설명**: 수직 속도를 위쪽으로 설정 (-1, -2, -3).

```c
            game->particles[i].life = 10;
```
**설명**: 파티클의 생명을 10프레임으로 설정.

```c
            game->particles[i].symbol = symbol;
            break;
        }
    }
}
```
**설명**: 파티클 문자를 설정하고 함수를 종료합니다.

---

## updateParticles 함수

```c
void updateParticles(GameState* game) {
```
**설명**: 모든 파티클을 업데이트하는 함수입니다.

```c
    for (int i = 0; i < MAX_PARTICLES; i++) {
```
**설명**: 모든 파티클을 순회합니다.

```c
        if (game->particles[i].life > 0) {
```
**설명**: 생명이 남은 파티클만 처리합니다.

```c
            game->particles[i].x += game->particles[i].velocityX;
            game->particles[i].y += game->particles[i].velocityY;
```
**설명**: 속도에 따라 위치를 업데이트합니다.

```c
            game->particles[i].velocityY++;
```
**설명**: 중력 효과 (아래로 가속).

```c
            game->particles[i].life--;
        }
    }
}
```
**설명**: 생명을 1 감소시킵니다.
