🗂️ Windows Server에서 GeoServer 배포 및 UTF-8 문제 해결 과정 정리
1. 개발 서버에서 GeoServer WAR 파일 복사

기존 리눅스 개발 서버에서 사용하던 geoserver.war 파일을 그대로 복사.
원격 접속된 Windows Server로 Ctrl + C / Ctrl + V 이용하여 복사 진행.

2. Tomcat 8.5 설치 파일 복사

리눅스에서 사용하던 Tomcat이 아닌 Windows용 Tomcat 8.5 폴더 전체를 복사.
동일하게 Ctrl + C / Ctrl + V로 윈도우 서버에 붙여넣음.

경로 예시:
D:\tomcat-8.5.88\

3. Tomcat 포트 설정 변경 (중요)

원격 접속한 Windows Server에서 내 로컬에서 접속 가능한 포트로 변경해야 했음.

✔️ server.xml 수정

경로:
D:\tomcat-8.5.88\conf\server.xml

변경 내용
👉 기존
<Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />
<Server port="8005" shutdown="SHUTDOWN">

👉 변경 후
<Connector port="8180" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />
<Server port="8006" shutdown="SHUTDOWN">


Web 포트: 8080 → 8180
Shutdown 포트: 8005 → 8006

이유: 기존 Windows 서버 포트 충돌 가능성 + 네트워크 포트 홀딩 문제 회피

4. Windows 방화벽 인바운드·아웃바운드 규칙 설정

GeoServer를 외부에서 접속하려면 Windows 방화벽에 포트를 열어야 함.

✔️ 인바운드 규칙 추가

제어판 → Windows Defender 방화벽
좌측 메뉴: 고급 설정
인바운드 규칙 → 새 규칙
규칙 종류: 포트
프로토콜: TCP
특정 로컬 포트: 8180
연결 허용
프로필: 도메인/개인/공용 모두 체크
이름: GeoServer_8180_IN

✔️ 아웃바운드 규칙 추가 (같이 해주는 것이 안전함)

절차 동일.
규칙 이름: GeoServer_8180_OUT
🔥 이 작업을 하지 않으면 외부(내 로컬 PC)에서 http://<WindowsServer_IP>:8180/geoserver 접근 불가

5. Tomcat 실행 및 GeoServer 자동 배포

D:\tomcat-8.5.88\webapps\ 경로에
geoserver.war 파일 복사.

Tomcat 실행:
D:\tomcat-8.5.88\bin\startup.bat


Tomcat이 WAR 파일을 자동으로 풀면서 폴더 생성:
D:\tomcat-8.5.88\webapps\geoserver\


로컬 PC에서 접속 테스트:
http://<WindowsServer_IP>:8180/geoserver

6. GeoServer에서 저장공간(Workspace) 및 저장소(Store) 생성
Workspace 생성

예:
Workspace2025

Store 생성
PostgreSQL/PostGIS 연결 정보 입력

레이어 Publish → 정상 확인

7. 한글이 들어간 SLD 사용 시 UTF-8 깨짐 문제 발생
증상

SLD의 <TextSymbolizer> 또는 <Title>에 한글 포함 시 오류

영어만 있을 때는 정상적, 콘솔 로그에도 특별한 오류 없음

원인

Windows Tomcat에서 GeoServer 실행 시 JVM 기본 인코딩이
MS949(CP949) 로 설정됨.

SLD(XML)는 반드시 UTF-8이어야 하는데 JVM과 충돌 발생.

8. 해결 방법: Tomcat JVM에 UTF-8 강제 설정 추가
✔️ startup.bat 직접 수정

경로:
D:\tomcat-8.5.88\bin\startup.bat


상단에 추가:
set JAVA_OPTS=-Dfile.encoding=UTF-8 -Dsun.jnu.encoding=UTF-8

✔️ (권장) setenv.bat 생성

파일 생성:
D:\tomcat-8.5.88\bin\setenv.bat


내용:
set JAVA_OPTS=-Dfile.encoding=UTF-8 -Dsun.jnu.encoding=UTF-8
setenv.bat가 있으면 startup.bat보다 우선 적용됨 → 안전하고 깔끔한 방식.

9. UTF-8 환경에서 SLD 스타일 정상 작동

Tomcat 재시작 후 한글 SLD 정상 파싱
GeoServer UI에서도 한글 Title/Rule 이름 모두 정상 표시

스타일 적용 오류 없음

10. GeoServer 포인트 심볼에 아이콘 적용하기

GeoServer는 styles 폴더의 이미지를 직접 참조할 수 있음.
📁 아이콘 파일 위치
D:\tomcat-8.5.88\webapps\geoserver\data\styles\icon.png

📌 SLD에서 아이콘 불러오기
<PointSymbolizer>
  <Graphic>
    <ExternalGraphic>
      <OnlineResource xlink:href="file:data/styles/icon.png" />
      <Format>image/png</Format>
    </ExternalGraphic>
    <Size>20</Size>
  </Graphic>
</PointSymbolizer>

또는 URL 경로 사용
<OnlineResource xlink:href="/geoserver/styles/icon.png" />

✔️ 결과

포인트 레이어에 아이콘 정상 적용
SLD 한글 + 아이콘 모두 정상 표시됨

📌 전체 과정 요약

🔸 개발 서버에서 geoserver.war 복사

🔸 Windows 서버에 Tomcat 8.5 복사

🔸 server.xml에서 포트 변경 (8080→8180, 8005→8006)

🔸 Windows 방화벽 인바운드/아웃바운드에서 TCP 8180 포트 개방

🔸 webapps에 war 넣고 Tomcat 실행 → GeoServer 자동 배포

🔸 Workspace / Store 생성 후 레이어 확인

🔸 SLD 한글 깨짐 → 원인은 Windows JVM 기본 인코딩(MS949)

🔸 JVM 옵션: -Dfile.encoding=UTF-8 -Dsun.jnu.encoding=UTF-8 추가하여 해결

🔸 GeoServer 스타일 아이콘은 geoserver/data/styles/아이콘.png 에 넣고/styles/icon.png 로 참조

🔸 최종적으로 모든 스타일 / 한글 / 아이콘 정상 표시됨