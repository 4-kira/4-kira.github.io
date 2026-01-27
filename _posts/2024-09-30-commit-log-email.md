---
title: "commit log의 email 변경"
date: 2025-09-30 00:00:00 +0900
categories: [implementation, tools]
tags: [commit]
---

> [Archive] 이 글은 2024년에 다른 플랫폼에서 작성한 글을 이전한 것입니다.  
> 내용 중 일부는 현재 기준과 맞지 않을 수 있습니다.

## 목표

커밋할 때 사용하는 이메일 주소를 터미널에서 확인하는 명령어는 다음과 같다.

전역 설정 확인
```
git config --global user.email
```

로컬 설정(특정 repository) 확인
```
git config user.email
```

gitKraken(Git GUI Client)를 사용해 log를 관리하는 과정에서  
Github primary email이 아닌 Organization의 회사 email을 등록하여 사용하던 중,  
어느 순간에 git contributions를 확인해보니 commit log가 찍히고 있지 않음을 확인.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FGYndV%2FbtsJR09KjVP%2FAAAAAAAAAAAAAAAAAAAAAKnSO40gGrbSn-c0ajYGUzubWnx-n73vUn2kMArSuU9H%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1769871599%26allow_ip%3D%26allow_referer%3D%26signature%3DjVpsYCzP85uAbymaiaOkL2WfQak%253D)

Git의 settings > access > emails에 회사 이메일을 등록해놓지 않았었기에 등록했지만 여전히 반영되지 않는다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FtpGkt%2FbtsJS8Momzp%2FAAAAAAAAAAAAAAAAAAAAAIoq3rz1cCjgh-ZSYwyhUl9YOVd7DDxCVJu-g5qaLAj0%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1769871599%26allow_ip%3D%26allow_referer%3D%26signature%3D0n3i17lkTQ8omtjFABiX5BywyfQ%253D)

[[GitHub Docs]Why are my contributions not showing up on my profile?](https://docs.github.com/en/account-and-profile/how-tos/contribution-settings/troubleshooting-missing-contributions)를 보면,  
"기여로 간주되는 요구 사항을 충족하는 커밋을 한 후, 기여가 표시되려면 최대 24시간이 걸릴 수 있다."  
라고 표현하고 있다.  
하지만 이걸 바로 반영하고 싶었다. 다음에 반영되었는지 확인하지 않을 것 같았기 때문.

때문에 특정 이메일로 commit된 로그의 이메일 주소만을 바꾸는 것이 가능한지 알아보았다.

<br/>

## 해결

특정 커밋의 이메일을 interactive rebase를 통해 수동으로 변경할 수 있다.

```
git rebase -i HEAD~n
```
n은 변경하고자 하는 commit의 개수.

전역 commit email을 정상적으로 돌려놓은 마지막 commit을 포함하여 아래로 21개의 commit을 조작한다.  
commit log가 vim편집기로 열리게 된다. (i: input, w: write, q: quit)  
log들(여기서는 21개의 row)에 대하여 수정해야 하는 커밋의 pick을 모두 edit으로 수정.
```
:%s/pick/emit/g
```
해당 명령어로 모든 워딩을 치환할 수 있다.

이어서 순서대로 다음의 명령어를 입력한다.
```
git commit --amend --author="{name} <{email}>"
```
해당 명령어를 통해 commit message를 수정할 수 있는 파일이 열리게 된다.

이미 '--author'을 통해 목표로 했던 email의 수정은 이뤄졌으므로, ':q'로 파일을 닫은 후 리베이스를 계속 진행.
```
git rebase --continue
```
이 행위를 원하는 commit의 수정 개수만큼 반복. 🤨  
각 커밋마다 수동으로 이메일을 변경해야 하므로, 자동화된 방법은 아니다.

모든 커밋을 수정한 후, 원격 저장소에 강제 푸시.
```
git push --force
```
결과적으로 다음과같이 해결할 수 있게 되었다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FbhG7Zz%2FbtsJSd2fm4z%2FAAAAAAAAAAAAAAAAAAAAAD6pq07JyWy66D3EMDP-_7Eind0sG9ZadJQywmTku4n-%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1769871599%26allow_ip%3D%26allow_referer%3D%26signature%3DNAiNhdDThfxO8NF%252Bjl19qnQyg8E%253D)

최초 commit의 시간대와 동일한 시간대로 contributions에 반영.

<br/>

## 발생할 수 있는 문제와 해결

### - rebase가 진행 중인 상태에서 git rebase로 재접근한 경우

> fatal: It seems that there is already a rebase-merge directory, and I wonder if you are in the middle of another rebase.
> If that is the case, please try git rebase (--continue | --abort | --skip) 
> If that is not the case, please rm -fr ".git/rebase-merge" and run me again. 
> I am stopping in case you still have something valuable there.

Git은 rebase 도중에 .git/rebase-merge 디렉토리를 생성해 현재 진행 상태를 관리하는데,  
해당 디렉토리가 남아 있는 경우 Git은 아직 이전 rebase가 끝나지 않았다고 판단한다.

interactive rebase 과정에서 커밋을 수정하다 보면, 흐름을 잊고 다시 git rebase -i를 실행하는 경우가 종종 발생한다.  
이 경우 선택지는 세 가지다.

- --continue : 현재 수정 중인 커밋을 마무리하고 rebase를 계속 진행
- --skip : 현재 커밋을 건너뛰고 다음 커밋으로 이동
- --abort : rebase를 완전히 취소하고, rebase 이전 상태로 되돌림

이번 경우처럼 처음부터 다시 rebase를 진행하고 싶다면,
기존 rebase 상태를 정리하는 것이 가장 안전하므로 다음 명령어를 사용한다.

```
git rebase --abort
```
이 명령어는 .git/rebase-merge를 제거하고, rebase 시작 이전의 브랜치 상태로 되돌린다.  
이후 다시 git rebase -i HEAD~n으로 재진입하여 작업.

<br/>

### - detached HEAD (branch가 없는)상태에서 push를 시도한 경우

> fatal: You are not currently on a branch. To push the history leading to the current (detached HEAD) state now, use git push origin HEAD:\<name-of-remote-branch>

이 에러는 현재 HEAD가 특정 브랜치를 가리키고 있지 않은 상태(detached HEAD)에서 git push를 시도했을 때 발생.

interactive rebase 과정에서 특정 커밋을 수정하면, 해당 커밋을 기준으로 HEAD를 분리한 상태에서 작업을 진행한다.  
이 상태에서 어느 브랜치에 push해야 하는지 Git이 알 수 없다.  
따라서 이 상태에서 push를 하려면, 현재 커밋을 기준으로 새로운 브랜치를 생성한 뒤 push해야 한다.
 
```
git checkout -b new-branch-name
git push origin new-branch-name
```
이렇게 하면 현재 수정된 커밋 히스토리를 새로운 브랜치로 명확히 귀속시킬 수 있고,  
이후 필요하다면 기존 브랜치에 강제 병합(--force)하거나 브랜치를 교체하는 방식으로 관리할 수 있다.

<br/>

### - 터미널에서 GitHub password를 입력했을 때 인증에 실패하는 경우

> remote: Please see
> https://docs.github.com/get-started/getting-started-with-git/about-remote-repositories#cloning-with-https-urls for information on currently recommended modes of authentication. fatal: Authentication failed for

이 에러는 HTTPS 방식으로 GitHub 원격 저장소에 접근하면서, 계정 비밀번호를 그대로 입력했을 때 발생한다.

GitHub는 2021년 이후로 보안 강화를 위해 계정 비밀번호 기반 인증을 완전히 중단하고,  
대신 Personal Access Token(PAT) 을 사용하도록 정책을 변경했다.

따라서 터미널에서 password를 요구받을 경우,  
username : GitHub 계정 ID  
password : 계정 비밀번호 ❌ → Personal Access Token ⭕

PAT는 다음 경로에서 생성할 수 있다.

`GitHub → Settings → Developer settings → Personal access tokens → Generate new token`

일반적인 Git 작업( push / pull / clone )만 필요하다면 권한은 repo 정도면 충분하다.  
생성된 토큰은 이후 찾을 수 없으므르, 로컬에 안전하게 저장하는 것이 좋다.

<br/>