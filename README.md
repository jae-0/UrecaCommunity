<div align="center">
  <h1>🤫 유레카 익명 커뮤니티 (U+RECA Anonymous)</h1>
  <p><b>LG U+ 유레카 3기 미니 프로젝트 (백엔드 비대면 윤재영)</b><br>
  에브리타임, 클라썸을 모티브로 한 수강생 전용 데스크탑 익명 커뮤니티 애플리케이션입니다.</p>
</div>

<br>

## 💡 프로젝트 목적 (Purpose)

**"질문에 대한 심리적 부담을 낮추고, 자유로운 소통을 돕습니다."**
유레카 수강생들이 수업 시간에 모르는 내용을 부담 없이 질의응답하고 소통할 수 있도록 **익명 기능**을 지원합니다. 게시판과 댓글 중심의 직관적인 사용성을 유지하면서도, 검색 및 내 활동 모아보기 등 필수적인 편의 기능을 제공하는 데 집중했습니다.

---

## 🎯 주요 기능 (Key Features)

* **🔐 회원 관리 및 인증**
    * 이메일과 비밀번호를 통한 로그인/회원가입
    * 수강생 전용 공간 유지를 위한 **초대코드 검증** 시스템
* **📝 게시판 및 게시글 (Q&A / 자유)**
    * 게시판별 목록 조회 및 키워드 기반 게시글 검색
    * 게시글 작성, 수정, 삭제 기능 지원
* **💬 체계적인 익명 댓글 시스템**
    * 게시글 내 **동일 사용자 익명번호 일관 유지** (예: 익명1, 익명2)
    * 직관적인 트리 구조의 댓글 및 대댓글(답글) 지원
* **🗑️ 스마트 댓글 삭제 정책**
    * 자식(대댓글)이 있는 경우: 소프트 삭제 처리 (`[삭제된 댓글입니다]`)
    * 자식(대댓글)이 없는 경우: DB에서 즉시 하드 삭제 처리
* **📂 내 활동 모아보기**
    * 내가 작성한 글과 댓글을 단 글을 한눈에 페이징하여 조회
* **🌙 다크 테마 UI 적용**
    * LG Red(`#A50034`)를 포인트 컬러로 활용한 눈이 편안한 다크 테마 디자인

---

## 🛠 기술 스택 (Tech Stack)

| Category | Technology |
| :--- | :--- |
| **Language** | Java 17+ |
| **UI Framework** | Java Swing |
| **Database** | MySQL 8.x |
| **Data Access** | 순수 JDBC (PreparedStatement), DAO 패턴 |
| **Build Tool** | None (순수 Java 프로젝트, IntelliJ IDEA 환경) |

---

## 🔄 화면 흐름 (Screen Flow)

```text
Launcher (앱 진입점)
   │
   ▼
 Login ──────▶ Signup (회원가입)
   │
   ▼
 Boards (게시판 목록)
   ├── [게시판 클릭] ──▶ Board (게시글 목록 / 검색)
   │                       ├── [글 클릭] ──▶ PostDetail (상세 내용 및 댓글 트리)
   │                       └── [글쓰기]  ──▶ ComposePost (작성 다이얼로그)
   │
   └── [내 활동 클릭] ──▶ ActivityBoard (내가 쓴 글 / 댓글 단 글 조회)
```

---

## 🕵️‍♂️ 익명 처리 방식 (Anonymous System)

같은 게시글 안에서 동일한 사용자가 여러 번 댓글을 작성하더라도, **항상 같은 익명 번호를 부여받아** 대화의 맥락이 끊기지 않도록 설계되었습니다.

> **예시 화면**
> **[익명1]** 이 부분 어떻게 해결하셨나요?
>  └─ **[익명2]** 환경 변수 설정 확인해 보세요!
>      └─ **[익명1]** 감사합니다! 해결했습니다. *(동일인물 = 동일번호)*

**💡 구현 로직**
1. `post_anon_map` 테이블을 활용하여 `(게시글 ID, 사용자 ID)` 조합으로 익명 인덱스를 매핑 및 관리합니다.
2. 댓글 작성 시 트랜잭션과 행 락(`FOR UPDATE`)을 사용하여 동시성 이슈를 제어합니다.
3. 해당 게시글에 처음 댓글을 작성하는 신규 사용자일 경우, `posts.anon_counter`를 원자적으로 증가시켜 새로운 번호를 부여합니다.

---

## 🗄 데이터 모델 (Data Model)

```text
users
  - id (PK), nickname, email, pw_hash, role

boards
  - id (PK), name, description

posts
  - id (PK), board_id (FK), author_id (FK)
  - title, content, is_anonymous
  - anon_counter, created_at, updated_at

comments
  - id (PK), post_id (FK), user_id (FK), parent_id (FK, 자기참조)
  - content, is_anonymous, anon_index, created_at

post_anon_map
  - PK (post_id, user_id)
  - anon_index (부여된 익명 번호)
```

---

## 🎨 UI 테마 (UI Theme)

| 역할 (Role) | 색상 (Color) | HEX Code |
| :--- | :---: | :--- |
| **배경** | 짙은 차콜 | `#0D0D0F` |
| **카드 배경** | 다크 차콜 | `#151518` |
| **주요 텍스트** | 밝은 흰색 | `#F2F2F5` |
| **보조 텍스트** | 회색 | `#B3B6BB` |
| **강조색 (Point)**| LG Red | `#A50034` |
| **구분선** | 어두운 회색 | `#27282C` |

---

## 📂 프로젝트 구조 (Project Directory)

```text
src/
├── app/
│   └── Launcher.java         # 애플리케이션 진입점 (로그인 화면 호출)
├── auth/
│   └── Session.java          # 싱글톤 패턴 기반 로그인 세션 관리
├── dao/
│   ├── UserDao.java          # 회원가입, 로그인 검증
│   ├── BoardDao.java         # 게시판 목록 및 정보 조회
│   ├── PostDao.java          # 게시글 CRUD 및 검색 로직
│   ├── CommentDao.java       # 댓글/대댓글 CRUD 및 익명 로직 처리
│   └── MyActivityDao.java    # 내 활동(작성 글/댓글 단 글) 조회
├── dto/
│   ├── LoginDto.java         # 로그인 사용자 정보 전송 객체
│   └── CommentDto.java       # 댓글 데이터 전송 객체
├── jdbc/
│   └── DBManager.java        # DB 연결 팩토리 및 자원 해제 유틸
└── ui/
    ├── Colors.java            # 공통 색상(HEX) 상수 관리
    ├── Scrollbars.java        # 다크 테마 커스텀 스크롤바
    ├── Login.java / Signup.java # 인증 화면
    ├── Boards.java / Board.java # 게시판 및 게시글 목록 (페이징/검색)
    ├── ComposePost.java       # 게시글 작성 팝업 다이얼로그
    ├── PostDetail.java        # 게시글 상세 열람 및 댓글 트리 UI
    └── ActivityBoard.java     # 내 활동 모아보기 화면
```

---

## ⚙️ 실행 방법 (Getting Started)

**1. 데이터베이스 설정**
MySQL에 데이터베이스를 생성하고, `jdbc/DBManager.java` 파일 내의 접속 정보를 본인의 로컬 환경에 맞게 수정합니다.
```java
static String url  = "jdbc:mysql://localhost:3306/ureka_community";
static String user = "root";
static String pwd  = "your_password";
```

**2. 애플리케이션 실행**
IDE(IntelliJ 등)에서 `src/app/Launcher.java` 파일의 `main()` 메서드를 실행합니다.

---

## 📄 산출물 문서 (Documents)

* `docs/miniproject_윤재영_ERD.png` — 데이터베이스 ERD
* `docs/MiniProject_1_윤재영_시연.pdf` — 기능 시연 및 발표 자료
