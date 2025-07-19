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
<img width="2568" height="1408" alt="image" src="https://github.com/user-attachments/assets/8d244e10-1080-4648-8ed6-21cc0a2adea8" />
```
USER32.dll
```

## 🔍 6. IMAGE_IMPORT_BY_NAME 추출

OriginalFirstThunk RVA = 0x0000551C

구조체 안에는 Hint와 함수명이 포함됨

```text
typedef struct _IMAGE_IMPORT_BY_NAME { 
   WORD Hint;     // ordinal 고유번호
   BYTE Name[l]; // function name string  / 가변 길이 문자열   함수이름 길이에 따라 다름
} lMAGE_IMPORT_BY_NAME, *PlMAGE_IMPORT_BY_NAME;
```


```

5C 55 00 00  
4E 55 00 00  
42 55 00 00  
30 55 00 00  
00 00 00 00

```

<img width="649" height="380" alt="image" src="https://github.com/user-attachments/assets/c56a4ce8-4990-45b9-b5e4-883098927447" />

## 📍 해석 과정
IMAGE_IMPORT_DESCRIPTOR.OriginalFirstThunk = 0x0000551C
이 RVA 주소부터 시작해서 4바이트씩 끊어보면 다음과 같은 항목들이 나옵니다.

| Offset (RVA 기준) | 값            | 의미                                          |
| --------------- | ------------ | ------------------------------------------- |
| 0x551C          | `0x0000555C` | IMAGE\_IMPORT\_BY\_NAME 구조체가 RVA 0x555C에 있음 |
| 0x5520          | `0x0000554E` | IMAGE\_IMPORT\_BY\_NAME 구조체가 RVA 0x554E에 있음 |
| 0x5524          | `0x00005542` | IMAGE\_IMPORT\_BY\_NAME 구조체가 RVA 0x5542에 있음 |
| 0x5528          | `0x00005530` | IMAGE\_IMPORT\_BY\_NAME 구조체가 RVA 0x5530에 있음 |
| 0x552C          | `0x00000000` | 끝을 나타냄 (null terminated)                    |


이 테이블은 IAT로 옮겨지기 전 "이름으로 import되는 함수들의 배열"입니다.

즉, 0x555C 주소에는 아래와 같이 Hint + 함수명 문자열이 들어 있습니다.

## 🔍 현재 구조 분석 (파일 오프셋 0x555C 기준)

0x555C: 13 01 47 65 74 44 6C 67 49 74 65 6D 54 65 78 74 41 00


| 오프셋      | 바이트         | 해석                                               |
| -------- | ----------- | ------------------------------------------------ |
| `0x555C` | `13 01`     | Hint 값: `0x0113` (little endian이므로 0x0113 = 275) |
| `0x555E` | ASCII 문자 시작 | `"GetDlgItemTextA"`                              |
| `0x556F` | `00`        | 문자열 종료 (NULL terminator)                         |


| 필드     | 값                   | 의미                                       |
| ------ | ------------------- | ---------------------------------------- |
| `Hint` | `0x0113`            | Export Table에서의 힌트 인덱스 (필수는 아님, 성능 최적화용) |
| `Name` | `"GetDlgItemTextA"` | Import할 함수 이름 (ASCII 문자열)                |


전체 구조체 크기:

2 bytes (Hint)

16 bytes ("GetDlgItemTextA")

1 byte (NULL 종료)
➡ 총 19 bytes




→ Hint = 0x0113
→ Name = "GetDlgItemTextA"

## ✅ 실습 요약

.rdata 영역의 IMAGE_IMPORT_DESCRIPTOR 구조를 직접 분석하였다.

RVA → FileOffset 계산을 통해 정확히 구조체가 어디에 있는지 판단함.

구조체 안에 있는 Name, FirstThunk, OriginalFirstThunk 필드를 해석함.

IMAGE_IMPORT_BY_NAME 구조체를 따라가 DLL 함수 이름까지 추출 성공함 (GetDlgItemTextA).

IMAGE_IMPORT_DESCRIPTOR 구조체는 끝에 NULL 구조체 (0x00000000 * 5)로 종료됨.

