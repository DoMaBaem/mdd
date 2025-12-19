# game.c 상세 설명 (Part 4 - 입력 처리 및 게임 오버)

## updateCoins 함수

```c
void updateCoins(GameState* game) {
```
**설명**: 코인 수집 및 포탈 진입을 처리하는 함수입니다.

---

### 코인 수집

```c
    for (int i = 0; i < game->coinCount; i++) {
        if (!game->coins[i].collected) {
```
**설명**: 아직 수집되지 않은 코인만 검사합니다.

```c
            if (abs(game->player.x - game->coins[i].x) <= 1 &&
                abs(game->player.y - game->coins[i].y) <= 1 &&
                game->coins[i].y <= game->player.y) {
```
**설명**: 플레이어가 코인과 가까이 있고, 코인이 플레이어와 같은 높이거나 위에 있을 때.

```c
                game->coins[i].collected = 1;
                game->player.coins++;
                game->player.score += 50;
                addParticle(game, game->coins[i].x, game->coins[i].y, '+');
            }
        }
    }
```
**설명**: 코인을 수집하고, 개수와 점수를 증가시키며, '+' 파티클을 생성합니다.

---

### 포탈 진입

```c
    if (game->portal.active && !game->inHiddenStage) {
```
**설명**: 포탈이 활성화되어 있고 히든 스테이지가 아닐 때.

```c
        if (abs(game->player.x - game->portal.x) <= 1 &&
            abs(game->player.y - game->portal.y) <= 1) {
            initHiddenStage(game);
        }
    }
```
**설명**: 플레이어가 포탈에 닿으면 히든 스테이지로 진입합니다.

---

### 히든 스테이지 타이머

```c
    if (game->inHiddenStage) {
        game->hiddenStageTimer--;
        if (game->hiddenStageTimer <= 0) {
            returnFromHiddenStage(game);
        }
    }
}
```
**설명**: 히든 스테이지에서 타이머를 감소시키고, 0이 되면 원래 스테이지로 복귀합니다.

---

## handleInput 함수

```c
void handleInput(GameState* game) {
    static int leftPressed = 0, rightPressed = 0;
```
**설명**: 키보드 입력을 처리하는 함수. static 변수로 키 입력 상태를 유지합니다.

```c
    if (_kbhit()) {
        char key = _getch();
```
**설명**: 키가 눌렸는지 확인하고, 눌린 키를 읽습니다.

---

### ESC 키 (게임 종료)

```c
        if (key == 27) {
            game->gameOver = 1;
        }
```
**설명**: ESC 키(ASCII 27)를 누르면 게임을 종료합니다.

---

### 방향키 입력

```c
        if (key == -32) {
            key = _getch();
```
**설명**: 방향키는 2바이트 입력이므로 두 번째 바이트를 읽습니다.

```c
            if (key == 75) {
                leftPressed = 1;
                game->player.direction = 'L';
            }
```
**설명**: 왼쪽 방향키(75) - 왼쪽 이동 플래그 설정.

```c
            if (key == 77) {
                rightPressed = 1;
                game->player.direction = 'R';
            }
```
**설명**: 오른쪽 방향키(77) - 오른쪽 이동 플래그 설정.

```c
            if (key == 72) {
                if (game->player.onGround) {
                    int jumpPower = game->inHiddenStage ? -5 : JUMP_POWER;
                    game->player.velocityY = jumpPower;
                    playSFX(SFX_JUMP);
                    addParticle(game, game->player.x, game->player.y + 1, '.');
                }
            }
```
**설명**: 위쪽 방향키(72) - 지상에 있을 때만 점프. 히든 스테이지에서는 점프력이 더 강합니다.

```c
            if (key == 80) {
                if (game->player.onGround) {
                    game->player.y += 2;
                }
            }
        }
```
**설명**: 아래쪽 방향키(80) - 플랫폼을 통과하여 아래로 내려갑니다.

---

### 스페이스바 (점프)

```c
        if (key == ' ') {
            if (game->player.onGround) {
                int jumpPower = game->inHiddenStage ? -5 : JUMP_POWER;
                game->player.velocityY = jumpPower;
                playSFX(SFX_JUMP);
                addParticle(game, game->player.x, game->player.y + 1, '.');
            }
        }
```
**설명**: 스페이스바로도 점프할 수 있습니다.

---

### A/D 키 (좌우 이동)

```c
        if (key == 'a' || key == 'A') {
            leftPressed = 1;
            game->player.direction = 'L';
        }
        
        if (key == 'd' || key == 'D') {
            rightPressed = 1;
            game->player.direction = 'R';
        }
```
**설명**: A/D 키로 좌우 이동합니다.

---

### W 키 (점프)

```c
        if (key == 'w' || key == 'W') {
            if (game->player.onGround) {
                int jumpPower = game->inHiddenStage ? -5 : JUMP_POWER;
                game->player.velocityY = jumpPower;
                addParticle(game, game->player.x, game->player.y + 1, '.');
            }
        }
```
**설명**: W 키로도 점프할 수 있습니다.

---

### R 키 (재시작)

```c
        if (key == 'r' || key == 'R') {
            initPlayer(&game->player);
            if (game->level == 1) {
                initPlatforms(game);
                initEnemies(game);
                initCoins(game);
            } else {
                initStage2(game);
            }
        }
```
**설명**: R 키로 현재 스테이지를 재시작합니다.

---

### 이동 속도 적용

```c
    if (leftPressed) {
        game->player.velocityX = -1;
        leftPressed = 0;
    }
    if (rightPressed) {
        game->player.velocityX = 1;
        rightPressed = 0;
    }
}
```
**설명**: 플래그가 설정되어 있으면 속도를 적용하고 플래그를 초기화합니다.

---

## checkGameOver 함수

```c
void checkGameOver(GameState* game) {
```
**설명**: 게임 오버 조건을 확인하는 함수입니다.

---

### 체력 0 처리

```c
    if (game->player.health <= 0) {
        game->player.lives--;
```
**설명**: 체력이 0 이하면 생명을 1개 감소시킵니다.

```c
        if (game->player.lives > 0) {
            game->player.health = 100;
            game->player.x = 5;
            game->player.y = HEIGHT - 3;
            game->player.velocityY = 0;
            game->player.velocityX = 0;
            game->player.invincible = 1;
            game->player.invincibleTimer = 60;
        }
```
**설명**: 생명이 남아있으면 체력을 회복하고 시작 위치로 리스폰합니다. 60프레임 무적 부여.

```c
        else {
            playSFX(SFX_DIE);
            game->gameOver = 1;
        }
    }
```
**설명**: 생명이 0개면 죽음 효과음을 재생하고 게임 오버합니다.

---

### 스테이지 클리어 확인

```c
    int allCoinsCollected = 1;
    for (int i = 0; i < game->coinCount; i++) {
        if (!game->coins[i].collected) {
            allCoinsCollected = 0;
            break;
        }
    }
```
**설명**: 모든 코인이 수집되었는지 확인합니다.

```c
    if (allCoinsCollected) {
        game->player.score += 500;
        game->stageCleared = 1;
```
**설명**: 모든 코인 수집 시 500점 보너스를 부여하고 클리어 플래그를 설정합니다.

```c
        if (game->level < 2) {
            Sleep(1500);
            nextStage(game);
        } else {
            game->gameOver = 1;
        }
    }
}
```
**설명**: Stage 1이면 1.5초 후 Stage 2로 진행하고, Stage 2면 게임을 종료합니다.

---

## 입력 처리 요약

### 이동 키
- **A / ←**: 왼쪽 이동
- **D / →**: 오른쪽 이동

### 점프 키
- **W / ↑ / Space**: 점프 (지상에서만)

### 기타 키
- **↓**: 플랫폼 통과하여 내려가기
- **R**: 현재 스테이지 재시작
- **ESC**: 게임 종료
