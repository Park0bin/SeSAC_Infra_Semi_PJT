# Hybrid Lab: 온프레미스(Vagrant) + AWS 진입(Route 53 / NLB) — Ansible 자동화

이 레포는 VirtualBox + Vagrant로 구성한 온프레미스 로컬 VM(Rocky Linux 9)을
Ansible로 자동 설정/배포하기 위한 코드와, AWS(Route 53/NLB)를 “진입점”으로
온프레미스 서비스로 트래픽을 연결하는 하이브리드 목표 구조를 정리한다.

## 참고(상세 가이드)
자세한 step by step 설치 가이드는 → [노션](https://www.notion.so/youngbin-sw/ansible-2c990d9338158027b88cfb4f8533ba97#2c990d9338158043922dea96bb290feb)으로


## 목적(왜 이걸 하는가)

- 온프레미스 “서비스 영역”을 **재현 가능**하게 구성하고, 컨트롤 노드에서
  **한 번에 관리**(bootstrap → 공통 세팅 → 웹/LB 배포)할 수 있게 만든다.
- AWS의 Route 53 + NLB를 **클라우드 진입점(Entry point)** 으로 두고,
  VPN 게이트웨이(EC2) 경유로 온프레미스 서비스에 트래픽을 연결하는
  **하이브리드 경로**를 실험/설계한다.
- 역할(Role) 기반 인프라 자동화(접근권한/패키지/서비스/방화벽/템플릿/핸들러)를
  구조적으로 정리한다.

## 아키텍처(개요)

- **온프레미스(이 레포가 실제로 자동화하는 범위)**
  - Controller 노드에서 Ansible로 관리 유저 생성 및 SSH 키 배포(초기 접근 bootstrap)
  - 공통 패키지/기본 설정 자동화
  - Web 노드: 웹 서비스 및 동적 HTML 배포(템플릿 기반)
  - LB 노드: HAProxy 설정을 템플릿으로 관리 및 배포

- **AWS 진입(목표/설계 범위)**
  - Route 53(Geo/Latency 정책 등) → 국가/리전별 NLB로 라우팅
  - (한국 리전) NLB → EC2 VPN 게이트웨이 → 온프레미스 LB로 VPN 연결
  - 필요 시 EC2에서 iptables(DNAT/SNAT)로 프라이빗 타깃 접근
  -
  ```
  aws [ ELB <---> EC2(HAProxy) ] <---> [ VM(HAProxy) <---> VM ] on-premise
  ```

<img width="2400" height="1792" alt="Image" src="https://github.com/user-attachments/assets/7fd6cc3e-2bea-430a-8389-0ca2f11aca6a" />

## 포함된 내용(구현 완료)

- **Bootstrap 자동화**
  - 관리용 원격 접속 유저 생성
  - SSH 키 배포로 비밀번호 없이 관리 가능하게 전환

- **Role 기반 구성**
  - `roles/common`: 공통 패키지/기본 설정
  - `roles/web`: 웹 노드 세팅 + 템플릿 기반 컨텐츠 배포
  - `roles/lb`: HAProxy 설치/설정 + 템플릿 기반 haproxy.cfg 배포
  - `roles/db`, `roles/vpn`: 현재는 뼈대 수준(미완)

- **Vault 기반 시크릿 관리**
  - 민감 변수는 Ansible Vault로 암호화(레포에 미포함)

- **Handlers**
  - 설정 변경 시 `haproxy/httpd/firewalld` 등 reload/restart 핸들러 호출

## 실습 토폴로지(Vagrant VM)

- controller: `192.168.56.100`
- LB-webs:   `192.168.56.101`
- LB-dbs:    `192.168.56.102`
- WEB-01:    `192.168.56.201`
- WEB-02:    `192.168.56.202`
- DB-01:     `192.168.56.151`
- DB-02:     `192.168.56.152`

`Vagrantfile`에 위 VM들이 정의되어 있다.

## 레포 구조

- `ansible.cfg` — Ansible 설정
- `inventory` — 인벤토리 파일
- `group_vars/` — 그룹 변수
- `playbooks/`
  - `bootstrap/` — 초기 접근(유저/키) 구성
  - `services/` — 웹/LB/DB 서비스 구성
- `roles/` — 역할 단위 구현(`common`, `web`, `lb`, `db`, `vpn`)
- `main.yaml` — 오케스트레이션 엔트리포인트(플레이북 import)

## 사전 준비
- Oracle VirtualBox
- Vagrant
- (컨트롤 노드에) Ansible 설치
- 초기에는 vagrant 계정(또는 비밀번호)로 접속 가능해야 함

## 커밋하지 않은 파일
아래 파일은 레포에 포함하지 않는다.
- `.vault_pass`
- `group_vars/secrets.yaml`


## 현재 상태 / 제한 사항
Web 및 LB(웹 경로) 자동화는 동작하는 수준으로 구현됨

DB 자동화는 미완(뼈대 수준)

VPN/WireGuard 및 AWS 연동은 목표 아키텍처로 문서화된 단계이며,

실제 배포 코드는 아직 완성되지 않음
