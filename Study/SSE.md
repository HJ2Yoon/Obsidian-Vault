기존 HTTP 웹 서버에서 **HTTP API** 만으로 동작되며 구현도 간단하다.
HTTP 프로토콜 통신을 이용한다.

## 클라이언트에서 SSE


## 서버에서의 SSE


Server-Sent Events (SSE)는 클라이언트에게 실시간 업데이트를 전송하기 위한 웹 기술 중 하나입니다. Express.js를 사용하여 SSE를 구현하려면 다음과 같은 단계를 따를 수 있습니다.

1. Express 설치:

```bash
npm install express
```

2. Express 애플리케이션 설정:

```javascript
const express = require('express');
const http = require('http');
const path = require('path');

const app = express();
const server = http.createServer(app);

const PORT = process.env.PORT || 3000;

app.use(express.static(path.join(__dirname, 'public')));

// SSE 라우트
app.get('/sse', (req, res) => {
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');

  // 클라이언트에 데이터 전송
  const sendEvent = (data) => {
    res.write(`data: ${JSON.stringify(data)}\n\n`);
  };

  // 예시: 1초마다 현재 시간 전송
  const intervalId = setInterval(() => {
    sendEvent({ time: new Date().toLocaleTimeString() });
  }, 1000);

  // 클라이언트 연결 종료 시 clearInterval
  req.on('close', () => {
    clearInterval(intervalId);
    res.end();
  });
});

// 기본 라우트
app.get('/', (req, res) => {
  res.sendFile(path.join(__dirname, 'public', 'index.html'));
});

// 서버 시작
server.listen(PORT, () => {
  console.log(`Server is running on http://localhost:${PORT}`);
});
```

3. `public/index.html` 파일 작성:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>SSE Example</title>
</head>
<body>
  <h1>Server-Sent Events (SSE) Example</h1>
  <div id="sse-data"></div>

  <script>
    const sseDataElement = document.getElementById('sse-data');
    const eventSource = new EventSource('/sse');

    eventSource.onmessage = (event) => {
      const data = JSON.parse(event.data);
      sseDataElement.innerHTML = `Server Time: ${data.time}`;
    };
  </script>
</body>
</html>
```

위의 코드는 Express를 사용하여 SSE를 구현한 예시입니다. `/sse` 엔드포인트는 SSE를 위한 엔드포인트로 사용되며, 클라이언트는 이 엔드포인트로 연결하여 실시간 업데이트를 수신합니다. 클라이언트는 이벤트 소스를 사용하여 서버로부터 데이터를 수신하고 이를 표시합니다.

이 코드는 간단한 예제이며, 실제 프로덕션 환경에서는 보안 및 성능 고려를 해야 합니다.