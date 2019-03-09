---
layout: post
title:  "STS에서 Github 연동하기"
tags:
  - Spring Tool Suite
  - Github
hero: https://source.unsplash.com/collection/582860/
overlay: aliceblue
---
[환경]
Spring Tool Suite(STS) 3.9.5 RELEASE
{: .lead}

이번 포스터에서는 STS의 Project와 Github의 Repository를 연동하는 방법을 다뤄보도록 하겠습니다.

# 1. Github Repository 등록

우선 STS에 연동하려는 Github Repository를 등록해야한다.
STS의 상단 우측을 보면 'Open Perspective' 버튼이 있다. 'Open Perspective'를 클릭하여 Repository를 등록해준다.
![Alt text](/uploads/sts_open_perspective.PNG)
위의 이미지를 보면 Open Perspective의 여러 메뉴가 있는데 Git메뉴를 선택해준 후 'Select one of the following to add a repository to the view' 문구 아래에 'Clone a Git repository' 를 클릭해준다. 
![Alt text](/uploads/sts_clone_git_repository.PNG)
위 사진의 창이 뜨면 Clone URI 를 선택해주고 next를 눌러준다.
![Alt text](/uploads/sts_clone_git_repository2.PNG)
그리고 연동 할 Github Repository의 URI를 복사하여 입력해주고 Github의 ID와 PW를 입력해준다. 
URI를 입력해주면 Host와 Repository path가 자동으로 입력될 것이다.
![Alt text](/uploads/sts_clone_git_repository3.PNG)
그 다음 next를 누르면 Branch를 선택하는 화면이 나온다.
연동할 Branch를 선택해주고 next를 눌러준다.
![Alt text](/uploads/sts_clone_git_repository4.PNG)
다음으로 Repository를 저장할 Local Diractory 위치를 입력해준다.
모두 입력이 끝나고 Finish 버튼을 눌러주면 Github Repository가 STS에 등록이 된다.
![Alt text](/uploads/sts_git_repositories.PNG)
정상적으로 등록이 끝나면 위의 사진과 같이 Repository가 등록된 것을 볼 수 있다.

--------------------------------------------------------------

# 2. STS Project와 Github Repository 연동

이제 Github Repository의 등록이 끝났으니 STS Project를 Commit 해보자.
![Alt text](/uploads/sts_git_add.PNG)
STS Project Explorer에서 Github Repository와 연동을 할 프로젝트를 우클릭 하여 'Team' 메뉴의 'Share Project'를 클릭해준다.
연동할 Repository와 Project를 선택 후 Finish를 눌러주고, 다시 프로젝트를 우클릭하여 'Team' 메뉴의 'Add to Index'를 클릭하여 Commit을 위해 프로젝트 파일들을 추가해준다.
![Alt text](/uploads/sts_git_commit.PNG)
마지막으로 Commit 할 파일들을 선택해주고 Commit Message를 입력하여 Commit해준다.