# music.h 및 music.c 상세 설명

## music.h - 헤더 파일

```c
#ifndef MUSIC_H
#define MUSIC_H
```
**설명**: 헤더 가드 시작.

```c
#include <windows.h>
#include <mmsystem.h>
#include <stdio.h>
```
**설명**: Windows 멀티미디어 API를 사용하기 위한 헤더를 포함합니다.

```c
#pragma comment(lib, "winmm.lib")
```
**설명**: winmm.lib 라이브러리를 링크합니다 (MCI 및 PlaySound 사용).

---

### SoundEffectType 열거형

```c
typedef enum {
    SFX_JUMP,
    SFX_HIT,
    SFX_DIE
} SoundEffectType;
```
**설명**: 효과음 타입을 정의합니다.
- SFX_JUMP: 점프 소리
- SFX_HIT: 피격 소리
- SFX_DIE: 죽음 소리

---

### 함수 선언

```c
void playBGM();
```
**설명**: 배경 음악을 재생합니다.

```c
void stopBGM();
```
**설명**: 배경 음악을 정지합니다.

```c
void updateBGM(void);
```
**설명**: BGM이 끝나면 자동으로 다시 재생합니다 (무한 반복).

```c
void playSFX(SoundEffectType type);
```
**설명**: 효과음을 재생합니다.

---

## music.c - 구현 파일

### 전역 변수 및 매크로

```c
#define BGM_ALIAS L"BGM_Music"
```
**설명**: MCI에서 사용할 BGM의 별칭(Alias)을 정의합니다.

```c
static int bgmInitialized = 0;
```
**설명**: BGM 초기화 여부를 저장하는 static 변수입니다.

---

### playBGM 함수

```c
void playBGM(void) {
    wchar_t cmd[512];
    wchar_t errMsg[256];
    MCIERROR err;
```
**설명**: MCI 명령어와 에러 메시지를 저장할 버퍼를 선언합니다.

```c
    mciSendStringW(L"close " BGM_ALIAS, NULL, 0, NULL);
```
**설명**: 이전에 열린 BGM이 있으면 닫습니다.

```c
    swprintf(cmd, 512,
        L"open \"%ls\" alias %ls",
        L"C:\\Users\\ladde\\source\\repos\\JaRuGuJo_finalProject\\JaRuGuJo_finalProject\\mainstage.wav",
        BGM_ALIAS
    );
```
**설명**: WAV 파일을 열고 별칭을 지정하는 MCI 명령어를 생성합니다.

```c
    err = mciSendStringW(cmd, NULL, 0, NULL);
    if (err) {
        mciGetErrorStringW(err, errMsg, 256);
        MessageBoxW(NULL, errMsg, L"MCI open error", MB_OK | MB_ICONERROR);
        return;
    }
```
**설명**: MCI 명령어를 실행하고, 에러가 발생하면 메시지 박스를 표시합니다.

```c
    swprintf(cmd, 512, L"play %ls", BGM_ALIAS);
    err = mciSendStringW(cmd, NULL, 0, NULL);
    if (err) {
        mciGetErrorStringW(err, errMsg, 256);
        MessageBoxW(NULL, errMsg, L"MCI play error", MB_OK | MB_ICONERROR);
        return;
    }
```
**설명**: BGM을 재생합니다. 에러 발생 시 메시지 박스를 표시합니다.

```c
    bgmInitialized = 1;
}
```
**설명**: BGM 초기화 플래그를 설정합니다.

**MCI (Media Control Interface)**:
- Windows에서 멀티미디어 장치를 제어하는 API
- 문자열 명령어로 오디오/비디오를 제어
- open, play, stop, close 등의 명령어 사용

---

### updateBGM 함수

```c
void updateBGM(void) {
    if (!bgmInitialized) return;
```
**설명**: BGM이 초기화되지 않았으면 종료합니다.

```c
    wchar_t cmd[128];
    wchar_t mode[64] = {0};

    swprintf(cmd, 128, L"status %ls mode", BGM_ALIAS);
    if (mciSendStringW(cmd, mode, 64, NULL) != 0) return;
```
**설명**: BGM의 현재 상태를 조회합니다.

```c
    if (wcscmp(mode, L"stopped") == 0) {
        swprintf(cmd, 128, L"play %ls", BGM_ALIAS);
        mciSendStringW(cmd, NULL, 0, NULL);
    }
}
```
**설명**: BGM이 정지 상태이면 다시 재생합니다 (무한 반복 구현).

**무한 반복 원리**:
1. 게임 루프에서 매 프레임마다 updateBGM() 호출
2. BGM이 끝나면 상태가 "stopped"로 변경
3. "stopped" 감지 시 자동으로 다시 재생

---

### stopBGM 함수

```c
void stopBGM() {
    wchar_t command[256];
    
    swprintf(command, 256, L"stop %ls", BGM_ALIAS);
    mciSendStringW(command, NULL, 0, NULL);
```
**설명**: BGM을 정지합니다.

```c
    swprintf(command, 256, L"close %ls", BGM_ALIAS);
    mciSendStringW(command, NULL, 0, NULL);
```
**설명**: BGM을 닫습니다.

```c
    bgmInitialized = 0;
}
```
**설명**: 초기화 플래그를 해제합니다.

---

### playSFX 함수

```c
void playSFX(SoundEffectType type) {
    const char* soundPath;

    switch (type) {
    case SFX_JUMP:
        soundPath = "jump.wav";
        break;
    case SFX_HIT:
        soundPath = "hitsound.wav";
        break;
    case SFX_DIE:
        soundPath = "deadsound.wav";
        break;
    default:
        return;
    }
```
**설명**: 효과음 타입에 따라 파일 경로를 선택합니다.

```c
    wchar_t filename[100];
    swprintf(filename, 100, L"%hs", soundPath);
```
**설명**: ANSI 문자열을 유니코드로 변환합니다.

```c
    PlaySoundW(filename, NULL, SND_FILENAME | SND_ASYNC | SND_NODEFAULT);
}
```
**설명**: PlaySound API로 효과음을 재생합니다.

**PlaySound 플래그**:
- SND_FILENAME: 파일 이름으로 재생
- SND_ASYNC: 비동기 재생 (게임이 멈추지 않음)
- SND_NODEFAULT: 파일이 없어도 기본 소리를 재생하지 않음

---

## 사운드 시스템 요약

### BGM (배경 음악)
- **API**: MCI (Media Control Interface)
- **파일**: mainstage.wav
- **특징**: 무한 반복 재생
- **제어**: playBGM(), stopBGM(), updateBGM()

### 효과음 (SFX)
- **API**: PlaySound
- **파일**: jump.wav, hitsound.wav, deadsound.wav
- **특징**: 비동기 재생 (게임 진행 방해 없음)
- **제어**: playSFX(SoundEffectType)

### 사용 예시
```c
// 게임 시작 시
playBGM();

// 게임 루프에서
updateBGM();  // 매 프레임마다 호출

// 점프 시
playSFX(SFX_JUMP);

// 피격 시
playSFX(SFX_HIT);

// 게임 오버 시
playSFX(SFX_DIE);
stopBGM();
```

### 주의사항
1. WAV 파일은 실행 파일과 같은 폴더에 있어야 합니다.
2. BGM 파일 경로는 절대 경로로 지정되어 있습니다.
3. 파일이 없어도 게임은 계속 진행됩니다 (경고만 표시).
