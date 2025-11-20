📦 Windows Server에서 GeoServer 배포 및 UTF-8 문제 해결 과정
1. 개발 서버에서 GeoServer WAR 파일 복사

리눅스 개발 서버에서 사용하던 geoserver.war 파일을 그대로 복사해 Windows Server에 붙여넣음
(RDP 환경: Ctrl + C / Ctrl + V)

2. Tomcat 8.5 설치 파일 복사

Windows에서 실행 가능한 Tomcat 8.5 전체 폴더를 복사해 Windows Server에 배치.

예시 경로

D:\tomcat-8.5.88\

3. Tomcat 포트 설정 변경 (중요)

외부에서 GeoServer에 접속하려면 Tomcat 내부 포트를 변경해주어야 함.

📌 수정 파일
D:\tomcat-8.5.88\conf\server.xml

🔧 기존 설정
<Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />
<Server port="8005" shutdown="SHUTDOWN">

🔧 변경 후
<Connector port="8180" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />
<Server port="8006" shutdown="SHUTDOWN">


Web 포트: 8080 → 8180

Shutdown 포트: 8005 → 8006

4. Windows 방화벽 인바운드·아웃바운드 규칙 설정
✔️ 인바운드 규칙 추가

제어판 → Windows Defender 방화벽

고급 설정

인바운드 규칙 → 새 규칙

규칙 종류: 포트

프로토콜: TCP

특정 로컬 포트: 8180

연결 허용

프로필: 도메인/개인/공용 모두 체크

이름: GeoServer_8180_IN

✔️ 아웃바운드 규칙 추가

같은 방식으로 추가

이름: GeoServer_8180_OUT

이 작업 없으면 외부에서
http://<WindowsServer_IP>:8180/geoserver
접근이 불가능함.

5. Tomcat 실행 및 GeoServer 자동 배포

GeoServer WAR 파일을 webapps에 넣으면 자동 배포됨.

📁 WAR 복사 위치
D:\tomcat-8.5.88\webapps\

▶️ Tomcat 실행
D:\tomcat-8.5.88\bin\startup.bat

📁 자동 생성된 GeoServer 폴더
D:\tomcat-8.5.88\webapps\geoserver\

🌐 접속 테스트
http://<WindowsServer_IP>:8180/geoserver

6. GeoServer에서 Workspace / Store 생성
Workspace 생성 예

Workspace2025

Store 생성

PostgreSQL/PostGIS 연결

레이어 Publish → 지도에서 정상 표시 확인

7. 한글이 들어간 SLD 사용 시 UTF-8 깨짐 문제
❗ 증상

<TextSymbolizer> 또는 <Title>에 한글이 포함되면 오류

영어만 사용할 때는 정상

GeoServer 콘솔에서도 별다른 에러 없음

❗ 원인

Windows에서 실행되는 Tomcat은 기본적으로
file.encoding=MS949 로 실행됨
→ SLD(XML)는 반드시 UTF-8 필요
→ 인코딩 충돌로 한글 파싱 실패

8. Tomcat JVM에 UTF-8 강제 설정 추가
✔️ startup.bat 수정

위치:

D:\tomcat-8.5.88\bin\startup.bat


내용 추가:

set JAVA_OPTS=-Dfile.encoding=UTF-8 -Dsun.jnu.encoding=UTF-8

✔️ (권장) setenv.bat 생성

위치:

D:\tomcat-8.5.88\bin\setenv.bat


내용:

set JAVA_OPTS=-Dfile.encoding=UTF-8 -Dsun.jnu.encoding=UTF-8


setenv.bat 가 있을 경우 startup.bat 보다 우선 적용됨
→ Windows 환경에서는 가장 안전한 방식

9. UTF-8 적용 후 정상 동작 확인

한글이 포함된 SLD 정상 파싱

GeoServer UI에서도 한글 Rule/Title 모두 정상 표시

스타일 적용 문제 없음

10. GeoServer 포인트 아이콘 적용 방법
📁 아이콘 파일 경로
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


또는 URL 기반:

<OnlineResource xlink:href="/geoserver/styles/icon.png" />

✅ 전체 과정 요약

개발 서버에서 geoserver.war 복사

Windows용 Tomcat 8.5 설치

server.xml 포트 변경 (8080→8180, 8005→8006)

방화벽 인바운드/아웃바운드 8180 포트 개방

Tomcat 실행 → GeoServer 자동 배포

Workspace / Store 생성

SLD 한글 깨짐 → Windows JVM 기본 인코딩(MS949) 때문

UTF-8 JVM 옵션 추가로 해결

styles/icon.png 적용 시 포인트 심볼 정상 표시