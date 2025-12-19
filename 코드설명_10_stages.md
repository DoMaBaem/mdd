# game.c 상세 설명 (Part 5 - 스테이지 시스템)

## initStage2 함수

Stage 2를 초기화하는 함수입니다. Stage 1보다 난이도가 높습니다.

```c
void initStage2(GameState* game) {
```
**설명**: Stage 2 초기화 함수.

---

### 플랫폼 배치 (20개)

```c
    game->platformCount = 20;
```
**설명**: Stage 2는 플랫폼이 20개입니다 (Stage 1은 24개).

```c
    game->platforms[0].x = 1; 
    game->platforms[0].y = HEIGHT - 2; 
    game->platforms[0].width = 15; 
    game->platforms[0].active = 1;
```
**설명**: 플랫폼 0번 - 왼쪽 바닥 (너비 15).

```c
    game->platforms[1].x = 25; 
    game->platforms[1].y = HEIGHT - 2; 
    game->platforms[1].width = 15; 
    game->platforms[1].active = 1;
```
**설명**: 플랫폼 1번 - 중앙 바닥 (너비 15).

```c
    game->platforms[2].x = 50; 
    game->platforms[2].y = HEIGHT - 2; 
    game->platforms[2].width = 15; 
    game->platforms[2].active = 1;
```
**설명**: 플랫폼 2번 - 오른쪽 바닥 (너비 15).

**나머지 플랫폼 (3~19번)**:
- Stage 1보다 플랫폼 간격이 넓습니다.
- 더 높은 곳에 배치되어 점프 난이도가 높습니다.
- 일부 플랫폼은 너비가 좁아 정확한 착지가 필요합니다.

---

### 몬스터 배치 (12개)

```c
    game->enemyCount = 12;
```
**설명**: Stage 2는 몬스터가 12개입니다.

#### Type 1 몬스터 (추적 몬스터 8개)

```c
    int pos[][2] = {{10,HEIGHT-6},{25,HEIGHT-7},{38,HEIGHT-9},{52,HEIGHT-8},
                    {67,HEIGHT-10},{15,HEIGHT-11},{30,HEIGHT-13},{47,HEIGHT-14}};
    int vel[] = {2,-2,2,-2,2,-2,2,-2};
```
**설명**: 8개의 추적 몬스터 위치와 속도를 배열로 정의합니다.

```c
    for (int i = 0; i < 8; i++) {
        game->enemies[i].x = pos[i][0]; 
        game->enemies[i].y = pos[i][1]; 
        game->enemies[i].velocityX = vel[i]; 
        game->enemies[i].velocityY = 0;
        game->enemies[i].active = 1; 
        game->enemies[i].type = 1;
        game->enemies[i].homingTimer = 0; 
        game->enemies[i].homingDuration = 0;
        game->enemies[i].respawnTimer = 0; 
        game->enemies[i].spawnX = pos[i][0]; 
        game->enemies[i].spawnY = pos[i][1];
    }
```
**설명**: 반복문으로 Type 1 몬스터 8개를 초기화합니다.

#### Type 2 몬스터 (비행 몬스터 4개)

```c
    int dpos[][2] = {{20,HEIGHT-15},{45,HEIGHT-18},{60,HEIGHT-12},{35,HEIGHT-20}};
    int dvel[][2] = {{1,-1},{-1,1},{1,1},{-1,-1}};
```
**설명**: 4개의 비행 몬스터 위치와 속도(X, Y)를 배열로 정의합니다.

```c
    for (int i = 0; i < 4; i++) {
        game->enemies[8+i].x = dpos[i][0]; 
        game->enemies[8+i].y = dpos[i][1];
        game->enemies[8+i].velocityX = dvel[i][0]; 
        game->enemies[8+i].velocityY = dvel[i][1];
        game->enemies[8+i].active = 1; 
        game->enemies[8+i].type = 2;
        game->enemies[8+i].homingTimer = 0; 
        game->enemies[8+i].homingDuration = 0;
        game->enemies[8+i].respawnTimer = 0; 
        game->enemies[8+i].spawnX = dpos[i][0]; 
        game->enemies[8+i].spawnY = dpos[i][1];
    }
```
**설명**: 반복문으로 Type 2 몬스터 4개를 초기화합니다 (인덱스 8~11).

---

### 코인 배치 (20개)

```c
    game->coinCount = 20;
```
**설명**: Stage 2는 코인이 20개입니다 (Stage 1은 38개).

```c
    int cpos[][2] = {{10,HEIGHT-6},{24,HEIGHT-7},{37,HEIGHT-9},{52,HEIGHT-8},
                     {67,HEIGHT-10},{14,HEIGHT-11},{30,HEIGHT-13},{47,HEIGHT-14},
                     {62,HEIGHT-12},{7,HEIGHT-16},{22,HEIGHT-18},{37,HEIGHT-19},
                     {52,HEIGHT-17},{67,HEIGHT-20},{17,HEIGHT-22},{42,HEIGHT-23},
                     {62,HEIGHT-24},{32,HEIGHT-13},{48,HEIGHT-14},{70,HEIGHT-10}};
```
**설명**: 20개의 코인 위치를 배열로 정의합니다.

```c
    for (int i = 0; i < 20; i++) {
        game->coins[i].x = cpos[i][0]; 
        game->coins[i].y = cpos[i][1]; 
        game->coins[i].collected = 0;
    }
}
```
**설명**: 반복문으로 코인 20개를 초기화합니다.

**Stage 2 특징**:
- 추적 몬스터(Type 1)가 8개로 증가
- 비행 몬스터(Type 2)가 4개 추가
- 플랫폼 간격이 넓어 점프 난이도 상승
- 코인 개수는 감소하지만 수집 난이도 증가

---

## initHiddenStage 함수

포탈을 통해 진입하는 히든 스테이지를 초기화합니다.

```c
void initHiddenStage(GameState* game) {
```
**설명**: 히든 스테이지 초기화 함수.

```c
    game->inHiddenStage = 1;
    game->hiddenStageTimer = 200;
    game->returnLevel = game->level;
```
**설명**: 히든 스테이지 플래그를 설정하고, 타이머를 200프레임(10초)으로 설정하며, 원래 스테이지를 저장합니다.

---

### 플랫폼 설정 (1개 - 바닥만)

```c
    game->platformCount = 1;
    game->platforms[0].x = 1;
    game->platforms[0].y = HEIGHT - 2;
    game->platforms[0].width = WIDTH - 2;
    game->platforms[0].active = 1;
```
**설명**: 바닥 플랫폼 하나만 배치합니다 (거의 전체 너비).

---

### 몬스터 제거

```c
    game->enemyCount = 0;
```
**설명**: 히든 스테이지에는 몬스터가 없습니다.

---

### 코인 배치 (50개)

```c
    for (int i = 0; i < MAX_COINS; i++) {
        game->coins[i].collected = 1;
    }
```
**설명**: 기존 코인을 모두 수집된 것으로 처리합니다.

```c
    game->coinCount = 50;
    for (int i = 0; i < 50; i++) {
        game->coins[i].x = 3 + (i % 25) * 3;
        game->coins[i].y = (i < 25) ? HEIGHT - 3 : HEIGHT - 7;
        game->coins[i].collected = 0;
    }
```
**설명**: 50개의 코인을 2줄로 배치합니다.
- 첫 번째 줄 (0~24): Y = HEIGHT - 3
- 두 번째 줄 (25~49): Y = HEIGHT - 7
- X 간격: 3칸씩

**코인 배치 패턴**:
```
줄 2: $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $
줄 1: $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $
바닥: ========================================
```

---

### 포탈 비활성화

```c
    game->portal.active = 0;
```
**설명**: 히든 스테이지에서는 포탈을 비활성화합니다 (재진입 방지).

---

### 플레이어 위치 초기화

```c
    game->player.x = 5;
    game->player.y = HEIGHT - 3;
}
```
**설명**: 플레이어를 시작 위치로 이동시킵니다.

**히든 스테이지 특징**:
- 제한 시간: 10초 (200프레임)
- 몬스터 없음
- 코인 50개 (최대 2500점)
- 점프력 증가 (JUMP_POWER -5)
- 빠른 코인 수집 가능

---

## returnFromHiddenStage 함수

히든 스테이지에서 원래 스테이지로 복귀하는 함수입니다.

```c
void returnFromHiddenStage(GameState* game) {
```
**설명**: 히든 스테이지 복귀 함수.

```c
    game->inHiddenStage = 0;
    game->hiddenStageTimer = 0;
    game->level = game->returnLevel;
```
**설명**: 히든 스테이지 플래그를 해제하고, 타이머를 초기화하며, 원래 스테이지로 복귀합니다.

```c
    if (game->level == 1) {
        initPlatforms(game);
        initEnemies(game);
        initCoins(game);
    } else {
        initStage2(game);
    }
```
**설명**: 원래 스테이지를 다시 초기화합니다.

```c
    game->player.x = 5;
    game->player.y = HEIGHT - 3;
}
```
**설명**: 플레이어를 시작 위치로 이동시킵니다.

**복귀 시 주의사항**:
- 히든 스테이지에서 수집한 코인은 유지됩니다.
- 원래 스테이지의 코인은 초기화됩니다 (다시 수집 가능).
- 플레이어의 체력과 생명은 유지됩니다.

---

## nextStage 함수

다음 스테이지로 진행하는 함수입니다.

```c
void nextStage(GameState* game) {
```
**설명**: 스테이지 진행 함수.

```c
    game->stageCleared = 0;
    game->level++;
```
**설명**: 클리어 플래그를 해제하고 레벨을 1 증가시킵니다.

```c
    game->player.x = 5;
    game->player.y = HEIGHT - 3;
    game->player.velocityX = 0;
    game->player.velocityY = 0;
```
**설명**: 플레이어를 시작 위치로 이동시키고 속도를 초기화합니다.

```c
    if (game->level == 2) {
        initStage2(game);
    } else {
        game->gameOver = 1;
    }
}
```
**설명**: 레벨 2면 Stage 2를 초기화하고, 그 이상이면 게임을 종료합니다.

---

## initGame 함수

게임 전체를 초기화하는 함수입니다.

```c
void initGame(GameState* game) {
```
**설명**: 게임 초기화 함수.

```c
    game->gameOver = 0;
    game->level = 1;
    game->timer = 0;
    game->particleCount = 0;
    game->stageCleared = 0;
    game->inHiddenStage = 0;
    game->hiddenStageTimer = 0;
    game->returnLevel = 1;
```
**설명**: 게임 상태 변수들을 초기화합니다.

```c
    for (int i = 0; i < MAX_PARTICLES; i++) {
        game->particles[i].life = 0;
    }
```
**설명**: 모든 파티클의 생명을 0으로 설정합니다.

```c
    initPlayer(&game->player);
    initPlatforms(game);
    initEnemies(game);
    initCoins(game);
}
```
**설명**: 플레이어, 플랫폼, 몬스터, 코인을 초기화합니다 (Stage 1).

---

## 스테이지 시스템 요약

### Stage 1
- **플랫폼**: 24개
- **몬스터**: 14개 (Type 0: 2개, Type 1: 12개)
- **코인**: 38개
- **포탈**: 1개 (히든 스테이지 진입)
- **난이도**: 보통

### Stage 2
- **플랫폼**: 20개
- **몬스터**: 12개 (Type 1: 8개, Type 2: 4개)
- **코인**: 20개
- **포탈**: 없음
- **난이도**: 높음 (비행 몬스터 추가)

### Hidden Stage
- **플랫폼**: 1개 (바닥만)
- **몬스터**: 0개
- **코인**: 50개
- **제한 시간**: 10초
- **특징**: 점프력 증가, 빠른 코인 수집

### 진행 순서
1. Stage 1 시작
2. 포탈 진입 → Hidden Stage (선택)
3. Stage 1 코인 모두 수집 → Stage 2
4. Stage 2 코인 모두 수집 → 게임 종료
