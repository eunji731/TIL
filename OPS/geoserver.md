# Windows Server에서 GeoServer 배포 및 UTF-8 문제 해결 과정

## 1. 개발 서버에서 GeoServer WAR 파일 복사
리눅스 개발 서버에서 사용하던 `geoserver.war` 파일을 Windows Server로 복사  
(RDP 환경: Ctrl + C / Ctrl + V)

---

## 2. Tomcat 8.5 설치 파일 복사
Windows에서 실행 가능한 Tomcat 8.5 폴더 전체 복사

예시 경로:
```
D:\tomcat-8.5.88\
```

---

## 3. Tomcat 포트 설정 변경 (중요)

### 수정 파일
```
D:\tomcat-8.5.88\conf\server.xml
```

### 기존 설정
```xml
<Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />
<Server port="8005" shutdown="SHUTDOWN">
```

### 변경 후
```xml
<Connector port="8180" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />
<Server port="8006" shutdown="SHUTDOWN">
```

- Web 포트: 8080 → 8180  
- Shutdown 포트: 8005 → 8006  

---

## 4. Windows 방화벽 인바운드·아웃바운드 규칙 설정

### 인바운드 규칙
- 규칙 종류: 포트  
- 프로토콜: TCP  
- 특정 포트: 8180  
- 연결 허용  
- 프로필: 도메인/개인/공용  
- 이름: `GeoServer_8180_IN`

### 아웃바운드 규칙
- 동일한 방식  
- 이름: `GeoServer_8180_OUT`

---

## 5. Tomcat 실행 및 GeoServer 자동 배포

WAR 복사 위치:
```
D:\tomcat-8.5.88\webapps\
```

Tomcat 실행:
```
D:\tomcat-8.5.88\bin\startup.bat
```

자동 생성된 폴더:
```
D:\tomcat-8.5.88\webapps\geoserver\
```

접속:
```
http://<WindowsServer_IP>:8180/geoserver
```

---

## 6. GeoServer에서 Workspace / Store 생성
- Workspace 생성  
- PostgreSQL Store 생성  
- 레이어 Publish 후 지도 확인  

---

## 7. 한글 SLD 깨짐 문제

### 원인
Windows Tomcat JVM 기본 인코딩 = `MS949`  
SLD는 UTF-8 필요 → 충돌 발생

---

## 8. JVM UTF-8 강제 설정

### startup.bat 수정
```
D:\tomcat-8.5.88\bin\startup.bat
```

추가:
```bat
set JAVA_OPTS=-Dfile.encoding=UTF-8 -Dsun.jnu.encoding=UTF-8
```

### (권장) setenv.bat 생성
```
D:\tomcat-8.5.88\bin\setenv.bat
```

내용:
```bat
set JAVA_OPTS=-Dfile.encoding=UTF-8 -Dsun.jnu.encoding=UTF-8
```

---

## 9. UTF-8 적용 후 정상 동작
- 한글 SLD 정상 파싱  
- Rule/Title 정상 표시  
- 스타일 문제 해결  

---

## 10. GeoServer 포인트 아이콘 적용

아이콘 경로:
```
D:\tomcat-8.5.88\webapps\geoserver\data\styles\icon.png
```

SLD 예시:
```xml
<PointSymbolizer>
  <Graphic>
    <ExternalGraphic>
      <OnlineResource xlink:href="file:data/styles/icon.png" />
      <Format>image/png</Format>
    </ExternalGraphic>
    <Size>20</Size>
  </Graphic>
</PointSymbolizer>
```

---

# 전체 요약
- geoserver.war 복사  
- Tomcat 8.5 복사  
- server.xml 포트 변경  
- Windows 방화벽 8180 개방  
- Tomcat 실행 → GeoServer 자동 배포  
- Workspace / Store 생성  
- SLD 한글 깨짐 → JVM 인코딩 문제  
- UTF-8 옵션으로 해결  
- 아이콘 적용 정상  
