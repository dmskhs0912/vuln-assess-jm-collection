# vuln-assess-jm-collection

## 개요
2025년 5월 19-23일에 수행한 JM Collection 대상 취약점 진단 및 분석 프로젝트 기록용 저장소.</br>
JM Collection을 이름으로 하는 가상의 클라우드 기반 온라인 쇼핑몰 웹 서비스를 대상으로 6명의 인원으로 취약점 진단을 수행하였다.</br>
웹 서버 / WAS / 클라이언트 사이드 웹 인터페이스로 3개의 팀을 나누어 진행하였다.</br>
KISA의 **'2021 주요정보통신기반시설 기술적 취약점 기술적 취약점 분석·평가 방법 상세가이드'**(이하 주통기 가이드)를 기반으로 각 서비스의 환경에 맞춘 취약점 분석을 수행하였다.</br>
진단 결과 총 61건의 취약점이 발견되었으며, 각 취약점의 분석 내용과 조치 사항 등을 보고서로 작성하였다.

## 점검 대상 클라우드 아키텍처

![클라우드 아키텍처 다이어그램](./img/cloud_architecture_diagram.png)

두 개의 AZ를 걸친 로드 밸런싱과 오토 스케일링으로 고가용성을 보장하는 클라우드 아키텍처.

### 주요 구성 요소
- **VPC (10.0.0.0/16)**  
  Web/App/DB 서버를 모두 포함하는 논리적 네트워크 범위
- **Public Subnet (10.0.0.0/24, 10.0.1.0/24)**  
  - **Web Server**  
    클라이언트로부터 받은 HTTP/HTTPS 요청 처리.</br>
    OS : Ubuntu 22.04 LTS
  - **Security Group: Web**  
    HTTP/HTTPS 인바운드 허용 / 8009(AJP) 포트 아웃바운드 허용
- **Private Subnet (10.0.4.0/24, 10.0.5.0/24)**  
  - **Wep Application Server**  
    Web 서버에서 보낸 웹 애플리케이션 동작 요청 처리</br>
    OS : Windows Server 2019 Base
  - **Security Group: WAS**</br>
    NLB의 보안 그룹 인바운드 허용 / 1521(Oracle) 포트 아웃바운드 허용
- **Private Subnet (10.0.8.0/24)**
  - **RDS (Amazon RDS)**  
    Oracle DB 기반 데이터베이스 인스턴스. WAS에서 요청한 쿼리 수행
  - **Security Group: RDS**</br>
    1521(Oracle) 포트 인바운드 허용
    
- **네트워크 계층**  
  1. **IGW**: 인터넷 게이트웨이  
  2. **ALB**: Web 계층 로드밸런싱 (Internet Facing)
  3. **NLB**: WAS 계층 로드밸런싱 (Internal)

## 진단 방법

- 주통기 가이드를 참고하여 각 서버 환경에 맞게 진단 수행.
- Burp Suite, Kali Linux, nikto, dirb, sqlmap 등의 진단 도구 사용.
- 운영 서비스에 장애를 발생시키지 않는 범위 내에서 실제 공격과 동일한 방법으로 진단.

## 진단 결과 요약 및 주요 취약점

### 진단 결과 요약
| 구분                             | High | Medium | Low | Total |
|----------------------------------|:----:|:------:|:---:|:-----:|
| Web Server                         |  13  |   6   |  3  |  22   |
| Web Application Server (WAS)     |   12  |   9   |  3  |  24   |
| Client-side Web Interface (Web)  |   10  |    5   |  0  |  16   |
| **합계**                         | **35** | **20** | **6** | **61** |

### 주요 취약점
| Target      | Issue       | Impact      | Measures      |
|-------------|-------------|-------------|---------------|
| Web Server | 불필요 특수 권한 파일 존재 | 권한 상승 | 불필요한 root 소유 파일 탐지 및 제거 |
| WAS | Tomcat 프로세스 권한 제한 미흡 | 시스템 침투 경로 확보시 모든 권한 획득 | Tomcat 로그온 계정 생성 및 권한 설정 |
| WAS | 서버 에러 페이지 출력 | 서버 내부 구조 및 코드 유출 | Tomcat /conf/server.xml 설정 보완 |
| Web | SQL Injection (4 Cases) | 데이터베이스 정보 유출 | Prepared Statement 적용해 코드 수정 |
| Web | 파라미터 조작 | 실제 판매 가격과 다른 가격의 주문 시도 가능 | 클라이언트 요청 파라미터 값을 DB의 값과 비교 후 처리 |
| Web | 파일 업로드 | 웹쉘 업로드 가능 | 파일의 타입/확장자/시그니처 검사 후 업로드 | 

## 최종 보고서

[최종 보고서 (PDF)](/report/Final_report.pdf)

## Authors

- 김현승 : 조장, WAS 진단, Web 취약점 조치 코드 작성
- 박정인 : Web Server 진단, 보고서 취합
- 유승민 : Web 진단, 진단 결과 요약
- 허태영 : WAS 진단, Web 진단 검토토
- 함형욱 : Web Server 진단
- 홍혜연 : Web 진단, 보고서 취합