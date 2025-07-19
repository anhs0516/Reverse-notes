
# Week01 - Day2 IAT(Import Address Table) 구조 및 실습 정리


## 📌 1. 시작 배경

- PE 구조 중 `IMAGE_OPTIONAL_HEADER`의 `DataDirectory[1]`는 Import Table의 시작 위치(RVA)와 크기를 제공한다.
- 해당 RVA를 따라가면 등장하는 구조가 `IMAGE_IMPORT_DESCRIPTOR`.
- IAT 분석을 위해 Import Table 흐름 및 구조를 정확히 이해해야 한다.

---

## 📎 2. 기본 용어 정리

| 용어 | 설명 |
|------|------|
| Import Table | 외부 DLL 함수 정보를 담고 있는 전체 영역 |
| IMAGE_IMPORT_DESCRIPTOR | Import Table을 구성하는 구조체 (DLL 단위) |
| ILT (Import Lookup Table) | Hint/Name Table. 함수 이름 주소를 가리킴 |
| IAT (Import Address Table) | DLL 함수의 실제 주소가 채워지는 테이블 |
| RVA | Relative Virtual Address. 기준은 ImageBase |
| VA | Virtual Address = ImageBase + RVA |

---

## 🔍 3. 분석 흐름 (예시 기반)

1. **IMAGE_OPTIONAL_HEADER.DataDirectory[1]**
   - RVA = `0x00005444`
   - Size = `0x3C`

2. **RVA `0x5444` 위치 확인**
   - 이 주소에는 `IMAGE_IMPORT_DESCRIPTOR` 구조체 배열이 존재한다.

3. `IMAGE_IMPORT_DESCRIPTOR` 구조체:

```c
typedef struct _IMAGE_IMPORT_DESCRIPTOR {
  DWORD OriginalFirstThunk; // ILT (Hint/Name)
  DWORD TimeDateStamp;
  DWORD ForwarderChain;
  DWORD Name;               // DLL 이름 (RVA)
  DWORD FirstThunk;         // IAT 시작 주소 (RVA)
};
```

## 🧩 4. `.rdata` 섹션과 Import Table

- `IMAGE_IMPORT_DESCRIPTOR` 배열은 보통 `.rdata` 또는 `.idata` 섹션에 위치한다.
- `.rdata` 안에 있는 경우, 구조체와 문자열(DLL 이름, 함수 이름)이 함께 존재한다.
- 각 DLL마다 `IMAGE_IMPORT_DESCRIPTOR` 구조체가 1개씩 있으며,
  마지막은 모든 필드가 0으로 된 NULL 구조체이다 (Import Table의 끝).


## 📌 5. 구조 흐름 요약

```text
PE Header
 └─ Optional Header
      └─ DataDirectory[1] → RVA: 0x5444

[0x5444] → IMAGE_IMPORT_DESCRIPTOR 배열
 ├─ Name → "KERNEL32.dll" 문자열 위치 (RVA)
 ├─ OriginalFirstThunk → ILT (함수 이름 목록 Hint/Name Table)
 └─ FirstThunk → IAT (실제 함수 주소를 저장할 공간)

IAT는 실행 중 DLL 로딩 후 실제 API 주소가 이 테이블에 기록됨
```

## 6. 혼동 방지 요점 정리

| 개념 |	설명 |
|------|------|
|IMAGE_IMPORT_DESCRIPTOR|	Import Table 항목 구조체 (DLL 단위)
|ILT (OriginalFirstThunk)|	Hint/Name Table. 함수 이름 정보를 가리킴
|IAT (FirstThunk)|	DLL 로딩 후 실제 API 주소를 저장하는 공간
|IAT 위치|	.idata 또는 .rdata 섹션 안에 있음
|IAT 역할|	디버깅, API 호출 추적 시 필수 위치


## 마무리 요약
Import Table은 DLL 함수 호출 정보의 핵심이다.

IMAGE_OPTIONAL_HEADER.DataDirectory[1]이 Import Table의 RVA를 제공한다.

해당 RVA 위치에는 IMAGE_IMPORT_DESCRIPTOR 배열이 존재하며,
각 항목은 하나의 DLL을 나타낸다.

FirstThunk는 IAT의 시작 주소이며, 실행 시 여기에 실제 API 주소가 채워진다.

분석 시 IAT는 브레이크포인트 설정, 흐름 추적의 핵심 위치이다.

PEView, PE-Bear 등의 툴로 .rdata 내 구조를 시각적으로 확인 가능하다.


