### Ghidra 설치 및 사용법

기드라란? 미국 국가 안보국(NSA)에서 만들어 오픈 소스로 공개한 역어셈블리어 프레임워크입니다

* 주요기능
  - 역어셈블리 : 실행 파일을 사람이 읽을 수 있는 소스코드로 변환
  - 디스어셈블리 : 기계어 코드를 어셈블리어로 변환
  - 디버깅 : 프로그램 실행을 추적하고 분석
 
이 프로그램 등장전에는 보편적으로 Hex-Rays사의 IDA를 사용 이는 2천만원대이나 기드라는 무료라고 합니다

### 설치

Dreamhack Ghidra 설치 페이지를 따라하시면 다운로드 할 수 있습니다.

주의사항
- Dreamhack에 나온 동일한 버젼으로 하셔야 편합니다... 최신버전들로만 다운받으니 실행되지 않더군요..


### 사용법

우선 프로젝트를 만들어줍니다

이후 디버깅할 파일을 실행해주면 아래와 같이 요약된 정보를 보여줍니다.

<img width="810" height="870" alt="image" src="https://github.com/user-attachments/assets/6b0dfdb2-c454-4f46-93f1-758ed8c9b215" />

더블 클릭 후 " Yes " 를 클릭해주면 아래와 같이 분석을 진행할 수 있습니다.

<img width="984" height="586" alt="image" src="https://github.com/user-attachments/assets/93f203d8-f5ec-4be3-96ac-af2632cfa152" />

분석을 클릭하게 되면 아래와 같이 창이 보입니다.

<img width="2553" height="1399" alt="image" src="https://github.com/user-attachments/assets/a27b0c70-24e7-470c-b4f2-cf597d45204a" />


- 그래프보기
도구 모음에서 Display Function Graph를 클릭하면 IDA에서 보는 Graph 형식과 동일하게 보입니다.

<img width="1010" height="1069" alt="image" src="https://github.com/user-attachments/assets/0dc25e85-5af6-40f6-b1d9-494fdbc2f778" />

- 문자열 추출

<img width="2541" height="1049" alt="image" src="https://github.com/user-attachments/assets/1bb73eaf-f734-4452-bc18-f2d12b0247ae" />



중요하다고 판단되는 기능들이 생기면 점차 추가하도록 하겠습니다.
