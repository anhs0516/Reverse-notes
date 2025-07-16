
# Week01 - Day2 IAT 구조 및 실습 정리

## 1. Import Table 개요

- PE 파일이 사용하는 외부 DLL API 정보를 담은 구조
- Optional Header의 DataDirectory[1]에서 Import Table RVA 확인 가능

## 2. IMAGE_IMPORT_DESCRIPTOR 구조

| 필드명 | 설명 |
|--------|------|
| Name | DLL 이름 (예: kernel32.dll) |
| OriginalFirstThunk | ILT (Hint/Name을 참조) |
| FirstThunk | IAT (실행 중 함수 주소로 대체됨) |

## 3. ILT vs IAT 차이

- ILT: Hint/Name 구조체 → 함수명
- IAT: 실행 시 실제 DLL 함수 주소 채워짐

## 4. 실습

분석 대상: `music_player.exe`  
도구: PE-Bear

| DLL | 함수 |
|-----|------|
| kernel32.dll | CreateFileA, WriteFile |
| user32.dll | MessageBoxA |

→ `.idata` 섹션에서 확인

## 5. 요약

- ILT는 참조용, IAT는 실행용
- 분석 시 IAT 주소 추적하여 API 호출 흐름 제어 가능
