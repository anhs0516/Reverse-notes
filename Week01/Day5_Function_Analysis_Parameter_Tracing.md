# Day 05 - 함수 내부 구조 분석 & 파라미터 추적

## 🧠 학습 목표

- 함수 호출 후 내부 흐름 분석 능력 습득
- 함수에 전달되는 인자의 위치 및 규약에 따른 추적 방법 학습
- 레지스터, 스택을 통한 파라미터 전달 방식 구분

---

## 📘 이론 정리

### 1. 함수 내부 기본 구조

- 함수 시작: prologue (`push ebp`, `mov ebp, esp`)
- 지역 변수 공간 확보: `sub esp, xx`
- 함수 종료: epilogue (`mov esp, ebp`, `pop ebp`, `ret`)

### 2. 파라미터 전달 방식에 따른 분석

| 규약          | 인자 전달 방식                | 호출자/피호출자 정리 | 예시                |
|---------------|-------------------------------|------------------------|---------------------|
| cdecl         | 인자를 **스택에 역순**으로 push | 호출자가 정리           | `call printf`       |
| stdcall       | 인자를 **스택에 역순**으로 push | 피호출자가 정리         | `call MessageBoxA`  |
| fastcall      | 첫 2개 인자 → ECX, EDX 사용     | 피호출자가 정리         | `call strcpy`       |
| thiscall(C++) | this → ECX, 나머지는 스택      | 피호출자가 정리         | `call obj->func()`  |

---

## 🔍 실습 예제: 파라미터 추적 및 함수 내부 분석

### 🔧 환경

- 디버거: x32dbg
- 샘플: `function_call.exe` (임의 제작된 테스트용 바이너리)

### 📌 실습 내용

#### 1. Entry Point → Main 함수 추적

- `Ctrl + E`로 Entry Point로 이동
- 코드 따라가다 보면 `call main` 또는 main 유사 함수 도달

#### 2. 함수 진입 후 구조 확인

```asm
main:
00401000   push ebp
00401001   mov ebp, esp
00401003   sub esp, 0x10     ; 지역 변수 공간
00401006   push offset str   ; "Hello"
0040100B   call print_msg

```


####  3. print_msg 함수 분석
```asm
print_msg:
00401010   push ebp
00401011   mov ebp, esp
00401013   mov eax, [ebp+8]     ; 인자 "Hello" 추출
00401016   push eax
00401017   call printf
0040101C   leave
0040101D   ret
```

[ebp+8]은 첫 번째 인자를 의미 (cdecl 기준)

printf는 cdecl → 호출자가 스택 정리

print_msg는 ret 앞에 add esp, 4가 없음 → 호출자가 인자 정리함

#### 4. x32dbg 파라미터 확인 방법
F8 → Step Over, F7 → Step Into

call print_msg 위에서 F7 → 함수 내부 진입

스택 창에서 [ESP+4] 또는 [EBP+8]로 인자 확인

레지스터 및 스택 확인 탭 활용

#### ✅ 실습 요약
함수 진입 후 push ebp → mov ebp, esp → sub esp, xx → 지역 변수 확보

call 직전 push된 인자를 [EBP+8], [EBP+0C] 등의 위치에서 확인 가능

인자 전달 순서와 반환 후 스택 정리는 함수 규약에 따라 다름

