<a id="readme-top"></a>

<div align="center">

# TIL Blog RAG

내 블로그 글을 검색하고, 내가 쓴 글을 근거로 답변하는  
**self-hosted RAG 검색 도구**

![Python](https://img.shields.io/badge/Python-3.11%2B-3776AB?style=for-the-badge&logo=python&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Early%20Development-yellow?style=for-the-badge)

</div>

<details>
  <summary>목차</summary>
  <ol>
    <li><a href="#프로젝트-소개">프로젝트 소개</a></li>
    <li><a href="#주요-기능">주요 기능</a></li>
    <li><a href="#동작-방식">동작 방식</a></li>
    <li><a href="#지원-플랫폼">지원 플랫폼</a></li>
    <li><a href="#시작하기">시작하기</a></li>
    <li><a href="#설정">설정</a></li>
    <li><a href="#사용법">사용법</a></li>
    <li><a href="#설계-원칙">설계 원칙</a></li>
    <li><a href="#프로젝트-구조">프로젝트 구조</a></li>
    <li><a href="#로드맵">로드맵</a></li>
    <li><a href="#기여하기">기여하기</a></li>
    <li><a href="#라이선스">라이선스</a></li>
    <li><a href="#acknowledgments">Acknowledgments</a></li>
  </ol>
</details>

## 프로젝트 소개

TIL과 기술 블로그 글이 쌓일수록 "이거 예전에 정리했던 것 같은데" 하는 순간이 늘어납니다. 키워드 검색은 정확한 단어를 기억해야만 찾을 수 있습니다.

**TIL Blog RAG**는 블로그 글을 인덱싱해서 다음 두 가지를 제공합니다.

- **검색**: 표현이 달라도 의미가 비슷한 글과 문단을 찾아 줍니다.
- **질문**: 찾은 문단만 근거로 답변하고, 원문 링크를 출처로 붙입니다.

설정 파일 하나로 내 블로그 구조에 맞출 수 있도록 만들고 있습니다. 데이터는 내 컴퓨터나 내가 고른 서버에만 저장되고, LLM API 키도 각자 자기 것을 씁니다.

> ⚠️ 현재 초기 개발 단계입니다. 아래 기능은 목표이며, 구현 상태는 [로드맵](#로드맵)에서 확인할 수 있습니다.

<p align="right">(<a href="#readme-top">맨 위로</a>)</p>

## 주요 기능

- **의미 검색**: "예약할 때 재고 충돌 막는 법"으로 검색해도 `낙관적 락` 글을 찾습니다.
- **키워드 검색**: `version_id_col` 같은 기술명, 함수명, 에러 메시지를 정확히 찾습니다.
- **Hybrid Search**: 벡터 검색과 BM25를 RRF로 결합합니다.
- **출처 기반 답변**: 검색된 문단에 없는 내용은 "모르겠다"고 답합니다.
- **메타데이터 필터링**: 태그, 카테고리, 날짜로 검색 범위를 좁힙니다.
- **증분 인덱싱**: 새로 쓰거나 수정한 글만 다시 임베딩해서 비용을 아낍니다.
- **블로그 위젯**: 블로그에 스크립트 한 줄로 검색창을 붙입니다.

<p align="right">(<a href="#readme-top">맨 위로</a>)</p>

## 동작 방식

```text
[인덱싱]  블로그 글 → 본문·메타데이터 분리 → 헤딩 기준 청킹 → 임베딩 → 벡터 DB

[검색]    질문 → 벡터 검색 + BM25 → RRF 결합 → Re-ranking → 관련 문단 + 원문 URL

[질문]    검색 결과 → 문맥 주입 프롬프트 → LLM 답변 + 출처
```

블로그와는 이렇게 연결됩니다. 블로그는 검색창만 담당하고, 검색은 내가 실행하는 API 서버가 담당합니다.

```text
내 블로그 (검색 위젯) ──요청──▶ TIL Blog RAG API ──▶ 벡터 DB
```

<p align="right">(<a href="#readme-top">맨 위로</a>)</p>

## 지원 플랫폼

TIL Blog RAG는 두 부분만 플랫폼마다 다르고, 청킹부터 검색까지는 모든 플랫폼이 같은 파이프라인을 씁니다.

```text
[콘텐츠 소스] → 청킹 → 임베딩 → 검색 → [검색창 연결]
 플랫폼별 로더          공통 파이프라인       플랫폼별 연결 방식
```

### 콘텐츠 소스: 글을 어디서 가져오나

| 소스 | 대상 플랫폼 | 가져오는 방법 | 상태 |
| --- | --- | --- | --- |
| `markdown` | GitHub Pages (Jekyll, Hugo), Obsidian, TIL 저장소 | 로컬 폴더의 `.md` 파일과 front matter | 개발 중 |
| `notion` | Notion | Notion 공식 API로 페이지를 가져와 Markdown으로 변환 | 예정 |
| `rss` | Tistory, velog 등 | RSS 피드의 HTML에서 본문만 추출 | 예정 |
| `html_backup` | Tistory | 블로그 백업 파일에서 전체 글 추출 | 검토 중 |

- **`markdown`** 은 front matter가 없는 글도 처리합니다. 이때는 파일명과 첫 헤딩에서 날짜와 제목을 추출합니다.
- **`rss`** 는 보통 최근 글 10~50개만 제공합니다. 처음 전체 인덱싱보다는 새 글을 받아오는 용도로 적합합니다.
- **velog**는 공식 API가 없어서 RSS만 지원합니다. 비공식 API는 사용하지 않습니다.

새 소스는 `ContentSource` 인터페이스만 구현하면 추가할 수 있습니다.

```python
class ContentSource(Protocol):
    def load(self) -> list[BlogDocument]:
        """플랫폼의 글을 본문 + 메타데이터 + 원문 URL 형태로 반환한다."""
```

### 검색창 연결: 블로그에 검색을 어떻게 붙이나

| 방식 | 사용 가능한 플랫폼 | 방법 | 상태 |
| --- | --- | --- | --- |
| 스크립트 위젯 | GitHub Pages, Tistory (스킨 편집) | 레이아웃에 `<script>` 한 줄 추가 | 예정 |
| 검색 전용 페이지 | 모든 플랫폼 | 별도 검색 페이지를 띄우고 링크 | 예정 |
| embed | Notion | 검색 전용 페이지를 `/embed` 블록으로 삽입 | 예정 |

Notion과 velog는 사용자가 JavaScript를 넣을 수 없어서 **검색 전용 페이지** 방식을 사용합니다.

### 플랫폼별 요약

| 플랫폼 | 콘텐츠 소스 | 검색창 연결 |
| --- | --- | --- |
| GitHub Pages | `markdown` | 스크립트 위젯 |
| Tistory | `rss` 또는 `html_backup` | 스크립트 위젯 |
| Notion | `notion` | embed |
| velog | `rss` (최근 글만) | 검색 전용 페이지 링크 |

<p align="right">(<a href="#readme-top">맨 위로</a>)</p>

## 시작하기

### 사전 요구사항

- Python 3.11 이상
- [uv](https://docs.astral.sh/uv/) (권장) 또는 pip
- OpenAI API 키 (임베딩·답변 생성용)

### 설치

```bash
git clone https://github.com/mjlee0914/til-blog-rag.git
cd til-blog-rag

uv sync
cp .env.example .env   # .env에 OPENAI_API_KEY 입력
```

> `.env`와 API 키는 절대 커밋하지 마세요.

<p align="right">(<a href="#readme-top">맨 위로</a>)</p>

## 설정

> 예정: 설정 형식은 개발하면서 바뀔 수 있습니다.

프로젝트 루트의 `til-rag.yaml`에서 내 블로그 구조를 지정합니다. 코드를 고치지 않고 이 파일만 바꿔서 다른 블로그에 쓸 수 있도록 하는 것이 목표입니다.

```yaml
source:
  type: markdown                    # markdown | notion | rss
  path: ../my-blog/_posts           # markdown일 때: 글이 있는 폴더
  include: ["**/*.md"]
  exclude: ["drafts/**"]
  # notion일 때: database_id: "..."   (NOTION_API_KEY는 .env에)
  # rss일 때:    feed_url: "https://myblog.tistory.com/rss"

url:
  base: https://username.github.io
  pattern: "/blog/{slug}"           # 검색 결과에서 원문으로 연결할 URL 규칙

metadata:
  date_from: front_matter           # front_matter | filename
  title_from: front_matter          # front_matter | first_heading

chunking:
  strategy: heading                 # heading | recursive
  chunk_size: 800
  chunk_overlap: 100

embedding:
  provider: openai
  model: text-embedding-3-small

vector_store:
  provider: chroma                  # chroma | pgvector
  path: ./data/chroma
```

<p align="right">(<a href="#readme-top">맨 위로</a>)</p>

## 사용법

> 예정: 아래 명령은 구현 목표입니다.

```bash
til-rag index                 # 글을 인덱싱 (바뀐 글만 다시 처리)
til-rag search "낙관적 락"     # 터미널에서 바로 검색
til-rag ask "예약 동시성은 어떻게 처리했지?"
til-rag serve                 # 블로그에서 쓸 검색 API 서버 실행
```

API 엔드포인트:

```text
POST /api/search   # 관련 글과 문단 검색 (임베딩만 사용, 저렴)
POST /api/ask      # 검색 결과를 근거로 답변 생성 (LLM 호출)
GET  /api/health   # 서버 상태 확인
```

블로그에 검색창 붙이기:

```html
<!-- GitHub Pages, Tistory: 레이아웃이나 스킨에 추가 -->
<div id="til-rag-search"></div>
<script src="https://<내-API-서버>/widget.js" data-api="https://<내-API-서버>"></script>
```

Notion이나 velog처럼 스크립트를 넣을 수 없는 플랫폼에서는 `https://<내-API-서버>/search` 검색 전용 페이지를 링크하거나, Notion의 `/embed` 블록으로 넣습니다.

<p align="right">(<a href="#readme-top">맨 위로</a>)</p>

## 설계 원칙

- **설정 우선**: 경로, URL 규칙, 모델명 같은 블로그별 정보는 코드가 아니라 `til-rag.yaml`에 둡니다.
- **각자 자기 키 사용**: 이 프로젝트는 API 키를 대신 보관하지 않습니다. 비용과 데이터는 사용자가 관리합니다.
- **교체 가능한 소스와 저장소**: 콘텐츠 소스와 벡터 저장소를 인터페이스 뒤에 두어, 플랫폼이나 DB(Chroma → pgvector)가 바뀌어도 구현체만 바꿔 끼울 수 있게 합니다.
- **검색과 생성 분리**: 저렴한 `/api/search`만 공개하고, 비용이 드는 `/api/ask`는 선택적으로 켤 수 있습니다.
- **평가로 검증**: 검색 로직을 바꿀 때마다 평가셋으로 품질 저하 여부를 확인합니다.

<p align="right">(<a href="#readme-top">맨 위로</a>)</p>

## 프로젝트 구조

```text
til-blog-rag/
├── src/til_rag/
│   ├── config.py       # til-rag.yaml 로딩
│   ├── sources/        # 콘텐츠 소스 (markdown, notion, rss)
│   ├── chunkers/       # 청킹 전략
│   ├── embeddings/     # 임베딩 provider
│   ├── stores/         # 벡터 저장소 인터페이스와 구현체
│   ├── retrievers/     # 벡터·BM25·Hybrid 검색
│   ├── chains/         # 검색 → 답변 생성
│   ├── api/            # FastAPI 서버
│   └── cli.py          # til-rag 명령
├── widget/             # 블로그용 검색 위젯과 검색 전용 페이지
├── examples/           # Jekyll·Hugo·TIL 저장소 설정 예시
├── evaluation/         # 평가셋과 평가 코드
├── tests/
│   └── fixtures/posts/ # 테스트용 샘플 Markdown
├── til-rag.example.yaml
├── .env.example
├── pyproject.toml
├── LICENSE
└── README.md
```

<p align="right">(<a href="#readme-top">맨 위로</a>)</p>

## 로드맵

### Milestone 1: 문서 로딩 ⬅ 진행 중

- [ ] `.md` 파일 탐색 (하위 폴더 포함, 제외 패턴 지원)
- [ ] YAML front matter와 본문 분리
- [ ] front matter가 없거나 잘못된 문서 처리
- [ ] 제목·날짜·태그·URL 메타데이터 정규화
- [ ] 로더 테스트 (한글 인코딩, 잘못된 YAML 포함)

**완료 기준:** 형식이 섞인 Markdown 폴더를 넣어도 일관된 문서 목록이 반환된다.

### Milestone 2: 인덱싱

- [ ] 헤딩을 보존하는 청킹과 전략별 비교
- [ ] 임베딩 생성과 벡터 저장소 저장
- [ ] 파일 해시 기반 증분 인덱싱
- [ ] `til-rag index` 명령

**완료 기준:** 명령 하나로 인덱싱되고, 수정한 글만 다시 처리된다.

### Milestone 3: 검색과 답변 MVP

- [ ] Top-k와 score threshold 기반 벡터 검색
- [ ] 태그·카테고리·날짜 필터링
- [ ] 출처를 붙인 답변 생성
- [ ] `/api/search`, `/api/ask` API

**완료 기준:** 질문하면 관련 문단, 답변, 원문 링크가 반환된다.

### Milestone 4: 검색 품질 개선

- [ ] BM25 키워드 검색
- [ ] Hybrid Search (RRF)와 Re-ranking
- [ ] 질의 유형별 검색 결과 비교

**완료 기준:** 기술명 검색과 자연어 질문 모두 벡터 검색만 쓸 때보다 결과가 좋다.

### Milestone 5: 평가

- [ ] 실제 블로그 기반 평가셋 구축
- [ ] RAGAS 검색·생성 지표 측정
- [ ] 변경 전후 회귀 검증

**완료 기준:** 검색·답변 품질을 수치와 실패 사례로 설명할 수 있다.

### Milestone 6: 누구나 쓸 수 있게 만들기

- [ ] `til-rag.yaml` 설정 파일과 예시 (Jekyll, Hugo, TIL 저장소)
- [ ] `til-rag` CLI
- [ ] `ContentSource` 인터페이스와 `notion`, `rss` 소스
- [ ] 블로그용 검색 위젯
- [ ] 검색 전용 페이지 (Notion embed, velog 링크용)
- [ ] `docker compose up`으로 API 서버 실행
- [ ] GitHub Action으로 글 push 시 자동 인덱싱
- [ ] pgvector 저장소 지원
- [ ] 설치·설정 문서

**완료 기준:** 다른 사람이 코드를 고치지 않고 설정만으로 자기 블로그에 붙일 수 있다.

<p align="right">(<a href="#readme-top">맨 위로</a>)</p>

## 기여하기

아직 초기 단계라 구조가 자주 바뀝니다. 버그 제보, 아이디어, "내 블로그 플랫폼도 지원해 주세요" 같은 요청은 [Issues](https://github.com/mjlee0914/til-blog-rag/issues)에 남겨 주세요.

1. 저장소를 Fork합니다.
2. 브랜치를 만듭니다. (`git checkout -b feat/notion-source`)
3. 변경 사항을 커밋합니다. (`git commit -m "feat: add notion source"`)
4. 브랜치에 Push합니다. (`git push origin feat/notion-source`)
5. Pull Request를 엽니다.

<p align="right">(<a href="#readme-top">맨 위로</a>)</p>

## 라이선스

MIT License로 배포합니다. 자세한 내용은 [`LICENSE`](LICENSE)를 참고하세요.

<p align="right">(<a href="#readme-top">맨 위로</a>)</p>

## Acknowledgments

- README 구조: [othneildrew/Best-README-Template](https://github.com/othneildrew/Best-README-Template)
- [LangChain](https://python.langchain.com/)
- [Chroma](https://www.trychroma.com/)
- [FastAPI](https://fastapi.tiangolo.com/)
- [RAGAS](https://docs.ragas.io/)

<p align="right">(<a href="#readme-top">맨 위로</a>)</p>
