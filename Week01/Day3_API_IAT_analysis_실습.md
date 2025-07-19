# 📦 HelloWorld.exe 리버싱 실습 - MessageBoxA API 분석

## 📌 목표
- MessageBoxA 함수의 연결 방식 이해
- IAT, EAT 구조 파악 및 분석 실습

## 🛠️ 사용 도구
- PEview / CFF Explorer
- x32dbg
- HxD
- HelloWorld.exe

## 🔍 MessageBoxA 연결 흐름

1. `user32.dll` → EAT → MessageBoxA 위치 확인
2. HelloWorld.exe → IAT → 해당 함수 주소 채워짐
3. CALL 명령어 → IAT 참조 → 실제 함수 호출

## 📁 EAT 분석

- 대상: user32.dll
- 위치: Export Directory
- 함수명: MessageBoxA
- RVA → VA 계산 확인

## 📥 IAT 분석

- 대상: HelloWorld.exe
- 위치: Import Directory → user32.dll
- 함수명: MessageBoxA
- 실행 전: 이름만 존재
- 실행 중: OS가 IAT에 실제 주소 채움

## 🧪 디버깅 실습

1. x32dbg로 HelloWorld.exe 열기
2. MessageBoxA 호출 확인
<img width="1219" height="563" alt="image" src="https://github.com/user-attachments/assets/8707f53b-d429-4d44-8101-448691d2a1a2" />

3. CALL 명령 추적 → 실제 주소 추적
<img width="1194" height="630" alt="image" src="https://github.com/user-attachments/assets/c7283c64-ad0f-4598-8db4-427b3e6ca917" />

4. Dump 창에서 IAT 확인
<img width="1318" height="678" alt="image" src="https://github.com/user-attachments/assets/d4d8dd55-d7d3-4eee-84ae-81b359ce2ca9" />

## 🧠 결론

- DLL의 함수는 EAT에 등록됨
- PE 파일은 IAT를 통해 DLL 함수 호출
- IAT는 실행 중 실제 함수 주소로 채워짐

| 항목                            | 내용                                                        |
| ----------------------------- | --------------------------------------------------------- |
| `771F05B0`                    | **MessageBoxA의 실제 주소** (user32.dll 내 함수 진입점)              |
| `0040821C`                    | 이 주소를 담고 있는 메모리 = **IAT 엔트리**                             |
| `jmp dword ptr ds:[0040821C]` | IAT를 통해 실제 API 호출 (간접 호출)                                 |
| IAT의 역할                       | 실행 중 OS가 DLL을 메모리에 올리고, 그 함수 주소를 IAT에 써넣음. 코드에서는 IAT만 참조함 |
