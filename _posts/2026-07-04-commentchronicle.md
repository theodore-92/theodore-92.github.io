---
layout: post
title: "댓글 연대기"
date: 2026-07-04
tags: [개발, 삽질]
---

## 블로그를 만들었다. 그런데, 댓글은?

블로그를 최초 구상하면서 댓글을 어찌할지 고민을 좀 했다.
1. 어차피 정적 페이지 묶음이고
2. 요새 누가 개발할 때 검색하냐 AI한테 물어보지
3. 내가 누군가 굳이 들어와서 읽고 댓글을 남길만한 뭔가를 생산하긴 할까?
싶어서 하지 말까 했는데…

걍 재밌으니까 구현하기로 함. 
처음엔 기본 기능인 [Disqus](https://disqus.com/) 를 사용하도록 되어있는데, 

{% include img.html name="2026-07-04-commentchronicle/disqus.png" alt="disqus comment" caption="연결 가능한 SSO 리스트들." %}

[Downdetector](https://downdetector.com/) 에서 사용해본 바 1회성으로 댓글 달기엔 너무 귀찮고 번거로워서 다른 시스템을 사용하기로 했다.
 

## Isso

대안으로 찾아낸 것이 [Isso](https://isso-comments.de/) 다. 주소로 미뤄보건데, 독일에서 작업되고 있는 프로젝트처럼 보인다. 
disqus로 구현된 프로젝트에서 마이그레이션 하기도 좋고, sqlite라 가볍고, 댓글 내 마크다운이나 메일 알림까지 지원한다고 했다. 무엇보다 무료다.

{% include img.html name="2026-07-04-commentchronicle/isso.png" alt="isso comment" caption="조금 투박하지만, 어쨌거나 작동만 하면 되잖아?" %}

처음엔 그냥 나이브하게 개인 시놀로지에 docker로 isso 서버 띄우고 떙쳐야지~ 싶었다. 

그런데… 그럼 재미가 없잖아. 굳이 블로그까지 만들어서 장난질하는데 재미를 놓치기엔 아쉬웠다. 
뭐 엄청난 기능을 할건 아니고 그냥 단순 댓글이니 공개 서버에 올리기로 했다. 
 

## CloudFlare Workers vs AWS Lambda

바로 제2의 뇌에게 비교를 하청줬다. 대화 중, 

{% include img.html name="2026-07-04-commentchronicle/systemselection.png" alt="clauderesponse" caption="鋭い指摘가 뭔데 씹덕아" %}

대화에서 일본어를 쓰지도 않았는데 일본어를 사용해서 답변해줬다. 鋭い指摘… 예 뭐시기 지적.. 날카로운 지적일 것이다. 
암튼 뭐 그렇단다. 사실 백엔드 개발자니까 익숙한 언어를 사용하려면 람다 쓰는게 맞았을 것이다. 

하지만 지금은 백엔드로 장난치기보단 js로 조금 더 조물조물하고 싶었다. 구성도 덜 복잡하고, cold start로 인한 지연이 적은 것도 맘에 들었다. 
댓글을 썼는데, '엇?' 하는 시점에서 등록되는 것 보다는 바로 등록되는게 당연히 낫다.
 

## 사양과 설계

보통 이런 부분은 직접 해본 적 없고 대부분 기획에서 넘어오다 보니… 이 부분이 가장 어려웠고, 수정도 제일 많이 했다. 

요구사항은 단순하다:
 1. 별도의 첨부파일이 없는 text-only 마크다운 문법의 댓글과 대댓글을 쓸 수 있어야 한다.
 2. 마스터(나)와 댓글 작성자는 수정과 삭제를 할 수 있어야 한다.
 3. (optional) 내가 달지 않은 댓글이 달리면 나에게 노티가 와야한다.
 4. (optional) 기본적인 수준의 스팸 방지가 있어야 한다.

#### 계층, 권한 & Misc

|  계층 |  권한 |
|---|---|
|  슈퍼유저 |  댓글 입력(노티없이), 모든 댓글의 수정/삭제 |
|  유저 |  댓글 입력, 작성 댓글 수정/삭제 |
|  익명 |  조회만 |

권한은 '비밀번호'를 통해 인증과 조건 취득 모두를 가능하도록 했다. 
HTTPS 사용할거니까 비밀번호 해싱도 다 서버로 넘겨버렸다. 굳이 양쪽에서 할 필요도 없고.

4번의 옵션을 위해 유저 계층에 IP 기반의 Rate-limit을 적용했다. 1분당 3번. 
Limit이 걸린 경우, 같은 Cloudflare의 Turnstile 이용해서 추가 인증하도록 했다.

3번의 옵션을 위해서는 [Resend](https://resend.com/) 를 붙이기로 했다. 일 100회 제한이 있지만, 1년에 100회도 안나갈거니 상관없다. 

#### DB

SQLite on Cloudflare D1
```sql
-- 댓글 본체
CREATE TABLE comments (
  id          INTEGER PRIMARY KEY AUTOINCREMENT,
  post_slug   TEXT NOT NULL,
  parent_id   INTEGER REFERENCES comments(id),  -- 대댓글이면 부모 id, 아니면 NULL
  author      TEXT NOT NULL,        -- 작성 후 불변
  content     TEXT NOT NULL,
  pw_hash     TEXT NOT NULL,
  is_owner    INTEGER DEFAULT 0,    -- 슈퍼유저 작성 여부. 추후 하이라이트에서도 사용.
  is_deleted  INTEGER DEFAULT 0,    -- soft delete용 플래그
  ip_hash     TEXT NOT NULL,        -- rate limit 및 추적용, 원본 IP는 저장하지 않음
  updated_at  DATETIME DEFAULT CURRENT_TIMESTAMP 
);

-- 수정/삭제 이력 (웹에 노출되지 않는 DB 내부 기록)
CREATE TABLE comment_history (
  id           INTEGER PRIMARY KEY AUTOINCREMENT,
  comment_id   INTEGER NOT NULL REFERENCES comments(id),
  action       TEXT NOT NULL CHECK(action IN ('edit', 'delete')),
  prev_content TEXT NOT NULL,       -- 변경/삭제 "전" 내용
  ip_hash      TEXT NOT NULL,       
  changed_at   DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- 스팸 방지용 Rate Limit
CREATE TABLE rate_limit (
  ip_hash    TEXT NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

```

#### APIs

|  메서드 | 기능 | 경로                         |
|---|---|----------------------------|
|  GET | R | /comments?slug=_post_slug_ |
|  POST | C | /comments                  |
|  PATCH | U | /comments/:id              |
|  DELETE | D | /comments/:id              |

 * Read를 제외한 모든 작업에 pw_hash를 요구할 것.

#### 화면

 이 부분은 claude design을 처음으로 한 번 써봤다. 목업도 머리속으로만 모호하게 그려놨다. 덕분에 수정하느라 시간 많이 먹었지만...

{% include img.html name="2026-07-04-commentchronicle/design.png" alt="claudedesignresult" caption="생성형 피그마 쓰는 느낌!" %}

 엄청나게 편했다. 의외로 특수 모델이 아니라 일반 모델을 사용하여 결과를 도출하는게 신기했다. 어쨌거나 코드뭉치라서 그런걸까.
 전체 페이지인지 단일 모듈인지 정하고 3개의 시안을 만들도록 주문했다. 
 
 이래저래 요구사항을 주고 얼추 대부분의 케이스를 커버할 수 있는 상태가 되었고, 내 커스텀을 몇가지 넣어서 완성했다. 아쉽게도 글을 작성하는 당시에는 React만 code와 연결하여 다이렉트로 수정이 가능한 것 처럼 보여서, hand-off 정보를 담은 패키지를 만들어서 다시 클로드 코드에게 먹였다.

#### Comment Backend

이 부분도 기본 구조만 내가 잡고 절찬리에 사용중인 클로드를 사용했다.
4건의 기본 요구사항과 쿼리뭉치, 구현 방식 (랭글러를 사용한 worker 세팅 등)..을 뭉뜽그려서 백엔드를 개발할 claude code에 전달할 md 문서를 하나 만들고, 이를 통해 개발하였다.

{% include img.html name="2026-07-04-commentchronicle/spec.png" alt="spec.md on ide" caption="SPEC.md의 일부. 굳이 사족을 달아야 했니?" %}

랭글러를 통해 배포하고, SECRET을 주입한 뒤 약간의 로컬 개발 이후 테스트했다. 

{% include img.html name="2026-07-04-commentchronicle/local.png" alt="local test" caption="" %}

이래저래 테스트했다. turnstile이 개입해버려서 로컬 환경을 따로 잡아주는 사고가 있었지만, 큰 문제 없이 개발된 듯. 마이너한 CSS 수정과 XSS 대응 등 잡아주고 라이브 테스트.