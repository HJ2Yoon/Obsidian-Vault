기존 HTTP 웹 서버에서 **HTTP API** 만으로 동작되며 구현도 간단하다.
HTTP 프로토콜 통신을 이용한다.

### Express 애플리케이션 설정:

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
    res.end();
  }

  res.send("Attempt to try SSE!");  

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
    client.write(data);
  });
};
```

위의 코드는 Express를 사용하여 SSE를 구현한 예시입니다. `/sse` 엔드포인트는 SSE를 위한 엔드포인트로 사용되며, 클라이언트는 이 엔드포인트로 연결하여 실시간 업데이트를 수신합니다. 클라이언트는 이벤트 소스를 사용하여 서버로부터 데이터를 수신하고 이를 표시합니다.

이 코드는 간단한 예제이며, 실제 프로덕션 환경에서는 보안 및 성능 고려를 해야 합니다.