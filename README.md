# AWSminiProjectForDVWA
Secure 3-Tier Architecture on AWS via CloudFormation. Designed for strict IAM environments with auto-deployed DVWA for security testing.
# 🛡️ Secure AWS 3-Tier Architecture with DVWA (CloudFormation)

이 프로젝트는 **AWS CloudFormation**을 사용하여 **보안이 강화된 3-Tier 아키텍처(Web-App-DB)**를 자동으로 구축하는 IaC(Infrastructure as Code) 프로젝트입니다.

특히, **엄격한 IAM 권한 제한이 있는 환경**에서도 배포가 가능하도록 설계되었으며, **DVWA(Damn Vulnerable Web App)**를 통해 웹 해킹 및 보안 실습을 바로 진행할 수 있는 환경을 제공합니다.

---

## 🏗️ Architecture Overview

이 프로젝트는 전형적인 엔터프라이즈급 3계층 구조를 따르며, 보안을 위해 퍼블릭/프라이빗 서브넷을 철저히 분리했습니다.

* **VPC**: 독립된 네트워크 환경 구성
* **Web Tier (Public)**: 외부 사용자의 요청을 받아 App Tier로 전달 (Apache Reverse Proxy)
* **App Tier (Private)**: 실제 애플리케이션(DVWA) 로직 처리, 외부 직접 접속 차단
* **DB Tier (Private)**: 데이터 저장을 위한 RDS (MySQL), 외부 접근 불가

### 🔑 Key Features
1.  **All-in-One CloudFormation**: 네트워크부터 애플리케이션 배포까지 스택 하나로 자동화
2.  **EC2 Instance Connect Endpoint (EIC)**: 보안에 취약한 Bastion Host 없이, 프라이빗 인스턴스에 안전하게 접속
3.  **NAT Instance**: 고비용의 NAT Gateway 대신, 커스텀 NAT 인스턴스를 구축하여 비용 절감 및 제어권 확보
4.  **Auto Scaling & ALB**: 트래픽 부하 분산 및 고가용성 확보 (Web/App Tier)
5.  **Strict Security Groups**: 최소 권한 원칙(Least Privilege)에 따라 IP 및 포트 접근을 엄격히 제어

---

## 📂 Project Structure

이 프로젝트는 학습 및 유지보수 편의를 위해 두 가지 형태의 템플릿을 제공합니다.

### 1. 통합 템플릿 (Recommended)
* `integrated-infra-file(osaka).yaml`: 네트워크, 보안그룹, DB, App, Web 등 모든 리소스를 한 번에 생성합니다. **(One-Click Deploy)**

### 2. 계층별 분할 템플릿 (For Study)
각 계층의 의존성을 이해하고 싶다면 아래 순서대로 배포하세요.
1.  `01-network-tier.yaml`: VPC, Subnet, NAT, EIC 생성
2.  `02-securitygroup.yaml`: 각 티어 간의 보안 그룹 규칙 정의
3.  `03-database.yaml`: RDS MySQL 생성
4.  `04-app-tier-no-internet.yaml`: 내부 ALB 및 DVWA 애플리케이션 배포
5.  `05-web-tier.yaml`: 외부 ALB 및 리버스 프록시 설정

---

## 🚀 How to Deploy (배포 방법)

### Prerequisites
* AWS 계정 및 CloudFormation 접근 권한
* (선택) AWS CLI 환경

### Steps
1.  **AWS Console** 접속 후 **CloudFormation** 서비스로 이동합니다.
2.  **[스택 생성]** -> **[새 리소스 사용(표준)]**을 클릭합니다.
3.  `integrated-infra-file(osaka).yaml` 파일을 업로드합니다.
4.  **Parameters(파라미터)** 설정:
    * `MyIpAddress`: 본인의 PC IP 주소 입력 (관리자 접속용, 예: `121.xxx.xxx.xxx/32`)
    * `DBPassword`: DB 관리자 비밀번호 설정 (8자 이상)
5.  스택 생성 완료(CREATE_COMPLETE) 후 **Outputs** 탭에서 `WebUrl`을 확인합니다.
6.  브라우저로 해당 URL에 접속하여 DVWA 화면을 확인합니다.

---

## 🧪 DVWA 초기 설정 및 테스트 가이드

배포가 완료되면 아래 절차를 통해 애플리케이션이 정상 동작하는지 확인합니다.

### 1. Database Setup
1.  제공된 `WebUrl`로 접속합니다.
2.  초기 로그인 화면이 나오면 `admin` / `password` 로 로그인합니다.
3.  좌측 메뉴 하단의 **[Setup / Reset DB]**를 클릭합니다.
4.  **[Create / Reset Database]** 버튼을 누릅니다. (자동화 스크립트가 있지만, 확실한 실습을 위해 초기화를 권장합니다.)

### 2. SQL Injection Test
이 아키텍처는 내부적으로 완벽하게 DB와 연결되어 있습니다.
* **Menu**: SQL Injection
* **User ID Input**: `1` (정상 출력 확인)
* **Injection Input**: `1' OR '1'='1` (모든 회원 정보 출력 확인)

> **Note**: SQL Injection이 작동하지 않는 경우, 입력값이 숫자가 아닌 문자(예: 'test')가 들어가면 결과가 나오지 않을 수 있습니다. 반드시 유효한 ID나 Injection 구문을 입력하세요.

---

## ⚠️ TroubleShooting & Notes

* **권한 문제 (777 Permission)**: DVWA는 파일 업로드 취약점 등 다양한 공격 실습을 위해 웹 루트 디렉토리에 `777` 권한이 부여되어 있습니다. 이는 **실습용 환경(Lab)**이기에 의도된 설정이며, 실제 운영 환경에서는 절대 권장하지 않습니다.
* **SELinux**: Amazon Linux 2023 환경에서 Apache 리버스 프록시가 정상 작동하도록 SELinux 정책(`httpd_can_network_connect`)이 UserData에 포함되어 자동 설정됩니다.

---

## 👤 Author

* **GitHub**: [본인의 깃허브 주소 입력]
* **Blog**: [블로그 주소가 있다면 입력]

AWS 인프라 구축 및 보안 아키텍처에 관심이 많습니다. 이슈나 PR은 언제든 환영합니다!
