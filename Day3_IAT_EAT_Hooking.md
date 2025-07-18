## Day03 - 📦 IAT, EAT, 그리고 API Hooking 기초


## 📘 IAT / EAT 개념 이해

| 항목 | 설명 |
|------|------|
| **IAT (Import Address Table)** | 외부 DLL의 함수 주소를 실행 시간에 불러오기 위해 사용하는 테이블 즉, 프로그램에 어떤 Library에서 어떤 함수를 사용하고 있는지 기술한 테이블|
| **EAT (Export Address Table)** | DLL이 외부에 제공하는 함수 목록 및 주소를 담고 있는 테이블 즉, 라이브러리 파일에서 제공하는 함수(기능)를 다른 프로그램에서 가져다가 사용할 수 있도록 해주는 핵심 메커니즘이다. 그래서 EAT를 통해서만 해당 라이브러리에서 내보내는 함수의 시작 주소를 정확히 알 수 있다.|


### 🔶 Import Table의 구조
- PE 파일의 **Optional Header → DataDirectory[1]** 에 Import Table의 RVA가 있음
- 이 RVA를 따라가면 `IMAGE_IMPORT_DESCRIPTOR` 구조체 배열이 있음
- 각 DLL에 대해 `Name`, `OriginalFirstThunk`, `FirstThunk` 등이 정의됨
- 그 후 실제 API 함수명과 Hint는 `IMAGE_IMPORT_BY_NAME` 구조체에 저장

---

## ⚙️ IAT의 동작 방식

1. 실행 시, PE Loader가 IAT를 따라가면서 필요한 DLL 로드
2. DLL의 `Export Address Table (EAT)`에서 실제 함수 주소 검색
3. 그 주소를 IAT의 `FirstThunk`에 채워 넣음
4. 이후 코드에서 호출할 때는 `call dword ptr [IAT의 주소]` 형식으로 간접 호출

### 🧠 IAT 관련 구조체들

```c
// IMAGE_IMPORT_DESCRIPTOR
typedef struct _IMAGE_IMPORT_DESCRIPTOR {
    DWORD OriginalFirstThunk; // Hint/Name Table (IMAGE_THUNK_DATA)
    DWORD TimeDateStamp;
    DWORD ForwarderChain;
    DWORD Name;               // DLL 이름 (RVA)
    DWORD FirstThunk;         // 실제 호출되는 함수 주소들이 저장됨
} IMAGE_IMPORT_DESCRIPTOR;

// IMAGE_IMPORT_BY_NAME
typedef struct _IMAGE_IMPORT_BY_NAME {
    WORD Hint;
    BYTE Name[1]; // NULL 종료 문자열
} IMAGE_IMPORT_BY_NAME;
```

## ⚙️ EAT의 동작 방식
1. DLL 내부에서 Export Table을 통해 외부에 제공할 함수 목록 관리
2. PE 구조상 DataDirectory[0]에 Export Table의 RVA 존재
3. EAT를 통해 외부에서 GetProcAddress() 호출 가능

```c
// IMAGE_EXPORT_DIRECTORY 구조체
typedef struct _IMAGE_EXPORT_DIRECTORY {
    DWORD Characteristics;
    DWORD TimeDateStamp;
    WORD  MajorVersion;
    WORD  MinorVersion;
    DWORD Name;                // DLL 이름
    DWORD Base;                // 함수 번호 시작값
    DWORD NumberOfFunctions;  // 실제 Export 함수의 개수
    DWORD NumberOfNames;       // Export 함수 중에서 이름을 가지는 함수
    DWORD AddressOfFunctions; // RVA 배열  , Export 함수 주소 배열(배열의 원소 개수 = NumberOfFunctions(함수개수))
    DWORD AddressOfNames;     // 함수 이름 주소 배열 (배열의 원소 개수 = NumberOfNames)
    DWORD AddressOfNameOrdinals; // Ordinal(고유번호) 배열(배열의 원소 개수 = NumberOfNames)
} IMAGE_EXPORT_DIRECTORY;
```

## 🧩 핵심 개념 요약

- **IAT는 실행 시 동적으로 채워진다**
- **EAT는 DLL이 메모리에 로드될 때 함께 올라간다**
- EXE의 IAT를 채우기 위해 OS는 DLL의 EAT를 참조한다

---

## 🔄 실행 중 IAT & EAT 동작 흐름

### 예시: EXE가 `MessageBoxA` (user32.dll) 함수 사용

### ⚙️ 전체 과정

```text
1️⃣ EXE 실행 → OS는 PE 헤더의 IMAGE_IMPORT_DESCRIPTOR 확인
    └─ 어떤 DLL이 필요한지 확인 (예: user32.dll)

2️⃣ 필요한 DLL 로드 (예: user32.dll)
    └─ user32.dll의 Export Address Table(EAT)도 함께 메모리에 로드

3️⃣ EXE의 OriginalFirstThunk → ILT (Import Lookup Table) 따라가기
    └─ IMAGE_IMPORT_BY_NAME 구조체에서 "MessageBoxA" 문자열 획득

4️⃣ OS는 user32.dll의 EAT 탐색
    └─ "MessageBoxA"의 실제 주소 확인 (예: 0x77B2A430)

5️⃣ 확인한 주소를 EXE의 FirstThunk (IAT)에 기록


[EXE 파일]                                [user32.dll 메모리 내]
────────────────────────────────────     ──────────────────────────────
 OriginalFirstThunk  ─────┐               "MessageBoxA" ← Import 함수명
                          │               
                          ▼               
                 IMAGE_IMPORT_BY_NAME     
                          │
                          ▼
                      [EAT 탐색]
                          │
                          ▼
                   0x77B2A430 ← MessageBoxA 실제 주소
                          │
                          ▼
                →→ FirstThunk (IAT)에 기록

```

## 📌 EAT vs IAT
✅ 장점:

IAT: 외부 DLL 사용자가 import할 때
EAT: DLL이 외부에 export할 때

📘 보완 정리:


| 항목     | EAT                                  | IAT                                    |
|----------|---------------------------------------|-----------------------------------------|
| 목적     | DLL이 다른 모듈에 제공하는 함수 목록   | 실행 파일이 외부 DLL에서 호출하는 함수 |
| 위치     | Export Directory                      | Import Directory                        |
| 사용 시기| API 제공자(DLL 측)                    | API 사용자(EXE 측)                     |
| 수정 여부| 불변 (DLL 내부 고정)                  | 실행 중 OS에 의해 API 주소로 덮어쓰기 |


## 🧩 API Hooking 기초

| 항목          | 설명                                       |
| ----------- | ---------------------------------------- |
| API Hooking | IAT나 EAT를 조작해, 특정 API 함수의 실행을 가로채는 기술    |
| 사용 목적       | 악성코드 탐지 우회, 함수 동작 분석, 디버깅, DLL Injection |

### 🔧 방법 1: IAT Hooking
- IAT에 있는 함수 포인터를 후킹 대상 함수 주소로 교체
  
FirstThunk의 항목을 조작
원래 주소 대신 내가 만든 함수 주소로 변경
모든 call [IAT] 호출이 내가 만든 함수로 연결됨

```
DWORD* IATEntry = (DWORD*)(ModuleBase + RVA_of_FirstThunk);
*IATEntry = (DWORD)MyHookFunction;
```


### 🔧 방법 2: Inline Hooking
- 원래 함수의 시작 주소에 JMP 명령어로 후킹 대상 함수로 점프


API 함수 진입 지점에 JMP 명령 삽입
원래 명령은 다른 위치에 백업
실행 흐름을 내가 만든 함수로 유도

```
Original: 55 8B EC    ; push ebp / mov ebp, esp
Hooked:   E9 XX XX XX XX ; JMP my_hook_func
```


### 3. EAT Hooking (비교적 드묾)
   - DLL이 Export하는 함수 주소를 변경
