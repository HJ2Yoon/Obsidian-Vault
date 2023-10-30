#### cros origin 정책
url 로 다른 서버에 올려진 img에 접근하는 경우, css를 접근하는 경우에는 cross-origin 정책이 지원함

다만 *fetch api(ajax method api call)*, *XMLHttpRequest* 같은 경우

``` js
// <img>는 cross-origin 정책이 지원
<img src="https://third-party-test.glitch.me/check.svg" alt="이미지">
// fetch type은 CORS에러 발생
fetch('https://third-party-test.glitch.me/check.svg')
```

*fetch*, *XMLHttpRequest* 왜 CORS에러가 발생
교차 **출처** 리소스 공유 정책 (출처 = ORIGIN)

**CORS 에러가 하면 출처가 서로 달라 발생하는 에러**
Cross Origin Resource sharing
통신, 접근하고 있는 리소스의 출처가 엇갈렸다라는 뜻 즉 출처가 서로 다르다라는 뜻

> 출처로 가져올 수 있는 액세스가 CORS 정책에 의해 차단되었습니다. 요청된 리소스에 'Access-Control-Allow-Origin' 헤더가 없습니다. 불투명한 응답이 필요에 적합한 경우, 요청 모드를 'no-cors'로 설정하여 CORS가 비활성화된 리소스를 가져오십시오.

Access-Control-Allow-Origin' 헤더가 없습니다.

URL 구성을 보면
HTTPS://www.domain.com:3000/route?name='xx'&page=1#first

Protocol + Host + Port 까지가 Origin(출처)이다 

**출처가 달라서 CORS정책에 차단된 것이면 출처를 같게 만들면 된다**

How? 
Same-Origin-Policy
출처가 같다는 것은 같은 서버 즉 서버 내부에 있는 리소스만을 신뢰하고 공유할 수 있다.
외부서버 리소스는 접근이 불가능하다.

보안적 문제로 신뢰할 수 없는 리소스들은 가져올(fetch)수 없게 만든다 SOP정책 따라야한다.

#### 출처 비교는 브라우저가 한다

>브라우저는 출처가 다르면 내가 요청한 리소스가 아니라며 CORS에러를 내면서 서버에서 온 응답을 거부한다.

대부분 문제는 서버에서 헤더 정보를 안 줘서 문제가 생김 출처에대한 정보를 응답 헤더에 안 담아서 문제가 발생 

이 응답은 무슨 출처의 요청으로부터의 응답입니다. 라는 정보를 헤더에 담아야한다
req에서 어디서 왔는지에 대한 출처를 가겨와서 응답할때 헤더에 담아줘야 에러를 발생 

- [!] CORS에러는 브라우저가 판단하니 서버간 요청은 에러가 발생하지 않는다.  서버간 통신은 CORS정책으로부터 자유로워진다 이를 이용한 프록시 서버

#### SOP정책을 회피하는 방법

##### CORS 기본 시퀀스

>클라이언트  응답 아무거나를  받는 것이 아니고 / 서버도 요청 아무거나 응답하는 것이 아니다 .

*Access-Control-Allow-Origin 필드에 있는 서버가 허용한 출처에 해당하면 => 시퀀스: 헤더에 요청에 담긴 출처를 그대로 담아서 보내준다. => 서버에서 받아주는 출처  

1. 클라이언트에서 HTTP요청의 헤더에 Origin을 담아 누가 요청했는지에 대한 출처를 남긴다
2. 서버는 응답 헤더에 너가 보낸 요청에 대한 응답이야 라고 헤더에 요청에 담긴 출처를 그대로 담아서 보내준다 (Acess-Control-Allow-Origin 필드에 포함된 출처라면)
3. 응답헤더에서 받아온 Acess-Control-Allow-Origin을 같은지 비교 => SOP

일반적으로 Acess-Control-Allow-Origin 필드에 요청에서 가져온 Origin 포함이 안되서 생긴 문제이다. CORS 설정은 잊지말고 백엔드에서 설정해줘야한다. 서버에서 해당 요청의 출처 허용 필요

#### Express CORS 설정
```js
const cors = require("cors"); // cors 설정을 편안하게 하는 패키지
const app = express();

app.use(cors({
    origin: "https://naver.com", // 접근 권한을 부여하는 도메인
    credentials: true, // 응답 헤더에 Access-Control-Allow-Credentials 추가
    optionsSuccessStatus: 200, // 응답 상태 200으로 설정
}));
```

##### 예비요청이란?

본요청을 보내기 전에 안전한 요청인지 확인한 후 본요청을 보냄
예비요청은 OPTIONS 메소드를 사용 => 보안적으로 뛰어나다

단점으로는 요청을 두번 보내기때문에 요청시간이 길어진다 따라서 인증토큰처럼 매번 인증하는 것이 아니라 예비요청을 유효기간을 설정하는 것  => 예비요청 응답을 캐싱하는 것 유효기간이 나면 다시 인증해 갱신해야함 JWT 토큰 시퀀스와 같다.

Access-Control-Max-Age 예비요청 응답에 유효기간을 설정 지나면 갱신

#### Access-Control
```js
// 헤더에 작성된 출처만 브라우저가 리소스를 접근할 수 있도록 허용함(서버가).
// * 이면 모든 곳에 공개되어 있음을 의미한다. 
Access-Control-Allow-Origin : https://naver.com

// 리소스 접근을 허용하는 HTTP 메서드를 지정해 주는 헤더
Access-Control-Request-Methods : GET, POST, PUT, DELETE

// 요청을 허용하는 해더.
Access-Control-Allow-Headers : Origin,Accept,X-Requested-With,Content-Type,Access-Control-Request-Method,Access-Control-Request-Headers,Authorization

//  클라이언트에서 preflight 의 요청 결과를 저장할 기간을 지정
// 60초 동안 preflight 요청을 캐시하는 설정으로, 첫 요청 이후 60초 동안은 OPTIONS 메소드를 사용하는 예비 요청을 보내지 않는다.
Access-Control-Max-Age : 60

// 클라이언트 요청이 쿠키를 통해서 자격 증명을 해야 하는 경우에 true. 
// 자바스크립트 요청에서 credentials가 include일 때 요청에 대한 응답을 할 수 있는지를 나타낸다.
Access-Control-Allow-Credentials : true

// 기본적으로 브라우저에게 노출이 되지 않지만, 브라우저 측에서 접근할 수 있게 허용해주는 헤더를 지정
Access-Control-Expose-Headers : Content-Length
```

- [!] Access-Control-Allow-Credentials: 인증된 요청일때 자격인증을 담는 응답헤더 필드

##### 인증요청이란? 
>클라이언트에 대한 서버 접근 제한 CORS 반대 CORS는 서버의 응답을 제한하지만 Credentials는 클라이언트가 자격이 있는지 판단한다. 

*별도의 옵션 없이 브라우저의 쿠키와 같은 인증과 관련된 데이터를 함부로 요청 데이터에 담지 않도록 되어있다*

클라이언트에서 서버에 요청시 헤더에  `withCredentials: true` 설정을 통해 요청 헤더에 자동으로 쿠키(인증 정보)를 담아 요청한다.

서버에서도 민감한 정보( 토큰, API key)를 포함한 응답을 할것인지 동의를 해야한다. => 서버입장에서는 응답에 민감한 정보를 담는 것에 대한 동의

*서버에서도  Access-Control-Allow-Credentials 헤더를 true로 함으로써 인증 옵션을 설정해주어야 한다.*
#### 서버에서 인증된 요청의 처리
응답 헤더의 Access-Control-Allow-Credentials 항목을 true로 설정해야 한다. => *인증된 클라이언트에 대한 요청의 응답입니다라고 알려준 것 

"인증된 클라이언트라는" 아주 제한적인 대상이기 떄문에 모든 출처를 허용한다라는 뜻에 와일드카드를 사용하는 것이 아니라 특정(정확하게 명시) 해야한다.
