# 🎬 영화 정보 관리 시스템 — Movie Info System

> **TMDB API + MySQL 연동 기반 Java 데스크톱 영화 정보 관리 프로그램**
> 팀 프로젝트 (3인) · 2025년 · 2학년 1학기

---

## 📌 프로젝트 소개

TMDB(The Movie Database) Open API를 활용하여 영화를 검색하고, 즐겨찾기 등록·리뷰 작성·상세 정보 조회가 가능한 **Java Swing 기반 데스크톱 영화 정보 관리 시스템**이다.

단순한 API 조회를 넘어, **DAO 패턴을 통한 계층 분리 설계**와 **JDBC 기반 DB 연동**에 집중하여 구현하였다.

---

## 👥 팀 역할 분담

| 이름 | 역할 |
| --- | --- |
| 김수빈 | GUI 시각적 구성 (화면 배치 및 레이아웃) |
| **지현우** | **DB 연동 (DAO 계층 전체) · GUI 이벤트 처리 · 비즈니스 로직 구현** |
| 최우진 | TMDB API 연동 및 DTO 정의 |

> GUI는 김수빈이 초기 구성하였고, 그 안의 이벤트 처리(검색, 즐겨찾기, 리뷰 저장 등)와 DB 호출, 데이터 연동은 **지현우**가 구현하였다.

---

## 💪 사용 기술

| 분류 | 기술 |
| --- | --- |
| Language | Java 8+ |
| GUI | Java Swing |
| Database | MySQL 8.0 |
| DB 연동 | JDBC (Raw, ORM 미사용) |
| 외부 API | TMDB Open API v3 |
| 개발 도구 | Eclipse IDE |
| 형상 관리 | Git / GitHub |

---

## 🏗️ 아키텍처 및 계층 구조

```
┌──────────────────────────────────┐
│         GUI Layer (Swing)        │
│  MainFrame / MovieDetailPanel    │
│  ReviewPanel / FavoritesPanel    │
└──────────────┬───────────────────┘
               │ 이벤트 처리 (ActionListener)
               ▼
┌──────────────────────────────────┐
│       Business Logic Layer       │
│  검색·즐겨찾기·리뷰 처리 로직       │
└──────┬───────────────┬───────────┘
       │               │
       ▼               ▼
┌─────────────┐  ┌──────────────────┐
│  DAO Layer  │  │   API Layer      │
│  MovieDAO   │  │  MovieAPI        │
│  ReviewDAO  │  │  ReviewAPI       │
│ FavoriteDAO │  │  TMDBConfig      │
│  DBUtil     │  └──────────────────┘
└──────┬──────┘         │
       │                ▼
       ▼         TMDB Open API
   MySQL DB      (HTTP GET 요청)
```

### 계층별 역할

- **GUI Layer**: Swing 컴포넌트 배치, 화면 렌더링
- **Business Logic Layer**: 이벤트 처리, 데이터 가공, 흐름 제어 ← **본인 담당 핵심**
- **DAO Layer**: DB CRUD 전담 (SQL 직접 작성) ← **본인 담당 핵심**
- **API Layer**: TMDB API HTTP 통신, JSON 파싱, DTO 변환

---

## 📂 프로젝트 구조

```
movie_info_system/
├── src/
│   └── movie_info_system/
│       ├── gui/                  # GUI 컴포넌트
│       │   ├── MainFrame.java        (이벤트 처리 - 지현우)
│       │   ├── MovieDetailPanel.java
│       │   ├── ReviewPanel.java
│       │   └── FavoritesPanel.java
│       ├── api/                  # TMDB API 연동
│       │   ├── MovieAPI.java
│       │   ├── ReviewAPI.java
│       │   └── TMDBConfig.java
│       ├── dto/                  # 데이터 전송 객체
│       │   ├── MovieDTO.java
│       │   ├── ReviewDTO.java
│       │   └── FavoriteDTO.java
│       └── dao/                  # DB 연동 (지현우 담당)
│           ├── MovieDAO.java
│           ├── ReviewDAO.java
│           ├── FavoriteDAO.java
│           └── DBUtil.java
├── resources/assets/
│   └── logo_tmdb.png
├── create_tables.sql
├── movie_db_dump.sql
└── pom.xml
```

---

## ✅ 주요 기능

| 기능 | 설명 | 구현 방식 |
| --- | --- | --- |
| 🔍 영화 검색 | 키워드 기반 영화 검색 | TMDB API + DB 로컬 데이터 통합 조회 |
| 📋 상세 정보 | 포스터, 개요, 평점, 출시일 표시 | TMDB API 응답 → DTO 변환 → GUI 렌더링 |
| ⭐ 즐겨찾기 | 즐겨찾기 등록/삭제 및 목록 조회 | FavoriteDAO → MySQL CRUD |
| 📝 리뷰 관리 | TMDB 리뷰 + 직접 작성 리뷰 통합 표시 | ReviewDAO DB 저장 + API 조회 병합 |

---

## 🗃️ DB 설계 (담당 구현)

```sql
-- 즐겨찾기 테이블
CREATE TABLE favorites (
    id          INT PRIMARY KEY AUTO_INCREMENT,
    movie_id    INT NOT NULL,
    title       VARCHAR(255),
    poster_path VARCHAR(255),
    created_at  DATETIME DEFAULT NOW()
);

-- 리뷰 테이블
CREATE TABLE reviews (
    id         INT PRIMARY KEY AUTO_INCREMENT,
    movie_id   INT NOT NULL,
    content    TEXT,
    rating     FLOAT,
    created_at DATETIME DEFAULT NOW()
);
```

### DBUtil — DB 연결 관리

```java
public class DBUtil {
    public static Connection getConnection() throws SQLException {
        Properties props = new Properties();
        props.load(new FileInputStream("db.properties"));
        return DriverManager.getConnection(
            props.getProperty("db.url"),
            props.getProperty("db.user"),
            props.getProperty("db.password")
        );
    }
}
```

- `db.properties`로 연결 정보 외부 분리 (하드코딩 방지)
- `try-with-resources`로 연결 자동 닫기 처리

---

## 🔄 데이터 흐름 (협업 구조)

| 항목 | 제공자 → 수신자 | 설명 |
| --- | --- | --- |
| `MovieDTO` | API(최우진) → GUI/DB(지현우) | JSON 응답 → DTO로 변환해 전달 |
| DAO 호출 입력 | GUI(지현우) → DB | 영화 ID, 리뷰 내용 등 전달 |
| `.properties` 설정 | 팀원 간 공유 | 개인 환경에 맞게 복사/수정 |

---

## ▶ 실행 방법

### 1. API Key 설정

```
apikey.properties.example → apikey.properties 로 복사
TMDB_API_KEY=your_tmdb_api_key
```

### 2. DB 설정

```
db.properties.example → db.properties 로 복사
db.url=jdbc:mysql://localhost:3306/movie_db
db.user=root
db.password=your_password
```

### 3. DB 생성

```bash
# 방법 1: 빈 테이블만 생성
mysql -u root -p < create_tables.sql

# 방법 2: 예제 데이터 포함 전체 복원
mysql -u root -p < movie_db_dump.sql
```

### 4. Eclipse에서 실행

- `resources/` 폴더 → `Build Path > Use as Source Folder` 설정
- `Main.java` 실행

---

## 💡 핵심 학습 포인트

- **DAO 패턴**: 비즈니스 로직과 DB 접근 코드를 분리하는 계층 설계 방법 학습
- **JDBC Raw 연동**: ORM 없이 직접 SQL 작성, `PreparedStatement`로 SQL Injection 방지
- **외부 API 연동**: TMDB JSON 응답을 파싱해 DTO로 변환하는 데이터 흐름 이해
- **팀 협업**: 인터페이스 기준 역할 분담 후 통합하는 방식 경험
- **설정 외부화**: `.properties` 파일로 환경별 설정 분리 (보안 관리)

---

## ⚠️ 주의사항

- `apikey.properties`, `db.properties`는 Git에 업로드 금지 (`.gitignore` 포함)
- JDBC 드라이버는 classpath에 반드시 포함
- TMDB API 호출은 일일 요청 제한이 있으므로 과도한 호출 주의

---

> 본 프로젝트는 교육 및 학습 목적의 프로젝트입니다.
