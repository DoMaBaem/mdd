# game.c 상세 설명 (Part 2 - updatePlayer)

## updatePlayer 함수

플레이어의 물리 엔진과 충돌 감지를 담당하는 핵심 함수입니다.

```c
void updatePlayer(GameState* game) {
    Player* player = &game->player;
```
**설명**: 플레이어 포인터를 가져와 코드를 간결하게 만듭니다.

---

### 중력 적용

```c
    player->velocityY += GRAVITY;
```
**설명**: 매 프레임마다 수직 속도에 중력(1)을 더합니다. 아래로 가속됩니다.

```c
    if (player->velocityY > 10) player->velocityY = 10;
```
**설명**: 최대 낙하 속도를 10으로 제한합니다 (터미널 속도).

---

### 위치 업데이트

```c
    player->y += player->velocityY;
```
**설명**: 수직 속도를 Y 좌표에 적용합니다.

```c
    player->x += player->velocityX;
```
**설명**: 수평 속도를 X 좌표에 적용합니다.

---

### 화면 경계 체크

```c
    if (player->x < 1) {
        player->x = 1;
        player->velocityX = 0;
    }
```
**설명**: 왼쪽 벽에 닿으면 위치를 보정하고 속도를 0으로 만듭니다.

```c
    if (player->x >= WIDTH - 1) {
        player->x = WIDTH - 2;
        player->velocityX = 0;
    }
```
**설명**: 오른쪽 벽에 닿으면 위치를 보정하고 속도를 0으로 만듭니다.

---

### 플랫폼 충돌 검사

```c
    player->onGround = 0;
```
**설명**: 기본적으로 공중 상태로 설정합니다.

```c
    for (int i = 0; i < game->platformCount; i++) {
```
**설명**: 모든 플랫폼을 순회하며 충돌을 검사합니다.

```c
        if (game->platforms[i].active) {
```
**설명**: 활성화된 플랫폼만 검사합니다.

```c
            int onPlatformX = (player->x >= game->platforms[i].x && 
                               player->x < game->platforms[i].x + game->platforms[i].width);
```
**설명**: 플레이어가 플랫폼의 X 범위 내에 있는지 확인합니다.

```c
            if (onPlatformX) {
```
**설명**: X 범위 내에 있으면 Y 충돌을 검사합니다.

```c
                if (player->y == game->platforms[i].y && player->velocityY >= 0) {
```
**설명**: 플레이어가 플랫폼 위에 정확히 있고 아래로 이동 중일 때.

```c
                    player->velocityY = 0;
                    player->onGround = 1;
```
**설명**: 낙하를 멈추고 지상 상태로 변경합니다.

```c
                else if (player->y > game->platforms[i].y && 
                         player->y <= game->platforms[i].y + 1 && 
                         player->velocityY > 0) {
```
**설명**: 플레이어가 플랫폼을 약간 통과했을 때 (보정 필요).

```c
                    player->y = game->platforms[i].y;
                    player->velocityY = 0;
                    player->onGround = 1;
                }
            }
        }
    }
```
**설명**: 플레이어를 플랫폼 위로 끌어올리고 지상 상태로 만듭니다.

---

### 낙사 처리

```c
    if (player->y >= HEIGHT - 1) {
```
**설명**: 플레이어가 화면 바닥으로 떨어졌을 때.

```c
        player->lives--;
```
**설명**: 생명을 1개 감소시킵니다.

```c
        if (player->lives > 0) {
```
**설명**: 생명이 남아있으면 리스폰합니다.

```c
            player->health = 100;
            player->x = 5;
            player->y = HEIGHT - 3;
            player->velocityY = 0;
            player->velocityX = 0;
```
**설명**: 체력을 회복하고 시작 위치로 돌아갑니다.

```c
        else {
            player->health = 0;
        }
    }
```
**설명**: 생명이 0개면 체력을 0으로 만들어 게임 오버를 유발합니다.

---

### 무적 시간 처리

```c
    if (player->invincibleTimer > 0) {
```
**설명**: 무적 타이머가 남아있으면.

```c
        player->invincibleTimer--;
```
**설명**: 타이머를 1 감소시킵니다.

```c
        if (player->invincibleTimer == 0) {
            player->invincible = 0;
        }
    }
```
**설명**: 타이머가 0이 되면 무적 상태를 해제합니다.

---

### 마찰력 적용

```c
    if (!player->invincible || player->invincibleTimer < 20) {
```
**설명**: 무적이 아니거나 무적 시간이 20프레임 미만일 때.

```c
        if (player->onGround && player->velocityX != 0) {
            player->velocityX = (int)(player->velocityX * 0.8);
```
**설명**: 지상에서 이동 중이면 속도를 0.8배로 감소 (강한 마찰).

```c
        } else if (!player->onGround) {
            player->velocityX = (int)(player->velocityX * 0.95);
        }
```
**설명**: 공중에서는 속도를 0.95배로 감소 (약한 공기 저항).

```c
    else {
        player->velocityX = (int)(player->velocityX * 0.8);
    }
}
```
**설명**: 무적 상태일 때는 0.8배로 감소합니다.

---

## 물리 엔진 요약

1. **중력**: 매 프레임 velocityY += 1
2. **위치 업데이트**: x += velocityX, y += velocityY
3. **경계 체크**: 벽에 닿으면 속도 0
4. **플랫폼 충돌**: Y 좌표를 플랫폼 위로 보정
5. **낙사**: 화면 밖으로 떨어지면 생명 감소
6. **마찰력**: 지상 0.8배, 공중 0.95배 감속
