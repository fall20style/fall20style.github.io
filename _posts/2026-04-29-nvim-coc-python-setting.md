## coc-pyright 환경 설정 정리

### 1. 필수 구성 요소 설치

* Node.js 설치: coc.nvim 구동을 위해 최신 안정판(LTS) 설치 필요함
* Neovim 플러그인: vim-plug 등을 이용해 `neoclide/coc.nvim` 설치 완료해야 함
* 확장 설치: Neovim 실행 후 `:CocInstall coc-pyright` 명령어로 설치함


### 2. 주요 기능 설정 (coc-settings.json)

`:CocConfig` 명령어로 설정 파일을 열고 아래 내용을 추가함

* 타입 검사 수준: `"python.analysis.typeCheckingMode": "basic"` (또는 "strict") 설정으로 오류 감지함
* 자동 포맷팅: `"python.formatting.provider": "black"` 지정하여 코드 스타일 통일함
* 저장 시 자동 수정: `"coc.preferences.formatOnSaveFiletypes": ["python"]` 추가로 저장 시 자동 정렬함
* 분석 엔진: `"python.analysis.autoImportCompletions": true` 설정으로 자동 임포트 활성화함


### 3. 파이썬 인터프리터 및 가상환경 설정

* 인터프리터 지정: `:CocCommand python.setInterpreter` 실행 후 사용할 파이썬 경로 선택함
* 가상환경 인식: 프로젝트 루트에 .venv 또는 venv 폴더 생성 시 자동으로 감지함
* 경로 수동 설정: 가상환경이 다른 곳에 있다면 `python.venvPath` 항목에 디렉토리 경로 입력함


### 4. 추천 키 매핑 (init.vim / init.lua)

* 정의 이동: gd 입력 시 함수나 클래스 정의로 즉시 이동함
* 문서 확인: K 키로 해당 심볼의 도큐먼트(Hover) 팝업 띄움
* 이름 변경: `<leader>rn` 사용하여 프로젝트 전체 변수명 일괄 변경함
* 코드 액션: `<leader>ac`로 미사용 임포트 삭제 등 빠른 수정 실행함


### 5. 트러블슈팅 및 관리

* 서버 재시작: 설정 변경 후 적용 안 되면 `:CocRestart` 실행함
* 로그 확인: 문제 발생 시 `:CocOpenLog` 통해 에러 원인 파악함
* 업데이트: `:CocUpdate` 명령어로 coc-pyright 최신 버전 유지함


