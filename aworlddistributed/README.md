# AWorld 분산 서버 (`aworlddistributed`)

이 디렉토리는 AWorld 프레임워크를 확장 가능하고 분산된 컨테이너화된 애플리케이션으로 실행하는 데 필요한 구성 요소들을 포함합니다. 에이전트 서빙, 태스크 관리, 그리고 서비스로서의 도구 제공을 위한 백엔드 인프라를 제공합니다.

![img.png](img.png)

## 1. 아키텍처 개요

AWorld 분산 서버는 마이크로서비스 아키텍처를 기반으로 구축되었습니다. Docker를 사용하여 배포하고 `docker-compose`를 통해 관리하도록 설계되었습니다.

주요 구성 요소는 다음과 같습니다:
-   **API 서버**: 에이전트 태스크 실행 요청을 받는 주 진입점 (FastAPI 사용 추정).
-   **태스크 관리**: 사용 가능한 에이전트 워커들에게 태스크를 분배하는 시스템.
-   **에이전트 워커**: 에이전트 로직을 실행하는 AWorld 프레임워크 인스턴스. `aworldspace/agents` 디렉토리에 정의되어 있습니다.
-   **데이터베이스**: 에이전트 메모리, 태스크 상태, 사용자 데이터를 영구 저장하기 위한 PostgreSQL 데이터베이스.
-   **MCP 도구 서버**: 각각 특정 도구(예: 웹 브라우징, 파일 시스템 접근, 코드 실행)를 제공하는 개별 마이크로서비스 모음. 에이전트는 MCP(Multi-agent Communication Protocol)를 통해 이 서버들과 통신합니다.

## 2. 디렉토리 구조

| 디렉토리                | 설명                                                                                                                                                                                                                          |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`aworldspace/`**      | 분산 서버의 핵심 애플리케이션 모듈. 웹 서버 로직, 데이터베이스 모델, API 라우트를 포함합니다.                                                                                                                                    |
| `aworldspace/agents`    | 분산 환경에 맞게 조정된 에이전트 정의를 포함합니다. 이 에이전트들은 태스크 관리 시스템에 의해 로드되고 실행되도록 설계되었습니다.                                                                                                   |
| `aworldspace/db`        | PostgreSQL 데이터베이스와 상호 작용하기 위한 SQLAlchemy 데이터베이스 모델(`models.py`) 및 연결 로직(`db.py`)을 정의합니다.                                                                                                         |
| `aworldspace/routes`    | 주 서버에 의해 노출되는 API 엔드포인트 정의(예: 태스크 제출 및 확인)를 포함합니다.                                                                                                                                             |
| **`client/`**           | 배포된 AWorld 서버와 프로그래밍 방식으로 상호 작용하기 위한 Python 클라이언트 코드(`aworld_client.py`)를 제공합니다. 이를 통해 사용자는 다른 애플리케이션에서 태스크를 제출하고 결과를 검색할 수 있습니다.                        |
| **`mcp_servers/`**      | 수많은 독립형 Python 서버를 포함하는 중요한 디렉토리. 각 서버는 특정 도구(예: `browser_server.py`, `search_server.py`, `e2b_code_server.py`)를 래핑하고 에이전트가 사용할 수 있도록 네트워크를 통해 노출합니다. |
| **`main.py`**           | AWorld 서버 애플리케이션을 시작하기 위한 주 진입점입니다.                                                                                                                                                                     |
| **`Dockerfile`**        | 주 AWorld 서버 Docker 이미지를 빌드하기 위한 레시피입니다.                                                                                                                                                                    |
| **`docker-compose.yaml`** | 주 API 서버, 데이터베이스 및 모든 개별 MCP 도구 서버를 포함한 모든 서비스의 배포를 조정하기 위한 설정 파일입니다.                                                                                                            |
| **`requirements.txt`**  | 분산 서버 및 모든 구성 요소에 필요한 Python 의존성 목록입니다.                                                                                                                                                                 |
| **`start.sh`**          | 서비스를 시작하기 위한 편의 스크립트입니다.                                                                                                                                                                                   |

## 3. 빠른 시작

1.  **Docker 이미지 빌드**:
    ```sh
    docker build --build-arg MINIMUM_BUILD=true -f Dockerfile --progress=plain -t aworldserver:main .
    ```

2.  **Docker Compose로 서비스 실행**:
    ```sh
    docker-compose up -d
    ```
    이 명령은 `docker-compose.yaml` 파일에 정의된 주 API 서버, 데이터베이스 및 기타 모든 서비스를 시작합니다.

3.  **서버와 상호 작용**:
    Python 클라이언트, cURL 또는 호환되는 웹 UI(예: OpenWebUI)를 통해 실행 중인 서버와 상호 작용할 수 있습니다.

    **cURL 사용 예시:**
    ```shell
    curl http://localhost:9299/v1/chat/completions \
      -H "Content-Type: application/json" \
      -H "Authorization: Bearer your-api-key" \
      -d '{
      "model": "gaia_agent",
      "messages": [
        {
          "role": "user",
          "content": "Your question or prompt here"
        }
      ]
    }'
    ```
