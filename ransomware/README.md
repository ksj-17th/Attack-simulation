# 랜섬웨어

https://github.com/lawndoc/RanSim

## installer.ps1

### 개요

`installer.ps1`은 AES를 사용하여 지정된 경로의 파일을 재귀적으로 암호화하거나 복호화하는 Ransomware Simulator이다.

스크립트는 통제된 환경에서 랜섬웨어에 대한 방어 및 백업을 테스트하기 위한 용도로 작성되어 있으며, 시뮬레이션용 데이터를 암호화하고 필요한 경우 동일한 스크립트로 다시 복호화할 수 있도록 구성되어 있다.

## 실행 모드

스크립트는 다음 두 가지 모드를 처리한다.

- `encrypt`
- `decrypt`

모드를 지정하지 않을 경우 encrypt 또는 decrypt 모드를 선택해야 한다는 오류 메시지를 출력한다.

---

## Parameters

### Target File Path

재귀적인 암호화를 시작할 경로를 지정한다.

기본 경로:

```text
C:\Users
```

### File Extension

암호화된 파일에 추가할 확장자를 지정한다.

기본 확장자:

```text
.encrypted
```

### Encryption Key

암호화와 복호화에 공통으로 사용하는 평문 AES 암호화 키를 지정한다.

스크립트에는 기본 키가 제공되어 있다.

---

## Target Files

암호화 대상으로 지정된 파일 형식은 다음과 같다.

```text
*.pdf
*.xls*
*.ppt*
*.doc*
*.accd*
*.rtf
*.txt
*.csv
*.jpg
*.jpeg
*.png
*.gif
*.avi
*.midi
*.mov
*.mp3
*.mp4
*.mpeg
*.mpeg2
*.mpeg3
*.mpg
*.ogg
*.json
*.php*
*.jsp*
*.bin
*.zip
*.exe
*.py*
```

스크립트에는 모든 파일을 대상으로 지정하기 위한 다음 설정도 주석으로 포함되어 있다.

```text
$TargetFiles = '*'
```

---

# Cryptography Functions

기존 `FileCryptography.psm1`의 암호화 관련 기능이 `installer.ps1`에 병합되어 있다.

## New-CryptographyKey

랜덤 암호화 키를 생성하는 함수이다.

### 지원 Algorithm

```text
AES
DES
RC2
Rijndael
TripleDES
```

기본 Algorithm은 다음과 같다.

```text
AES
```

### Parameters

#### Algorithm

키를 생성할 암호화 Algorithm을 지정한다.

#### KeySize

생성할 키의 bit 크기를 지정한다.

#### AsPlainText

지정할 경우 `SecureString` 대신 문자열 형태로 키를 반환한다.

### Outputs

기본적으로 다음 형식을 반환한다.

```text
System.Security.SecureString
```

`AsPlainText`를 사용할 경우 다음 형식을 반환한다.

```text
System.String
```

### Notes

```text
Author: Tyler Siegrist
Date: 9/22/2017
```

---

# Protect-File

대칭키 Algorithm을 사용하여 파일을 암호화하는 함수이다.

## Parameters

### FileName

암호화할 파일을 지정한다.

### Key

암호화에 사용할 `SecureString` 형식의 Cryptography Key를 지정한다.

### KeyAsPlainText

암호화에 사용할 문자열 형식의 Cryptography Key를 지정한다.

### CipherMode

암호화 과정에서 사용할 Block Cipher Mode를 지정한다.

### PaddingMode

Cryptographic operation에 필요한 전체 byte 수보다 message data block이 짧을 경우 적용할 Padding 방식을 지정한다.

### Suffix

암호화된 파일에 사용할 Suffix를 지정한다.

기본값은 선택된 Algorithm 이름을 기반으로 한다.

### RemoveSource

암호화 이후 원본 파일을 제거한다.

---

## Protect-File 동작

암호화 과정에서는 다음 작업이 수행된다.

1. 지정된 파일을 가져온다.
2. 암호화 결과 파일의 경로를 생성한다.
3. 원본 파일과 대상 파일의 FileStream을 연다.
4. 새로운 IV(Initialization Vector)를 생성한다.
5. 암호화 파일에 IV 길이와 IV를 기록한다.
6. Encryptor를 생성한다.
7. `CryptoStream`을 통해 파일 데이터를 암호화한다.
8. 암호화 작업이 완료되면 열린 Stream을 종료한다.
9. `RemoveSource`가 지정된 경우 암호화되지 않은 원본 파일을 제거한다.
10. 암호화된 파일 정보를 결과로 반환한다.

반환되는 파일 정보에는 다음 항목이 추가된다.

```text
SourceFile
Algorithm
Key
CipherMode
PaddingMode
```

암호화 과정에서 오류가 발생하면 실패한 대상 파일을 제거하고 다음 파일 처리를 계속한다.

### Notes

```text
Author: Tyler Siegrist
Date: 9/22/2017
```

---

# Unprotect-File

`Protect-File`을 통해 암호화된 파일을 복호화하는 함수이다.

## Parameters

### FileName

복호화할 파일을 지정한다.

### Key

복호화에 사용할 `SecureString` 형식의 Cryptography Key를 지정한다.

### KeyAsPlainText

복호화에 사용할 문자열 형식의 Cryptography Key를 지정한다.

### CipherMode

암호화에 사용했던 Block Cipher Mode를 지정한다.

기본값:

```text
CBC
```

### PaddingMode

암호화에 사용했던 Padding 방식을 지정한다.

기본값:

```text
PKCS7
```

### Suffix

암호화된 파일에 사용된 Suffix를 지정한다.

Suffix를 별도로 지정하지 않으면 Algorithm 이름을 이용한다.

### RemoveSource

복호화가 완료된 이후 암호화된 원본 파일을 제거한다.

---

## Unprotect-File 동작

복호화 과정에서는 다음 작업이 수행된다.

1. Cryptography Key를 설정한다.
2. 선택된 Algorithm을 생성한다.
3. Cipher Mode와 Padding Mode를 설정한다.
4. 암호화 키의 크기와 값을 설정한다.
5. Suffix가 지정되지 않은 경우 Algorithm을 이용하여 Suffix를 결정한다.
6. 대상 파일의 이름이 지정된 Suffix로 끝나는지 확인한다.
7. Suffix를 제거하여 복호화된 파일의 경로를 생성한다.
8. 암호화된 파일과 복호화 결과 파일의 FileStream을 연다.
9. 암호화된 파일에서 IV 길이를 읽는다.
10. 저장되어 있는 IV를 읽는다.
11. 해당 IV를 Cryptography 객체에 설정한다.
12. Decryptor를 생성한다.
13. `CryptoStream`을 통해 데이터를 복호화한다.
14. 복호화 작업이 완료되면 열린 Stream을 종료한다.
15. `RemoveSource`가 지정된 경우 암호화된 원본 파일을 제거한다.
16. 복호화된 파일 정보를 반환한다.

반환되는 파일 정보에는 다음 항목이 추가된다.

```text
SourceFile
```

복호화 과정에서 오류가 발생하면 실패한 대상 파일을 제거하고 다음 파일 처리를 계속한다.

### Notes

```text
Author: Tyler Siegrist
Date: 9/22/2017
```

---

# Encrypt Mode

`mode`가 `encrypt`인 경우 지정된 Target Path와 하위 디렉터리에서 대상 파일을 재귀적으로 검색한다.

검색 과정에서 이미 암호화 확장자가 적용된 파일은 제외한다.

디렉터리는 대상에서 제외한다.

검색된 파일의 수를 저장한 후 각각의 파일에 대해 암호화를 수행한다.

다음 두 파일은 암호화 처리에서 제외한다.

```text
joke.exe
joker.png
```

파일 암호화에는 다음 설정이 사용된다.

```text
Algorithm: AES
Suffix: Extension parameter
RemoveSource: enabled
```

처리가 끝나면 암호화 대상으로 확인된 파일 수를 출력한다.

---

# Decrypt Mode

`mode`가 `decrypt`인 경우 지정된 Target Path와 하위 디렉터리에서 암호화 확장자가 붙은 파일을 재귀적으로 검색한다.

디렉터리는 대상에서 제외한다.

각 파일에 대해 복호화를 수행하며 다음 설정을 사용한다.

```text
Algorithm: AES
Suffix: Extension parameter
RemoveSource: enabled
```

---

# installer.iss

`installer.iss`에는 Windows 레지스트리 Run Key 등록을 통한 지속성 확보 설정이 포함되어 있다.

## 1. 레지스트리 Run Key 등록을 통한 지속성 확보

사용되는 Windows 레지스트리 위치:

```text
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
```

해당 레지스트리 키에 새로운 값을 추가한다.

현재 사용자가 Windows에 로그인할 때마다 다음 파일이 자동 실행되도록 등록하는 방식이다.

```text
%TEMP%\joke.exe
```

이를 통해 지속성(Persistence)을 확보한다.

설정에는 다음 항목도 포함되어 있다.

```text
WorkingDir: %TEMP%
StatusMsg: 적절한 인터페이스를 구성 중입니다...
Flags: nowait runhidden
```

---

## 2. FileCryptography.psm1 병합

기존의 `FileCryptography.psm1` 파일에 있던 암호화 기능을 `installer.ps1`에 병합한다.
