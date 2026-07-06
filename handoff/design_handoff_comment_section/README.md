# Handoff: Jekyll Hydeout 댓글 시스템 (Comment Section)

## Overview
Jekyll 블로그(Hydeout 테마) 포스트 하단에 들어갈 댓글 섹션 목업. 관리자 댓글, 일반 댓글, 대댓글(1단계), soft-delete된 댓글, 댓글 작성/수정/삭제 인터랙션, rate-limit 시 노출되는 Cloudflare Turnstile 상태까지 하나의 화면에서 확인할 수 있도록 구성했다.

## About the Design Files
이 번들의 `Comment Section.dc.html`은 **디자인 레퍼런스(정적 HTML 목업)**이며, 그대로 복사해서 쓰는 프로덕션 코드가 아니다. 실제 구현 시에는 이 HTML이 보여주는 레이아웃/상태/인터랙션을 그대로 재현하되, 대상 코드베이스(Jekyll 사이트 + 정적 JS, 또는 별도 프레임워크)의 기존 패턴과 라이브러리를 사용해 새로 작성해야 한다. React 컴포넌트가 아니라 순수 HTML/inline style로 작성되어 있으므로, 프레임워크가 있다면 그 구조에 맞게, 없다면 Jekyll 사이트에 맞는 vanilla JS/템플릿(예: Liquid include)으로 옮기는 것을 권장한다.

## Fidelity
**Hi-fi**: 색상, 타이포그래피, 간격, 상태별 UI가 최종안으로 확정됨. 실제 구현 시 아래 값들을 그대로 사용할 것.

## Screens / Views
단일 화면 — 포스트 본문 하단에 삽입되는 댓글 섹션 하나.

### 레이아웃
- 전체 wrapper: `max-width: 560px`, 가운데 정렬, 배경 `#f7f5f0`, 바깥 패딩 `40px 24px`
- 상단에 댓글 개수 헤더 (`댓글 N개`, 13px/700)
- 댓글 리스트 → 댓글 작성 폼(2개 상태 예시) → Turnstile 인증 블록 순서로 세로 배치, 각 블록 사이 `margin-bottom/margin-top: 12~18px`

### 컴포넌트

**1. 최상위 댓글 카드**
- 배경 `#fff`, `border: 1px solid rgba(0,0,0,.09)`, `border-radius: 8px`, `padding: 16px 18px`
- 관리자 댓글은 강조: `border: 1px solid rgba(38,139,210,.35)` + 좌측 `border-left: 4px solid #268bd2`
- 헤더 행: 작성자명(700/14px) + (관리자면) `ADMIN` 배지(흰 글씨, 배경 `#268bd2`, `border-radius:10px`, `padding:2px 9px`, 10.5px/700) + 작성일시(`#999`, 11.5px) — `display:flex; gap:8px; align-items:baseline`
- 본문: 14px, `line-height:1.65`
- 액션 버튼(수정/삭제): `display:flex; gap:8px`, 각 버튼 `background:#eef5fb; color:#268bd2; border:none; border-radius:14px; padding:4px 12px; font-size:11.5px; font-weight:600; white-space:nowrap` (⚠ nowrap 필수 — 없으면 "삭제"가 "삭"/"제"로 줄바꿈됨)

**2. 대댓글 (1단계만 존재)**
- 부모 카드 안쪽에 `margin-left:14px`로 들여쓰기, 배경 `#fbfaf7`, `border:1px solid rgba(0,0,0,.06)`, `border-radius:8px`, `padding:12px 14px`
- 폰트 사이즈만 약간 작음(13.5px 본문, 13.5px 이름) 외 구조는 최상위 댓글과 동일 — 수정/삭제 버튼도 동일하게 노출
- 대댓글에는 답글 작성 UI가 없음(2단계 depth 제한)

**3. 인라인 수정 상태** (예: 박지훈 댓글)
- 카드 border를 `#268bd2`로 강조, 상단에 "수정 중" 라벨(11px, `#268bd2`, 700)
- 본문 자리에 `<textarea>`로 교체: `border:1px solid #268bd2; border-radius:6px; padding:8px 10px; font-size:13.5px; min-height:52px`
- 하단 우측 정렬(`justify-content:flex-end`)로: 비밀번호 확인 input(`width:110px`) → 저장 버튼(파란 배경) → 취소 버튼(텍스트만, `white-space:nowrap` 필수)

**4. 삭제 확인 상태** (예: 정하늘 댓글)
- 카드 border를 경고색 `#d64545`로 강조, 상단에 "삭제 확인" 라벨(11px, `#d64545`, 700)
- 본문 아래 경고문구: "삭제하면 되돌릴 수 없습니다. 비밀번호를 입력해 삭제를 확인해주세요." (12px, `#767676`)
- 하단 우측 정렬: 비밀번호 확인 input → 삭제 버튼(배경 `#d64545`, 흰 글씨) → 취소 버튼

**5. 삭제된 댓글 (soft delete)**
- 배경 `#efefec`, `border:1px dashed rgba(0,0,0,.15)`, `border-radius:8px`, `padding:14px 18px`
- 내용 대신 플레이스홀더 텍스트만 표시: "삭제된 댓글입니다." (13px, `#9a9a9a`, italic)
- 그 아래 대댓글이 있다면 정상적으로 계속 표시됨 (부모가 삭제되어도 자식 댓글은 유지)

**6. 댓글 작성 폼** (2가지 렌더링 모드 상태를 나란히 제시)
- 카드: 배경 `#fff`, `border:1px solid rgba(0,0,0,.09)`, `border-radius:8px`, `padding:18px`
- 상단: "댓글 작성" 타이틀(13px/700) + 마크다운 모드 토글(세그먼트 컨트롤: "실시간 미리보기" / "제출 후 렌더링", 활성 상태 배경 `#268bd2` 흰 글씨, `border-radius:14px`)
- 이름 input: `width:100%`, `border:1px solid rgba(0,0,0,.15)`, `border-radius:6px`, `padding:8px 10px`, 13px
- **실시간 미리보기 모드**: 2단 그리드(`grid-template-columns:1fr 1fr; gap:10px`) — 좌측 textarea(마크다운 입력), 우측 렌더링된 미리보기 박스(배경 `#fbfaf7`, 점선 대신 실선 border)
- **제출 후 렌더링 모드**: textarea 1단만 (`width:100%`), 아래 안내문구("※ 미리보기 없이 텍스트만 입력 — 등록 후에 마크다운이 렌더링됩니다.", 11px `#a3a3a3`)
- textarea placeholder(두 모드 공통): `**굵게**, _기울임_, `코드`, [링크](https://example.com) 처럼 마크다운을 사용할 수 있어요.` — 마크다운 문법 힌트를 placeholder로 제공, 별도 힌트 텍스트는 없음
- 하단 액션 행: `justify-content:flex-end; gap:8px; margin-top:14px` — 비밀번호 input(`type=password`, `width:110px`) 바로 옆에 "댓글 등록" 버튼(배경 `#268bd2`, 흰 글씨, `border-radius:16px`, `padding:8px 20px`, 13px/700). **비밀번호 필드는 항상 제출 버튼 바로 왼쪽에 위치.**

**7. Cloudflare Turnstile 인증 상태** (rate limit 초과 시)
- 경고 배경 박스: `background:#fff8ec; border:1px solid #f0d9a8; border-radius:8px; padding:14px 16px`
- 안내문구: "요청이 많아 추가 인증이 필요합니다." (11.5px, `#8a6d1f`)
- 그 아래 Turnstile 위젯 자리 표시: 흰 배경 박스(`border:1px solid rgba(0,0,0,.15)`, `border-radius:6px`, `width:250px`) 안에 체크박스 자리(20×20 회색 border) + "사람인지 확인해주세요" 텍스트 + 우측에 "CLOUDFLARE" 워터마크(9px, `#999`)
- 이 상태는 항상 표시되는 것이 아니라, rate limit 초과 시에만 폼 아래 조건부로 나타나는 것으로 설계됨

## Interactions & Behavior
- **수정 클릭** → 해당 댓글이 인라인 편집 상태(#3)로 전환, 본문이 textarea로 바뀌고 비밀번호 확인 필드 노출
- **삭제 클릭** → 해당 댓글이 삭제 확인 상태(#4)로 전환, 경고문구 + 비밀번호 확인 필드 노출 (즉시 삭제 없음 — 반드시 확인 단계 거침)
- **저장/삭제 확정** → 비밀번호 검증 (자신의 댓글 비밀번호 또는 슈퍼유저 비밀번호) → 통과 시 반영, 실패 시 에러 표시(에러 상태는 이번 목업에 미포함 — 구현 시 추가 필요)
- **취소** → 원래 표시 상태로 롤백, 입력값 폐기
- **마크다운 모드 토글** → 실시간 미리보기 ↔ 제출 후 렌더링 전환. 기본값은 실시간 미리보기
- **수정/삭제 버튼 노출 조건**: 자신이 작성한 댓글에만 노출 (비밀번호로 작성자 본인 확인). 슈퍼유저는 모든 댓글에 노출되지만 별도 비밀번호로 인증
- **Rate limit 초과 시** → 댓글 작성 폼 아래 Turnstile 인증 블록이 나타나고, 인증 완료 전까지 등록 버튼은 비활성 처리 권장 (비활성 스타일은 이번 목업에 미포함)

## State Management
- 댓글별 상태: `normal | editing | confirming_delete | deleted`
- `is_owner` (boolean) — 관리자 여부, ADMIN 배지 노출 여부 결정
- `is_deleted` (boolean, soft delete) — true면 플레이스홀더로 렌더링, 자식 대댓글은 별도 유지
- `parent_id` — 있으면 대댓글(들여쓰기), 대댓글의 대댓글은 금지(최대 depth 2)
- 폼 상태: `markdown_mode: 'live' | 'after_submit'` (기본 `live`)
- `rate_limited: boolean` → true면 Turnstile 블록 노출
- 시간 표시는 항상 절대시간 `YYYY-MM-DD HH24:MI` 형식 (상대시간 사용 안 함)

## Design Tokens
- 링크/포인트 컬러: `#268bd2`
- 경고/삭제 컬러: `#d64545`
- 텍스트: 본문 `#2b2b2b`, 보조 텍스트 `#767676` / `#999` / `#a3a3a3`
- 배경: 페이지 `#f7f5f0`, 카드 `#fff`, 대댓글/미리보기 박스 `#fbfaf7`, 삭제됨 박스 `#efefec`, Turnstile 경고 박스 `#fff8ec`
- 폰트: `-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif` (Hydeout 테마의 시스템 산세리프 기조 유지)
- 폰트 크기: 이름 14px(최상위)/13.5px(대댓글), 본문 14px/13.5px, 메타정보 11.5px, 버튼 11.5~13px
- Border radius: 카드 8px, 필드 6px, pill 버튼 14~16px
- 버튼/필드 간 gap: 8px (같은 행), 블록 간 margin: 12~18px

## Assets
프로필 이미지 없음 (요구사항에 따라 의도적으로 제외). 별도 아이콘/이미지 자산 없음 — Turnstile 워터마크는 텍스트로만 표시(실제 구현 시 Cloudflare 공식 위젯으로 교체).

## Files
- `Comment Section.dc.html` — 전체 목업 (단일 파일, inline style)
- `screenshot.png` — 전체 화면 스크린샷 (참고용)
