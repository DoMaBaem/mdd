# game.c 상세 설명 (Part 3 - updateEnemies)

## updateEnemies 함수

몬스터의 AI, 이동, 플레이어와의 충돌을 처리하는 함수입니다.

```c
void updateEnemies(GameState* game) {
    for (int i = 0; i < game->enemyCount; i++) {
```
**설명**: 모든 몬스터를 순회합니다.

---

### 리스폰 처리

```c
        if (!game->enemies[i].active && game->enemies[i].respawnTimer > 0) {
```
**설명**: 죽은 몬스터의 리스폰 타이머가 남아있으면.

```c
            game->enemies[i].respawnTimer--;
```
**설명**: 타이머를 1 감소시킵니다.

```c
            if (game->enemies[i].respawnTimer == 0) {
                game->enemies[i].active = 1;
                game->enemies[i].x = game->enemies[i].spawnX;
                game->enemies[i].y = game->enemies[i].spawnY;
            }
        }
```
**설명**: 타이머가 0이 되면 원래 위치에서 부활합니다.

---

### Type 2 (비행 몬스터) 이동

```c
        if (game->enemies[i].active) {
            if (game->enemies[i].type == 2) {
```
**설명**: 활성화된 Type 2 몬스터 처리.

```c
                game->enemies[i].x += game->enemies[i].velocityX;
                game->enemies[i].y += game->enemies[i].velocityY;
```
**설명**: X와 Y 방향으로 동시에 이동 (대각선 이동).

```c
                if (game->enemies[i].x <= 2 || game->enemies[i].x >= WIDTH - 3) {
                    game->enemies[i].velocityX *= -1;
                }
```
**설명**: 좌우 벽에 닿으면 X 속도를 반전시킵니다 (반사).

```c
                if (game->enemies[i].y <= 1 || game->enemies[i].y >= HEIGHT - 1) {
                    game->enemies[i].velocityY *= -1;
                }
```
**설명**: 상하 벽에 닿으면 Y 속도를 반전시킵니다 (반사).

---

### Type 1 (추적 몬스터) AI

```c
            } else if (game->enemies[i].type == 1) {
```
**설명**: Type 1 몬스터 처리.

```c
                int speed = abs(game->enemies[i].velocityX);
```
**설명**: 현재 속도의 절댓값을 저장 (방향 변경 시 사용).

```c
                if (game->enemies[i].homingDuration > 0) {
```
**설명**: 추적 모드가 활성화되어 있으면.

```c
                    game->enemies[i].homingDuration--;
```
**설명**: 추적 지속 시간을 1 감소시킵니다.

```c
                    if (game->player.x > game->enemies[i].x) {
                        game->enemies[i].velocityX = speed;
                    }
                    else if (game->player.x < game->enemies[i].x) {
                        game->enemies[i].velocityX = -speed;
                    }
```
**설명**: 플레이어 방향으로 이동합니다 (호밍).

```c
                } else {
                    game->enemies[i].homingTimer++;
```
**설명**: 추적 모드가 아니면 타이머를 증가시킵니다.

```c
                    if (game->enemies[i].homingTimer >= 120) {
                        game->enemies[i].homingDuration = 80;
                        game->enemies[i].homingTimer = 0;
                    }
                }
            }
```
**설명**: 120프레임마다 80프레임 동안 추적 모드를 활성화합니다.

---

### Type 0, 1 (일반/추적) 이동

```c
            if (game->enemies[i].type != 2) {
```
**설명**: Type 2가 아닌 몬스터는 수평으로만 이동합니다.

```c
                game->enemies[i].x += game->enemies[i].velocityX;
```
**설명**: X 좌표를 속도만큼 이동시킵니다.

```c
                if (game->enemies[i].x <= 2 || game->enemies[i].x >= WIDTH - 3) {
                    game->enemies[i].velocityX *= -1;
                }
            }
```
**설명**: 벽에 닿으면 방향을 반전시킵니다.

---

### 플레이어와의 충돌 검사

```c
            int dx = abs(game->player.x - game->enemies[i].x);
            int dy = game->player.y - game->enemies[i].y;
```
**설명**: 플레이어와 몬스터 사이의 거리를 계산합니다.
- dx: 수평 거리 (절댓값)
- dy: 수직 거리 (플레이어 - 몬스터)

---

### 몬스터 밟기 (처치)

```c
            if (dx <= 1 && dy == -1 && game->player.velocityY > 0) {
```
**설명**: 플레이어가 몬스터 바로 위에서 아래로 떨어지고 있을 때.

```c
                game->enemies[i].active = 0;
                game->enemies[i].respawnTimer = 100;
```
**설명**: 몬스터를 비활성화하고 100프레임 후 리스폰 설정.

```c
                game->player.velocityY = -4;
```
**설명**: 플레이어를 위로 튕겨 올립니다 (연속 점프 가능).

```c
                if (game->enemies[i].type == 0) {
                    game->player.score += 100;
                } else if (game->enemies[i].type == 1) {
                    game->player.score += 150;
                } else {
                    game->player.score += 200;
                }
```
**설명**: 몬스터 타입에 따라 점수를 부여합니다.
- Type 0: +100점
- Type 1: +150점
- Type 2: +200점

```c
                for (int j = 0; j < 5; j++) {
                    addParticle(game, game->enemies[i].x, game->enemies[i].y, '*');
                }
            }
```
**설명**: 처치 효과로 '*' 파티클 5개를 생성합니다.

---

### 몬스터 충돌 (피격)

```c
            else if (dx <= 1 && dy == 0 && !game->player.invincible) {
```
**설명**: 플레이어가 몬스터와 같은 높이에서 충돌하고 무적이 아닐 때.

```c
                int damage = (game->enemies[i].type == 0) ? 15 : 
                             (game->enemies[i].type == 1) ? 25 : 30;
                game->player.health -= damage;
```
**설명**: 몬스터 타입에 따라 데미지를 입힙니다.
- Type 0: -15 HP
- Type 1: -25 HP
- Type 2: -30 HP

```c
                game->player.invincible = 1;
                game->player.invincibleTimer = 30;
```
**설명**: 무적 상태를 30프레임(1.5초) 동안 활성화합니다.

```c
                playSFX(SFX_HIT);
```
**설명**: 피격 효과음을 재생합니다.

```c
                if (game->player.x < game->enemies[i].x) {
                    game->player.velocityX = -2;
                }
                else {
                    game->player.velocityX = 2;
                }
                game->player.velocityY = -2;
```
**설명**: 넉백 효과 - 몬스터 반대 방향으로 밀려나고 위로 튕깁니다.

```c
                for (int j = 0; j < 3; j++) {
                    addParticle(game, game->player.x, game->player.y, '!');
                }
            }
        }
    }
}
```
**설명**: 피격 효과로 '!' 파티클 3개를 생성합니다.

---

## 몬스터 AI 요약

### Type 0 (일반 몬스터 'M')
- 좌우로 일정 속도로 이동
- 벽에 닿으면 방향 전환
- 데미지: 15 HP
- 점수: 100점

### Type 1 (추적 몬스터 'W')
- 기본적으로 Type 0과 동일
- 120프레임마다 80프레임 동안 플레이어 추적
- 데미지: 25 HP
- 점수: 150점

### Type 2 (비행 몬스터 'X')
- 대각선으로 자유롭게 이동
- 벽과 천장에 반사
- 데미지: 30 HP
- 점수: 200점
