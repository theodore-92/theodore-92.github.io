# 블로그 시작 가이드

## 1. 저장소 이름 변경
이 폴더 전체를 `사용자명.github.io` 라는 이름의 **public** 저장소로 푸시하세요.
(예: 사용자명이 `gildong`이면 저장소 이름은 `gildong.github.io`)

## 2. 초기 설정값 수정
`_config.yml`에서 아래 항목을 본인 정보로 바꾸세요.
- `title`, `description`
- `url` (https://사용자명.github.io)
- `author.name`

## 3. 로컬에서 미리보기 (선택)

```bash
bundle install                  # install Ruby gems
bundle exec jekyll build        # build to _site/
```

```bash
bundle exec jekyll serve --future  # local preview at http://localhost:4000
```

http://localhost:4000 에서 확인 가능합니다.

## 4. GitHub Pages 배포 설정
저장소의 Settings > Pages 에서 Source를 `Deploy from a branch`로 두고
브랜치를 `main`, 폴더를 `/ (root)`로 지정하면 자동 빌드/배포됩니다.

## 5. 새 글 작성
`_posts/` 폴더에 `YYYY-MM-DD-제목.md` 형식으로 파일을 만들고
맨 위에 다음 형태의 front matter를 넣으세요.

```markdown
---
layout: post
title: "글 제목"
date: 2026-06-25
tags: [태그1, 태그2]
---

본문 내용
```

## 6. 이미지 추가하기
1. 이미지 파일을 `assets/images/posts/날짜-제목/` 폴더에 넣기
2. 글에서 아래처럼 호출

```liquid
{% raw %}{% include img.html name="날짜-제목/파일명.png" alt="설명" caption="캡션(선택)" %}{% endraw %}
```

나중에 이미지 호스팅을 다른 서비스로 옮기게 되면
`_config.yml`의 `image_base_url` 한 줄만 바꾸면 모든 글에 한꺼번에 적용됩니다.

## 7. 댓글(Isso) 연동
Isso 서버를 따로 띄운 뒤, `_config.yml`에서 아래 줄의 주석을 해제하고
본인 서버 주소를 넣으면 모든 글에 댓글창이 나타납니다.

```yaml
isso_url: "https://comments.본인도메인.com"
```

## 8. 디자인 커스터마이징
`assets/css/main.scss` 파일에서 사이드바 색상, 링크 색상, 사이드바 위치(좌/우) 등을
SASS 변수로 바로 수정할 수 있습니다.
