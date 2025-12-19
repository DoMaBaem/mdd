# ranking.h 및 ranking.c 상세 설명

## ranking.h - 헤더 파일

```c
#ifndef RANKING_H
#define RANKING_H
```
**설명**: 헤더 가드 시작.

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <windows.h>
```
**설명**: 필요한 라이브러리를 포함합니다.

---

### 상수 정의

```c
#define TOP_K 10
```
**설명**: 랭킹 TOP 10만 유지합니다.

```c
#define RANKING_FILE "rankings.txt"
```
**설명**: 랭킹 데이터를 저장할 파일 이름입니다.

---

### RankingEntry 구조체

```c
typedef struct {
    char name[50];
    int score;
} RankingEntry;
```
**설명**: 랭킹 엔트리 구조체. 이름과 점수를 저장합니다.

---

### MinHeap 구조체

```c
typedef struct {
    RankingEntry data[TOP_K];
    int size;
} MinHeap;
```
**설명**: Min Heap 구조체. 최대 10개의 엔트리를 저장합니다.
- data: 랭킹 데이터 배열
- size: 현재 저장된 엔트리 개수

**Min Heap을 사용하는 이유**:
- 루트 노드가 최솟값 (10등)
- 새 점수가 10등보다 높으면 루트를 교체
- O(log 10) = O(1) 시간에 삽입 가능
- TOP 10만 유지하여 메모리 효율적

---

### 함수 선언

```c
void swap(RankingEntry* a, RankingEntry* b);
```
**설명**: 두 엔트리를 교환합니다.

```c
void heapifyUp(MinHeap* heap, int idx);
```
**설명**: 힙 속성을 유지하기 위해 위로 올립니다.

```c
void heapifyDown(MinHeap* heap, int idx);
```
**설명**: 힙 속성을 유지하기 위해 아래로 내립니다.

```c
void insertHeap(MinHeap* heap, char* name, int score);
```
**설명**: 새 점수를 힙에 삽입합니다.

```c
RankingEntry extractMin(MinHeap* heap);
```
**설명**: 최솟값(루트)을 추출합니다.

```c
void loadRankings(MinHeap* heap);
```
**설명**: 파일에서 랭킹을 로드합니다.

```c
void saveRankings(MinHeap* heap);
```
**설명**: 랭킹을 파일에 저장합니다.

```c
void displayRankings(MinHeap* heap, char* currentPlayer, int currentScore);
```
**설명**: TOP 10 랭킹을 화면에 표시합니다.

---

## ranking.c - 구현 파일

### swap 함수

```c
void swap(RankingEntry* a, RankingEntry* b) {
    RankingEntry temp = *a;
    *a = *b;
    *b = temp;
}
```
**설명**: 두 엔트리의 값을 교환합니다.

---

### heapifyUp 함수

```c
void heapifyUp(MinHeap* heap, int idx) {
    if (idx == 0) return;
```
**설명**: 루트에 도달하면 종료합니다.

```c
    int parent = (idx - 1) / 2;
```
**설명**: 부모 노드의 인덱스를 계산합니다.

```c
    if (heap->data[idx].score < heap->data[parent].score) {
        swap(&heap->data[idx], &heap->data[parent]);
        heapifyUp(heap, parent);
    }
}
```
**설명**: 자식이 부모보다 작으면 교환하고 재귀적으로 위로 올립니다.

**시간 복잡도**: O(log N) - 트리의 높이만큼 반복

---

### heapifyDown 함수

```c
void heapifyDown(MinHeap* heap, int idx) {
    int smallest = idx;
    int left = 2 * idx + 1;
    int right = 2 * idx + 2;
```
**설명**: 현재 노드와 자식 노드의 인덱스를 계산합니다.

```c
    if (left < heap->size && heap->data[left].score < heap->data[smallest].score)
        smallest = left;
    if (right < heap->size && heap->data[right].score < heap->data[smallest].score)
        smallest = right;
```
**설명**: 자식 중 가장 작은 값을 찾습니다.

```c
    if (smallest != idx) {
        swap(&heap->data[idx], &heap->data[smallest]);
        heapifyDown(heap, smallest);
    }
}
```
**설명**: 자식이 더 작으면 교환하고 재귀적으로 아래로 내립니다.

**시간 복잡도**: O(log N)

---

### insertHeap 함수 (핵심 알고리즘)

```c
void insertHeap(MinHeap* heap, char* name, int score) {
```
**설명**: 새 점수를 힙에 삽입하는 함수입니다.

```c
    if (heap->size < TOP_K) {
```
**설명**: 힙이 가득 차지 않았으면 (10개 미만).

```c
        strcpy(heap->data[heap->size].name, name);
        heap->data[heap->size].score = score;
        heapifyUp(heap, heap->size);
        heap->size++;
```
**설명**: 마지막에 추가하고 heapifyUp으로 힙 속성을 유지합니다.

```c
    } else if (score > heap->data[0].score) {
```
**설명**: 힙이 가득 찼고, 새 점수가 10등(루트)보다 높으면.

```c
        strcpy(heap->data[0].name, name);
        heap->data[0].score = score;
        heapifyDown(heap, 0);
    }
}
```
**설명**: 루트를 새 점수로 교체하고 heapifyDown으로 힙 속성을 유지합니다.

**시간 복잡도**: O(log 10) = O(1)

**알고리즘 설명**:
1. 10개 미만: 끝에 추가 후 heapifyUp
2. 10개 이상 + 새 점수 > 10등: 루트 교체 후 heapifyDown
3. 10개 이상 + 새 점수 ≤ 10등: 무시 (TOP 10에 들지 못함)

---

### extractMin 함수

```c
RankingEntry extractMin(MinHeap* heap) {
    RankingEntry min = heap->data[0];
```
**설명**: 루트(최솟값)를 저장합니다.

```c
    heap->data[0] = heap->data[heap->size - 1];
    heap->size--;
    heapifyDown(heap, 0);
    return min;
}
```
**설명**: 마지막 노드를 루트로 이동하고, heapifyDown으로 힙 속성을 유지한 후 최솟값을 반환합니다.

**시간 복잡도**: O(log N)

---

### loadRankings 함수

```c
void loadRankings(MinHeap* heap) {
    heap->size = 0;
    FILE* file = fopen(RANKING_FILE, "r");
    if (file == NULL) return;
```
**설명**: 힙을 초기화하고 파일을 엽니다. 파일이 없으면 종료합니다.

```c
    char name[50];
    int score;
    while (fscanf(file, "%s %d", name, &score) == 2) {
        insertHeap(heap, name, score);
    }
    fclose(file);
}
```
**설명**: 파일에서 이름과 점수를 읽어 힙에 삽입합니다.

**시간 복잡도**: O(N log 10) - N개의 데이터를 읽어 각각 O(log 10)에 삽입

---

### saveRankings 함수

```c
void saveRankings(MinHeap* heap) {
    FILE* file = fopen(RANKING_FILE, "w");
    if (file == NULL) return;

    for (int i = 0; i < heap->size; i++) {
        fprintf(file, "%s %d\n", heap->data[i].name, heap->data[i].score);
    }
    fclose(file);
}
```
**설명**: 힙의 모든 데이터를 파일에 저장합니다.

**시간 복잡도**: O(10) = O(1)

---

### displayRankings 함수

```c
void displayRankings(MinHeap* heap, char* currentPlayer, int currentScore) {
    MinHeap tempHeap;
    tempHeap.size = heap->size;
    for (int i = 0; i < heap->size; i++) {
        tempHeap.data[i] = heap->data[i];
    }
```
**설명**: 원본 힙을 보존하기 위해 임시 힙에 복사합니다.

```c
    RankingEntry top10[TOP_K];
    int count = tempHeap.size;
    
    for (int i = 0; i < count; i++) {
        top10[i] = extractMin(&tempHeap);
    }
```
**설명**: extractMin을 반복하여 오름차순으로 정렬된 배열을 만듭니다.

**시간 복잡도**: O(10 log 10) = O(1)

```c
    system("cls");
    SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), 14);
    printf("\n\n");
    printf("   ==========================================\n");
    printf("            TOP 10 RANKINGS\n");
    printf("   ==========================================\n\n");
```
**설명**: 화면을 지우고 타이틀을 노란색으로 출력합니다.

```c
    for (int i = 0; i < 10; i++) {
        int idx = count - 1 - i;
```
**설명**: 내림차순으로 출력하기 위해 역순으로 인덱스를 계산합니다.

```c
        if (idx >= 0) {
            if (strcmp(top10[idx].name, currentPlayer) == 0 && 
                top10[idx].score == currentScore) {
                SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), 11);
                printf("   >> %d. %-20s %d pts <<\n", i + 1, top10[idx].name, top10[idx].score);
                SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), 14);
            }
```
**설명**: 현재 플레이어의 기록은 하늘색(11)으로 강조 표시합니다.

```c
            else {
                SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), 15);
                printf("      %d. %-20s %d pts\n", i + 1, top10[idx].name, top10[idx].score);
            }
        }
```
**설명**: 다른 기록은 흰색(15)으로 출력합니다.

```c
        else {
            SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), 8);
            printf("      %d. ---\n", i + 1);
        }
    }

    SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), 15);
    printf("\n   ==========================================\n");
}
```
**설명**: 빈 순위는 어두운 회색(8)으로 "---"를 출력합니다.

---

## Min Heap 알고리즘 요약

### 시간 복잡도
- **삽입**: O(log 10) = O(1)
- **추출**: O(log 10) = O(1)
- **로드**: O(N log 10) - N개 데이터
- **저장**: O(10) = O(1)
- **표시**: O(10 log 10) = O(1)

### 공간 복잡도
- **O(10)** - 항상 TOP 10만 유지

### 장점
1. TOP K 유지에 최적화
2. 메모리 효율적 (고정 크기)
3. 빠른 삽입/조회
4. 파일 기반 영속성
