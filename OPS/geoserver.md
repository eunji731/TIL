🗂️ Windows Server에서 GeoServer 배포 및 UTF-8 문제 해결 과정
1. 개발 서버에서 GeoServer WAR 파일 복사

리눅스 개발 서버에서 사용하던 geoserver.war 파일을 복사해 Windows Server에 붙여넣음
(RDP 환경: Ctrl + C / Ctrl + V)

2. Tomcat 8.5 설치 파일 복사

Windows에서 실행 가능한 Tomcat 8.5 전체 폴더를 복사해 Windows Server에 배치

예시 경로:

D:\tomcat-8.5.88\

3. Tomcat 포트 설정 변경 (중요)

외부에서 GeoServer에 접근 가능하도록 Tomcat 내부 포트 변경 작업 수행.

📌 수정 파일
D:\tomcat-8.5.88\conf\server.xml

🔧 기존 설정
<Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />
<Server port="8005" shutdown="SHUTDOWN">

🔧 변경 후 설정
<Connector port="8180" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />
<Server port="8006" shutdown="SHUTDOWN">


Web 포트: 8080 → 8180

Shutdown 포트: 8005 → 8006

이유: 기존 서버 포트 충돌 방지 및 외부 접근 가능성 확보

4. Windows 방화벽 인바운드 & 아웃바운드 규칙 설정

GeoServer(8180 포트) 외부 접속을 위해 방화벽 규칙 추가가 필요함.

✔️ 인바운드 규칙 추가

제어판 → Windows Defender 방화벽 → 고급 설정

인바운드 규칙 → 새 규칙

규칙 종류: 포트

프로토콜: TCP

특정 로컬 포트: 8180

연결 허용

프로필: 도메인 / 개인 / 공용 모두 선택

이름: GeoServer_8180_IN

✔️ 아웃바운드 규칙 추가

동일한 절차

이름: GeoServer_8180_OUT

🔥 이 작업 없으면 외부(내 로컬)에서
http://<WindowsServer_IP>:8180/geoserver 접근 불가

5. Tomcat 실행 및 GeoServer 자동 배포

GeoServer WAR 파일을 Tomcat에 넣으면 자동 배포됨.

✔️ WAR 배치 위치
D:\tomcat-8.5.88\webapps\geoserver.war

✔️ Tomcat 실행
D:\tomcat-8.5.88\bin\startup.bat

✔️ 배포 결과
D:\tomcat-8.5.88\webapps\geoserver\

✔️ 접속 확인
http://<WindowsServer_IP>:8180/geoserver

6. GeoServer에서 Workspace & Store 생성
Workspace 생성 예시

이름: Workspace2025

Store 생성

PostgreSQL/PostGIS 정보 입력

레이어 Publish → 정상 표시되는지 검증

7. SLD에서 한글 포함 시 UTF-8 깨짐 문제
❗ 발생한 문제

<TextSymbolizer> 또는 <Title>에 한글이 포함되면 SLD 반영 실패

영어는 정상 적용됨

콘솔 오류 없음 → 인코딩 문제 가능성 높음

🎯 원인

Windows Tomcat은 기본적으로
file.encoding=MS949 (CP949)
로 실행됨

→ SLD(XML)는 반드시 UTF-8이어야 함
→ 인코딩 불일치로 한글 파싱 실패

8. Tomcat JVM에 UTF-8 강제 설정 추가
✔️ 방법 1) startup.bat 수정

위치:

D:\tomcat-8.5.88\bin\startup.bat


상단에 추가:

set JAVA_OPTS=-Dfile.encoding=UTF-8 -Dsun.jnu.encoding=UTF-8

✔️ 방법 2) setenv.bat 사용 (권장)

파일 생성:

D:\tomcat-8.5.88\bin\setenv.bat


내용:

set JAVA_OPTS=-Dfile.encoding=UTF-8 -Dsun.jnu.encoding=UTF-8


✔ setenv.bat가 있으면 startup.bat보다 우선 적용됨
✔ Windows 환경에서는 setenv.bat로 관리하는 것이 가장 안전

9. UTF-8 환경에서 SLD 정상 동작

Tomcat 재시작 후 한글 SLD 정상 렌더링

한글 <Title> <Label> 모두 정상 출력

스타일 적용 문제 없음

10. GeoServer 포인트 심볼 아이콘 적용

GeoServer는 data/styles 폴더의 이미지 파일을 직접 참조할 수 있음.

📁 아이콘 위치
D:\tomcat-8.5.88\webapps\geoserver\data\styles\icon.png

📌 SLD 적용 예시
<PointSymbolizer>
  <Graphic>
    <ExternalGraphic>
      <OnlineResource xlink:href="file:data/styles/icon.png" />
      <Format>image/png</Format>
    </ExternalGraphic>
    <Size>20</Size>
  </Graphic>
</PointSymbolizer>

또는 URL 기반
<OnlineResource xlink:href="/geoserver/styles/icon.png" />

📌 전체 과정 요약
🔹 1) geoserver.war 복사
🔹 2) Windows용 Tomcat 8.5 복사
🔹 3) 포트 변경 (8080→8180 / 8005→8006)
🔹 4) 방화벽에서 8180(TCP) 인/아웃바운드 허용
🔹 5) Tomcat 실행 → GeoServer 자동 배포
🔹 6) Workspace & Store 생성 후 레이어 정상 동작
🔹 7) SLD 한글 깨짐 → Windows JVM 기본 인코딩(MS949) 때문
🔹 8) JVM 옵션에 UTF-8 강제 설정 추가하여 해결
🔹 9) styles/icon.png 적용하여 포인트 아이콘 정상 표시
🔹 10) 모든 스타일/한글/아이콘 정상 적용 완료