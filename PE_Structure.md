# Week 01 - PE 파일 구조 요약


## 1.PE(Portable Executable)란?

- 윈도우에서 실행되는 모든 EXE, DLL 파일은 PE 형식을 따른다.
- 구조는 아래와 같이 구성됨

┌─────────────┐
│ DOS Header │ (MZ) - 64바이트
├─────────────┤
│ NT Header │ (PE) - Signature + FileHeader + OptionalHeader
├─────────────┤
│ Section Table│ (.text, .data 등)
├─────────────┤
│ 실제 섹션들 │
└─────────────┘

## 2. 주요 구조 설명

###[1] DOS Header ('MZ')
- 실행 실패 시 메시지 출력용
- 'e_lfanew' : PE header의 오프셋 주소를 가리킴 (중요!)

###[2] NT Header (`PE\0\0`)
- PE Signature + FileHeader + OptionalHeader 로 구성됨

#### NT Header 세부
*** NT Header 구조체 
typedef struct _IMAGE_NT_HEADERS {
  DWORD                   Signature;                     // PE Signature : 50450000 ("PE"00)
  IMAGE_FILE_HEADER       FileHeader;
  IMAGE_OPTIONAL_HEADER32 OptionalHeader;
} IMAGE_NT_HEADERS32, *PIMAGE_NT_HEADERS32;


*** FileHeader 구조체 

⦁	 IMAGE_FILE_HEADER       FileHeader;
typedef struct _IMAGE_FILE_HEADER {
⦁	  WORD  Machine;
⦁	  WORD  NumberOfSections;
DWORD TimeDateStamp;     빌드 했던 시간
 DWORD PointerToSymbolTable;
 DWORD NumberOfSymbols;
⦁	  WORD  SizeOfOptionalHeader;
⦁	  WORD  Characteristics;
} IMAGE_FILE_HEADER, *PIMAGE_FILE_HEADER;


| 필드 | 설명 |
|------|------|
| Machine | 대상 CPU 아키텍처 |
| NumberOfSections | 섹션 수 |
| TimeDateStamp | 컴파일 타임 | 빌드했던 시간
| SizeOfOptionalHeader | OptionalHeader 크기 |
| Characteristics | 실행 파일 플래그 |

#1 Machine 
CPU 호환칩의 고유한 번호 저장
예) 4C 01       리틀엔디언으로 01 4C

#2 NumberOfSections
PE파일(실행가능한파일) 코드, 데이터, 리소스 등이 각각 섹션에 나뉘어 저장되는데 
NumberOfSections는 PE파일 섹션의 개수를 알려준다. 이 값은 무조건 0보다 커야만 한다

예) NumberOfSections =1; 
원래는 sections = 3; 
섹션의 개수와 실제 섹션이 다르면 실행이 안된다 

#3 SizeOfOptionalHeader
IMAGE_OPTIONAL_HEADER32의 크기를 명시함
예) 32비트와 64비트로 프로그램이 만들어지기 때문 

#4 Characteristics *
파일 속성을 나타내는 값
실행 가능한 형태(EXE) 혹은 DLL 파일인지 정보를 저장한다

예) 0x0002  실행가능한 파일, 0x2000 DLL 파일 


*** OptionalHeader  구조체
 IMAGE_OPTIONAL_HEADER32 OptionalHeader;


typedef struct _IMAGE_OPTIONAL_HEADER {
*  WORD                 Magic;
  BYTE                 MajorLinkerVersion;
  BYTE                 MinorLinkerVersion;
  DWORD                SizeOfCode;
  DWORD                SizeOfInitializedData;
  DWORD                SizeOfUninitializedData;
*  DWORD                AddressOfEntryPoint;
  DWORD                BaseOfCode;
  DWORD                BaseOfData;
 * DWORD                ImageBase;
  *DWORD                SectionAlignment;
  *DWORD                FileAlignment;
  WORD                 MajorOperatingSystemVersion;
  WORD                 MinorOperatingSystemVersion;
  WORD                 MajorImageVersion;
  WORD                 MinorImageVersion;
  WORD                 MajorSubsystemVersion;
  WORD                 MinorSubsystemVersion;
  DWORD                Win32VersionValue;
 * DWORD                SizeOfImage;
  *DWORD                SizeOfHeaders;
  DWORD                CheckSum;
  * WORD                 Subsystem;
  WORD                 DllCharacteristics;
  DWORD                SizeOfStackReserve;
  DWORD                SizeOfStackCommit;
  DWORD                SizeOfHeapReserve;
  DWORD                SizeOfHeapCommit;
  DWORD                LoaderFlags;
 * DWORD                NumberOfRvaAndSizes;
  IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES];
} IMAGE_OPTIONAL_HEADER32, *PIMAGE_OPTIONAL_HEADER32;


|------|------|
| AddressOfEntryPoint | 실행 시작 위치 (RVA) |
| ImageBase | 메모리 적재 기준 주소 |
| SectionAlignment | 메모리에서 섹션 정렬 단위 |
| FileAlignment | 파일 내 섹션 정렬 단위 |
| SizeOfImage | 전체 메모리 크기 |
| Subsystem | 실행 환경 (GUI, CLI 등) |

> ⚠️ EntryPoint = 실제 디버깅 시작 위치!


#1 Magic   
IMAGE_OPTIONAL_HEADER32 10B
IMAGE_OPTIONAL_HEADER64 20B
32비트냐 64비트로 만들어졌냐에 따라 크기가 다름 

#2 AddressOfEntryPoint
EP(Entry Point)의 RVA(상대주소) 값을 가지고 있다.
프로그램의 최초로 실행되는 코드에 시작주소로 매우 중요하다.

#3 ImageBase
EIP : CPU가 다음 실행할 명령어 Register
프로세스의 가상메모리는 0~FFFFFF 범위이다. (32bit)
IB(ImageBase) 광활한 메모리에서 PE파일(실행가능한파일)이 로드되는 시작주소를 나타낸다.
PE 로더 PE파일을 실행하기위해 프로세스를 생성하고 파일을 메모리에 로드한 후 EIP(RIP 64bit) Reg 값을 ImageBase + AddressOfEntryPoint 값으로 세팅한다.

⦁	EP & IB 차이점
⦁	BaseAddress(기본주소) + RVA(상대주소) = VA(절대주소)
IB : 실행파일이 로드될때 BaseAddress(기본주소)를 나타냄
EP : 실행파일이 시작될때 실행되는 함수의 주소를 나타냄


#4 SectionAlignment & FileAlignment
PE파일의 바디 부분은 섹션으로 나뉜다.
파일에서 섹션의 최소 단위를 나타내는 것이 FileAlignment 
메모리에서 섹션의 최소 단위를 나타내는 것이 SectionAlignment

#5 SizeOfImage
PE파일이 메모리 로딩되었을때 가상메모리 PE IMAGE가 차지하는 크기를 나타낸다
파일의크기 != 메모리의 로딩된 파일의 크기

#6 SizeOfHeader
PE파일 전체의 크기를 나타냄

#7 Subsystem
.sys 인지 dll or exe구별
sys인지 판단

#8 NumberOfRvaAndSizes
IMAGE_OPTIONAL_HEADER32의 마지막 멤버인 DataDirectory의 배열 개수를 나타냄

#9 DataDirectory
IMAGE_OPTIONAL_HEADER32 배열


## 4. Section Table (.text, .rdata 등)

| 섹션 | 설명 |
|------|------|
| .text | 코드 (실행 명령어) |
| .data | 초기화된 전역변수 |
| .rdata | 문자열, 상수 등 |
| .idata | Import Table (DLL 함수 정보) |
| .reloc | 재배치 정보 (ASLR 대응) |


Section Header
각 섹션의 속성(Property)을 정의한 것이 섹션헤더
text , data, rsrc 3가지로 나누어서 섹션을 저장하는 이유
1. 프로그램 복잡함 감소
2. 프로그램 안전성 
따라서 code/data/rsrc 마다 각각의 특성(접근권한이 다름)
text : 실행, 읽기 권한
data : 비실행, 읽기  권한 
rsrc : 비실행, 읽기 권한

섹션의 구조체
typedef struct _IMAGE_SECTION_HEADER {
  BYTE  Name[IMAGE_SIZEOF_SHORT_NAME];
  union {
    DWORD PhysicalAddress;
  *  DWORD VirtualSize;  : 메모리에서 섹션이 차지하는 크기
  } Misc;
*  DWORD VirtualAddress; : 메모리에서 섹션의 시작주소 (RVA = 상대주소)
*  DWORD SizeOfRawData; : 파일에서 섹션이 차지하는 크기
*  DWORD PointerToRawData; : 파일에서 섹션의 시작주소
  DWORD PointerToRelocations;
  DWORD PointerToLinenumbers;
  WORD  NumberOfRelocations;
  WORD  NumberOfLinenumbers;
*  DWORD Characteristics;   NT 헤더의 File Header 구조체 내에 있는 것과는 다름 , 섹션의 속성을(Bit OR)
} IMAGE_SECTION_HEADER, *PIMAGE_SECTION_HEADER;

**** VirtualAddress, PointerToRawData
VA(절대주소)랑은 다른애임

RVA TO RAW : PE파일이 메모리에 로딩되었을 때, 메모리주소(RVA 상대주소)와 오프셋을 잘 매핑할 수 있어야함
RAW (file offset)
PointerToRawData : 파일에서 섹션의 시작위치
VA(Virtual Address 절대주소x ) : 메모리에서 섹션의 시작위치
<공식>
RAW = RVA - VA(메모리 시작지점(ImageBase - VA)) + PointerToRawData


## 🛠️ 5. 실습 예시 (PE-Bear 사용)

> 분석 대상: `music_player.exe`  
> 툴: [PE-Bear](https://github.com/hasherezade/pe-bear)

### 확인 정보:

| 항목 | 값 |
|------|----|
| ImageBase | 0x00400000 |
| AddressOfEntryPoint | 0x00001000 |
| EntryPoint (VA) | `0x00400000 + 0x00001000 = 0x00401000` |
| .text RVA | 0x1000 |

→ **디버거에서 이 주소부터 실행되므로 브레이크포인트 걸기 용이**


## 🧠 6. 요약

- PE 구조는 리버싱 시 EntryPoint, Import Table 등 실전 분석의 출발점
- `ImageBase + RVA = VA` 공식을 이해해야 분석 위치를 정확히 지정할 수 있다.
- PE-Bear, PE View 등으로 구조 확인 연습 필수


