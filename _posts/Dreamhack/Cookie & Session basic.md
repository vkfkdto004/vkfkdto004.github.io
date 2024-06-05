# Cookie & Session

# Cookie

클라이언트 IP 주소와  User-Agent는 매번 변경될 수 있는 고유하지 않은 정보일 뿐만 아니라, HTTP 프로토콜의 Connectionless와 Stateless 특징 때문에 웹 서버는 클라이언트를 일일히 기억할 수 없다.

- Connectionless : 하나의 요청에 하나의 응답을 한 후 연결을 종료하는 것. 새 요청이 있을 때 마다 항상 새로운 연결을 맺는다.
- Stateless : 통신이 끝난 후 상태 정보를 저장하지 않는 것을 의미. 이전 연결에서 사용한 데이터를 다른 연결에 요구할 수 없음

### Cookie란?

key와 value로 이루어진 일종의 단위로, 서버가 클라이언트에게 쿠키를 발급하면 클라이언트는 서버에 요청을 보낼때마다 쿠키를 같이 전송한다. 그 쿠키 값을 확인해 클라이언트를 구분할 수 있게된다.

# Session

쿠키에서의 단점은 클라이언트가 인증 정보를 변조하여 보안적인 문제를 초래한다는 것이다.

쿠키를 보완하기 위해 클라이언트가 아닌 서버에 저장되는 세션을 사용한다

### Session이란?

인증정보를 서버에 저장하고 해당 데이터에 접근할 수 있는 키(유추할 수 없는 랜덤한 문자열)을 만들어 클라이언트에게 전달하는 방식으로 사용하는 서버측 인증 방식

# Exercise

## 1. Cookie

문제에서 제공된 소스코드

```bash
#!/usr/bin/python3
from flask import Flask, request, render_template, make_response, redirect, url_for

app = Flask(__name__)

try:
    FLAG = open('./flag.txt', 'r').read()
except:
    FLAG = '[**FLAG**]'

users = {
    'guest': 'guest',
    'admin': FLAG
}
# guest 유저와 admin 계정이 있는 것으로 확인

@app.route('/')
def index():
    username = request.cookies.get('username', None) # 이용자가 전송한 쿠키의 username 입력값을 가져온다
    if username:
        return render_template('index.html', text=f'Hello {username}, {"flag is " + FLAG if username == "admin" else "you are not admin"}')
        # admin 유저로 username 입력값이 존재할 경우 flag를 띄어
    return render_template('index.html')

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'GET':
        return render_template('login.html')
    elif request.method == 'POST':
        username = request.form.get('username')
        password = request.form.get('password')
        try:
            pw = users[username]
        except:
            return '<script>alert("not found user");history.go(-1);</script>'
        if pw == password:
            resp = make_response(redirect(url_for('index')) )
            resp.set_cookie('username', username)
            return resp 
        return '<script>alert("wrong password");history.go(-1);</script>'

app.run(host='0.0.0.0', port=8000)

```

![1](/assets/img/Dreamhack/Cookie%20&%20Session%20basic/Untitled.png)

쿠키의 name → username, value는 guest로 되어있다. admin으로 쿠키값을 변경 시 flag 값을 띄어주므로 value 값을 admin으로 변경

![Untitled](/assets/img/Dreamhack/Cookie%20&%20Session%20basic/Untitled1.png)

![Untitled](/assets/img/Dreamhack/Cookie%20&%20Session%20basic/Untitled2.png)

간단하게 flag 값 획득

## 2. session-basic

```bash
#!/usr/bin/python3
from flask import Flask, request, render_template, make_response, redirect, url_for

app = Flask(__name__)

try:
    FLAG = open('./flag.txt', 'r').read()
except:
    FLAG = '[**FLAG**]'

users = {
    'guest': 'guest',
    'user': 'user1234',
    'admin': FLAG
}
# guest, user, admin 계정이 있는 것으로 확인된다.

# this is our session storage
session_storage = {
}

@app.route('/')
def index():
    session_id = request.cookies.get('sessionid', None) # 쿠키에서 세션ID를 조회한다
    try:
        # get username from session_storage
        username = session_storage[session_id] # 세션 스토리지에서 해당 세션ID를 통해 username을 조회한다.
    except KeyError:
        return render_template('index.html')

    return render_template('index.html', text=f'Hello {username}, {"flag is " + FLAG if username == "admin" else "you are not admin"}')
# 역시나 admin으로 로그인시 flag값을 띄어준다

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'GET':
        return render_template('login.html')
    elif request.method == 'POST':
        username = request.form.get('username')
        password = request.form.get('password')
        try:
            # you cannot know admin's pw
            pw = users[username]
        except:
            return '<script>alert("not found user");history.go(-1);</script>'
        if pw == password:
            resp = make_response(redirect(url_for('index')) )
            session_id = os.urandom(32).hex()
            session_storage[session_id] = username
            resp.set_cookie('sessionid', session_id)
            return resp
        return '<script>alert("wrong password");history.go(-1);</script>'

# 아래의 주석 코드을 지우지 않은 것은 일종의 취약점이다.
# 관리자 페이지를 구성한 코드 같은데,  만약 /admin으로 접속한 username이 admin이 아니라면index.html 페이지로 접속이 된다
# 또한 세션 정보가 포함된 session_storage는 usernaem이 admin인 관리자만 조회가 가능하도록 되어있다. 하지만 주석처리되어있으니 
#관리가자 아니라도 확인이 가능하다.
@app.route('/admin')
def admin():
    # developer's note: review below commented code and uncomment it (TODO)

    #session_id = request.cookies.get('sessionid', None)
    #username = session_storage[session_id]
    #if username != 'admin':
    #    return render_template('index.html')

    return session_storage

if __name__ == '__main__':
    import os
    # create admin sessionid and save it to our storage
    # and also you cannot reveal admin's sesseionid by brute forcing!!! haha
    session_storage[os.urandom(32).hex()] = 'admin'
    print(session_storage)
    app.run(host='0.0.0.0', port=8000)

```

결론적으로 위 코드를 봤을때 session_storage를 확인해 session_id를 변경하여 admin 계정으로 로그인해야 flag 값을 획득할 수 있는 것이다.

![Untitled](/assets/img/Dreamhack/Cookie%20&%20Session%20basic/Untitled3.png)

guest 계정으로 로그인 했을 시 sessionid 이다 여러번 로그인을 시도해봤지만 로그인 할때마다 sessionid가 변경되는 것을 확인했다.

![Untitled](/assets/img/Dreamhack/Cookie%20&%20Session%20basic/Untitled4.png)

위 코드에서 확인했을때 `/admin`은 원래 admin 계정으로 로그인해야 들어갈 수 있지만 주석처리 되어있기 떄문에 누구나 접속이 가능하다. 위 사진은 로그인했던 기록 즉 session_storage를 보여주는 것이다. admin에 매핑되어있는 값을 이용하여 flag 값을 획득해보겠다.

![Untitled](/assets/img/Dreamhack/Cookie%20&%20Session%20basic/Untitled5.png)

![Untitled](/assets/img/Dreamhack/Cookie%20&%20Session%20basic/Untitled6.png)

flag 값 획득!