
# week 01 - x86 Assembly 정리

## 1. 어셈블리 기본 개념

- x86은 32비트 레지스터 기반 명령어 구조
- 명령어는 주로 MOV , ADD, JMP, CALL , CMP 등

## 2. 주요 레지스터 정리
레지스터 : cpu가 사용하려고 가지고 있는 데이터

eax : 계산에 대한 저장을 하는 데이터
ebx : 베이스레지스터 (주소계산)
ecx : 카운터레지스터 (반복 루프 등)
edx : 데이터 레지스터 (보조연산)
esi : source inex (출발지)
edi  : destination index (목적지)
ebp : 스택의 아랫부분 (베이스) 포인터
esp : 스택의 윗부분 (탑) 포인터
eip : cpu가 다음에 실행할 명령어 주소 
EFLAGS : 플래그레지스터

## 3. 주요 명령어 예시

MOV EAX , 1     : EAX에 1 저장
PUSH EAX        : EAX 값을 스택에 push
POP EBX         : 스택에서 EBX로 POP
CMP EAX, EBX    : 두 값을 비교
JZ 0x00401000   :  Zero이면 해당 주소로 점프
CALL 0x00401000 : 함수 호출
RET             : 함수 복귀


## 4. 조건 분기 명령어

### CMP + JUMP 조합
CMP EAX, 5        : EAX가 5인지 비교
JZ 0x401000       : 같으면 0x401000으로 점프
JNZ 0x401050      : 다르면 0x401050으로 점프

주요 조건 분기 명령어
명령어                의미                    설명
JZ                   Zero 일 때 점프          CMP 결과가 같으면 (ZF=1)
JNZ                  Zero 아닐때 점프         CMP 결과 다르면 (ZF=0)
JE                   Equal(JZ와 동일)       
JNE                  Not Equal(JNZ와 동일)
JA                    Above(>)              부호없는비교
JB                    Below(<)              부호없는비교
JG                  Greater than            부호있는비교
JL                  Less than               부호있는비교
JMP                무조건 점프               분기없음

## 5. 함수 호출과 스택 구조

### 호출 흐름

CALL 0x401000        : 현재 주소를 스택에 PUSH
-> EIP(다음에 실행할 명령어 주소)가 함수로 이동
-> 함수 끝나면 RET ->  이전 주소 POP -> 복귀

### 스택의 구조
↑ 증가 방향
| 함수 인자 (Arg3)
| 함수 인자 (Arg2)
| 함수 인자 (Arg1)
| Return Address (CALL에 의해 PUSH됨)
| EBP (이전 함수 기준 포인터)
↓ 감소 방향

### 스택 관련 명령어
PUSH : 값을 스택에 저장, ESP -= 4
POP : 값을 스택에서 꺼냄 , ESP += 4
EBP : 함수 시작 기준 포인터 역할

## 6. 간단한 디버깅 흐름 예시
1. Entry Point 도달 CALL sub_401000
2. 내부에서 CMP EAX, 1234 -> JNZ 실패 ->메시지 출력
3. 인증 성공 흐름 : PUSH, CALL, RET 구조 반복

## 요약
- x86 어셈블리 이해는 리버싱 분석의 기본
- MOV, CALL, CMP, JZ, PUSH/POP 흐름 이해가 핵심
- 스택(ESP/EBP), 조건 분기(JZ/JNZ), 함수 복귀 구조(RET)까지 이해하면
  실전 분석에서 함수 흐름을 쉽게 따라갈 수 있음
