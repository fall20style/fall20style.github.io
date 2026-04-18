---
title: git 여러 commit 합치기
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

Squash는 주로 `git rebase -i`(인터랙티브 모드)를 사용함.

## Squash 진행 순서

   1. 리베이스 시작: 합치고 싶은 커밋의 직전 해시나 개수를 지정함.
   * `git rebase -i HEAD~3` (최근 3개의 커밋을 대상으로 함)
   2. 편집기 창 확인: 대상 커밋 목록이 vim에 나타남.
   
   ```
   pick abc1234 첫 번째 커밋
   pick def5678 두 번째 커밋 (합칠 대상)
   pick ghi9012 세 번째 커밋 (합칠 대상)
   ```
   
   3. 명령어 수정: 남겨둘 첫 번째 커밋은 pick으로 두고, 합칠 커밋들의 pick을 squash(또는 s)로 수정.
   
   ```
   pick abc1234 첫 번째 커밋
   s def5678 두 번째 커밋
   s ghi9012 세 번째 커밋
   ```
   
   4. 저장 후 닫기: 편집기를 저장하고 닫으면 커밋 메시지를 통합하는 창이 새로 뜸.
   5. 메시지 정리: 하나로 합쳐질 최종 커밋 메시지를 작성하고 저장.
   6. 완료: Successfully rebased 메시지가 뜨면 성공.

## 주의사항

* 히스토리 변경: Squash도 Rebase의 일종이라 커밋 해시가 새로 생성됨.
* 강제 푸시: 이미 원격에 올렸던 커밋들이라면 `git push -f`가 필요함.


## Related Posts
- [git rebase와 merging의 차이]({% link _posts/2026-04-01-git-rebase-and-merging.md %})
- [원본 저장소에서 Fork 없이 바로 PR을 만드는 방법]({% link _posts/2026-04-07-creat-pr-on-org-repo.md %})
- [github에 comment를 가져오기]({% link _posts/2026-04-02-github-comment-query.md %})
