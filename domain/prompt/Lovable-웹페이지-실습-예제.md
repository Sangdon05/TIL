너는 Lovable.dev, v0, Cursor 등 AI 코딩 빌더에 최적화된 Web Application PRD(제품 요구사항 문서) 작성 전문가야.
내가 전달하는 [앱 아이디어]를 바탕으로, Lovable이 한 번에 정확한 코드를 생성할 수 있도록 완벽하게 구조화된 ""Lovable 전용 웹 앱 기획서""를 작성해 줘.

### [기본 기술 스택 전제]
- Frontend: React (Vite), Tailwind CSS, shadcn/ui, Lucide Icons
- Backend/Database: Supabase (Auth, Postgres DB, Storage)

---

### [출력 형식 및 작성 가이드]

1. **앱 개요 (Overview)**
   - 앱 이름, 목적, 주요 타겟 유저, 핵심 가치 제안

2. **페이지 구조 및 라우팅 (Page Architecture)**
   - 경로별 페이지 구성 (예: `/dashboard`, `/settings`)
   - 각 페이지에 배치될 주요 UI 컴포넌트 목록 (shadcn/ui 기반)

3. **데이터베이스 스키마 설계 (Supabase DB Schema)**
   - 필요한 테이블 명, 컬럼 명, 데이터 타입, 제약 조건 (PK, FK, Unique)
   - RLS(Row Level Security) 정책 요약

4. **페이지별 UI/UX 및 기능 상세**
   - 각 화면의 레이아웃 및 사용자 흐름 (User Flow)
   - 각 화면별 4가지 상태 정의 (Initial/Empty, Loading, Success, Error)

5. **Lovable 단계별 입력 프롬프트 (Step-by-Step Prompts)**
   *Lovable에 순차적으로 입력할 수 있는 실전 프롬프트를 작성해 줘.*
   - **Phase 1 (MVP/기초 레이아웃 & Supabase 연동):** 전체 레이아웃, 더미 데이터 기반 UI, DB 스키마 생성용 프롬프트
   - **Phase 2 (핵심 CRUD 및 데이터 로직):** 실제 데이터 C.R.U.D 처리 및 상태 관리 구현 프롬프트
   - **Phase 3 (디테일 및 예외 처리):** 예외 처리, 로딩 스피너, 토스트 알림, 반응형 모바일 최적화 프롬프트

---

[앱 아이디어]:
(여기에 만들고자 하는 앱 아이디어를 자유롭게 적으세요. 예: 개인 사용자를 위한 구독 서비스 관리 및 지출 분석 대시보드)"
"""=== TaskFlow (칸반 보드) Lovable 단계별 프롬프트 모음 ===

---
[Phase 1: 전체 UI 레이아웃 및 더미 데이터 기반 칸반 보드 구축]
---
React, Tailwind CSS, Lucide Icons, shadcn/ui 컴포넌트를 사용하여 개인 및 팀을 위한 모던 칸반 보드 웹 애플리케이션 ""TaskFlow(태스크플로우)""를 만들어 주세요. 모든 UI 텍스트와 레이블은 한국어로 작성해 주세요.

[레이아웃 구조]
- 상단 네비게이션 바: 앱 로고(""TaskFlow""), 보드 이동 드롭다운, 프로필 아바타, ""+ 새 보드"" 버튼 포함.
- 좌측 사이드바: 내 보드 목록 바로가기, 설정 메뉴 포함.

[페이지 구현 (더미 데이터 우선 사용)]
1. `/boards (보드 목록)`:
   - 보드 카드가 그리드 형태로 나열됨. 각 카드는 보드 제목, 설명, 색상 태그, 생성일 포함.
   - ""+ 새 보드 생성"" 다이얼로그 모달 포함 (제목, 설명, 대표 색상 선택).
2. `/board/:boardId (칸반 보드 상세)`:
   - 상단 필터바: 검색창, 우선순위 필터(높음/보통/낮음), ""+ 컬럼 추가"" 버튼.
   - 가로 스크롤 가능한 칸반 영역: 기본 컬럼 3개 (""할 일"", ""진행 중"", ""완료"").
   - 컬럼 카드 구성: 카드 제목, 우선순위 배지(높음: 빨강, 보통: 노랑, 낮음: 파랑), 마감일 배지, 체크리스트 현황(예: 1/3).
   - 태스크 상세 모달 (`Dialog`): 태스크 제목, 설명, 우선순위 변경, 마감일 달력 선택(`Popover` + `Calendar`), 체크리스트(Subtasks) 추가 및 체크박스 토글 기능.
3. 드래그 앤 드롭 구현 준비: `@hello-pangea/dnd` 또는 `@dnd-kit`을 사용하여 컬럼 간 태스크 카드 이동 시각 효과 적용.


---
[Phase 2: Supabase 연동, 인증(Auth) 및 데이터 CRUD 로직 구현]
---
TaskFlow 애플리케이션에 Supabase를 연동하여 회원가입/로그인 및 데이터 영구 저장을 구현해 주세요. 모든 UI는 한국어로 유지해 주세요.

[작업 내용]
1. Supabase 인증 활성화: `/login` 페이지에서 이메일/비밀번호 로그인 및 회원가입 구현.
2. Supabase 데이터베이스 스키마 구성:
   - `profiles` (id, email, display_name, avatar_url)
   - `boards` (id, user_id, title, description, color, created_at, updated_at)
   - `columns` (id, board_id, title, position, created_at)
   - `tasks` (id, column_id, board_id, title, description, priority, due_date, position, created_at, updated_at)
   - `subtasks` (id, task_id, title, is_completed, position)
3. RLS(Row Level Security) 정책 적용: 유저가 자신의 보드 및 속한 컬럼/태스크만 C.R.U.D 처리 가능하도록 구현 (`auth.uid() = user_id`).
4. 실제 데이터 C.R.U.D 연동:
   - 보드 생성, 컬럼 추가/수정/삭제, 카드 추가/수정/삭제.
   - 태스크 카드를 다른 컬럼으로 드래그 앤 드롭 시 Supabase `tasks` 테이블의 `column_id` 및 `position` 컬럼이 즉시 업데이트되도록 구현.
   - 태스크 상세 모달 내 체크리스트 항목 추가, 삭제, 완료 토글 처리.


---
[Phase 3: 예외 처리, 디테일 Polish, 토스트 알림 및 반응형 최적화]
---
TaskFlow 애플리케이션의 사용자 경험(UX)을 완성하고 예외 처리 및 반응형 스타일을 적용해 주세요.

[요구사항]
1. UX 및 로딩 상태:
   - 보드 및 칸반 카드 로딩 시 shadcn `Skeleton` 로더 표시.
   - 컬럼이나 카드가 없는 경우 ""카드를 추가해 보세요"" 안내 문구와 깔끔한 Empty State 표시.
2. 토스트 알림 연동:
   - shadcn `Toast` / `Sonner` 연동 (예: ""새 카드가 생성되었습니다"", ""카드가 이동되었습니다"", ""삭제 완료"").
3. 반응형 최적화:
   - 모바일 기기(768px 미만)에서는 칸반 컬럼이 세로 탭 스크롤 또는 스와이프 형태 카드 뷰로 전환.
   - 모바일 햄버거 메뉴를 통한 보드 목록 사이드바 전환.
4. 애니메이션 및 터치 드래그 지원:
   - 태스크 드래그 시 부드러운 호버 및 그림자 효과 적용. 모바일 터치 기반 드래그 앤 드롭 지원.