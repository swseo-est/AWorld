# 태스크 핸들러 모듈 (`task.py`)

이 스크립트는 `TaskEventRunner`의 전체 라이프사이클을 관리하는 핵심 핸들러인 `DefaultTaskHandler`를 정의합니다. 이 핸들러는 에이전트의 작업 수행이 아닌, 태스크 자체의 시작, 성공, 실패, 취소 등 시스템 레벨의 이벤트를 처리하는 "운영체제"와 같은 역할을 합니다.

## 주요 구성 요소

### `TaskHandler` (기본 클래스)

`DefaultHandler`를 상속하며, 태스크와 관련된 핸들러들을 위한 기본 클래스 역할을 합니다. 태스크 설정에 정의된 훅(hook)들을 초기화하는 기능을 주로 담당합니다.

---

### `DefaultTaskHandler`

`TaskHandler`의 구체적인 구현체이며, `TaskEventRunner`의 핵심 제어 센터입니다.

-   **등록**: `__{Constants.TASK}__`라는 이름으로 `HandlerFactory`에 등록됩니다. 이로 인해 `category`가 `Constants.TASK`인 모든 메시지를 이 핸들러가 처리하게 됩니다.

-   **주요 역할**: `TaskEventRunner`의 전체 실행 흐름을 제어합니다. 에이전트나 도구의 구체적인 로직을 수행하는 대신, 태스크의 상태를 관리하고 시스템 전반의 이벤트를 처리합니다.

-   **핵심 로직 (`_do_handle`)**: 메시지의 `topic`에 따라 분기되는 `if/elif` 구조를 통해 다양한 시스템 이벤트를 처리합니다.
    -   **`TopicType.SUBSCRIBE_TOOL`**: 태스크 실행 중에 동적으로 새로운 도구를 등록할 수 있게 합니다.
    -   **`TopicType.ERROR`**: 시스템의 다른 부분에서 에러 메시지가 발행되면, 이 핸들러가 이를 감지합니다. 에러를 기록하고, 최종 `TaskResponse`를 실패 상태로 설정한 뒤, `self.runner.stop()`을 호출하여 `TaskEventRunner` 전체를 중지시킵니다.
    -   **`TopicType.FINISHED`**: 에이전트가 작업이 완료되었다고 판단하여 `FINISHED` 메시지를 보내면, 이 핸들러가 이를 받아 최종 `TaskResponse`를 성공 상태로 설정하고 실행기를 중지시킵니다.
    -   **`TopicType.START`**: 태스크의 가장 첫 시작을 처리하며, 에이전트 실행 루프를 개시하는 초기 메시지를 `yield`합니다.
    -   **`TopicType.CANCEL`**: 외부로부터의 태스크 취소 요청을 처리하고, 실행기를 정상적으로 종료시킵니다.
    -   **`TopicType.HUMAN_CONFIRM`**: 사람의 확인이나 입력이 필요할 때 실행을 일시 중지합니다.
    -   **`TopicType.OUTPUT`**: 출력(logging, UI 등)을 위해 생성된 메시지를 그대로 통과시킵니다.

요약하자면, `DefaultTaskHandler`는 `EventRunner`라는 엔진을 제어하는 운영체제와 같습니다. 에이전트나 도구처럼 구체적인 작업을 수행하지는 않지만, 태스크의 시작, 정상적인 종료, 예외 발생 시의 종료 등 전체 프로세스의 상태를 관리함으로써 시스템의 안정성과 제어 가능성을 보장합니다. 이는 잘 설계된 이벤트 기반 시스템의 핵심적인 관심사 분리(Separation of Concerns) 원칙을 보여줍니다.
