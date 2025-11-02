# Ansible Role: Jenkins CI

Docker 기반 Jenkins CI/CD 서버를 자동으로 배포하고 구성하는 Ansible Role입니다. RHEL/CentOS, Debian/Ubuntu 등 주요 리눅스 배포판을 지원합니다.

## 목차

- [프로젝트 개요](#프로젝트-개요)
- [주요 기능](#주요-기능)
- [시스템 요구사항](#시스템-요구사항)
- [설치 방법](#설치-방법)
- [Role 변수](#role-변수)
- [의존성](#의존성)
- [디렉토리 구조](#디렉토리-구조)
- [사용 예제](#사용-예제)
- [고급 설정](#고급-설정)
- [트러블슈팅](#트러블슈팅)
- [라이선스](#라이선스)
- [개발자 정보](#개발자-정보)

## 프로젝트 개요

이 Ansible Role은 Docker Compose를 이용하여 Jenkins CI/CD 서버를 손쉽게 배포하고 관리할 수 있도록 설계되었습니다. 주요 특징은 다음과 같습니다:

- **컨테이너 기반 배포**: Docker를 이용한 격리된 Jenkins 환경 구성
- **다중 플랫폼 지원**: RHEL, CentOS, Debian, Ubuntu, Alpine, ArchLinux 지원
- **유연한 설정**: 다양한 환경 변수를 통한 커스터마이징 가능
- **통합 지원**: AWS, ArgoCD, Kubernetes, SSH, Flyway 등과의 통합
- **자동화**: 전체 설치 과정 자동화로 수동 작업 최소화

## 주요 기능

### 1. Docker 기반 Jenkins 배포
- Docker Compose를 통한 Jenkins 컨테이너 관리
- 커스텀 Jenkins Docker 이미지 지원
- 이미지 레지스트리(Docker Hub, GHCR 등) 연동

### 2. 다양한 통합 옵션
- **AWS 통합**: AWS CLI 설정 및 인증 정보 구성
- **Kubernetes 통합**: kubeconfig 파일 마운트
- **ArgoCD 통합**: ArgoCD CLI 설정 지원
- **SSH 통합**: SSH 키 기반 인증 설정
- **Flyway 통합**: 데이터베이스 마이그레이션 도구 지원
- **Traefik 통합**: 리버스 프록시 및 SSL/TLS 설정

### 3. 볼륨 관리
- Jenkins 홈 디렉토리 영구 저장
- Maven 로컬 저장소 공유
- Git 설정 및 Maven settings.xml 자동 구성
- Docker 소켓 마운트를 통한 Docker-in-Docker 지원

### 4. 네트워크 설정
- 커스텀 Docker 네트워크 지원
- Extra hosts 설정으로 호스트명 해석
- 포트 매핑 커스터마이징

## 시스템 요구사항

### 필수 요구사항
- **Docker**: 최신 버전 (20.10 이상 권장)
- **Docker Compose**: v2.x 이상 권장
- **Ansible**: 2.1 이상
- **Python**: 3.6 이상
- **Docker SDK for Python**: ansible을 통해 자동 설치됨

### 지원 운영체제
- Red Hat Enterprise Linux (RHEL) - 모든 버전
- CentOS - 모든 버전
- Debian - 모든 버전
- Ubuntu - 모든 버전
- Alpine Linux - 모든 버전
- ArchLinux - 모든 버전

## 설치 방법

### 1. Ansible Galaxy를 통한 설치 (권장)

```bash
ansible-galaxy install codesuiteapp.jenkins
```

### 2. Git을 통한 직접 설치

```bash
cd /etc/ansible/roles
git clone https://github.com/codesuiteapp/ansible-role-jenkins.git codesuiteapp.jenkins
```

### 3. requirements.yml을 통한 설치

`requirements.yml` 파일 생성:

```yaml
---
roles:
  - name: codesuiteapp.jenkins
    src: https://github.com/codesuiteapp/ansible-role-jenkins.git
    version: main
```

설치 실행:

```bash
ansible-galaxy install -r requirements.yml
```

## Role 변수

### 필수 변수

이 변수들은 반드시 설정해야 합니다:

```yaml
dkr_username: "your_docker_username"    # Docker 레지스트리 사용자명
dkr_passwd: "your_docker_password"      # Docker 레지스트리 비밀번호
passwd_adm_user: "admin_password"       # 관리자 사용자 비밀번호
```

### 이미지 및 컨테이너 설정

```yaml
# Docker 이미지 설정
image_registry_server: "ghcr.io/codesuiteapp"   # 기본 레지스트리 서버
image_registry_server2: "devops"                 # 보조 레지스트리 서버
jenkins_image: "jenkins-docker"                  # Jenkins 이미지 이름
jenkins_ver: "latest"                            # Jenkins 이미지 버전
docker_type: "ghcr.io"                          # Docker 레지스트리 타입

# 컨테이너 설정
container_name: "jenkins"                        # 컨테이너 이름
jenkins_port: 9876                              # Jenkins 웹 포트
```

### 디렉토리 설정

```yaml
app_home_dir: /app                              # 애플리케이션 홈 디렉토리
app_data_dir: /data                             # 데이터 디렉토리
jenkins_home: "{{ app_home_dir }}/jenkins-docker"     # Jenkins 설정 디렉토리
jenkins_data: "{{ app_data_dir }}/jenkins_home"       # Jenkins 데이터 디렉토리
nexus_data: "{{ app_data_dir }}/m2/repository"        # Maven 로컬 저장소
```

### 사용자 설정

```yaml
create_user: false                              # 사용자 자동 생성 여부
adm_user: "appadm"                              # 관리자 사용자명
adm_group: "appadm"                             # 관리자 그룹명
adm_uid: "1000"                                 # 사용자 UID
```

### 통합 기능 활성화/비활성화

```yaml
use_ssh: false                                  # SSH 키 설정 사용
use_kubeconfig: false                           # Kubernetes 설정 사용
use_argocd: false                               # ArgoCD 설정 사용
use_flyway: false                               # Flyway 데이터 디렉토리 사용
use_aws: false                                  # AWS CLI 설정 사용
use_traefik: false                              # Traefik 리버스 프록시 사용
use_traefik_tls: false                          # Traefik TLS/SSL 사용
use_dkr_net: false                              # 커스텀 Docker 네트워크 사용
use_extra_hosts: false                          # Extra hosts 설정 사용
```

### Docker Compose 설정

```yaml
docker_compose_bin: "docker-compose"            # Docker Compose 바이너리
pull_image: false                               # 이미지 자동 pull 여부
tag_image: true                                 # 이미지 태그 여부
push_image: false                               # 이미지 push 여부
```

### 네트워크 설정

```yaml
dkr_network: "devops_net"                       # Docker 네트워크 이름
extra_hosts:                                    # 추가 호스트 설정
  - "devops.com:10.11.22.33"
```

### AWS 설정 (use_aws: true일 때)

```yaml
aws_profile: "default"                          # AWS 프로파일 이름
enc_access_key: "YOUR_ACCESS_KEY"               # AWS Access Key ID
enc_secret_access_key: "YOUR_SECRET_KEY"        # AWS Secret Access Key
```

### Git 설정

```yaml
jenkins_git_user: "Jenkins"                     # Git 사용자명
jenkins_git_email: "manager@devops.com"         # Git 이메일
```

### 환경 설정

```yaml
env_profile: "dev"                              # 환경 프로파일 (dev/stage/prod)
```

## 의존성

이 Role은 다음 Role에 의존합니다:

- **codesuiteapp.user**: 사용자 및 그룹 관리를 위한 Role
  - 저장소: https://github.com/codesuiteapp/ansible-role-user.git
  - 버전: main

의존성은 자동으로 설치되지만, 수동으로 설치하려면:

```bash
ansible-galaxy install -r meta/main.yml
```

## 디렉토리 구조

```
ansible-role-jenkins/
├── defaults/           # 기본 변수 정의
│   └── main.yml
├── handlers/           # 핸들러 정의 (서비스 시작/중지/재시작)
│   └── main.yml
├── meta/              # Role 메타데이터 및 의존성
│   └── main.yml
├── tasks/             # 주요 작업 정의
│   └── main.yml
├── templates/         # Jinja2 템플릿 파일
│   ├── .env.j2                    # Docker Compose 환경 변수
│   ├── docker-compose.yml.j2      # Docker Compose 설정
│   ├── gitconfig.j2               # Git 설정
│   └── settings.xml.j2            # Maven settings
├── tests/             # 테스트 플레이북
│   └── test.yml
├── vars/              # 내부 변수
│   └── main.yml
├── LICENSE            # GPL-2.0 라이선스
└── README.md          # 이 파일
```

## 사용 예제

### 기본 사용법

가장 간단한 플레이북 예제:

```yaml
---
- name: Install Jenkins Server
  hosts: jenkins_server
  become: true

  vars:
    dkr_username: "myusername"
    dkr_passwd: "mypassword"
    passwd_adm_user: "P@ssw0rd1"

  roles:
    - codesuiteapp.jenkins
```

### 로컬 설치 예제

로컬 머신에 Jenkins를 설치하는 경우:

```yaml
---
- name: Install Jenkins Server Locally
  hosts: localhost
  connection: local
  become: true

  vars:
    today: "{{ lookup('pipe', 'date +\"%Y%m%d\"') }}"
    dkr_username: "your_username"
    dkr_passwd: "your_password"
    passwd_adm_user: "P@ssw0rd1"
    jenkins_port: 9876

  roles:
    - codesuiteapp.jenkins
```

### AWS 통합 예제

AWS CLI를 사용하는 Jenkins 설정:

```yaml
---
- name: Install Jenkins with AWS Integration
  hosts: jenkins_server
  become: true

  vars:
    dkr_username: "myusername"
    dkr_passwd: "mypassword"
    passwd_adm_user: "P@ssw0rd1"

    # AWS 설정
    use_aws: true
    aws_profile: "production"
    enc_access_key: "AKIAIOSFODNN7EXAMPLE"
    enc_secret_access_key: "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"

  roles:
    - codesuiteapp.jenkins
```

### Kubernetes 및 ArgoCD 통합 예제

Kubernetes 클러스터와 ArgoCD를 사용하는 DevOps 환경:

```yaml
---
- name: Install Jenkins for Kubernetes/ArgoCD
  hosts: jenkins_server
  become: true

  vars:
    dkr_username: "myusername"
    dkr_passwd: "mypassword"
    passwd_adm_user: "P@ssw0rd1"

    # Kubernetes 설정
    use_kubeconfig: true

    # ArgoCD 설정
    use_argocd: true

    # SSH 키 설정
    use_ssh: true

  roles:
    - codesuiteapp.jenkins
```

### Traefik 리버스 프록시 예제

Traefik을 사용한 SSL/TLS 설정:

```yaml
---
- name: Install Jenkins with Traefik
  hosts: jenkins_server
  become: true

  vars:
    dkr_username: "myusername"
    dkr_passwd: "mypassword"
    passwd_adm_user: "P@ssw0rd1"

    # Traefik 설정
    use_traefik: true
    use_traefik_tls: true
    traefik_domain: "jenkins.example.com"
    traefik_http_port: 8080

    # Docker 네트워크
    use_dkr_net: true
    dkr_network: "traefik_net"

  roles:
    - codesuiteapp.jenkins
```

### 완전한 설정 예제

모든 기능을 활성화한 프로덕션 환경 예제:

```yaml
---
- name: Install Production Jenkins Server
  hosts: jenkins_production
  become: true

  vars:
    # Docker 레지스트리 설정
    dkr_username: "{{ vault_dkr_username }}"
    dkr_passwd: "{{ vault_dkr_passwd }}"
    docker_type: "ghcr.io"

    # 사용자 설정
    create_user: true
    passwd_adm_user: "{{ vault_admin_password }}"
    adm_user: "jenkins"
    adm_group: "jenkins"
    adm_uid: "2000"

    # 환경 설정
    env_profile: "prod"
    jenkins_port: 8080

    # 통합 기능
    use_aws: true
    aws_profile: "production"
    enc_access_key: "{{ vault_aws_access_key }}"
    enc_secret_access_key: "{{ vault_aws_secret_key }}"

    use_kubeconfig: true
    use_argocd: true
    use_ssh: true
    use_flyway: true

    # Traefik 설정
    use_traefik: true
    use_traefik_tls: true
    traefik_domain: "jenkins.mycompany.com"

    # 네트워크 설정
    use_dkr_net: true
    dkr_network: "prod_network"

    use_extra_hosts: true
    extra_hosts:
      - "gitlab.internal:192.168.1.10"
      - "nexus.internal:192.168.1.20"

    # Git 설정
    jenkins_git_user: "Jenkins CI"
    jenkins_git_email: "jenkins@mycompany.com"

  roles:
    - codesuiteapp.jenkins
```

## 고급 설정

### 1. 커스텀 Jenkins 이미지 사용

자체 Jenkins 이미지를 사용하려면:

```yaml
vars:
  image_registry_server: "registry.mycompany.com"
  jenkins_image: "custom-jenkins"
  jenkins_ver: "v2.401.1"
  pull_image: true
```

### 2. 이미지 태그 및 푸시

이미지를 다른 레지스트리로 태그하고 푸시:

```yaml
vars:
  image_registry_server: "ghcr.io/myorg"
  image_registry_server2: "registry.internal.com"
  tag_image: true
  push_image: true
```

### 3. 볼륨 커스터마이징

데이터 디렉토리를 커스터마이징:

```yaml
vars:
  app_home_dir: /opt/jenkins
  app_data_dir: /var/lib/jenkins
  jenkins_data: "{{ app_data_dir }}/home"
  nexus_data: "{{ app_data_dir }}/m2"
```

### 4. Flyway 데이터베이스 마이그레이션

```yaml
vars:
  use_flyway: true
  flyway_data: "/data/flyway"
  flyway_data_kos: "/data/flyway-kos"
```

### 5. 환경별 설정 관리

개발, 스테이징, 프로덕션 환경별 변수 파일 사용:

```bash
# group_vars/development.yml
env_profile: dev
jenkins_port: 9876

# group_vars/production.yml
env_profile: prod
jenkins_port: 8080
use_traefik: true
```

## 트러블슈팅

### 일반적인 문제 해결

#### 1. Docker Compose 모듈 오류

**증상**: `docker_compose_v2` 모듈 실행 실패

**해결방법**:
- handlers는 자동으로 shell 명령어로 폴백됩니다
- Docker Compose v2를 설치했는지 확인: `docker compose version`

#### 2. 권한 문제

**증상**: 디렉토리 생성 또는 파일 작성 권한 오류

**해결방법**:
```yaml
- name: Run with become
  hosts: jenkins_server
  become: true  # root 권한으로 실행
```

#### 3. Docker 로그인 실패

**증상**: Docker 레지스트리 인증 실패

**해결방법**:
- GHCR의 경우 Personal Access Token 사용
- Docker Hub의 경우 올바른 사용자명/비밀번호 확인
- `docker_type` 변수를 올바르게 설정

#### 4. 포트 충돌

**증상**: 포트가 이미 사용 중

**해결방법**:
```yaml
vars:
  jenkins_port: 9876  # 다른 포트로 변경
```

#### 5. Python Docker SDK 설치 실패

**증상**: `pip install docker` 실패

**해결방법**:
```bash
# 수동 설치
pip3 install docker

# 또는 패키지 매니저 사용
apt-get install python3-docker  # Debian/Ubuntu
yum install python3-docker      # RHEL/CentOS
```

### 로그 확인

Jenkins 컨테이너 로그 확인:

```bash
docker logs jenkins
docker logs -f jenkins  # 실시간 로그
```

Docker Compose 상태 확인:

```bash
cd /app/jenkins-docker
docker-compose ps
docker-compose logs
```

### 재시작 방법

Jenkins 서비스 재시작:

```yaml
- name: Restart Jenkins
  hosts: jenkins_server
  become: true

  tasks:
    - name: Restart container
      ansible.builtin.shell: |
        cd /app/jenkins-docker
        docker-compose restart
```

## 보안 고려사항

### 1. 민감한 정보 관리

Ansible Vault를 사용하여 비밀번호 암호화:

```bash
# vault 파일 생성
ansible-vault create vars/secrets.yml

# 내용
dkr_username: myuser
dkr_passwd: mypassword
passwd_adm_user: adminpass
enc_access_key: AWS_KEY
enc_secret_access_key: AWS_SECRET
```

플레이북에서 사용:

```yaml
- name: Install Jenkins
  hosts: jenkins_server
  become: true

  vars_files:
    - vars/secrets.yml

  roles:
    - codesuiteapp.jenkins
```

### 2. 파일 권한

중요한 파일들의 권한이 자동으로 설정됩니다:
- AWS credentials: 0600
- SSH keys: 0600
- Config 파일: 0644

### 3. 네트워크 보안

- 방화벽 설정 권장
- Traefik을 통한 SSL/TLS 사용 권장
- 내부 네트워크 사용 시 `use_dkr_net: true` 설정

## 업그레이드 가이드

### Jenkins 버전 업그레이드

```yaml
vars:
  jenkins_ver: "2.401.2"  # 새 버전으로 변경
  pull_image: true        # 새 이미지 pull
```

플레이북 재실행:

```bash
ansible-playbook -i inventory playbook.yml
```

### Role 업데이트

```bash
ansible-galaxy install --force codesuiteapp.jenkins
```

## 라이선스

이 프로젝트는 **GPL-2.0-or-later** 라이선스 하에 배포됩니다.

자세한 내용은 [LICENSE](LICENSE) 파일을 참조하세요.

## 개발자 정보

- **개발자**: CodeSuiteApp
- **회사**: CodeSuiteApp
- **저장소**: https://github.com/codesuiteapp/ansible-role-jenkins

### 기여하기

버그 리포트, 기능 요청, Pull Request는 언제나 환영합니다!

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

### 지원

문제가 발생하거나 질문이 있으시면:
- GitHub Issues: https://github.com/codesuiteapp/ansible-role-jenkins/issues

---

**마지막 업데이트**: 2025-11-02
