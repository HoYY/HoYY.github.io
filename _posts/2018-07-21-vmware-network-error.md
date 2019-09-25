---
layout: post
title:  "VMware 외장하드로 옮긴 후 Network 연결 오류"
tags:
  - VMware
hero: https://source.unsplash.com/collection/261936/
overlay: skyblue
---

VMware를 사용하다 보면 용량이 너무 커져 하드의 공간이 부족해지는 일을 많은 분들이 겪었을 것이라 생각됩니다.  
저 또한 이와 같은 상황에 처해져 외장 SSD에 사용 중이던 VMware 옮기게 되었는데 구글 검색을 통해 찾은 글을 참고하여 copy it/move it 을 묻는 창에서 move it을 선택하였더니 network가 연결이 되지 않는 문제가 발생하였습니다. 때문에 이 게시물에서는 이 문제의 해결방안을 다뤄보겠습니다.
{: .lead}
<!–-break-–>

# 1. Virtual Network Editor

우선 VMware 상단의 Edit를 클릭하여 **Virtual Network Editor**를 클릭한다.

![Alt text](/uploads/vm_net_editor.PNG)

--------------------------------------------------------------

# 2. Change Settings

위의 창을 띄웠다면 이제 오른쪽 아래의 **Change Settings** 버튼을 클릭해주고 경고 창이 뜨면 권한을 허용해준다.

--------------------------------------------------------------

# 3. Restore Defaults

2번의 과정을 거치면 이제 왼쪽 아래의 Restore Defaults 버튼이 활성화될 것이다. 마지막으로 **Restore Defaults** 버튼을 클릭하면 VMware의 Network 세팅이 다시 이루어지면서 문제가 해결된다. 참고로 Vmware 머신들의 IP가 바뀐다.


[참고 사이트]  
<http://dbsc.tistory.com/114>
