📦 Windows Server에서 GeoServer 배포 및 UTF-8 문제 해결 과정
1. 개발 서버에서 GeoServer WAR 파일 복사

리눅스 개발 서버에서 사용하던 geoserver.war 파일을 복사해 Windows Server에 붙여넣음.
(RDP 환경: Ctrl + C / Ctrl + V)

2. Tomcat 8.5 설치 파일 복사

Windows에서 실행 가능한 Tomcat 8.5 전체 폴더를 복사해 Windows Server에 배치.

예시 경로:

D:\tomcat-8.5.88\

3. Tomcat 포트 설정 변경 (중요)

외부에서 GeoServer에 접속하려면 Tomcat 내부 포트를 변경해야 함.

수정 파일
D:\tomcat-8.5.88\conf\server.xml

기존 설정
<Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />
<Server port="8005" shutdown="SHUTDOWN">

변경 후
<Connector port="8180" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />
<Server port="8006" shutdown="SHUTDOWN">


Web 포트: 8080 → 8180

Shutdown 포트: 8005 → 8006

4. Windows 방화벽 인바운드·아웃바운드 규칙 설정
인바운드 규칙 추가

제어판 → Windows Defender 방화벽

고급 설정

인바운드 규칙 → 새 규칙

규칙 종류: 포트

프로토콜: TCP

특정 로컬 포트: 8180

연결 허용

프로필: 도메인/개인/공용

이름: GeoServer_8180_IN

아웃바운드 규칙 추가

동일한 방식으로 생성

이름: GeoServer_8180_OUT

이 작업 없으면 외부에서
http://<WindowsServer_IP>:8180/geoserver
접근이 불가능함

5. Tomcat 실행 및 GeoServer 자동 배포

GeoServer WAR 파일을 webapps에 넣으면 자동 배포됨.

WAR 복사 위치
D:\tomcat-8.5.88\webapps\geoserver.war

Tomcat 실행
D:\tomcat-8.5.88\bin\startup.bat

자동 생성된 GeoServer 폴더
D:\tomcat-8.5.88\webapps\geoserver\

접속 확인
http://<WindowsServer_IP>:8180/geoserver

6. GeoServer Workspace / Store 생성
Workspace 생성

예: Workspace2025

Store 생성

PostgreSQL 연결

PostGIS 레이어 정상 확인

Publish 후 지도에서 정상 렌더링되는지 확인

7. SLD 한글 포함 시 UTF-8 깨짐 문제
증상

<TextSymbolizer> 또는 <Title> 에 한글 포함하면 SLD 적용 실패

영어는 정상

GeoServer 로그에도 명확한 에러 없음

원인

Windows Tomcat 기본 인코딩이
MS949(CP949)
SLD(XML)은 UTF-8 필요

8. JVM에 UTF-8 강제 설정 추가
startup.bat 수정
D:\tomcat-8.5.88\bin\startup.bat


내용 추가:

set JAVA_OPTS=-Dfile.encoding=UTF-8 -Dsun.jnu.encoding=UTF-8

(권장) setenv.bat 생성
D:\tomcat-8.5.88\bin\setenv.bat

set JAVA_OPTS=-Dfile.encoding=UTF-8 -Dsun.jnu.encoding=UTF-8


setenv.bat가 존재하면 startup.bat보다 우선 적용됨

9. UTF-8 적용 후 정상 동작

한글 SLD 정상 파싱

Rule/Title 전부 정상 출력

스타일 적용 오류 완전 해결

10. GeoServer 포인트 아이콘 적용
아이콘 위치
D:\tomcat-8.5.88\webapps\geoserver\data\styles\icon.png

SLD 적용 예시
<PointSymbolizer>
  <Graphic>
    <ExternalGraphic>
      <OnlineResource xlink:href="file:data/styles/icon.png" />
      <Format>image/png</Format>
    </ExternalGraphic>
    <Size>20</Size>
  </Graphic>
</PointSymbolizer>


또는:

<OnlineResource xlink:href="/geoserver/styles/icon.png" />

✅ 전체 과정 요약

geoserver.war 복사

Windows Tomcat 8.5 복사

server.xml 포트 변경 (8080→8180, 8005→8006)

Windows 방화벽 8180 포트 개방

Tomcat 실행 → GeoServer 자동 배포

Workspace / Store 생성

한글 SLD 깨짐 → Windows 인코딩(MS949) 때문

JVM UTF-8 옵션 적용으로 해결

아이콘(symbol)도 정상 적용됨