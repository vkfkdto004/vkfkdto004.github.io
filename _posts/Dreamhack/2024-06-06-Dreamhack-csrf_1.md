# Dreamhack - csrf-1
CSRF 관련 취약점이 존재하는 csrf-1 문제를 풀며 관리자만 수행할 수 있는 메모 작성 기능을 이용하는 것이 목표이다.

문제에서 제공하는 Flask 프레임워크의 코드를 살펴본다.
## / 엔드포인트
```bash
@app.route("/")
def index():
    return render_template("index.html")
```
index.html 페이지를 반환해주는 코드이다

## /vuln
```bash
@app.route("/vuln")
def vuln():
    param = request.args.get("param", "").lower()
    xss_filter = ["frame", "script", "on"]
    for _ in xss_filter:
        param = param.replace(_, "*")
    return param
```
이용자가 입력한 값을 param 파라미터의 값을 출력한다. 이때 XSS가 발생하지 않도록 frame, script, on의 키워드를 * 문자로 치환하여 키워드 필터링을 한 것 같다. 이 `/vuln` 엔드포인트가 아마 취약점에 대한 힌트이지 않을까 싶다
왜냐면 `<>`와 같은 다른 키워드와 태그는 사용할 수 있기 때문에 CSFR 공격이 가능 할 것같다.

## /admin/notice_flag
```bash
@app.route("/admin/notice_flag")
def admin_notice_flag():
    global memo_text
    if request.remote_addr != "127.0.0.1":
        return "Access Denied"
    if request.args.get("userid", "") != "admin":
        return "Access Denied 2"
    memo_text += f"[Notice] flag is {FLAG}\n"
    return "Ok"
```
`/admin/notice_flag` 엔드포인트이다. 코드를 살펴보면 로컬에서 접근하고, userid가 admin일 경우에 memo_text 즉 `/memo`에 flag 값을 적어주는 것이다.
페이지 자체는 모든 사람이 들어갈 수 있고, userid 파라미터에도 admin을 넣는 것은 가능하지만, 제일 우선 적으로 로컬호스트의 자격을 얻지 못하거나, 단순히 접속하는 것만으로 는  flag 값을 획득 할 수 없을 것이다.

# 익스플로잇 

## 취약점 진단

취약점인 `/vuln` 엔드포인트를 이용해 익스플로잇을 하기전에 우선 취약점에 대해 진단을 한다.
Dreamhack에서 제공하는 Request Bin을 이용하여 URL에 접속하는 정보를 쉽게 얻어 올 수 있게 됬다.
![alt text](/assets/img/Dreamhack/CSRF/Request%20Bin.png)

위의 주소를 `<img>` 태그를 이용하여 `<img src="https://bceukdd.request.dreamhack.games">` 값을 `vuln page`에 접속해서 주소창에 다음곽 같이 입력한다
![alt text](/assets/img/Dreamhack/CSRF/image.png)
이후 Request Bin을 확인한다.
![alt text](/assets/img/Dreamhack/CSRF/image1.png)
CSRF 공경 코드 삽입에 대한 응답 결과이다.


## 익스플로잇 코드 작성 및 익스플로잇
코드를 확인했을 떄 `/admin/notice_flag` 페이지에서는 로컬호스트와 userid 파라미터는 admin인지 검사하는 부분이 있었다 그 부분을 충족시키기 위해 아래와 같은 코드를 작성한다
```bash
<img src="/admin/notice_flag?userid=admin" />
```
이 코드는 `/flag` 페이지에 접속해 넣어준다.
![alt text](/assets/img/Dreamhack/CSRF/image2.png)

이 코드를 삽입한 후 good 이라는 alert 메시지가 뜬다 `/memo` 엔드포인트에 접속해 flag값이 정상적으로 찍혀있는지 확인한다
![alt text](/assets/img/Dreamhack/CSRF/image3.png)
정상적으로 flag 값이 찍혀있다.

이 ctf는 일반 유저가 아닌 로컬호스트의 이용자만 이용할 수 있는 기능을 해당 이용자의 의도와 무관하게 실행하도록 하는 CSRF 취약점을 이용한 공격이었다. 이 문제 이외에도 실제 게시판 서비스에서 공격자가 특정 글을 공지사항으로 업로드 하거나, 삭제하는 등의 기능을 실행할 수 도 있으며, 공격자의 계정을 관리자 계정으로 승격시켜주는 행위같이 응용이 가능하다.

