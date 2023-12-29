# [12/29] Kubernetes (가상 인프라 구축, 쿠버네티스 의미 및 특징)

## 쿠버네티스용 가상 인프라 구축

### 수업 - VirtualBox 사용

- OS : Ubuntu20.04 LTS
- CPU : 2
- RAM : 4096MB (4GB)
- 디스크 : 50GB + 10GB + 10GB

### 실습 - AWS EC2 Ubuntu 환경 구축

- OS : Ubuntu20.04 LTS
- 인스턴스 유형 : t2.micro (1vCPU, 1 GiB Memory)
- 스토리지 : 30GiB / gp2

## 외부 네트워크 연결

- 네트워크 설정 파일 실행
    - VirtualBox 에서는 `/etc/netplan/00-installer-config.yaml` 경로로 파일 실행
    - AWS EC2 에서는 `/etc/netplan/50-cloud-init.yaml` 경로로 파일 실행

        ```bash
        sudo vi /etc/netplan/50-cloud-init.yaml
        ```

    - VirtualBox 에서 설정 시 다음과 같이 파일 수정

        ```yaml
        network:
          version: 2
          ethernets:
            enp0s3:
              addresses:
              - "10.0.2.101/24" # NAT 게이트웨이에서 접근하는 IP
              nameservers:
                addresses:
                - 8.8.8.8
                - 8.8.4.4
              gateway4: 10.0.2.1
            enp0s8:
              addresses:
              - "192.168.56.101/24" # 실제 IP
        ```

    - 설정 후 적용

        ```bash
        sudo netplan apply
        
        netplan get
        ```

- **AWS EC2 는 보안그룹 아웃바운드 설정으로 처리할 수 있어 설정하지 않음**

## 환경설정

### 방화벽 해제

```bash
# 방화벽 해제
sudo ufw disable

# 방화벽 상태 확인
sudo ufw status
```

### 패스워드 입력 없이 sudo 명령어 수행

- sudo 관련 설정 파일 실행

    ```bash
    sudo vi /etc/sudoers
    ```

- 사용자 패스워드 관련 설정 추가

    ```bash
    Defaults        env_reset
    Defaults        mail_badpass
    Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
    
    # User privilege specification
    root    ALL=(ALL:ALL) ALL
    # 추가
    # <본인계정> ALL=(ALL) NOPASSWD:ALL
    ubuntu  ALL=(ALL)       NOPASSWD:ALL
    
    # Members of the admin group may gain root privileges
    %admin ALL=(ALL) ALL
    
    # Allow members of group sudo to execute any command
    %sudo   ALL=(ALL:ALL) ALL
    ```


### 도메인 주소 지정

- 도메인 설정 파일 실행
    - 원래 /etc/hosts 의 권한은 root 이나 앞전에 PASSWORD 설정을 제거했기 때문에 ubuntu 사용자로 접근 가능

    ```bash
    vi /etc/hosts
    ```

- IP 에 매칭되는 도메인 주소 설정
    - **내부 네트워크 사이에 호출하는 경우가 없으므로 도메인은 지정하지 않음**

    ```bash
    127.0.0.1 localhost
    
    # The following lines are desirable for IPv6 capable hosts
    ::1 ip6-localhost ip6-loopback
    fe00::0 ip6-localnet
    ff00::0 ip6-mcastprefix
    ff02::1 ip6-allnodes
    ff02::2 ip6-allrouters
    ff02::3 ip6-allhosts
    
    # 추가
    54.180.147.235  hd08-01
    13.124.117.86   hd08-02
    3.35.209.180    hd08-03
    ```


### 인스턴스 간 네트워크 설정

- **AWS EC2 의 경우 보안그룹 인바운드 규칙으로 외부 설정을 추가할 수 있으므로 수정하지 않음**

## Kubernetes (쿠버네티스)

- 대규모 클러스터 환경에서 컨테이너화된 어플리케이션을 자동으로 배포하고 확장, 관리하는 데 필요한 여러 가지 요소들을 자동화하는 오픈소스 플랫폼
- 사용자 부하에 따라 자동으로 어플리케이션과 서버의 규모를 확장할 수 있음
- 네트워크, 스토리지, 모니터링 등 시스템 운영에 필수적인 여러 컴포넌트를 편리하게 구축, 관리

## 쿠버네티스의 특징

- 소스코드를 기반으로 클러스터 운영
    - 기존의 가상 머신 환경처럼 명령어를 이용해 순차적으로 설정을 변경하면서 관리하는 것이 아님
    - 개발팀, 데브옵스팀, 보안팀 등 모든 이해관계자가 이해할 수 있는 코드 기반으로 운영 가능
- 의도한 상태(desired state) 로 클러스터 관리
    - 최초 의도한 상태와 현재 실행 중인 상태를 쿠버네티스 컨트롤러가 자동으로 끊임없이 확인하여 원복
    - 시스템의 자원 사용 현황을 모니터링해서 자원 여유가 있는 노드에 어플리케이션을 적절하게 배치하고 필요한 물리 리소스는 자동으로 증가시킴

## 실습 방법

### 1) Kubespray 를 이용한 n 개의 가상 머신 쿠버네티스 클러스터 설치

- 컨테이너 런타임으로 containerd 사용
    - https://containerd.io/
- 쿠버네티스 감사(audit) 로그 활성화
- 쿠버네티스 로드밸런서 용도의 MetalLB 사용을 위한 설정 추가 필요

### 2) k3s 를 이용한 단일 노드에 쿠버네티스 클러스터 설치

### 3) kubectl 를 로컬호스트에 설치하여 사용

- 쿠버네티스를 관리하기 위한 커맨드라인 도구
- https://kubespray.io/#/
