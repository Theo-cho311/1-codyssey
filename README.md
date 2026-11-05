\# Docker 개발 환경 구축 실습



\## 프로젝트 목표

터미널 기본 조작과 Linux 권한을 익히고, Docker로 웹 서버를 실행한다.

포트 매핑, 바인드 마운트, 볼륨의 동작을 직접 확인한다.



\## 실행 환경

\- PC: 캠퍼스 Windows 공용 PC

\- 셸: Windows PowerShell, Ubuntu 컨테이너 내부 Bash

\- Git: 2.56.0.windows.4

\- Docker: 29.9.2

\- Linux 실습 이미지: ubuntu:24.04

\- 웹 서버 베이스 이미지: nginx:alpine



\## 수행 체크리스트

\- \[x] 터미널 파일·폴더 기본 조작

\- \[x] Linux 파일·폴더 권한 변경

\- \[x] Docker 설치 및 엔진 점검

\- \[x] hello-world 실행

\- \[x] Ubuntu 내부 명령 및 exec/attach 비교

\- \[x] Dockerfile 작성 및 이미지 빌드

\- \[x] 포트 매핑과 브라우저 접속

\- \[x] 바인드 마운트 변경 반영

\- \[x] 컨테이너 삭제 후 볼륨 데이터 유지

\- \[x] 컨테이너 로그 및 자원 사용량 확인

\- \[x] Git 저장소 생성 및 작성자 설정

\- \[x] 실행·검증 방법과 증거 정리

\- \[x] 트러블슈팅 정리

\- \[x] VSCode GitHub 연동

\- \[x] GitHub 업로드



\## 웹 서버 접속 증거



\### 이미지에 복사한 웹페이지 — 8080

!\[8080 웹페이지](web-8080.png)



\### 바인드 마운트로 변경한 웹페이지 — 8081

!\[바인드 마운트 변경 후](bind-after.png)



\## 웹 서버 실행 방법



Docker Desktop을 실행한 뒤, PowerShell에서 이 저장소 폴더로 이동한다.



\### 이미지 빌드

```powershell

docker build -t codyssey-web:1.0 .

```



\### 컨테이너 실행

```powershell

docker run -d --name codyssey-web -p 8080:80 codyssey-web:1.0

```



\### 접속 확인

브라우저에서 http://localhost:8080 에 접속한다.



\- 베이스 이미지: nginx:alpine

\- 커스텀 내용: 기본 웹페이지를 직접 작성한 site/index.html로 교체했다.

\- 포트 연결: Windows의 8080번을 컨테이너의 80번에 연결했다.

\- 같은 이름의 컨테이너가 이미 있으면 새로 만들기 전에 기존 상태를 확인한다.



\## 바인드 마운트 검증



PowerShell에서 저장소 폴더를 현재 위치로 둔 뒤 실행했다.



```powershell

docker run -d --name codyssey-bind -p 8081:80 --mount "type=bind,source=$($PWD.Path)\\site,target=/usr/share/nginx/html,readonly" codyssey-web:1.0

```



\- source: Windows의 site 폴더

\- target: 컨테이너의 웹페이지 제공 폴더

\- readonly: 컨테이너에서는 읽기만 허용



\### 관찰 결과

1\. Windows의 HTML을 수정해도 기존 8080 페이지는 바뀌지 않았다.

&#x20;  이미지 빌드 때 파일을 복사했기 때문이다.

2\. 바인드 마운트로 실행한 8081에서는 Updated page!가 보였다.

3\. HTML을 Bind mount works!로 수정하고 새로고침하자 변경이 반영됐다.

4\. 이미지 재빌드나 컨테이너 재시작은 하지 않았다.



변경 후 화면은 위의 바인드 마운트 접속 증거를 참고한다.



\## 볼륨 데이터 유지 검증



\### 1. 볼륨 생성

```powershell

docker volume create codyssey-data

```



\### 2. 볼륨에 파일 저장 및 확인

```powershell

docker run --name volume-write --mount type=volume,source=codyssey-data,target=/data ubuntu:24.04 bash -c "echo persistent-data > /data/hello.txt; cat /data/hello.txt"

```



출력:

```text

persistent-data

```



\### 3. 데이터를 쓴 컨테이너 삭제

```powershell

docker rm volume-write

```



\### 4. 새 컨테이너에서 같은 볼륨 읽기

```powershell

docker run --rm --mount type=volume,source=codyssey-data,target=/data ubuntu:24.04 cat /data/hello.txt

```



출력:

```text

persistent-data

```



\### 결론

기존 컨테이너를 삭제한 뒤에도 새 컨테이너에서 같은 데이터를 읽었다.

컨테이너와 볼륨의 수명은 별개이며, 이 실험에서는 볼륨을 삭제하지 않았다.



\## 트러블슈팅



\### 1. PowerShell 명령 오타



\- 문제: set-conent를 입력하자 명령을 찾을 수 없다는 오류가 나왔다.

\- 원인 가설: 명령 이름이나 옵션을 잘못 입력했을 수 있다.

\- 확인: Set-Content에서 t가 빠졌고, Path 앞의 하이픈도 누락됐다.

\- 해결: Set-Content -Path hello.txt -Value "hello, codyssey!"로 수정했다.

\- 결과: 오류 없이 실행됐고 Get-Content로 저장한 글을 확인했다.



\### 2. Docker 명령을 찾을 수 없음



\- 문제: docker --version 실행 시 명령을 인식하지 못했다.

\- 원인 가설: Docker가 설치되지 않았거나 명령 검색 경로에 등록되지 않았을 수 있다.

\- 확인: 시작 메뉴에서도 Docker Desktop 앱을 찾지 못했다.

\- 해결: 캠퍼스의 설치 허용을 확인한 뒤 Docker Desktop을 설치하고 실행했다.

&#x20; PowerShell도 새로 열었다.

\- 결과: docker --version과 docker info가 정상 실행됐고,

&#x20; hello-world 컨테이너에서 Hello from Docker!를 확인했다.



\## 실행 기록



\[PowerShell 명령 및 출력 기록 보기](mission-log.txt)



기록은 Start-Transcript를 실행한 시점부터 저장했다.

그 이전에 수행한 터미널 기본 조작, Linux 권한 변경 등의 출력은 이 파일에 포함되지 않는다.

공개용 기록에서 이메일 주소는 가림 처리했다.



\## VSCode와 GitHub 연동



VSCode에서 과제 폴더를 열고 main 브랜치가 인식되는 것을 확인했다.

계정 메뉴에서 본인의 GitHub 로그인 상태도 확인했다.



!\[VSCode GitHub 연동 확인](vscode-github.png)



\## Linux 권한 변경 검증



Ubuntu 컨테이너의 파일과 폴더 권한을 변경하고 전후를 확인했다.



| 대상 | 변경 전 | 변경 후 |

|---|---|---|

| /root/practice 폴더 | 700: drwx------ | 755: drwxr-xr-x |

| /root/practice/hello.txt 파일 | 600: -rw------- | 644: -rw-r--r-- |



권한 숫자는 읽기 4, 쓰기 2, 실행 1을 더한 값이다.

세 자리는 소유자, 그룹, 그 외 사용자 순서다.

폴더의 실행 권한은 진입과 내부 항목 접근을 뜻한다.



\[권한 변경 명령 및 실제 출력 기록](verification-log.txt)





