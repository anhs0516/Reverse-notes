
### 🔁 어셈블리어 Jump 명령어 종류 정리

#### 1. 📌 기본 개념

Jump 명령어는 프로그램의 흐름을 다른 위치(주소)로 변경할 때 사용합니다.

조건에 따라 점프하거나, 무조건 점프하기도 합니다.

#### 2. 🔹 무조건 점프 (Unconditional Jump)

| 점프종류 | 설명            | 예시             |
| -------- | ------------- | -------------- |
| `JMP`    | 무조건 해당 주소로 점프 | `jmp 0x401000` |


#### 3. 🔹 조건부 점프 (Conditional Jump)

조건 레지스터(예: ZF, SF, OF, CF 등)에 따라 점프 여부가 결정됩니다.

🟢 산술 조건 기반

| 점프종류      | 의미                                | 조건 플래그            | 설명        |
| ------------- | --------------------------------- | ----------------- | --------- |
| `JE` / `JZ`   | Jump if Equal / Zero              | ZF = 1            | 값이 같을 때   |
| `JNE` / `JNZ` | Jump if Not Equal / Not Zero      | ZF = 0            | 값이 다를 때   |
| `JG` / `JNLE` | Jump if Greater (signed)          | ZF = 0, SF = OF   | 큰 경우      |
| `JGE` / `JNL` | Jump if Greater or Equal (signed) | SF = OF           | 크거나 같을 경우 |
| `JL` / `JNGE` | Jump if Less (signed)             | SF ≠ OF           | 작은 경우     |
| `JLE` / `JNG` | Jump if Less or Equal (signed)    | ZF = 1 or SF ≠ OF | 작거나 같은 경우 |


🟡 부호 없는 조건

| 점프종류      | 의미                                | 조건 플래그           | 설명         |
| ------------- | --------------------------------- | ---------------- | ---------- |
| `JA` / `JNBE` | Jump if Above (unsigned)          | CF = 0, ZF = 0   | 크면 점프      |
| `JAE` / `JNB` | Jump if Above or Equal (unsigned) | CF = 0           | 크거나 같으면 점프 |
| `JB` / `JNAE` | Jump if Below (unsigned)          | CF = 1           | 작으면 점프     |
| `JBE` / `JNA` | Jump if Below or Equal (unsigned) | CF = 1 or ZF = 1 | 작거나 같으면 점프 |


4. 🔹 특별 조건 점프

| 점프종류      | 의미                           | 조건 플래그 | 설명             |
| ------------- | ---------------------------- | ------ | -------------- |
| `JC`          | Jump if Carry                | CF = 1 | 캐리 발생 시        |
| `JNC`         | Jump if Not Carry            | CF = 0 | 캐리 없을 시        |
| `JO`          | Jump if Overflow             | OF = 1 | 오버플로우 발생 시     |
| `JNO`         | Jump if Not Overflow         | OF = 0 | 오버플로우 없을 시     |
| `JS`          | Jump if Sign                 | SF = 1 | 음수일 경우         |
| `JNS`         | Jump if Not Sign             | SF = 0 | 양수일 경우         |
| `JP` / `JPE`  | Jump if Parity / Parity Even | PF = 1 | 패리티 비트가 짝수일 경우 |
| `JNP` / `JPO` | Jump if Not Parity / Odd     | PF = 0 | 패리티가 홀수일 경우    |


#### 5. 🔹 루프 및 카운트 점프

| 점프종류            | 의미                     | 설명           |
| ------------------- | ---------------------- | ------------ |
| `LOOP`              | CX 레지스터가 0이 아니면 점프     | `loop label` |
| `LOOPE` / `LOOPZ`   | CX ≠ 0 이고 ZF = 1 이면 점프 |              |
| `LOOPNE` / `LOOPNZ` | CX ≠ 0 이고 ZF = 0 이면 점프 |              |
| `JCXZ`              | CX = 0 이면 점프           |              |
| `JECXZ`             | ECX = 0 이면 점프          |              |
| `JRCXZ`             | RCX = 0 이면 점프 (64-bit) |              |


#### 6. 📍 참고: 점프의 주소 형태

상대 주소 점프: jmp short, jmp near, jmp far

간접 점프: 주소가 레지스터나 메모리에 있음 → jmp eax, jmp [esp]

Call과의 차이: call은 리턴 주소를 스택에 저장하고 점프함. jmp는 그냥 이동.

#### ✅ 분석 팁 (리버싱 관점)

조건 점프 앞의 cmp, test를 해석해서 조건을 파악해야 함

ZF, CF, SF 등 플래그 설정 명령들을 주의 깊게 살펴야 흐름이 보임

루프 안에서 조건점프는 반복 조건 확인 용도로 자주 사용됨


현재까지 리버싱 공부하면서는 무조건점프, 산술 조건 기반 점프, 부호 없는 조건 점프 관련 내용만 봐왔지만 검색 결과 다른 점프들도 알아둬야할 것 같아 정리해보았습니다.

