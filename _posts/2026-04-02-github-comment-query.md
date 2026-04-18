---
title: github에 comment를 가져오기
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - git
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

GitHub CLI(gh)를 쓰면 편함.

## 1. 설치 및 로그인
gh 깔려 있어야 함.

```
gh auth login
```

## 2. PR 코멘트 긁어오기
정정 분석 결과가 PR 댓글로 달린다면 아래 명령어로 확인 가능.

* 전체 코멘트 보기: `gh pr view <PR번호> --comments`
* 특정 작성자(봇) 결과만 필터링:

```
gh pr view <PR번호> --json comments --jq '.comments[] | select(.author.login == "봇아이디") | .body'
```

## 3. API 직접 호출
더 상세한 데이터가 필요하면 API를 직접 때리는 게 나음.

```
gh api repos/:owner/:repo/issues/<PR번호>/comments --jq '.[].body'
```

## 4. 기타

* 파일 저장: 뒤에 `> result.txt` 붙여서 저장하면 됨.
* 체크 결과: 댓글 말고 'Checks' 탭에 있으면 `gh pr checks <PR번호>` 쓰면 됨.

## Related Posts
- [git 여러 commit 합치기]({% link _posts/2026-04-01-git-merging-commits-into-one.md %})
- [git rebase와 merging의 차이]({% link _posts/2026-04-01-git-rebase-and-merging.md %})
- [원본 저장소에서 Fork 없이 바로 PR을 만드는 방법]({% link _posts/2026-04-07-creat-pr-on-org-repo.md %})
