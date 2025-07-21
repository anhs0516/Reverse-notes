# 📦 실행 압축 (Executable Compression)

---

## 📌 데이터 압축이란?

컴퓨터 공학의 핵심 기술 중 하나로, 모든 파일은 내부적으로 **바이너리(기계어)** 형태로 구성되어 있으며,  
**압축 알고리즘**을 통해 **파일 크기를 줄여 보관 및 전송을 용이하게** 만든다.

---

## 📂 압축의 종류

### 1. 비손실 압축 (Lossless Data Compression)
- 데이터의 손실 없이 완벽히 복원 가능한 압축 방식  
- 예: ZIP, PNG, FLAC

### 2. 손실 압축 (Lossy Data Compression)
- 일부 데이터 손실을 감수하여 고효율 압축  
- 예: MP3, JPG, MPEG

### 3. 실행 압축 (Executable Compression)
- 실행 가능한 파일(EXE, DLL 등)을 압축하여 크기를 줄이고, 복호화된 상태로 메모리에서 실행되는 방식

---

## 🧰 패커 & 프로텍터 (실행압축)

### 🎯 패커 (Packer)
- **PE 파일(Portable Executable)** 전용 압축기
- 정확히는 **Run-time Packer**로, 실행 시 압축 해제됨
- **대표 예시**: `UPX`, `ASPACK`

#### ✅ 장점
- 실행 파일의 크기를 줄임
- 리버싱을 어렵게 만듦 (간단한 보호 수단)

#### ❌ 단점
- 메모리에서 **디코딩 루프**가 돌며 압축이 해제됨
- 메모리 덤프 분석에 취약

---

### 🔐 프로텍터 (Protector)
- 패커 + **Anti-Reversing 기법** → 리버싱 방지 강화
- 보호 기능이 강력한 대신 **파일 크기가 오히려 커질 수 있음**

#### ✅ 대표 예시
- `Themida`, `VMProtect (VMP)`

---

## 🔍 OEP (Original Entry Point) 찾기

패킹된 파일의 진짜 진입점(OEP)을 찾기 위해 사용하는 기술 중 하나

### Pushad / Popad 사용
- `pushad`: 현재 `EAX ~ EDI` (ESP, EBP 포함)의 값을 스택에 저장
- `popad`: 저장된 값을 스택에서 복원

→ 패커가 원래 코드로 돌아오기 전, `popad`를 통해 레지스터 복원 → **OEP 직전 위치**

---

## 🔄 PE Relocation (재배치)

PE 파일이 메모리에 로드될 때, PE 헤더에 정의된 **ImageBase** 위치가 이미 사용 중이면 **다른 주소로 로딩됨**  
→ 이 때 사용되는 기술이 **Relocation (재배치)**

### 📍 특징
- DLL은 매번 주소가 달라질 수 있으나, `ImageBase + RVA = VA` 공식은 항상 유효
- **EXE 파일은 ASLR 이전엔 항상 고정 주소에 로드**

### 🛡 ASLR (Address Space Layout Randomization)
- 메모리 주소를 **무작위로 할당**하여 취약점 공격을 어렵게 만듦
- 실행할 때마다 베이스 주소 변경됨

---

## 🛠 PE 재배치 동작 원리

1. 프로그램에서 하드코딩된 **주소 위치를 찾음**
2. 해당 주소에서 **ImageBase 값을 뺌**
3. 실제 로딩된 베이스 주소를 더해 **재계산된 주소로 사용**

---

## 📑 Base Relocation Table 구조

### 위치
IMAGE_NT_HEADERS → OPTIONAL_HEADER → DataDirectory[6]

### 구조체
```c
typedef struct _IMAGE_BASE_RELOCATION { 
    DWORD VirtualAddress;   // 기준 RVA 주소
    DWORD SizeOfBlock;      // 해당 블록의 크기
    // WORD TypeOffset[1]; // 이 아래 WORD 배열이 따라옴 (다수의 항목)
} IMAGE_BASE_RELOCATION; typedef IMAGE_BASE_RELOCATION UNALIGNED * PIMAGE_BASE_RELOCATION;

```

VirtualAddress: 재배치가 필요한 코드 블록의 기준 RVA

SizeOfBlock: 전체 블록의 크기 (헤더 + 배열 전체 크기)

TypeOffset[]: 실제 재배치 항목들 (WORD 배열)

✅ 이 구조는 실행 파일이 다른 주소에 로딩되었을 때도 코드가 정확히 작동하도록 보장하는 중요한 역할을 합니다.



