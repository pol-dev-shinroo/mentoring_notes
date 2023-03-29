# mentoring_notes

1. apis 구분하기
- view functions : 패이지를 보여주는 apis 
- api functions : CRUD 기능을 하는 apis. (기본 url이 "/api"이면 좋습니다.)
예시: "/api/stopwatch" - stopwatch 기능을 담당하는 api.

2. app.py 폴더 구분하기 (옵션입니다 필수 아닙니다!)
예시:

```py
# export 시킬 파일: test.py 
from flask import Flask, jsonify

app = Flask(__name__)

# API route to get a message
@app.route('/message', methods=['GET'])
def get_message():
    return jsonify({'message': 'This is a test message'})

# API route to get a number
@app.route('/number', methods=['GET'])
def get_number():
    return jsonify({'number': 42})
```

```py
# app.py에서

from flask import Flask
from test import get_message, get_number

app = Flask(__name__)

# Register the message API route
app.add_url_rule('/message', endpoint='message', view_func=get_message, methods=['GET'])

# Register the number API route
app.add_url_rule('/number', endpoint='number', view_func=get_number, methods=['GET'])
``` 

add_url_rule 이라는 메서드로 타파일에서 만들어준 api 함수들을
app.py에서 설정해주시면 됩니다. 

3. 혹시 jwt token 사용하신다면 아래 링크를 참조 하셔도 됩니다.
링크: https://github.com/pol-dev-shinroo/IntroDog/blob/main/app.py

클라언트 쪽 로직
- 로그인 
```js
$('#login-form').on('submit', function(e) {
    e.preventDefault();
    var data = new FormData(this);
    fetch('/api/login', {
        method: 'POST',
        body: data
    })
    .then(response => response.json())
    .then(data => {
        // 백엔드에서 데이터배이스 비밀번호 인증시
        // 쿠키에 토큰을 저장합니다.
        document.cookie = "token=" + data.token + "; path=/";
        // 저장후 실행될 로직...(홈페이지로 이동?) 
    })
    .catch(error => {
        console.error(error);
    });
});
```
- 토큰인증확인 (모든 페이지에서 가능)
 ```js
$(document).ready(function() {
    // 브라우저에 html업로드 되면 토큰 가져오기
    // getCookie는 아래에 만들어져 있습니다.
    // 왜 토큰애 통상적으로 저장하는지는 다른 저장소 (ex localstorage) 와 비교해보시기 바랍니다.
    var token = getCookie('token');
    if (token) {
        // 토큰 같은 키는 보통 headers 로 보냅니다. 
        fetch('/api/test_token', {
            method: 'POST',
            headers: { 'Authorization': 'Bearer ' + token }
        })
        .then(response => response.json())
        .then(data => {
            // 성공시 보통 유저 정보를 가져옵니다. 
            console.log(data)
            
        })
        .catch(error => {
            console.error(error);
            // 로그인 인증이 안되면 로그인 패이지로 이동?
        });
    } else {
        // 쿠키에 저장된 토큰이 없으므로 로그인 패이지로 이동?
    }

    // 이름으로 쿠키 값 찾는 함수
    function getCookie(name) {
        var cookieArr = document.cookie.split(";");

        for (var i = 0; i < cookieArr.length; i++) {
            var cookiePair = cookieArr[i].split("=");
            if (name == cookiePair[0].trim()) {
                return decodeURIComponent(cookiePair[1]);
            }
        }

        // Return null if cookie not found
        return null;
    }
});

```

백엔드쪽 로직
```py
from flask import Flask, request, jsonify
import jwt
import datetime

app = Flask(__name__)

app.config['SECRET_KEY'] = 'mysecretkey'

# 데이터 베이스 목데이터입니다. 해싱 구현하신거 적용하면 됩니다!
users = {
    'john': 'password123',
    'jane': 'qwerty456',
    'bob': 'abcdef789'
}

# Login route
@app.route('/api/login', methods=['POST'])
def login():
    username = request.form.get('username')
    password = request.form.get('password')
    
    if username not in users or users[username] != password:
        return jsonify({'message': 'Invalid credentials'})
    
    # Generate JWT token
    token = jwt.encode({'user': username, 'exp': datetime.datetime.utcnow() + datetime.timedelta(minutes=30)}, app.config['SECRET_KEY'])
    
    return jsonify({'token': token})

# Test token route
@app.route('/api/test_token', methods=['POST'])
def test_token():
    token = request.headers.get('Authorization').split(' ')[1]
    
    try:
        data = jwt.decode(token, app.config['SECRET_KEY'], algorithms=['HS256'])
        return jsonify({'message': 'Token is valid', 'user': data['user']})
    except jwt.ExpiredSignatureError:
        return jsonify({'message': 'Token has expired'}), 401
    except jwt.InvalidTokenError:
        return jsonify({'message': 'Invalid token'}), 401

if __name__ == '__main__':
    app.run(debug=True)
```
