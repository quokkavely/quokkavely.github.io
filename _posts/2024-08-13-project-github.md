---
layout : single
title : "[Project] Github Repository 생성 및 연결"
categories: Project
tag : [project1, practice]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

---

## 2024-08-09 ~ 2024-08-29  첫 번째 프로젝트


이번 프로젝트에서 Repository를 내 GitHub에 올리기로 했는데, 팀원들이 편하게 작업할 수 있도록 미리 클라이언트와 서버를 나눠서 준비해두었다. 서버는 스프링 부트 파일을 다운받아 `build.gradle`을 세팅해 두었고, 클라이언트도 `npm run start` 명령어로 기본 파일을 만들어 놓았다. 그런데 Repository를 만들 때 몇 가지 실수를 저질렀다.

원래는 GitHub에 새로운 Repository를 만들고 아래와 같은 명령어를 사용해서 초기 설정을 해야 했다:

```bash
echo "# abc" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/quokkavely/abc.git
git push -u origin main
```

하지만, 이 과정을 생략한 채 그냥 확인을 눌러버려서 로컬 파일과 GitHub 원격 저장소가 제대로 연결되지 않았다. 파일이 추적되지 않았고, 한참 동안 왜 그런지 이유를 몰라서 헤맸다.

### Gitbash에서 파일과 원격 저장소 연결하기

이런 상황에 직면했을 때, 다음과 같은 절차를 밟으면 된다:

1. **Git 초기화 및 원격 저장소 연결:**
먼저, 파일이 있는 폴더로 이동한 다음 `git init`을 실행해서 Git을 초기화한다.
    
    ```bash
    git init
    ```
    
    이후, 원격 저장소와 연결을 설정한다.
    
    ```bash
    git remote add origin <repository 주소>
    ```
    
2. **파일 추가 및 커밋, 푸쉬:**
파일을 원격 저장소로 푸쉬하기 위해서는 먼저 파일을 Git에 추가하고, 커밋한 후, 푸쉬해야 한다.
    
    ```bash
    git add .
    git commit -m "Initial commit"
    git push -u origin main
    ```
    

### 이미 `.git` 폴더가 있을 때의 문제

이번에 에러가 발생한 원인은, 이전 실습 때 VScode에서 `npm run start` 명령어를 사용해 파일을 만들면서 이미 `.git` 폴더가 생성되어 있었기 때문이었다. 이 때문에 파일들이 원격 저장소와 제대로 연결되지 않았고, 추적되지도 않았다.

이런 경우, `.git` 폴더를 삭제하고 다시 Git 초기화를 하면 문제가 해결된다.

### `.git` 폴더 삭제하기 (Git 초기화 제거)

흔히 "깃발 제거"라고도 하는 `.git` 폴더를 삭제하는 명령어는 아래와 같다:

```bash
rm -rf .git
```

이 명령어를 사용하면 현재 폴더에서 Git 관련 설정이 모두 제거된다. 이후, 위에서 설명한 절차에 따라 Git 초기화를 다시 해주면 된다.

### 에러 해결 후 Git 사용법 정리

이 과정에서 몇 가지 중요한 교훈을 얻었다. 우선, Git 명령어를 정확하게 이해하고 사용하는 것이 얼마나 중요한지 다시 한 번 깨달았다. Repository를 만들고 원격 저장소에 제대로 연결하려면, 기본적인 명령어와 그 의미를 확실히 알아야 한다. 또한, 커밋 규칙이나 브랜치 사용법 등을 미리 정해두고 팀원들과 공유하는 것이 중요하다. 이 과정이 처음에는 시간이 좀 걸릴 수 있지만, 나중에 발생할 수 있는 충돌이나 혼란을 줄여주기 때문이다.

Git 사용이 아직 익숙하지 않아서 이런저런 시행착오를 겪게 되었는데,, 제발 충돌이 안생겼으면 좋겠다,

<br>
<br>
<br>
<br>
<br>
<br>
<br>
