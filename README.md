# tutorial-uv

uv를 처음 사용하는 소프트웨어 엔지니어를 위한 최소 Python 프로젝트 예제입니다.

## 설치

https://docs.astral.sh/uv/getting-started/installation/


## 튜토리얼 스펙

- 패키지 매니저: uv
- Python 하한 버전: 3.13+
- 런타임 의존성: requests
- 개발 의존성: pytest


## 빠른 시작 (복붙 실행)

아래 4줄만 실행하면 프로젝트가 바로 동작합니다.

```bash
uv python install 3.13
uv sync --dev
uv run python -V
uv run python main.py
```

예상 출력:

```text
Python 3.13.x
Hello from tutorial-uv!
```

## 이 프로젝트에서 중요한 규칙

1. `python`, `pip`를 직접 실행하지 않습니다.
2. 항상 `uv run`, `uv add`, `uv sync`를 사용합니다.
3. 의존성은 `pyproject.toml`과 `uv.lock`으로 관리합니다.
- 자세한 명령어 설명은 [instruction.md](instruction.md)를 참고하세요.

## 자주 쓰는 명령

의존성 설치(잠금 파일 기준):

```bash
uv sync --dev
```

- 의미: 잠금 파일 기준으로 개발 의존성까지 환경을 맞춘다.
- 왜 필요한가: 로컬/CI 실행 결과를 일치시킨다.
- 언제 쓰는가: 처음 실행 전, 의존성 변경 후.

런타임 의존성 추가:

```bash
uv add requests
```

- 의미: 실행 시 필요한 패키지를 추가한다.
- 왜 필요한가: 앱 실행에 필요한 라이브러리를 프로젝트에 선언한다.
- 언제 쓰는가: 프로덕션에서도 필요한 라이브러리 도입 시.

개발 의존성 추가:

```bash
uv add --dev pytest
```

- 의미: 개발/테스트 전용 패키지를 추가한다.
- 왜 필요한가: 운영 환경과 개발 도구를 분리 관리할 수 있다.
- 언제 쓰는가: 테스트, 린트, 타입체크 도구 추가 시.

프로젝트 실행:

```bash
uv run python main.py
```

- 의미: 프로젝트 가상환경 컨텍스트에서 실행한다.
- 왜 필요한가: 셸 활성화 여부와 상관없이 같은 환경으로 실행된다.
- 언제 쓰는가: 앱/스크립트 실행 시 기본 방식.

의존성 import 확인:

```bash
uv run python -c "import requests; print(requests.__version__)"
```

## 자주 발생하는 에러 + 해결법 (요약)

`uv python install 3.13` 실패:
- 의미: Python 런타임 설치 단계 실패
- 왜 필요한가: 이후 모든 명령이 동일 런타임 기준으로 동작하려면 선행 필요
- 언제 쓰는가: 새 PC, CI 초기화
- 해결 핵심: `uv --version` 확인, `uv python list`로 설치 가능 버전 확인

`uv sync --dev` 실패:
- 의미: 의존성 동기화 실패
- 왜 필요한가: 실행/테스트 재현성 확보
- 언제 쓰는가: 최초 실행 전, lock 변경 후
- 해결 핵심: 프로젝트 루트에서 실행(`ls pyproject.toml`), 버전 충돌 시 제약 재확인

`uv run python main.py` 실패:
- 의미: 실행 컨텍스트 또는 의존성 문제
- 왜 필요한가: 실제 앱 실행 검증 단계
- 언제 쓰는가: 로컬 실행, CI smoke test
- 해결 핵심: `uv sync --dev` 재실행, `uv run python -V`로 버전 확인

- 상세 트러블슈팅은 [instruction.md](instruction.md)의 "자주 발생하는 에러 + 해결법" 섹션을 참고하세요.

## 복붙 진단 명령 모음

아래 블록은 문제 상황에서 가장 먼저 실행하는 진단 세트입니다.

```bash
pwd
uv --version
cat .python-version
uv run python -V
ls pyproject.toml uv.lock
uv sync --dev
uv run python -c "import requests; print('requests ok')"
uv run python -c "import pytest; print('pytest ok')"
```

- 의미: 환경/버전/프로젝트 파일/핵심 의존성 상태를 한 번에 점검합니다.
- 왜 필요한가: 에러 원인을 빠르게 범주화할 수 있습니다.
- 언제 쓰는가: 설치/동기화/실행 에러가 발생했을 때 첫 진단으로 사용합니다.

- 상세 해석은 [instruction.md](instruction.md)의 "복붙 진단 명령 모음" 섹션을 참고하세요.

## 파일 구성

- `pyproject.toml`: 프로젝트 메타데이터 및 의존성 선언
- `uv.lock`: 해석된 정확한 의존성 버전 잠금 파일
- `.python-version`: 프로젝트 기본 Python 버전 핀
- `main.py`: 실행 예제 엔트리포인트
- `instruction.md`: 초심자용 상세 실행 가이드

## CI 최소 예시

```bash
uv python install 3.13
uv sync --dev
uv run python main.py
```
