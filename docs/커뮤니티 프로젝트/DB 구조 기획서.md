# DB 구조 기획서 (Novel Story)

`기획서.md`의 데이터 모델 개요(3장)와 페이지별 기능 명세(4장)를 기반으로 실제 테이블 구조를 설계한다. Supabase(Postgres) 기준이며, 모든 테이블명 앞에 `ns_` 접두사를 붙인다.

## 1. 설계 원칙
- **테이블명 접두사**: 모든 테이블은 `ns_`로 시작 (Novel Story 식별, 같은 Supabase 프로젝트에 다른 서비스가 들어와도 네임스페이스 충돌 방지)
- **기본 키**: `uuid default gen_random_uuid()` — Supabase Auth의 `auth.users.id`와 동일한 타입 체계 유지
- **타임스탬프**: 모든 테이블에 `created_at timestamptz default now()`, 수정이 발생하는 테이블에는 `updated_at`도 포함
- **회원 정보**: 이메일/비밀번호는 Supabase Auth의 `auth.users`가 관리한다. `ns_profiles`는 닉네임 등 서비스 전용 정보만 갖는 1:1 확장 테이블이다. (기획서 결정에 따라 전화번호 컬럼은 두지 않는다)
- **폴리모픽 대상 분리**: "게시글 또는 댓글"을 동시에 대상으로 하는 좋아요·신고는 하나의 테이블에 대상 종류를 문자열로 넣는 대신, **대상별로 테이블을 분리**한다. 실제 FK 제약과 UNIQUE 제약을 걸 수 있어 참조 무결성이 보장된다.
- **집계 캐싱**: 목록에서 자주 쓰이는 좋아요 수·댓글 수·평균 평점은 컬럼으로 캐싱하고 트리거로 동기화한다. (매 요청마다 COUNT 쿼리를 돌리지 않기 위함)

## 2. 테이블 목록

| 테이블명 | 설명 | 관련 페이지(기획서 4장) |
|---|---|---|
| `ns_profiles` | 회원 프로필(닉네임 등 앱 전용 정보) | 4.1, 4.8 |
| `ns_novels` | 소설/잡지 마스터 데이터 | 4.4, 4.7 |
| `ns_posts` | 게시글 | 4.3, 4.4, 4.5 |
| `ns_comments` | 댓글/대댓글 | 4.5, 4.6 |
| `ns_ratings` | 소설/잡지 평점 | 4.7 |
| `ns_post_likes` | 게시글 좋아요 | 4.5 |
| `ns_comment_likes` | 댓글 좋아요 | 4.6 |
| `ns_post_reports` | 게시글 신고 | 4.5, 4.9 |
| `ns_comment_reports` | 댓글 신고 | 4.6, 4.9 |

## 3. 테이블 상세 정의

### 3.1 `ns_profiles`
`auth.users`를 1:1로 확장하는 프로필 테이블. 회원가입 시 Supabase 트리거(`on auth.users insert`)로 자동 생성한다.

| 컬럼 | 타입 | 제약 | 설명 |
|---|---|---|---|
| `id` | uuid | PK, FK → `auth.users.id` | Auth 계정과 1:1 매핑 |
| `nickname` | text | not null, unique | 닉네임(가입 시 중복 확인) |
| `role` | text | not null, default `'member'`, check in (`'member'`,`'admin'`) | 관리자 페이지 접근 권한 구분 |
| `created_at` | timestamptz | not null, default now() | 가입일 |
| `updated_at` | timestamptz | not null, default now() | 프로필 수정일 |

> 회원 탈퇴는 `auth.users` 하드 삭제 대신, 계정을 비활성화하고 `nickname`을 "탈퇴한 사용자"로 치환하는 소프트 삭제를 권장한다. 작성한 글/댓글의 작성자 정보가 끊기지 않기 때문이다.

### 3.2 `ns_novels`
소설/잡지 마스터 데이터. 게시글 작성 시 검색 후 선택하며, 없으면 이 테이블에 새로 등록한다.

| 컬럼 | 타입 | 제약 | 설명 |
|---|---|---|---|
| `id` | uuid | PK, default gen_random_uuid() | |
| `media_type` | text | not null, check in (`'novel'`,`'magazine'`) | 소설/잡지 구분 |
| `title` | text | not null | 제목 |
| `author` | text | null 허용 | 작가 |
| `publisher` | text | null 허용 | 출판사/발행처 |
| `genre` | text | null 허용 | 장르 |
| `cover_image_url` | text | null 허용 | 표지 이미지 |
| `avg_rating` | numeric(2,1) | not null, default 0 | 평균 평점(캐시) |
| `rating_count` | integer | not null, default 0 | 평점 개수(캐시) |
| `created_by` | uuid | FK → `ns_profiles.id` | 최초 등록자 |
| `created_at` | timestamptz | not null, default now() | |
| `updated_at` | timestamptz | not null, default now() | |

> 제목만으로 UNIQUE 제약을 걸지 않는다(동명 소설 가능성). 중복 등록 정리는 4.9 관리자 페이지에서 수동으로 처리한다.

### 3.3 `ns_posts`
| 컬럼 | 타입 | 제약 | 설명 |
|---|---|---|---|
| `id` | uuid | PK, default gen_random_uuid() | |
| `novel_id` | uuid | not null, FK → `ns_novels.id` on delete restrict | 연결된 소설/잡지 |
| `author_id` | uuid | not null, FK → `ns_profiles.id` on delete restrict | 작성자 |
| `title` | text | not null | 게시글 제목 |
| `content` | text | not null | 본문 |
| `image_url` | text | null 허용 | 첨부 이미지 |
| `like_count` | integer | not null, default 0 | 좋아요 수(캐시) |
| `comment_count` | integer | not null, default 0 | 댓글 수(캐시) |
| `created_at` | timestamptz | not null, default now() | |
| `updated_at` | timestamptz | not null, default now() | |
| `deleted_at` | timestamptz | null 허용 | 신고 처리 등으로 인한 소프트 삭제 |

### 3.4 `ns_comments`
| 컬럼 | 타입 | 제약 | 설명 |
|---|---|---|---|
| `id` | uuid | PK, default gen_random_uuid() | |
| `post_id` | uuid | not null, FK → `ns_posts.id` on delete cascade | 게시글이 삭제되면 댓글도 함께 삭제 |
| `author_id` | uuid | not null, FK → `ns_profiles.id` on delete restrict | 작성자 |
| `parent_id` | uuid | null 허용, FK → `ns_comments.id` on delete restrict | 대댓글일 경우 원댓글 참조 |
| `depth` | smallint | not null, default 0, check in (0, 1) | 0=원댓글, 1=대댓글 — **1단계 제한을 DB 레벨에서도 강제** |
| `content` | text | not null | |
| `like_count` | integer | not null, default 0 | 좋아요 수(캐시) |
| `created_at` | timestamptz | not null, default now() | |
| `updated_at` | timestamptz | not null, default now() | |
| `deleted_at` | timestamptz | null 허용 | 소프트 삭제 — 대댓글이 있으면 "삭제된 댓글입니다"로 표시 |

> 트리거로 `parent_id`가 가리키는 댓글의 `depth`가 0인지 검증한다. `depth = 1`인 댓글에는 다시 대댓글을 달 수 없도록 막는다.

### 3.5 `ns_ratings`
| 컬럼 | 타입 | 제약 | 설명 |
|---|---|---|---|
| `id` | uuid | PK, default gen_random_uuid() | |
| `novel_id` | uuid | not null, FK → `ns_novels.id` on delete cascade | |
| `user_id` | uuid | not null, FK → `ns_profiles.id` on delete cascade | |
| `score` | smallint | not null, check between 1 and 5 | 별점 |
| `created_at` | timestamptz | not null, default now() | |
| `updated_at` | timestamptz | not null, default now() | 재평가 시 갱신 |
| — | — | **UNIQUE**(`novel_id`, `user_id`) | 유저당 소설 1개에 평점 1회 |

### 3.6 `ns_post_likes`
| 컬럼 | 타입 | 제약 | 설명 |
|---|---|---|---|
| `id` | uuid | PK, default gen_random_uuid() | |
| `post_id` | uuid | not null, FK → `ns_posts.id` on delete cascade | |
| `user_id` | uuid | not null, FK → `ns_profiles.id` on delete cascade | |
| `created_at` | timestamptz | not null, default now() | |
| — | — | **UNIQUE**(`post_id`, `user_id`) | 중복 좋아요 방지, 재클릭 시 row 삭제로 토글 |

### 3.7 `ns_comment_likes`
| 컬럼 | 타입 | 제약 | 설명 |
|---|---|---|---|
| `id` | uuid | PK, default gen_random_uuid() | |
| `comment_id` | uuid | not null, FK → `ns_comments.id` on delete cascade | |
| `user_id` | uuid | not null, FK → `ns_profiles.id` on delete cascade | |
| `created_at` | timestamptz | not null, default now() | |
| — | — | **UNIQUE**(`comment_id`, `user_id`) | 중복 좋아요 방지, 재클릭 시 row 삭제로 토글 |

### 3.8 `ns_post_reports`
| 컬럼 | 타입 | 제약 | 설명 |
|---|---|---|---|
| `id` | uuid | PK, default gen_random_uuid() | |
| `post_id` | uuid | not null, FK → `ns_posts.id` on delete cascade | |
| `reporter_id` | uuid | not null, FK → `ns_profiles.id` on delete cascade | 신고자 |
| `reason` | text | not null | 신고 사유 |
| `status` | text | not null, default `'pending'`, check in (`'pending'`,`'resolved'`,`'rejected'`) | 처리 상태 |
| `resolved_by` | uuid | null 허용, FK → `ns_profiles.id` | 처리한 관리자 |
| `resolved_at` | timestamptz | null 허용 | 처리 시각 |
| `created_at` | timestamptz | not null, default now() | |
| — | — | **UNIQUE**(`post_id`, `reporter_id`) | 동일 유저의 중복 신고 방지 |

### 3.9 `ns_comment_reports`
`ns_post_reports`와 동일한 구조에서 대상만 댓글로 바뀐다.

| 컬럼 | 타입 | 제약 | 설명 |
|---|---|---|---|
| `id` | uuid | PK, default gen_random_uuid() | |
| `comment_id` | uuid | not null, FK → `ns_comments.id` on delete cascade | |
| `reporter_id` | uuid | not null, FK → `ns_profiles.id` on delete cascade | |
| `reason` | text | not null | |
| `status` | text | not null, default `'pending'`, check in (`'pending'`,`'resolved'`,`'rejected'`) | |
| `resolved_by` | uuid | null 허용, FK → `ns_profiles.id` | |
| `resolved_at` | timestamptz | null 허용 | |
| `created_at` | timestamptz | not null, default now() | |
| — | — | **UNIQUE**(`comment_id`, `reporter_id`) | |

## 4. 관계도 (ERD 요약)

```
auth.users (Supabase 관리)
  └─ 1:1 ─ ns_profiles
              ├─ 1:N ─ ns_novels.created_by
              ├─ 1:N ─ ns_posts.author_id
              ├─ 1:N ─ ns_comments.author_id
              ├─ 1:N ─ ns_ratings.user_id
              ├─ 1:N ─ ns_post_likes.user_id
              ├─ 1:N ─ ns_comment_likes.user_id
              ├─ 1:N ─ ns_post_reports.reporter_id
              └─ 1:N ─ ns_comment_reports.reporter_id

ns_novels
  ├─ 1:N ─ ns_posts.novel_id
  └─ 1:N ─ ns_ratings.novel_id

ns_posts
  ├─ 1:N ─ ns_comments.post_id
  ├─ 1:N ─ ns_post_likes.post_id
  └─ 1:N ─ ns_post_reports.post_id

ns_comments
  ├─ 1:N ─ ns_comments.parent_id  (자기 참조, depth 0 → 1)
  ├─ 1:N ─ ns_comment_likes.comment_id
  └─ 1:N ─ ns_comment_reports.comment_id
```

## 5. 집계 컬럼 동기화 (트리거)
캐시 컬럼은 원본 테이블의 INSERT/DELETE(또는 소프트 삭제) 시점에 트리거로 갱신한다.

| 캐시 컬럼 | 갱신 시점 |
|---|---|
| `ns_posts.like_count` | `ns_post_likes` insert/delete |
| `ns_posts.comment_count` | `ns_comments` insert / `deleted_at` 설정·해제 |
| `ns_comments.like_count` | `ns_comment_likes` insert/delete |
| `ns_novels.avg_rating`, `ns_novels.rating_count` | `ns_ratings` insert/update/delete 시 재계산 |

## 6. Row Level Security(RLS) 정책 개요

| 테이블 | SELECT | INSERT | UPDATE | DELETE |
|---|---|---|---|---|
| `ns_profiles` | 전체 공개 | 가입 트리거로 자동 생성 | 본인만 | 관리자만(제재) |
| `ns_novels` | 전체 공개 | 로그인 유저 누구나 | 관리자만 | 관리자만 |
| `ns_posts` | `deleted_at is null` 조건으로 전체 공개 | 로그인 유저 누구나 | 작성자 본인 | 작성자 본인 또는 관리자 |
| `ns_comments` | `deleted_at is null` 조건으로 전체 공개 | 로그인 유저 누구나 | 작성자 본인 | 작성자 본인 또는 관리자 |
| `ns_ratings` | 전체 공개 | 본인 것만 | 본인 것만 | 본인 또는 관리자 |
| `ns_post_likes` / `ns_comment_likes` | 전체 공개 | 본인만(토글 insert) | — | 본인만(토글 delete) |
| `ns_post_reports` / `ns_comment_reports` | 관리자만(+ 본인이 접수한 신고는 본인도 조회 가능) | 로그인 유저 누구나 | 관리자만(처리) | — |

## 7. 인덱스 계획
- 모든 FK 컬럼(`novel_id`, `post_id`, `comment_id`, `author_id`, `user_id`, `reporter_id` 등)에 기본 인덱스
- `ns_posts (novel_id, created_at desc)` — 소설별 최신 글 조회
- `ns_posts (created_at desc)` — 목록 최신순 정렬
- `ns_posts (like_count desc)` — 목록 인기순 정렬
- `ns_novels (avg_rating desc)` — 목록 평점순 정렬
- `ns_comments (post_id, created_at)` — 게시글별 댓글 순서 조회

---
참고: 페이지별 기능 요구사항은 `기획서.md`, 보완 논의 배경은 `기획서_보완점.md` 참고.
