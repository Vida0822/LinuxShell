# 서버 운영해보기 (Rinux, AWS)







## 1. Rinux 사용해보기



### Window에서 Rinux 쓰기 

1. Window 터미널 설치 

![image-20231009122544257](D:\Programming\images\서버 운영해보기 (Rinux, AWS)\image-20231009122544257.png)



#### 2. WLS2 활성화 

* WSL2(Windows Subsystem for Linux 2)



관리자 권한으로 Window Terminal 실행 

​	↓

명령어 

```powershell
> dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
> dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```



```powershell
PS C:\Users\SHIN HEEMIN> dism.exe /online /enable-feature /featur
배포 이미지 서비스 및 관리 도구
버전: 10.0.19041.844

이미지 버전: 10.0.19045.3448

기능을 사용하도록 설정하는 중
[==========================100.0%==========================]
작업을 완료했습니다.

PS C:\Users\SHIN HEEMIN> dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

배포 이미지 서비스 및 관리 도구
버전: 10.0.19041.844

이미지 버전: 10.0.19045.3448

기능을 사용하도록 설정하는 중
[==========================100.0%==========================]
작업을 완료했습니다.
```







#### 3. 리눅스 (우분투) 다운로드 

재부팅을 하고 `wsl`을 실행하면 먼저 아래 주소로 이동해서 WSL을 설치하라는 안내 메시지가 나옴

=> WSL을 사용하려면 리눅스 배포판을 설치해야함

1. 리눅스 커널 세팅   - [이전 버전 WSL의 수동 설치 단계 | Microsoft Learn](https://learn.microsoft.com/ko-kr/windows/wsl/install-manual)
2. 리눅스(우분투) 다운로드  - 

```powershell
PS C:\Users\SHIN HEEMIN> wsl --install
Ubuntu이(가) 이미 설치되어 있습니다.
Ubuntu을(를) 시작하는 중...
Installing, this may take a few minutes...
```





오류!!!!! : WslRegisterDistribution failed with error

(머든 세팅은 지옥같다 개발자가 단명한다면 분명 세팅 때문.)

```powershell
WslRegisterDistribution failed with error: 0x800701bc Error: 0x800701bc WSL 2? ?? ?? ?? ????? ?????. ??? ??? https://aka.ms/wsl2kernel? ??????
```

=> 해결방법 

1. 리눅스 Microsoft에서 따로 설치해준거 지우고, 배포판 다운 받고 terminal 통해 wsl가 자동 설치하게 해줌 
2. 하위 시스템이 꼬여서 아예 취소 후 재설정 

![image-20231009132535737](D:\Programming\images\서버 운영해보기 (Rinux, AWS)\image-20231009132535737.png)

​						ㄴ 아래꺼 취소 -> 재부팅 -> 다시 체크 -> 재부팅 



=> 교훈 : 실행 순서가 꼬이지 않게 차례대로 하자 





#### 4. 계정 설정 

```powershell
Please create a default UNIX user account. The username does not need to match your Windows username.
For more information visit: https://aka.ms/wslusers
Enter new UNIX username: hm
New password:
Retype new password: # ss9543
passwd: password updated successfully
Installation successful!
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

Welcome to Ubuntu 22.04.2 LTS (GNU/Linux 5.15.90.1-microsoft-standard-WSL2 x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

This message is shown once a day. To disable it please create the
/home/hm/.hushlogin file.
hm@DESKTOP-DGFPD6H:~$
```







#### 5. PoserShell에서 Ubuntu 사용 가능 

![image-20231009132850421](D:\Programming\images\서버 운영해보기 (Rinux, AWS)\image-20231009132850421.png)







#### * 과제 : 리눅스에서 간단한 파이썬 프로그램을 작성한 후 실행 

리눅스에는 별도의 편집기가 없으니 vi, vim  으로 작성  --> 이거 반드시 익숙해져야 함



 





## 2. AWS로에 서버 구축   

**EC2 : 클라우드 가상 서버 **

* 대부분의 기업에선 클라우드에서 리눅스 사용 가능 --> 클라우드의 방화벽 같은 추가 기능 사용가능 

* asw로부터 리눅스를 할당받아 거기에 mysql db 설치해보자 !  

  ( 데이터베이스 - 'RDS' : mysql을 사용할 수 있는 서버를 임대받을 수 있음 - 아주 큰 aws의 db 중 일부분을 임대받아서 사용하는거임)



### AWS에 서버 구축 





#### 1. aws 가입 

![image-20231009135222802](D:\Programming\images\서버 운영해보기 (Rinux, AWS)\image-20231009135222802.png)



* 신용카드 
* 무료 계정 (한도 내 사용 + 기간 내 서버 밀고 탈퇴)
* otp 설정 필수 

![image-20231009135440754](D:\Programming\images\서버 운영해보기 (Rinux, AWS)\image-20231009135440754.png)





#### 2. 리전 설정

: 서버를 만들 위치 (Data center의 가상서버)

ㄴ 멀리 떨어지면 인터넷이 느려질 수도 있음 / 서버 인스턴스를 못찾는 경우  







#### 3. EC2 --> 인스턴스 시작 

![image-20231009140116101](D:\Programming\images\서버 운영해보기 (Rinux, AWS)\image-20231009140116101.png)





* 서버 이름 및 OS 설치 (AWS가 자동으로 설치해줌)

![image-20231009140157666](D:\Programming\images\서버 운영해보기 (Rinux, AWS)\image-20231009140157666-1696827718026-1.png)



![image-20231009140414303](D:\Programming\images\서버 운영해보기 (Rinux, AWS)\image-20231009140414303.png)

* 프리티어 : 무료 프로그램 (작은 서버를 1년동안 무료로 서버 만들 수 있음)

​	※ 여러대로 쓸 경우 나눠서 1년 소비 (각 4달씩)



* 인스턴스 유형 : 서버의 규모 (프리티어로...)

![image-20231009140719148](D:\Programming\images\서버 운영해보기 (Rinux, AWS)\image-20231009140719148.png)







* 키페어 

  : 비밀번호만 하면 불안전하니까 특정 파일을 갖고 있는 경우에만 로그인이 가능한 추가 인증 설정해줌 (전문적인 접속 도구 )

  => 보안성 강화 



![image-20231009141249779](D:\Programming\images\서버 운영해보기 (Rinux, AWS)\image-20231009141249779.png)

​	

ㄴ 프라이빗 키 : ppk --> putty 프로그램과 엮어서 사용가능 

​	(putty : 웹 기반으로 rinux 접속해서 사용할 때 불편한점이 있음 

​		=> 편하게 접속 , 다룰 수 있는 전문적인 접속 도구   )

​	ㄴ 나중에 putty를 통해 서버에 접속할 때도 이 key 페어 파일을 사용할 예정 ! 



![image-20231009141529016](D:\Programming\images\서버 운영해보기 (Rinux, AWS)\image-20231009141529016.png)







* 네트워크 설정 

![image-20231009141600548](D:\Programming\images\서버 운영해보기 (Rinux, AWS)\image-20231009141600548.png)

​	

![image-20231009141800306](D:\Programming\images\서버 운영해보기 (Rinux, AWS)\image-20231009141800306.png)

​	ㄴ 공인 IP : 알아서 제공해줌 --> 내 노트북에서 이 서버에 접속할 때 요거 사용해줌

​	※ 고정 IP 가 아니다 ! 

​	ㄴ프라이빗 IP : 얘도 있음 ! (내부적으로 쓰는 IP) 

​			--> amazon cloud 서비스 내에서 작업할 땐 이거 써도 됨 

​	(인스턴스 '상태' 껐다가 키면 IP 바뀔 수도 있음)

 * 인스턴스 상태 : 중지(멈추는거)  vs  종료 (아예 없애는거)
   * 실습하다가 문제 생기면 날리고 새로 만들기(과금 안됨)



​	

ㄴ 방화벽 설정

![image-20231009141647587](D:\Programming\images\서버 운영해보기 (Rinux, AWS)\image-20231009141647587.png)

하드디스크는 8 gb만큼 생성 

(비용과 직결! )





#### 세부 정보 확인하기 

* 보안 

![image-20231009142504609](D:\Programming\images\서버 운영해보기 (Rinux, AWS)\image-20231009142504609.png)

--> 지금은 putty 프로그램을 통해 내가 내 서버에 접속하는데에 대한 보안만 있음 (1행)

 : 2개 더 추가해야함 (웹 브라우저 접속, mysql db에 접속) 

==> '보안 그룹'에서 추가해주면 됨 









### 3. 연결

#### 1. 웹에서 접속해서 사용 : ec2 인스턴스 연결 

![image-20231009142918088](D:\Programming\images\서버 운영해보기 (Rinux, AWS)\image-20231009142918088.png)

![image-20231009143011390](D:\Programming\images\서버 운영해보기 (Rinux, AWS)\image-20231009143011390.png)

ㄴ 우왕 우분투 잘 되있다! 





#### 2. putty 이용해서 연결 : ssn 클라이언트

![image-20231009143249157](D:\Programming\images\서버 운영해보기 (Rinux, AWS)\image-20231009143249157.png)

대표적인 ssn 클라이언트 : open ssn / putty 





##### putty 설치

![image-20231009143407761](D:\Programming\images\서버 운영해보기 (Rinux, AWS)\image-20231009143407761.png)

* putty 

  : 원격지에 있는 rinux 서버에 접속 해 그 rinux 서버를 쉽게 다룰 수 있게 (편할 수 있게) 도와주는 프로그램 

  --> 내가 입력한걸 서버로 암호화해 전달하기 때문에 ('보안 접속')

  ​	도청 위험이 없다
  
  
  
* 원래는 telnet  :  보안이 되지 않아 내 입력 내용을 도청 가능 

  => ssh : 보안이 적용되어있는 원격접속 프로그램 

  ㄴ 조건 

  ​	* 서버에선 ssd가 돌아가야함 (aws가 기본 제공, 직접 linux 깔때는 이거 반드시 설치)

  ​	* 접속자는 putty 또는 ssn(맥) 으로 접속 

  ㄴ 접속 방법 : key  ! (서버에서 발급받아 내 컴퓨터에 저장해놓고 접속할때마다 제출)

 

기본정보 2 개 : 1. 원격지에 있는 ip 주소 2. keyfile

![image-20231009143906201](D:\Programming\images\서버 운영해보기 (Rinux, AWS)\image-20231009143906201.png)

1) 공인 ip 주소 적어주고 open 

--> 계정명으로 로그인하려하면 오류 ! 왜냐면 우리가 이 계정 접속할땐 키페어를 사용하도록 설정해줬기 때문이다 





2) keypair 설정해주기 

![image-20231009144653512](D:\Programming\images\서버 운영해보기 (Rinux, AWS)\image-20231009144653512.png)





![image-20231009150019502](D:\Programming\images\서버 운영해보기 (Rinux, AWS)\image-20231009150019502.png)

```powershell
login as: ubuntu
Authenticating with public key "linuxTest"
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.15.0-1036-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon Oct  9 05:59:25 UTC 2023

  System load:  0.0               Processes:             98
  Usage of /:   21.2% of 7.57GB   Users logged in:       0
  Memory usage: 24%               IPv4 address for eth0: 172.31.45.25
  Swap usage:   0%


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
New release '22.04.3 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


Last login: Mon Oct  9 05:30:04 2023 from 18.206.107.28
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.


```



```powershell

ubuntu@ip-172-31-45-25:~$ su # fail - 비번 x 
Password:
su: Authentication failure
ubuntu@ip-172-31-45-25:~$ sudo su # 슈퍼권한 (우분투 계정이 root니까 가능)
root@ip-172-31-45-25:/home/ubuntu # root 계정으로 작업 (계정 ubun)
root@ip-172-31-45-25:/home/ubuntu exit
exit # 일반 계정으로 돌아옴


```







#### 청구서 매일 확인 : 과금 확인 (탈퇴할때까진)

![image-20231009150458938](D:\Programming\images\서버 운영해보기 (Rinux, AWS)\image-20231009150458938.png)

각종 서비스 요금 이것저것 나와도 prettier로 최종 요금이 0.00으로 나와야함