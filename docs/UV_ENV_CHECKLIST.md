# uv 사전 점검 체크리스트 (uv Pre-Check)

> 강의·워크숍 환경, 특히 **사내망/폐쇄망**에서 [uv](https://docs.astral.sh/uv/) 로 실습하기 전에 **1회** 확인할 것들. 수강생 다수가 같은 환경에서 동시에 받을 때의 실패를 미리 막는 용도.  
> 실습 전 인터넷이 되는 곳에서 `uv --version` 과 첫 `uv run` 을 **1회** 실행해 환경을 데워두세요(이후엔 오프라인도 동작). 사내망에서 막히면 강사에게 알려주세요. (README.md 에서 방법 참고)

## TL;DR
- uv 가 네트워크를 쓰는 건 **최초 1회, 단 3곳**: ① uv 설치 ② 파이썬 확보 ③ 패키지 설치. 한 번 받으면 `~/.cache/uv` 캐시로 **이후 오프라인** 동작.
- **표준 라이브러리만 쓰는 스크립트는 패키지 네트워크가 0** — uv 와 파이썬만 있으면 인터넷 없이 돈다.
- 폐쇄망이면 **사내 PyPI 미러 + 시스템 파이썬(only-system)** 조합으로 공개망 접속 0 으로도 운영 가능.

## 1. uv 가 네트워크를 쓰는 시점

| 시점 | 무엇을 받나 | 접속 호스트 | 오프라인/회피 |
|---|---|---|---|
| uv 설치 | uv 바이너리 | `astral.sh`, `github.com`, `objects.githubusercontent.com` | 바이너리를 사내에 1회 배포 |
| 파이썬 확보 | uv-managed CPython | `github.com`, `objects.githubusercontent.com` | 시스템 파이썬 사용(`UV_PYTHON_PREFERENCE=only-system`) |
| 패키지 설치 | 의존성(wheel/sdist) | `pypi.org`, `files.pythonhosted.org` | 사내 PyPI 미러(`UV_INDEX_URL`) |
| 표준 라이브러리만 쓰는 스크립트 | (없음) | — | 네트워크 불필요 |

> uv 는 시스템에 파이썬이 있어도 기본적으로 **자체 관리 파이썬을 내려받는다.** 시스템 파이썬을 쓰게 하려면 아래 `UV_PYTHON_PREFERENCE=only-system` 를 설정한다.

## 2. 1대로 사전검증 (방화벽 egress 로그를 함께 캡처)

실제 접속 도메인은 uv 버전마다 미세하게 다를 수 있으므로, **점검용 1대에서 아래를 돌리고 방화벽 egress 로그를 캡처**해 정확한 화이트리스트를 확정하는 것이 가장 확실하다.

```bash
uv --version                         # ① 부트스트랩 확인(설치돼 있나)
uv run --no-project script.py        # 의존성 없는 스크립트 — 패키지 네트워크 0 이어야 정상
uv run main.py                       # pyproject 의존성 사용 — 최초 1회 패키지 다운로드
uv sync                              # 프로젝트 의존성 일괄 설치(최초 1회)
uv run --with <패키지> tool.py       # 일회성 의존성 즉석 설치(최초 1회)
```

한 번 성공하면 캐시(`~/.cache/uv`)에 저장되어 이후 오프라인으로 동작한다.

## 3. 네트워크가 막혔을 때 — 영향과 대안

| 상황 | 영향 | 대안 |
|---|---|---|
| 표준 라이브러리만 쓰는 스크립트 | uv·파이썬만 있으면 **오프라인 OK** | uv 자체가 없으면 `python3 script.py` 로 폴백 |
| 의존성(패키지) 필요 | 다운로드 실패 | 사내 PyPI 미러, 또는 venv+pip 폴백(아래) |
| uv 설치/파이썬 자체가 차단 | 모든 `uv run` 불가 | uv 바이너리 사내 배포 + 시스템 파이썬 사용 |

venv + pip 폴백 (uv 가 막힐 때, PEP 668 회피):
```bash
python3 -m venv .venv && source .venv/bin/activate
pip install <패키지>
```

## 4. 사내 엔지니어에게 요청할 화이트리스트 (택1)

**A) 공개 도메인 허용 (HTTPS/443)**
- `astral.sh` — uv 설치 스크립트
- `github.com`, `objects.githubusercontent.com` — uv 바이너리 + uv-managed Python 다운로드
- `pypi.org` — 패키지 인덱스(메타데이터)
- `files.pythonhosted.org` — **패키지 실제 파일(wheel/sdist). 이게 빠지면 설치 전부 실패**
- (보조) `api.github.com`

**B) 폐쇄망 권장 (공개망 접속 0)**
- uv 바이너리를 1회 받아 **사내 배포**(설치 스크립트 대신 바이너리 직접) → `astral.sh`·`github.com` 불필요
- **시스템 파이썬** 사용 + `UV_PYTHON_PREFERENCE=only-system` → 파이썬 다운로드/GitHub 화이트리스트 불필요
- 사내 **PyPI 미러**(Nexus/Artifactory 등)에 필요한 패키지를 적재 후 `UV_INDEX_URL` 지정 → 공개 PyPI 불필요

## 5. 유용한 환경변수·플래그

| 변수 / 플래그 | 효과 |
|---|---|
| `UV_PYTHON_PREFERENCE=only-system` | uv-managed 파이썬 다운로드 금지, 시스템 파이썬만 사용 |
| `UV_PYTHON_DOWNLOADS=never` | 파이썬 자동 다운로드 차단 |
| `UV_INDEX_URL=<미러>` (또는 `UV_DEFAULT_INDEX`) | 패키지 인덱스를 사내 미러로 지정 |
| `UV_OFFLINE=1` | 네트워크 시도 자체를 막고 캐시만 사용 |
| `uv run --no-project script.py` | 프로젝트 동기화 생략 — 의존성 없는 스크립트/훅에 적합(빠름) |
| `uv run --with <pkg> script.py` | 프로젝트 변경 없이 일회성 의존성만 즉석 설치 |
| `~/.cache/uv` | 캐시 위치 — 한 번 받으면 오프라인 재사용(미리 데우기 가능) |

---
*이 문서는 특정 강의에 종속되지 않는 범용 체크리스트입니다. 환경에 맞게 자유롭게 수정해 쓰세요.*
