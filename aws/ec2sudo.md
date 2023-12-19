# ec2 sudo 문제
AWS 비용 문제로 인해 instance type을 바꾸고 다시 가동한뒤 sudo 명령어를 사용하려고 했는데

다음과 같은 문제가 console 창에 뜨면서 문제가 발생했다. 
`"/usr/bin/sudo must be owned by uid 0 and have the setuid bit set"`

왜..? 갑자기 왜???? 라는 물음이 들면서 짜증이 좀 났지만 어쩌겠는가... 수정을 해야지

서칭결과 두가지 방법이 나왔다. 

첫번째 ec2 직렬 콘솔에 접속하여 문제를 해결하기!
두번째 사용자 스크립트를 통해 문제를 해결하기

참고 :<a>https://repost.aws/ko/knowledge-center/ec2-sudo-commands</a>

이런식으로 문제를 해결가능하다고 aws 에서 친절하게 알려주고 있었다. 자 그러면 먼저 ec2 직렬콘솔에 접근해보자!

<img src="./img폴더/직렬콘솔.png">

이거이거.. X됐네..... 라는 생각이 들었고 재빨리 두번째 방법이 되기 만을 기도하며 두번쨰 방법으로 갔다.

해결방법은 먼저 ec2 console창에서 다음과 같은 명령어를 친다.
```sh
cat /etc/os-release
```
이렇게 명령어를 치면 두가지 경우의 수가 나온다

```sh
Red Hat 기반: ID="rhel", ID="fedora", ID="centos" 등
Debian 기반: ID="debian", ID="ubuntu" 등
```
이렇게 두가지가 나올텐데, 사실 두가지만 나오는건 아니다. 본인의 경우

```sh
NAME="Amazon Linux"
VERSION="2"
ID="amzn"
ID_LIKE="centos rhel fedora"
```
이렇게 나오는것이였다. 이게 뭐야..? 라고 생각했지만 ID_LIKE에 fedora가 있고 우리의 친절한(?) aws에서 어떤 리눅스 커널에 어떤 방법이 가능한지 알려주고 있었다. 

<img src="./img폴더/사용자%20데이터스크립트.png">

자 그런 다음 이제 어떤 스크립트를 사용하는지 알았으니 

```sh
Content-Type: multipart/mixed; boundary="//"
MIME-Version: 1.0

--//
Content-Type: text/cloud-config; charset="us-ascii"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Content-Disposition: attachment; filename="cloud-config.txt"

#cloud-config
cloud_final_modules:
- [scripts-user, always]

--//
Content-Type: text/x-shellscript; charset="us-ascii"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Content-Disposition: attachment; filename="userdata.txt"

#!/bin/bash
PATH=$PATH:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:
rpm --setugids sudo && rpm --setperms sudo
find /etc/sudoers.d/ -type f -exec /bin/chmod 0440 {} \;
find /etc/sudoers.d/ -type f -exec /bin/chown root:root {} \;
--//
```
이 스크립트를 어디에 적용시켜야 하는지 하나하나 따라가보자.

먼저 인스턴스를 중지 시킨다. <span style=color:red>**종료 아니다.**</span>  

종료시키고 나면 옆에 작업 부분이 보일것이다.

<img src="./img폴더/aws사용자/인스턴스%20작업.png"/>

작업 부분을 클릭하고 **인스턴스 설정** 으로 들어가서 **사용자 데이터 편집**을 실행시킨다. 

그러고 나면 이런 화면이 나오게 된다. 

<img src="./img폴더/aws사용자/사용자%20데이터%20편집.png">

여기에 아래 사용자 데이터 배치 부분에 위레 올려져있는 스크립트를 작성하고 저장을 누른후 다시 실행을 시켜주면 문제가 해결되게 된다!