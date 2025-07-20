# 🔐 4일차: 스택, 호출 규약, 함수 리턴 메커니즘 이론 및 실습

---

## ✅ 학습 목표
- 스택 구조 이해 (LIFO, push/pop)
- 함수 호출 시 스택 프레임 생성 및 해제 흐름 이해
- `call`, `ret`, `push`, `pop` 명령어 실습
- cdecl, stdcall 등 호출 규약 이해 및 EIP 복구 원리 분석

---

## 🧠 이론

### 📌 스택(Stack) 구조
- 메모리에서 **하향식**으로 자람 (높은 주소 → 낮은 주소)
- **LIFO(Last In First Out)** 구조
- 주요 명령어: `push`, `pop`
- 스택 포인터(`ESP`)가 스택의 최상단 주소를 가리킴

  
push eax ; ESP = ESP - 4 → [ESP] = EAX
pop ebx ; EBX = [ESP] → ESP = ESP + 4

### 📌 함수 호출과 스택 프레임

- 함수가 호출되면 아래와 같은 스택 프레임이 생성됨

main():
call func → return address가 스택에 push됨

func():
push ebp → 이전 ebp 값 저장
mov ebp, esp → 현재 스택 프레임 시작점 저장
...
pop ebp → 이전 ebp 복구
ret → return address pop → EIP 복구


- `call` 명령어는 **리턴 주소(EIP)** 를 스택에 푸시하고, 함수로 점프
- `ret` 명령어는 스택에서 리턴 주소를 꺼내서 `EIP`로 복구

---

### 1. Calling Convention 이란?
Calling Convention은 함수 호출 시 **인자 전달 방식**, **스택 정리 방식**, **레지스터 사용 규약**을 정의한 규칙입니다.

### 2. 주요 Calling Convention 종류

| 종류        | 인자 전달 | 스택 정리 주체 | 특징 및 사용 예 |
|-------------|-----------|----------------|------------------|
| `cdecl`     | 오른쪽 → 왼쪽 (스택) | 호출자(caller) | C 언어 기본 방식 |
| `stdcall`   | 오른쪽 → 왼쪽 (스택) | 피호출자(callee) | WinAPI 대부분 |
| `fastcall`  | 왼쪽 -> 오른쪽 일부 인자를 레지스터(ECX, EDX)로 전달 | 피호출자 | 속도 최적화용 |
| `thiscall`  | 클래스 멤버 함수, `this` → ECX | 피호출자 | C++ 객체 지향에서 사용 |
| `ms64`      | RCX, RDX, R8, R9 → 레지스터 전달 | 호출자 | x64 Windows 환경 |
---


### 각 Calling Convention 예시

✅ cdecl (C Declaration)
인자 전달: 오른쪽에서 왼쪽 순서로 스택에 push

스택 정리: **호출자(caller)**가 한다

주로 사용되는 곳: C언어 기본 규약


```c
int sum(int a, int b) {
    return a + b;
}

int main() {
    sum(1, 2);
}

```

```
; 어셈블리 (Visual Studio)
push 2            ; b
push 1            ; a
call sum
add esp, 8        ; caller가 인자 정리
```


✅ stdcall (Standard Call)
인자 전달: 오른쪽에서 왼쪽 순서로 스택에 push

스택 정리: **피호출자(callee)**가 한다

주로 사용되는 곳: Windows API

```c

__stdcall int sum(int a, int b);

push 2
push 1
call sum         ; sum 함수 내부에서 ret 8 수행
; 호출자는 스택을 정리하지 않음

```

```
sum:
    mov eax, [esp+4] ; a
    add eax, [esp+8] ; b
    ret 8            ; 인자 8바이트 정리

```

✅ fastcall
인자 전달: 첫 번째와 두 번째 인자를 **레지스터(ecx, edx)**로 전달, 나머지는 스택

스택 정리: callee

주로 사용되는 곳: 마이크로소프트 컴파일러 최적화

```c

__fastcall int sum(int a, int b, int c);

mov ecx, 1
mov edx, 2
push 3             ; 세 번째 인자
call sum
; sum 내부에서 ret 4 수행

```


✅ thiscall
인자 전달: C++ 클래스 멤버 함수에서 this 포인터를 ecx 레지스터에 전달

스택 정리: callee

주로 사용되는 곳: C++ 클래스 메서드

```c
class A {
public:
    int add(int x) {
        return this->val + x;
    }
};

```

```
mov ecx, this      ; ecx에 this 포인터 저장
push x
call A::add
; callee가 ret 4 수행

```


## 🔧 실습 예제

### 📁 실습 파일
```c
// filename: call_test.c
#include <stdio.h>

void print_msg(char *msg) {
    printf("Message: %s\n", msg);
}

int main() {
    print_msg("Hello Reversing!");
    return 0;
}


```

### 🔍 실습 목표

✅ 목표 1: 함수 호출 전후 스택 확인
main 함수의 call print_msg 실행 시점

스택에 리턴 주소(EIP)가 push 되는지 확인

ret 시 해당 주소로 EIP가 복원되는지 확인

✅ 목표 2: 호출 규약 이해
cdecl 방식 → 함수 호출 후 스택 인자 제거 주체 확인

main 함수가 add esp, 4로 스택 인자를 직접 제거하는지 확인



### 📸 실습 포인트 (x32dbg 기준)
main 함수의 call print_msg 명령어에서 BP 설정

call 직후 → 스택 최상단(ESP)에 리턴 주소가 들어있는지 확인

print_msg 내부 ret 명령어 직전 ESP 상태 확인

ret 이후 → EIP가 main의 다음 줄로 이동하는지 확인

add esp, 4로 main이 스택 정리하는지 확인


### 📝 정리
call → 리턴 주소를 스택에 푸시 + EIP 이동

ret → 리턴 주소를 pop 하여 EIP에 넣음

호출 규약에 따라 인자 정리 주체가 다름

실시간으로 push/pop 변화 및 EIP 이동 확인 가능



