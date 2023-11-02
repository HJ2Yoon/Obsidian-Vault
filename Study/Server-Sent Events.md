
## SSE 개요 
클라이언트가 동적인 컨탠츠를 제공하기 위해서는 데이터를 계속 서버로부터 받아와야하는 문제점이 발생한다. 여기서 서버의 데이터가 변경될 때마다 클라이언트가 바로 적용할 수 있는 방법을 고찰

### Short Polling
서버로부터 Rest API로 데이터를 요청하는 방식은 서버의 데이터가 언제 바뀌는 시점을 모르니 **주기적으로 API를 호출해야한다**. 이 방식이 각 클라이언트마다 발생하게 되면 WAS 서버의 무리가 생긴다.

> 기존 Rest API로 서버 요청시 자주 바뀌는 데이터와 같이 요청 주기가 짧으면 클라이언트가 많아질수록 WAS의 부담은 늘어만 간다.

### Server-Sent-Events(SSE)
서버와 한번 연결(Data stream)을 만들면 서버에서 이밴트(변경) 발생할떄 마다 데이터를 전송하는 방법 👉 WebHook 방식과 동일하다.

#### a) Client: SSE subscribe

**Event Stream: Subscribe**
```js
const sse = new EventSource("server url")
// server url = protocol + domain + port
```

#### b) Server: Subscription에 대한 응답

**Event Stream: Fromat**
```js
 res.setHeader("Content-Type", "text/event-stream");
 res.setHeader("Cache-Control", "no-cache");
 res.setHeader("Connection", "keep-alive");
```

이벤트 스트림에서의 Response 미디어 타입은 `text/event-stream`이고 응답을 하고 연결을 끊는 옵션이 아닌 `Connection: keep-alive`으로 연결을 유지한다.

**Evenet Stream: Write Response**
```js
event: type1
name1: value1
name2: value2
\n
event: type2
name1: value1
name2: value2
\n
```

서버는 클라이언트에게 비동기적으로 데이터를 전송할 수 있다. 이때 데이터를 Response Body(본문)에 `UTF-8`로 인코딩된 택스트 데이터만  작성 가능하다. 

한 이밴트의 데이터 필드는 줄바꿈문자 `\n` 한개로 구분하고 서로 다른 이밴트 줄바꿈 문자 `\n\n` 두개로 구분된다. 

## ✨write( )에 대한 이해
res.write( )는 서버가 클라이언트에 Response를 보내주는 역할만 하고 connection을 유지한다. 즉 한번의 요청에 여러 Response에 나눠서 응답을 보내주는 것이다. 

**SSE: Response**
```js
res.write('
	name1: value1

	event: update
	name1: value1
	
')
```

이벤트 스트림이 연결이 지속되는 상태에서 write( )를 통해 Response 본문에 이밴트 내용을 형식에 맞춰서 이밴트 발생시 마다 Response를 클라이언트에 보내는 것이다.

- [!] 단 이밴트 전달을 할시 응답후 **connection을 종료**해버리는 send( ), json( ), sendFile( )은 사용하면 안된다!!

### Express 애플리케이션 설정

```typescript
import express, { Request, Response } from "express";
import { getClientIp } from "request-ip";

const app = express();
const port = 3000;

const clients = new Map<string, Response>();

app.get("/sse", (req: Request, res: Response) => {
  res.setHeader("Content-Type", "text/event-stream");
  res.setHeader("Cache-Control", "no-cache");
  res.setHeader("Connection", "keep-alive");

  const ip = getClientIp(req) ?? "not-found";
  if (!clients.has(ip)) {
    clients.set(ip, res);
    console.log(`📞 Client connected ${ip}`);
  } else {
    console.log("❌ Connected client is already exist");
    return res.end();
  }
  
  req.on("close", () => {
    clients.delete(ip);
    console.log("Current clients list", clients);
  });
});

app.listen(port, "0.0.0.0", () => {
  console.log(`Example app listening at http://localhost:${port}`);
});

// Server side event occur
const sendEvent = (data: { [key: string]: string }) => {
  clients.forEach((client) => {
    client.write(`${JSON.stringfy(data)\n\n}`);
  });
};
```
