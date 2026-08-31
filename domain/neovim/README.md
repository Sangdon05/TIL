---
id: "neovim-guide-001"
title: "Neovim (v0.11+) 설정 및 개발 환경 가이드"
status: "active"
domain: "neovim"
tags:
  - "neovim"
  - "lua"
  - "lsp"
  - "ide"
  - "lazy"
aliases:
  - "Neovim Guide"
  - "Nvim 설정 가이드"
created_at: "2026-08-29"
updated_at: "2026-08-29"
summary: "Neovim 0.11+ 네이티브 API 기반 모던 IDE 개발 환경 설정(init.lua) 구조, 플러그인 구성 및 단축키 가이드"
related_code:
  - "~/.config/nvim/init.lua"
---

# Neovim (v0.11+) 설정 및 사용 가이드

Neovim 0.11+ 및 0.12+ 버전에 맞춘 모던 개발자 환경 설정(`init.lua`) 구조와 주요 플러그인, 단축키 사용법을 정리한 문서입니다.

---

## 1. 설정 구조 개요

설정 파일은 단일 Lua 파일(`~/.config/nvim/init.lua`) 기반으로 구성되어 있으며, 고속 플러그인 매니저인 `lazy.nvim`을 통해 관리됩니다.

```
~/.config/nvim/
├── init.lua          # 기본 옵션, 키맵, 플러그인 및 LSP 통합 설정
└── lazy-lock.json    # 설치된 플러그인 버전 잠금 파일
```

### Neovim 0.11+ 주요 변경 및 최적화 포인트
- **`vim.uv` 도입**: 기존 `vim.loop` 대신 최신 비동기 시스템 API인 `vim.uv` 사용.
- **네이티브 LSP 활성화**: `require("lspconfig")` 프레임워크 래퍼 대신 Nvim 0.11+ 표준 내장 API(`vim.lsp.config`, `vim.lsp.enable`) 적용.
- **진단 점프 함수 현대화**: Deprecated된 `vim.diagnostic.goto_prev/next` 대신 통합 `vim.diagnostic.jump({ count = ±1, float = true })` 사용.
- **`LspAttach` 자동 명령어**: LSP 서버가 버퍼에 로드될 때만 버퍼 로컬 단축키를 바인딩하여 충돌 및 오버헤드 방지.

---

## 2. 플러그인 구성 (Plugin Stack)

| 플러그인 | 역할 | 설명 |
| :--- | :--- | :--- |
| **`folke/lazy.nvim`** | 패키지 매니저 | 고성능 비동기 플러그인 관리 및 부트스트랩 |
| **`catppuccin/nvim`** | 테마 (Colorscheme) | 눈이 편안한 Mocha 다크 테마 |
| **`nvim-lualine/lualine.nvim`** | 상태 표시줄 | 하단 모드, 파일 정보, Git 브랜치, LSP 진단 상태 표시 |
| **`lewis6991/gitsigns.nvim`** | Git 통합 | 줄 단위 추가/수정/삭제 기호 표시 |
| **`nvim-tree/nvim-tree.lua`** | 파일 탐색기 | 사이드바 파일 트리 뷰 |
| **`nvim-telescope/telescope.nvim`** | 퍼지 파인더 | 파일 검색, 프로젝트 텍스트(Live Grep) 검색, 버퍼 탐색 |
| **`nvim-treesitter/nvim-treesitter`** | 구문 분석 | 고속 정밀 코드 하이라이팅 및 문법 기반 인덴트 |
| **`windwp/nvim-autopairs`** | 괄호 자동 완성 | 괄호(`()`, `{}`, `[]`) 및 따옴표 자동 닫기 |
| **`williamboman/mason.nvim`** | 도구 설치 관리자 | 언어 서버(LSP), 린터, 포맷터 바이너리 자동 관리 |
| **`hrsh7th/nvim-cmp`** | 자동 완성 엔진 | LSP, 버퍼 단어, 경로 기반 팝업 자동완성 및 Snippet 지원 |
| **`neovim/nvim-lspconfig`** | 언어 서버 설정 | Mason과 연계된 LSP 설정 및 키 바인딩 |

---

## 3. 핵심 단축키 가이드 (Cheatsheet)

> **리더 키(`<Leader>`)**: `Space` (스페이스바)

### 3.1 일반 편집 및 창 관리
| 단축키 | 모드 | 설명 |
| :--- | :---: | :--- |
| `<C-s>` | Normal | 파일 저장 (`:w`) |
| `<Esc>` | Normal | 검색 하이라이트 끄기 (`:nohlsearch`) |
| `<C-h>` | Normal | 왼쪽 분할 창으로 이동 |
| `<C-j>` | Normal | 아래쪽 분할 창으로 이동 |
| `<C-k>` | Normal | 위쪽 분할 창으로 이동 |
| `<C-l>` | Normal | 오른쪽 분할 창으로 이동 |

### 3.2 파일 및 프로젝트 탐색
| 단축키 | 설명 |
| :--- | :--- |
| `<Leader>e` | 파일 탐색기(Nvim-Tree) 열기 / 닫기 |
| `<Leader>ff` | 프로젝트 내 파일 이름 검색 (Telescope) |
| `<Leader>fg` | 프로젝트 전체 텍스트 검색 (Live Grep) |
| `<Leader>fb` | 열린 버퍼(Buffer) 목록 검색 |
| `<Leader>fh` | Neovim 도움말(Help) 검색 |

### 3.3 코드 인텔리전스 & LSP
| 단축키 | 설명 |
| :--- | :--- |
| `gd` | 함수/변수 정의(Definition)로 이동 |
| `gD` | 함수/변수 선언(Declaration)으로 이동 |
| `gi` | 인터페이스 구현체(Implementation)로 이동 |
| `gr` | 심볼 참조 목록(References) 탐색 |
| `K` | 함수 시그니처 / 타입 정보 팝업 (Hover) |
| `<Leader>rn` | 변수/함수명 일괄 변경 (Rename) |
| `<Leader>ca` | 코드 액션 및 빠른 수정 (Code Action) |
| `<Leader>f` | 현재 파일 코드 포맷팅 (Format) |
| `]d` | 다음 에러/경고로 이동 |
| `[d` | 이전 에러/경고로 이동 |
| `<Leader>d` | 현재 줄의 에러/경고 상세 메시지 플로팅 창 띄우기 |

### 3.4 자동 완성 (입력 모드)
| 단축키 | 설명 |
| :--- | :--- |
| `<Tab>` | 다음 자동완성 항목 선택 (스니펫 확장 가능 시 스니펫 점프) |
| `<S-Tab>` | 이전 자동완성 항목 선택 |
| `<CR>` (엔터) | 현재 선택된 자동완성 항목 적용 |
| `<C-Space>` | 자동 완성 수동 호출 |
| `<C-e>` | 자동 완성 팝업 닫기 |

---

## 4. 확장 및 유지보수 방법

### 새 언어 LSP 추가하기
1. `init.lua`의 `mason-lspconfig` 및 `servers` 목록에 추가할 언어 서버명을 등록합니다.
   ```lua
   -- 예: Go(gopls), Rust(rust_analyzer) 추가
   local servers = { "lua_ls", "ts_ls", "pyright", "gopls", "rust_analyzer" }
   ```
2. Neovim 실행 후 `:Mason` 명령어를 통해 GUI 상에서 설치 상태를 확인하거나 추가 도구를 설치할 수 있습니다.

### 플러그인 업데이트 및 관리
- `:Lazy` : 플러그인 관리 대시보드 열기
- `:Lazy sync` : 플러그인 설치/업데이트/동기화
- `:TSUpdate` : Treesitter 파서 업데이트
- `:checkhealth` : Neovim 및 LSP 환경 진단
