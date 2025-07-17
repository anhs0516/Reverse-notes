## 🧪 Day 2 - 실습: .rdata 섹션 내 IMAGE_IMPORT_DESCRIPTOR 분석

## 🔍 1. 배경 정보

| 항목                                    | 값            |
| ------------------------------------- | ------------ |
| ImageBase                             | `0x00400000` |
| `DataDirectory[1]` (Import Table RVA) | `0x00005444` |
| `.rdata` 섹션 RVA                       | `0x00005000` |
| `.rdata` PointerToRawData             | `0x00005000` |
| `.rdata` SizeOfRawData                | `0x00001000` |


📌 Import Table의 RVA 0x00005444는 .rdata 섹션의 RVA 범위 0x00005000 ~ 0x00005FFF 안에 위치합니다.

## 🧮 2. RVA → File Offset 변환
계산식
FileOffset = RVA - Section.VirtualAddress + Section.PointerToRawData
           = 0x5444 - 0x5000 + 0x5000
           = 0x5444

📌 즉, 이 RVA는 .rdata 섹션의 시작과 정렬되어 있어 File Offset도 0x5444입니다.

## 📂 3. HxD에서 0x5444의 Raw Data
HxD에서 0x5444 위치에 존재하는 데이터는 다음과 같음:

```
1C 55 00 00 00 00 00 00 00 00 00 00 6E 55 00 00 9C 50 00 00
```

이를 구조체 단위로 해석하면 다음과 같음.

## 📑 4. IMAGE_IMPORT_DESCRIPTOR 구조체 해석

```c
typedef struct _IMAGE_IMPORT_DESCRIPTOR {
    union {
        DWORD Characteristics;      // 0x00 (OriginalFirstThunk)
        DWORD OriginalFirstThunk;
    };
    DWORD TimeDateStamp;           // 0x04
    DWORD ForwarderChain;         // 0x08
    DWORD Name;                   // 0x0C (DLL 이름 RVA)
    DWORD FirstThunk;             // 0x10 (IAT 시작 RVA)
};
```

🧷 해석 결과

| 필드명                  | 값 (Little Endian) | 해석                             |
| -------------------- | ----------------- | ------------------------------ |
| `OriginalFirstThunk` | `0x0000551C`      | Hint/Name Table RVA            |
| `TimeDateStamp`      | `0x00000000`      | (보통 0)                         |
| `ForwarderChain`     | `0x00000000`      | 없음                             |
| `Name`               | `0x0000556E`      | DLL 이름 문자열의 RVA                |
| `FirstThunk`         | `0x0000509C`      | Import Address Table (IAT) RVA |


## 🔍 5. DLL 이름 추출

Name RVA = 0x0000556E

.rdata RVA = 0x00005000

```
Offset = 0x556E - 0x5000 = 0x056E
File Offset = 0x5000 + 0x056E = 0x556E
```

HxD에서 0x556E 위치를 문자열로 보면:

```
KERNEL32.dll
```

## 🔍 6. IMAGE_IMPORT_BY_NAME 추출

OriginalFirstThunk RVA = 0x0000551C

구조체 안에는 Hint와 함수명이 포함됨

```text
IMAGE_IMPORT_BY_NAME {
  WORD Hint;
  CHAR Name[]; // Null-terminated
}


예시 (HxD에서 0x551C 위치):

```
00 00 4C 6F 61 64 4C 69 62 72 61 72 79 41 00
```

→ Hint = 0x0000
→ Name = "LoadLibraryA"

## ✅ 실습 요약

.rdata 영역의 IMAGE_IMPORT_DESCRIPTOR 구조를 직접 분석하였다.

RVA → FileOffset 계산을 통해 정확히 구조체가 어디에 있는지 판단함.

구조체 안에 있는 Name, FirstThunk, OriginalFirstThunk 필드를 해석함.

IMAGE_IMPORT_BY_NAME 구조체를 따라가 DLL 함수 이름까지 추출 성공함 (LoadLibraryA).

IMAGE_IMPORT_DESCRIPTOR 구조체는 끝에 NULL 구조체 (0x00000000 * 5)로 종료됨.

