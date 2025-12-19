# main.c 상세 설명

## 개요
메인 프로그램 파일로, 게임의 진입점과 메뉴 시스템, 게임 루프를 담당합니다.

---

## 코드 및 설명

### 헤더 및 전처리기

```c
#define _CRT_SECURE_NO_WARNINGS
```
**설명**: Visual Studio에서 strcpy, scanf 등의 보안 경고를 무시합니다.

```c
#include <stdio.h>
#include <stdlib.h>
#include <conio.h>
#include <windows.h>
#include <time.h>
#include <string.h>
#include "game.h"
#include "graphics.h"
#include "ranking.h"
#include "music.h"
```
**설명**: 필요한 라이브러리와 커스텀 헤더 파일을 포함합니다.

---

### showMenu 함수

```c
void showMenu(MinHeap* rankings) {
    int selected = 0;
```
**설명**: 메뉴 화면 표시 함수. selected는 현재 선택된 항목 (0~2).

```c
    while (1) {
        system("cls");
```
**설명**: 무한 루프로 메뉴를 계속 표시. 화면을 지웁니다.

```c
        setColor(14);
        printf("\n\n");
        printf("   ==========================================\n");
        printf("          PLATFORM JUMP GAME\n");
        printf("   ==========================================\n\n");
        setColor(15);
```
**설명**: 노란색(14)으로 타이틀을 출력하고 흰색(15)으로 변경합니다.

```c
        if (selected == 0) {
            setColor(11);
            printf("   >> 1. START GAME\n");
            setColor(15);
            printf("      2. VIEW RANKINGS\n");
            printf("      3. EXIT\n");
        }
```
**설명**: 첫 번째 메뉴 선택 시 하늘색(11)으로 강조 표시합니다.

```c
        char key = _getch();
        if (key == 'w' || key == 'W') {
            selected--;
            if (selected < 0) selected = 2;
        }
```
**설명**: W 키로 위로 이동. 0보다 작아지면 2로 순환합니다.

```c
        else if (key == 's' || key == 'S') {
            selected++;
            if (selected > 2) selected = 0;
        }
```
**설명**: S 키로 아래로 이동. 2보다 커지면 0으로 순환합니다.

```c
        else if (key == 13) {
            if (selected == 0) {
                return;
            }
```
**설명**: Enter 키(13) 입력 시 START GAME이면 함수 종료하고 게임 시작합니다.

```c
            else if (selected == 1) {
                system("cls");
                displayRankings(rankings, "", 0);
                printf("\n   Press any key to return to menu...\n");
                _getch();
            }
```
**설명**: VIEW RANKINGS 선택 시 랭킹 표시 후 키 입력 대기합니다.

```c
            else {
                exit(0);
            }
```
**설명**: EXIT 선택 시 프로그램을 종료합니다.

---

### run 함수

```c
void run() {
    GameState game;
    MinHeap rankings;
    char playerName[50];
    int playAgain = 1;
```
**설명**: 게임 메인 실행 함수. 필요한 변수들을 선언합니다.

```c
    srand(time(NULL));
    hideCursor();
```
**설명**: 난수 생성기 초기화 및 커서 숨기기.

```c
    loadRankings(&rankings);
```
**설명**: rankings.txt에서 기존 랭킹 데이터를 Min Heap에 로드합니다.

```c
    FILE* fp = fopen("mainstage.wav", "r");
    if (fp == NULL) {
        printf("\n [경고] mainstage.wav 파일을 찾을 수 없습니다! 폴더를 확인하세요.\n");
        _getch();
    } else {
        fclose(fp);
    }
```
**설명**: BGM 파일 존재 여부를 확인합니다. 없어도 게임은 진행됩니다.

```c
    while (playAgain) {
        system("cls");
        setColor(14);
        printf("\n\n");
        printf("   ==========================================\n");
        printf("          PLATFORM JUMP GAME\n");
        printf("   ==========================================\n\n");
        setColor(15);
        printf("   Enter your name: ");
        scanf("%s", playerName);
```
**설명**: 재시작 루프. 플레이어 이름을 입력받습니다.

```c
        showMenu(&rankings);
```
**설명**: 메뉴 화면을 표시합니다.

```c
        memset(prevBuffer, 0, sizeof(prevBuffer));
        memset(prevColorBuffer, 0, sizeof(prevColorBuffer));
```
**설명**: 더블 버퍼링용 이전 프레임 버퍼를 초기화합니다.

```c
        initGame(&game);
        strcpy(game.player.name, playerName);
```
**설명**: 게임 상태를 초기화하고 플레이어 이름을 복사합니다.

```c
        playBGM();
```
**설명**: 배경 음악을 재생합니다.

```c
        while (!game.gameOver) {
            handleInput(&game);
```
**설명**: 게임 루프 시작. 키보드 입력을 처리합니다.

```c
            if (game.stageCleared && _kbhit()) {
                char key = _getch();
                if (key == 'n' || key == 'N') {
                    nextStage(&game);
                }
            }
```
**설명**: 스테이지 클리어 시 N 키로 다음 스테이지 진행합니다.

```c
            updatePlayer(&game);
            updateEnemies(&game);
            updateCoins(&game);
            updateParticles(&game);
            checkGameOver(&game);
```
**설명**: 게임 로직을 업데이트합니다 (플레이어, 몬스터, 코인, 파티클, 게임오버 체크).

```c
            drawGame(&game);
            updateBGM();
```
**설명**: 화면을 그리고 BGM을 업데이트합니다 (무한 반복).

```c
            game.timer++;
            Sleep(50);
        }
```
**설명**: 타이머 증가 및 50ms 대기 (약 20 FPS).

```c
        stopBGM();
```
**설명**: 게임 오버 시 BGM을 정지합니다.

```c
        insertHeap(&rankings, game.player.name, game.player.score);
        saveRankings(&rankings);
```
**설명**: 플레이어 점수를 Min Heap에 삽입하고 파일에 저장합니다.

```c
        system("cls");
        setColor(12);
        printf("\n\n");
        printf("   ===================================\n");
        printf("          GAME OVER!\n");
        printf("   ===================================\n");
        setColor(15);
        printf("\n");
        printf("   Player: %s\n", game.player.name);
        printf("   Final Score: %d\n", game.player.score);
        printf("   Coins Collected: %d\n", game.player.coins);
        printf("   Stage Reached: %d\n", game.level);
        printf("\n");
        printf("   Press any key to see rankings...\n");
        _getch();
```
**설명**: 게임 오버 화면을 표시합니다 (빨간색 타이틀, 플레이어 정보).

```c
        displayRankings(&rankings, game.player.name, game.player.score);
```
**설명**: TOP 10 랭킹을 표시합니다. 현재 플레이어는 강조됩니다.

```c
        printf("\n   Play again? (Y/N): ");
        char choice = _getch();
        if (choice != 'y' && choice != 'Y') {
            playAgain = 0;
        }
    }
}
```
**설명**: 재시작 여부를 묻습니다. Y가 아니면 루프를 종료합니다.

---

### main 함수

```c
int main() {
    run();
    return 0;
}
```
**설명**: 프로그램의 진입점. run() 함수를 호출합니다.
