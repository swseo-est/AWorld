# ChromaDB 벡터 데이터베이스 구현체 (`chroma.py`)

이 스크립트는 `base.py`에 정의된 `VectorDB` 인터페이스의 구체적인 구현체인 `ChromaVectorDB` 클래스를 포함합니다. 이는 AWorld 프레임워크가 유명한 오픈소스 벡터 데이터베이스인 ChromaDB를 백엔드로 사용할 수 있도록 하는 어댑터(Adapter) 역할을 합니다.

## 주요 구성 요소

### `ChromaVectorDB` (구체적인 클래스)

`VectorDB` 추상 클래스를 상속받아, ChromaDB에 특화된 로직으로 모든 추상 메서드를 구현합니다.

-   **어댑터 패턴(Adapter Pattern)**: 이 클래스는 AWorld 프레임워크의 표준 `VectorDB` 인터페이스와 `chromadb` 라이브러리의 특정 API 사이를 변환해주는 완벽한 예시입니다. 이를 통해 프레임워크의 핵심 메모리 로직은 ChromaDB의 세부 구현에 대해 전혀 알 필요 없이, 일관된 방식으로 벡터 저장소 기능을 사용할 수 있습니다.

-   **유연한 초기화 (`__init__`)**:
    -   설정(config)에 따라 원격 ChromaDB 서버에 HTTP로 연결하거나(`chromadb.HttpClient`), 로컬 디스크에 영구적인 인스턴스를 생성하여(`chromadb.PersistentClient`) 사용할 수 있습니다.
    -   이러한 유연성 덕분에 간단한 로컬 개발 환경과 실제 운영 환경 모두에서 쉽게 사용할 수 있습니다.

-   **메서드 구현**:
    -   `VectorDB`에 정의된 모든 추상 메서드(예: `search`, `insert`, `delete_collection` 등)를 `chromadb` 클라이언트 라이브러리의 해당 API를 호출하여 구현합니다.
    -   **데이터 변환**: `_convert2EmbeddingResult`와 같은 내부 헬퍼 메서드를 사용하여, `chromadb` 라이브러리가 반환하는 고유한 데이터 구조를 AWorld 프레임워크의 표준 데이터 모델(`EmbeddingsResults`, `EmbeddingsResult`)로 변환합니다. 이는 프레임워크의 다른 부분이 ChromaDB의 데이터 형식에 종속되지 않도록 보장하는 중요한 과정입니다.
    -   **점수 정규화**: `search` 메서드 내에서 ChromaDB가 사용하는 코사인 거리(distance) 값을 프레임워크에서 사용하는 유사도 점수(similarity score)로 변환하는 로직을 포함하고 있습니다.

요약하자면, `chroma.py`는 AWorld 프레임워크가 ChromaDB를 벡터 저장소 백엔드로 원활하게 사용할 수 있도록 연결해주는 "어댑터" 또는 "드라이버"입니다. 이를 통해 프레임워크의 모듈성과 확장성을 유지하면서 강력한 벡터 검색 기능을 활용할 수 있습니다.
