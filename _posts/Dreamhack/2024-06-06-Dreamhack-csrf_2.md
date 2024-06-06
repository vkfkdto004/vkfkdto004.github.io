# Dreamhack - csrf-2
이번 문제 또한 csrf 관련 문제인데, 이전과 다른 점은 관리자 계정으로 로그인을 우선 해야한다는 시나리오가 있어야한다.

# 코드 분석

### users
```python
users = {
    'guest': 'guest',
    'admin': FLAG
}
```
로그인이 가능한 두개의 유저가 뜬다.

### /
```python
@app.route("/")
def index():
    session_id = request.cookies.get('sessionid', None)
    try:
        username = session_storage[session_id]
    except KeyError:
        return render_template('index.html', text='please login')

    return render_template('index.html', text=f'Hello {username}, {"flag is " + FLAG if username == "admin" else "you are not an admin"}')
```
기본 페이지에 대한 코드인데, username이 admin이 되어야 flag를 보여주는 것같고, 세션ID는 세션 스토리지에 저장이 된다.

### /vuln
```python
@app.route("/vuln")
def vuln():
    param = request.args.get("param", "").lower()
    xss_filter = ["frame", "script", "on"]
    for _ in xss_filter:
        param = param.replace(_, "*")
    return param
```
csrf-1과 같이 XSS 공격을 시도하지 못하겠끔 키워드 필터링이 걸려 문자열이 치환되는 것을 볼 수 있다.
여전히 키워드 필터링을 벗어나 `<img>` 태그를 사용할 수 있는 취약점이 존재하며, 익스플로잇 까지 연계가 가능할 것같다.

### /flag
```python
@app.route("/flag", methods=["GET", "POST"])
def flag():
    if request.method == "GET":
        return render_template("flag.html")
    elif request.method == "POST":
        param = request.form.get("param", "")
        session_id = os.urandom(16).hex()
        session_storage[session_id] = 'admin'
        if not check_csrf(param, {"name":"sessionid", "value": session_id}):
            return '<script>alert("wrong??");history.go(-1);</script>'

        return '<script>alert("good");history.go(-1);</script>'
```
/flag에서는 method에 따라 다른 동작이 있는데, 주목해야할점은 POST이다 세션ID가 생성되고 생성한 세션ID를 키로 사용하고, admin 값을 세션 스토리지 딕셔너이레 저장한다.

### /login
```python
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
            session_id = os.urandom(8).hex()
            session_storage[session_id] = username
            resp.set_cookie('sessionid', session_id)
            return resp 
        return '<script>alert("wrong password");history.go(-1);</script>'
```
/login의 엔드포인트인데 로그인을 처리하며, 사용자가 유효한 사용자 이름과 비밃너호를 제출하면 세션을 설정하고 다르다면 index 페이지로 리다이렉팅 된다.

### /change_password
```python
@app.route("/change_password")
def change_password():
    pw = request.args.get("pw", "")
    session_id = request.cookies.get('sessionid', None)
    try:
        username = session_storage[session_id]
    except KeyError:
        return render_template('index.html', text='please login')

    users[username] = pw
    return 'Done'
```
비밀번호를 변경하는 페이지이다 사용자의 세션이 맞는지 확인한 후에 새로운 비밀번호를 설정하는 것이다.


# 익스플로잇
이전 코드에서 보았듯이, /flag에서 admin의 세션ID가 생성되고 세션 스토리지에 저장되는 것을 확인했다. 그렇다면 비밀번호를 알아야하는데 이전 비밀번호에 대한 힌트는 전혀 정보가 없고, 이 또한 코드에서 보았듯이 비밀번호를 변경할 수 있는 페이지가 있는 것을 확인했다.
/flag 페이지에 접속해 아래와 같은 코드를 삽입해 익스플로잇을 실시한다.
```bash
<img src="/change_password?pw=admin">
```
![alt text](/assets/img/Dreamhack/CSRF/csrf-2/image.png)
pw를 admin으로 변경하였으니 설정한 비밀번호로 로그인을 해본다.
![alt text](/assets/img/Dreamhack/CSRF/csrf-2/image-1.png)
username과 password 모두 admin으로 삽입하니 관리자 계정으로 로그인 성공한 것을 확인할 수 있다. 