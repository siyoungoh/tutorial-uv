# tutorial-uv 실행 가이드 (복붙 즉시 실행)

이 문서는 uv를 처음 쓰는 소프트웨어 엔지니어가 이 저장소를 바로 실행할 수 있도록 작성되었다.

대상 프로젝트 정보:
- Python 하한: 3.13+
- 런타임 의존성: requests
- 개발 의존성: pytest

핵심 규칙:
- `python`, `pip`를 직접 실행하지 않는다.
- 항상 `uv run`, `uv add`, `uv sync`를 사용한다.

---

## 0) 사전 조건

1. uv가 설치되어 있어야 한다.
2. 현재 디렉터리가 프로젝트 루트여야 한다. (`pyproject.toml`이 보여야 함)

---

## 1) 바로 실행 (가장 짧은 경로)

아래 명령을 순서대로 실행하면 된다.

```bash
uv python install 3.13
uv sync --dev
uv run python -V
uv run python main.py
```

정상 결과:
- `uv run python -V` 결과가 `Python 3.13.x`
- `uv run python main.py` 결과가 `Hello from tutorial-uv!`

설명:
- `uv python install 3.13`: uv 관리 Python 런타임을 준비
- `uv sync --dev`: `pyproject.toml` + `uv.lock` 기준으로 개발 환경 동기화
- `uv run ...`: 프로젝트 `.venv` 컨텍스트에서 명령 실행

### 명령별 해설

`uv python install 3.13`
- 의미: uv가 Python 3.13 런타임을 로컬에 설치한다.
- 왜 필요한가: 시스템 Python 상태와 무관하게, 팀이 의도한 런타임을 확보할 수 있다.
- 언제 쓰는가: 새 PC 세팅, CI 러너 초기화, 로컬에 3.13이 없을 때.

`uv sync --dev`
- 의미: `pyproject.toml`과 `uv.lock`에 정의된 의존성을 개발용 그룹까지 포함해 `.venv`에 동기화한다.
- 왜 필요한가: 실행 환경을 잠금 파일 기준으로 재현해서 "내 컴퓨터에서만 되는" 상태를 줄인다.
- 언제 쓰는가: 저장소를 처음 받았을 때, lock 파일 변경 후, CI 테스트 전에.

`uv run python main.py`
- 의미: 프로젝트 가상환경 컨텍스트에서 Python 명령을 실행한다.
- 왜 필요한가: 셸 활성화 여부와 무관하게 올바른 인터프리터/패키지로 실행된다.
- 언제 쓰는가: 애플리케이션 실행, 스크립트 실행, 테스트 실행 시 기본 실행 방식.

---

## 2) 현재 환경 검증

```bash
cat .python-version
uv run python -V
```

검증 기준:
- `.python-version`은 프로젝트 기본 버전 핀이다.
- 실제 실행 버전은 `uv run python -V` 결과를 기준으로 판단한다.

중요:
- `.python-version`만 바꿔도 기존 `.venv`가 자동으로 바뀌지는 않는다.
- 버전 변경 후에는 가상환경을 재생성하거나 동기화해야 한다.

---

## 3) 의존성 관리

런타임 의존성 추가:

```bash
uv add requests
```

개발 의존성 추가:

```bash
uv add --dev pytest
```

의미:
- 런타임 의존성: 애플리케이션 실행에 필요한 패키지
- 개발 의존성: 테스트/린트/타입체크 등 개발 단계에서만 필요한 패키지

변경 파일:
- `pyproject.toml`
- `uv.lock`

왜 필요한가:
- 의존성 선언(`pyproject.toml`)과 해석 결과(`uv.lock`)를 함께 관리해야 팀/CI/배포 환경을 일관되게 유지할 수 있다.

언제 쓰는가:
- 새 라이브러리를 도입할 때
- 런타임 패키지를 개발 전용 패키지와 분리할 때
- 의존성 버전을 갱신해야 할 때

---

## 4) 팀 표준 운영안 (최소 명령 세트)

로컬 개발:

```bash
uv python install 3.13
uv sync --dev
uv run python main.py
```

CI:

```bash
uv python install 3.13
uv sync --dev
uv run python main.py
```

배포(프로덕션):

```bash
uv python install 3.13
uv sync
uv run python -m your_app
```

주의:
- 프로덕션에서는 보통 `--dev`를 사용하지 않는다.

---

## 5) 새 프로젝트를 uv로 시작할 때 (참고)

이 저장소는 이미 초기화되어 있어 아래 명령은 보통 필요 없다.

```bash
uv init
```

`uv add`는 `pyproject.toml`이 있어야 동작한다.

`uv init`
- 의미: uv 프로젝트 기본 파일(`pyproject.toml`)을 생성한다.
- 왜 필요한가: `uv add` 같은 프로젝트 모드 명령이 동작하려면 프로젝트 메타데이터가 필요하다.
- 언제 쓰는가: 새 디렉터리에서 프로젝트를 처음 만들 때. (이 저장소는 이미 초기화되어 있어 보통 생략)

---

## 6) 자주 발생하는 에러 + 해결법

아래는 이 가이드에서 사용하는 핵심 명령 기준으로, 자주 발생하는 실패 케이스와 해결 절차를 정리한 섹션이다.

### `uv python install 3.13` 실패

의미:
- Python 3.13 런타임 다운로드/설치 단계에서 실패한 상태다.

왜 필요한가:
- 이후 `uv sync`, `uv run`이 일관된 런타임에서 동작하려면 먼저 설치가 성공해야 한다.

언제 쓰는가:
- 새 PC 세팅, CI 초기화, 로컬에 3.13 런타임이 없을 때.

자주 나오는 에러:
- `uv: command not found`
- 네트워크/인증서 관련 다운로드 실패

해결법:
1. `uv --version`으로 uv 설치 확인
2. 회사망/프록시 환경이면 네트워크 정책 확인 후 재시도
3. 설치 가능 버전 확인: `uv python list`

### `uv sync --dev` 실패

의미:
- `pyproject.toml`/`uv.lock` 기준 환경 동기화 중 실패한 상태다.

왜 필요한가:
- 의존성 설치가 실패하면 실행/테스트 결과를 신뢰할 수 없다.

언제 쓰는가:
- 프로젝트 최초 실행 전, 의존성 변경 후, CI 실행 전.

자주 나오는 에러:
- `No pyproject.toml found in current directory or any parent directory`
- 의존성 해석 충돌(버전 제약 불일치)

해결법:
1. 현재 경로 확인: `ls pyproject.toml`
2. 프로젝트 루트에서 다시 실행
3. 버전 충돌 시 `pyproject.toml` 제약 확인 후 `uv lock` 또는 `uv sync` 재실행

### `uv run python main.py` 실패

의미:
- 프로젝트 컨텍스트 실행 중 인터프리터/코드/의존성 문제로 실패한 상태다.

왜 필요한가:
- 앱 실행 확인은 환경 세팅 완료 여부를 검증하는 최종 단계다.

언제 쓰는가:
- 로컬 실행, 스크립트 실행, CI smoke test.

자주 나오는 에러:
- `ModuleNotFoundError: ...`
- 잘못된 Python 버전으로 인한 실행 에러

해결법:
1. `uv sync --dev` 재실행
2. 버전 확인: `uv run python -V`
3. import 확인: `uv run python -c "import requests; print('ok')"`

### `uv add ...` 실패

의미:
- 의존성 추가 및 잠금 파일 갱신 과정에서 실패한 상태다.

왜 필요한가:
- 의존성을 선언적으로 관리하지 않으면 팀 환경/CI 재현성이 깨진다.

언제 쓰는가:
- 런타임/개발 의존성 신규 도입 또는 교체 시.

자주 나오는 에러:
- `No pyproject.toml found in current directory or any parent directory`
- 인덱스 접근 실패 또는 패키지 이름 오타

해결법:
1. 프로젝트 루트 여부 확인: `ls pyproject.toml`
2. 패키지 이름 재확인
3. 네트워크/패키지 인덱스 접근 가능 여부 확인

---

## 7) 복붙 진단 명령 모음

아래 명령은 문제를 빠르게 분류하기 위한 진단용 스니펫이다.

### A. 기본 상태 한 번에 점검

의미:
- uv 설치, 현재 경로, Python 버전, 프로젝트 파일 존재 여부를 한 번에 확인한다.

왜 필요한가:
- 문제 원인을 환경/경로/버전/파일 누락으로 빠르게 분리할 수 있다.

언제 쓰는가:
- 어떤 에러든 최초 진단 시작점으로 사용.

```bash
pwd
uv --version
cat .python-version
uv run python -V
ls pyproject.toml uv.lock
```

### B. 의존성/환경 재동기화 진단

의미:
- 잠금 파일 기준으로 환경을 다시 맞추고 핵심 import를 검증한다.

왜 필요한가:
- `ModuleNotFoundError`나 실행 불일치 문제를 가장 빨리 해결할 수 있는 경로다.

언제 쓰는가:
- `uv run` 실행 실패, 패키지 인식 실패, 버전 불일치 의심 시.

```bash
uv sync --dev
uv run python -V
uv run python -c "import requests; print('requests ok')"
uv run python -c "import pytest; print('pytest ok')"
```

### C. 실행 경로 진단

의미:
- 실제로 어떤 인터프리터가 선택되어 실행되는지 확인한다.

왜 필요한가:
- `.python-version`을 바꿨는데도 버전이 안 바뀐 것처럼 보이는 문제를 확인할 수 있다.

언제 쓰는가:
- Python 버전 관련 혼선이 있을 때.

```bash
cat .python-version
uv run python -V
.venv/bin/python -V
```

해석:
- `uv run python -V`와 `.venv/bin/python -V`가 같으면 프로젝트 실행 환경은 정상적으로 일치한다.

### D. 프로젝트 초기화 누락 진단 (`uv add` 실패 시)

의미:
- `uv add`가 실패하는 대표 원인인 `pyproject.toml` 부재를 확인한다.

왜 필요한가:
- `uv add`는 프로젝트 모드 명령이라 메타데이터 파일이 필수다.

언제 쓰는가:
- `No pyproject.toml found...` 에러 발생 시.

```bash
ls pyproject.toml || echo 'pyproject.toml 없음'
```

필요 시(새 프로젝트에서만):

```bash
uv init
```

### E. 네트워크/인덱스 이슈 진단

의미:
- 패키지 다운로드 경로가 막혀 있는지 간접 확인한다.

왜 필요한가:
- 사내망/프록시/인증서 환경에서 설치 실패가 자주 발생한다.

언제 쓰는가:
- 설치/동기화가 네트워크 관련 메시지로 실패할 때.

```bash
uv python list
uv sync --dev
```

해석:
- `uv python list`가 정상인데 `uv sync`만 실패하면 의존성 해석/인덱스 문제일 가능성이 높다.
