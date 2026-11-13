<a id="readme-top"></a>

<div align="center">

# TIL Blog RAG

내가 작성한 TIL과 프로젝트 포스팅을 검색하고,  
검색된 문서를 근거로 답변하는 기술 블로그 전용 RAG 프로젝트

![Python](https://img.shields.io/badge/Python-3.11%2B-3776AB?style=for-the-badge&logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-Planned-009688?style=for-the-badge&logo=fastapi&logoColor=white)
![Status](https://img.shields.io/badge/Status-Document%20Loading-yellow?style=for-the-badge)

</div>

<details>
  <summary>목차</summary>
  <ol>
    <li><a href="#프로젝트-소개">프로젝트 소개</a></li>
    <li><a href="#주요-기능">주요 기능</a></li>
    <li><a href="#rag-파이프라인">RAG 파이프라인</a></li>
    <li><a href="#기술-스택">기술 스택</a></li>
    <li>
      <a href="#시작하기">시작하기</a>
      <ul>
        <li><a href="#사전-요구사항">사전 요구사항</a></li>
        <li><a href="#설치">설치</a></li>
      </ul>
    </li>
    <li><a href="#사용법">사용법</a></li>
    <li><a href="#프로젝트-구조">프로젝트 구조</a></li>
    <li><a href="#로드맵">로드맵</a></li>
    <li><a href="#테스트">테스트</a></li>
    <li><a href="#라이선스">라이선스</a></li>
    <li><a href="#acknowledgments">Acknowledgments</a></li>
  </ol>
</details>

## 프로젝트 소개

**TIL Blog RAG**는 블로그의 TIL 및 프로젝트 Markdown 포스팅을 지식 데이터로 활용하는 검색·질의응답 서비스입니다.

단순히 키워드가 포함된 글만 찾는 대신, 문서의 의미와 메타데이터를 함께 사용해 관련 문단을 검색합니다. 이후 검색된 문맥을 LLM에 전달하여 근거가 있는 답변과 원문 링크를 제공합니다.

현재는 RAG 인덱싱 파이프라인의 첫 단계인 **Markdown 문서 로딩**을 구현하는 단계입니다.

### 프로젝트 목표

- 블로그 Markdown의 YAML front matter와 본문을 안정적으로 분리합니다.
- 헤딩 구조를 보존하면서 검색에 적합한 단위로 문서를 청킹합니다.
- 벡터 검색과 BM25를 결합한 Hybrid Search를 구현합니다.
- 답변마다 실제 블로그 포스팅과 관련 문단을 출처로 제공합니다.
- 변경된 문서만 다시 처리하는 증분 인덱싱을 구현합니다.
- 평가셋과 RAGAS를 활용해 변경 전후의 검색·생성 품질을 검증합니다.

<p align="right">(<a href="#readme-top">맨 위로</a>)</p>

## 주요 기능

> 아래 항목은 프로젝트의 목표 기능이며, 구현 상태는 <a href="#로드맵">로드맵</a>에서 확인할 수 있습니다.

- **의미 검색**: 표현이 달라도 의미가 비슷한 TIL과 프로젝트 글 검색
- **키워드 검색**: 기술명, 함수명, 에러 메시지 등 정확한 용어 검색
- **Hybrid Search**: 벡터 검색과 BM25 결과를 결합해 검색 품질 향상
- **출처 기반 답변**: 검색된 블로그 문단만 근거로 답변 생성
- **메타데이터 필터링**: 카테고리, 태그, 날짜 등으로 검색 범위 제한
- **원문 연결**: 결과에서 실제 블로그 포스팅으로 이동
- **증분 갱신**: 추가되거나 수정된 Markdown만 다시 인덱싱

<p align="right">(<a href="#readme-top">맨 위로</a>)</p>

## RAG 파이프라인

```text
블로그 Markdown
    ↓
문서 로딩 및 front matter 파싱
    ↓
헤딩 기반 텍스트 청킹
    ↓
임베딩 생성
    ↓
벡터 DB 저장
    ↓
벡터 검색 + BM25 검색
    ↓
결과 결합 및 Re-ranking
    ↓
문맥 주입과 답변 생성
    ↓
답변 + 관련 문단 + 원문 URL
```

<p align="right">(<a href="#readme-top">맨 위로</a>)</p>

## 기술 스택

현재 학습 및 구현 진행에 따라 확정할 예정입니다.

| 구분 | 기술 | 상태 |
| --- | --- | --- |
| Language | Python 3.11+ | 예정 |
| Document parsing | python-frontmatter, Pydantic | 2장 1강에서 구현 |
| API | FastAPI | 3장에서 구현 예정 |
| Embedding | 미정 | 2장 4강에서 결정 |
| Vector DB | 미정 | 2장 5강에서 결정 |
| Keyword search | BM25 | 5장에서 구현 예정 |
| Evaluation | RAGAS | 7장에서 구현 예정 |

<p align="right">(<a href="#readme-top">맨 위로</a>)</p>

## 시작하기

현재는 프로젝트 초기 단계이므로 아래 명령은 기본 개발 환경을 준비하는 용도입니다. 실제 의존성이 추가되면 설치 방법도 함께 갱신합니다.

### 사전 요구사항

- Python 3.11 이상
- Git

### 설치

1. 저장소를 복제합니다.

   ```bash
   git clone <repository-url>
   cd til-blog-rag
   ```

2. 가상환경을 만들고 활성화합니다.

   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   ```

3. 문서 로딩에 필요한 패키지를 설치합니다.

   ```bash
   pip install python-frontmatter pydantic pytest
   ```

4. 환경변수 예시 파일이 추가된 이후 로컬 설정 파일을 생성합니다.

   ```bash
   cp .env.example .env
   ```

> API 키와 `.env` 파일은 Git에 커밋하지 않습니다.

<p align="right">(<a href="#readme-top">맨 위로</a>)</p>

## 사용법

문서 로더가 구현되면 샘플 Markdown 디렉터리를 다음과 같이 읽습니다.

```python
documents = load_markdown_directory("data/sample_posts")

for document in documents:
    print(document.metadata.get("title"))
    print(document.content[:100])
```

인덱싱 파이프라인 통합 후에는 단일 명령으로 문서를 인덱싱할 예정입니다.

```bash
python -m scripts.index_posts --posts-path data/sample_posts
```

API 구현 후 제공할 엔드포인트입니다.

```text
POST /api/search  # 관련 문서와 문단 검색
POST /api/ask     # 검색 문맥을 사용한 답변 생성
GET  /api/health  # 서비스 상태 확인
```

<p align="right">(<a href="#readme-top">맨 위로</a>)</p>

## 프로젝트 구조

```text
til-blog-rag/
├── app/
│   ├── loaders/        # Markdown 문서 로딩
│   ├── chunkers/       # 텍스트 청킹
│   ├── embeddings/     # 임베딩 생성
│   ├── stores/         # 벡터 DB 저장
│   ├── retrievers/     # 검색 전략
│   ├── chains/         # 검색·생성 체인
│   └── api/            # FastAPI 엔드포인트
├── data/
│   ├── sample_posts/   # 테스트용 Markdown
│   └── indexes/        # 로컬 인덱스
├── evaluation/         # 평가셋과 평가 코드
├── scripts/            # 인덱싱 실행 스크립트
├── tests/
├── .env.example
├── .gitignore
├── pyproject.toml
└── README.md
```

<p align="right">(<a href="#readme-top">맨 위로</a>)</p>

## 로드맵

### Milestone 1 — 블로그 문서 수집

- [ ] 블로그의 TIL·프로젝트 Markdown 파일 탐색
- [ ] YAML front matter와 본문 분리
- [ ] 제목, 날짜, 태그, 카테고리, URL 메타데이터 정규화
- [ ] 손상된 문서와 누락된 메타데이터 예외 처리
- [ ] 샘플 포스팅을 사용한 문서 로더 테스트

**완료 기준:** Markdown 디렉터리를 입력하면 검색에 사용할 문서 목록이 일관된 형식으로 반환된다.

### Milestone 2 — 검색용 인덱스 구축

- [ ] Markdown 헤딩을 보존하는 청킹 구현
- [ ] 청크 크기와 overlap 비교 실험
- [ ] 청크 임베딩 생성
- [ ] 본문, 벡터, 메타데이터를 벡터 DB에 저장
- [ ] 파일 해시를 이용한 증분 인덱싱

**완료 기준:** 명령 한 번으로 블로그 글을 인덱싱하고, 수정된 글만 다시 처리할 수 있다.

### Milestone 3 — 관련 글 검색 MVP

- [ ] 사용자 질의를 임베딩으로 변환
- [ ] Top-k 및 score threshold 기반 벡터 검색
- [ ] 카테고리·태그·날짜 필터링
- [ ] 관련 문단, 점수, 원문 URL 반환
- [ ] 검색 API 구현

**완료 기준:** 질문을 입력하면 관련 블로그 문단과 원문 링크가 검색 결과로 반환된다.

### Milestone 4 — 출처 기반 RAG 답변

- [ ] 검색 문맥 주입 프롬프트 설계
- [ ] 검색 결과를 이용한 답변 생성
- [ ] 근거가 부족한 질문에 대한 응답 규칙 적용
- [ ] 답변과 출처를 함께 반환하는 API 구현

**완료 기준:** 검색된 블로그 내용만 근거로 답변하고, 사용한 포스팅을 출처로 표시한다.

### Milestone 5 — 검색 품질 고도화

- [ ] BM25 키워드 검색 추가
- [ ] 벡터 검색과 BM25를 결합한 Hybrid Search
- [ ] RRF 가중치 실험
- [ ] Re-ranking 및 문맥 압축 적용
- [ ] 질의 유형별 검색 전략 비교

**완료 기준:** 기술명 검색과 자연어 질문 모두에서 기본 벡터 검색보다 나은 결과를 확인한다.

### Milestone 6 — 평가 및 회귀 검증

- [ ] 실제 블로그 내용을 기반으로 평가 질문셋 구축
- [ ] 검색 성공·실패 사례 기록
- [ ] RAGAS 검색 및 생성 지표 측정
- [ ] 환각과 출처 오류 분석
- [ ] 파이프라인 변경 전후 회귀 검증

**완료 기준:** 검색·답변 품질을 수치와 실패 사례로 설명하고, 변경 후 품질 저하 여부를 확인할 수 있다.

### Milestone 7 — 블로그 연동 및 배포

- [ ] 블로그 검색 UI 구현
- [ ] 검색 모드와 RAG 질문 모드 제공
- [ ] 배포 환경의 검색 API 연결
- [ ] 새 포스팅 인덱싱 자동화
- [ ] 운영 환경에서 검색·출처 링크 검증

**완료 기준:** 실제 블로그 방문자가 검색과 질문 기능을 사용할 수 있고, 새 글이 자동으로 반영된다.

<p align="right">(<a href="#readme-top">맨 위로</a>)</p>

## 테스트

테스트가 추가된 이후 다음 명령으로 실행합니다.

```bash
pytest
```

초기 문서 로더에서는 다음 항목을 검증합니다.

- 정상 Markdown 파일 로딩
- front matter와 본문 분리
- 여러 파일과 하위 디렉터리 탐색
- Markdown이 아닌 파일 제외
- 한글 인코딩 보존
- 누락되거나 잘못된 메타데이터 처리

<p align="right">(<a href="#readme-top">맨 위로</a>)</p>

## 라이선스

프로젝트 라이선스는 아직 결정하지 않았습니다. 라이선스가 추가되기 전까지는 프로젝트 코드의 사용·복제·배포 조건이 명시적으로 허가되지 않은 상태입니다.

<p align="right">(<a href="#readme-top">맨 위로</a>)</p>

## Acknowledgments

- README structure adapted from [othneildrew/Best-README-Template](https://github.com/othneildrew/Best-README-Template)
- [FastAPI](https://fastapi.tiangolo.com/)
- [RAGAS](https://docs.ragas.io/)

<p align="right">(<a href="#readme-top">맨 위로</a>)</p>
