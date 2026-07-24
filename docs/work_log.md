# 프로젝트 작업일지

---

## Day 0 (2026-07-13)

### 목표
- 프로젝트 기획
- 개발 환경 준비
- 프로젝트 문서 작성

### 수행 내용
- 프로젝트 폴더 생성
- docs, docker, logs, screenshots 폴더 생성
- project_plan.md 작성
- requirements_spec.md 작성
- architecture.md 작성
- Docker 및 Ubuntu 환경 점검

### 문제점
- 없음

### 해결 내용
- 없음

### 다음 계획
- ELK Stack(Docker) 구축

## Day 1 (2026-07-14)

### 목표
- ELK Stack 구축 환경 변경
- Docker 기반 ELK Stack 구축
- Kibana 접속 환경 구성

### 수행 내용
- 기존 UTM Ubuntu Docker 환경에서 ELK Stack 구축 시도
- Elasticsearch 컨테이너 실행 오류 확인
- 문제 원인 분석 및 구축 환경 변경 결정
- MacBook Docker Desktop 환경으로 ELK Stack 구축 환경 변경
- Elasticsearch, Logstash, Kibana 컨테이너 구성
- Logstash Pipeline 설정 파일 작성
- ELK Stack 정상 실행 확인
- Kibana 접속 확인

---

### 문제점 1. Docker 실행 권한 문제

#### 문제 상황
Docker 컨테이너 상태 확인을 위해 `docker ps` 명령어 실행 시 권한 오류가 발생하였다.

#### 원인
현재 사용자 계정이 Docker 소켓(`/var/run/docker.sock`) 접근 권한을 가지고 있지 않아 Docker 명령어 실행이 불가능하였다.

#### 해결 내용
- `sudo usermod -aG docker user1` 명령어를 통해 사용자를 docker 그룹에 추가
- docker 그룹 등록 여부 확인
- 재부팅 후 사용자 그룹 권한 적용 확인

#### 결과
Docker 사용자 권한 문제를 해결하고 정상적으로 Docker 명령어 실행이 가능해졌다.

---

### 문제점 2. Elasticsearch 실행 오류

#### 문제 상황
UTM Ubuntu 환경에서 Docker 기반 ELK Stack 구축 중 Elasticsearch 컨테이너가 정상적으로 실행되지 않았다.

발생 오류:
SIGILL (0x4)
Java Runtime Environment crash
linux-aarch64


#### 원인
Apple Silicon 기반 MacBook에서 실행되는 UTM Ubuntu ARM64 환경과 Elasticsearch Docker 이미지 실행 환경 간 호환 문제가 발생하였다.

#### 해결 내용
ELK Stack 실행 위치를 UTM Ubuntu 환경에서 MacBook Docker Desktop 환경으로 변경하였다.

변경된 구조:
MacBook Docker Desktop
├── Elasticsearch
├── Logstash
└── Kibana

Ubuntu Server (UTM)
└── Filebeat 및 로그 수집 대상 서버


#### 결과
- Elasticsearch 실행 문제 해결
- ELK Stack 정상 구축 환경 확보
- Ubuntu는 로그 발생 서버, MacBook은 관제 시스템 역할로 분리하여 보안관제 구조 개선

---

### 문제점 3. Logstash 실행 오류

#### 문제 상황
ELK Stack 구축 후 Logstash 컨테이너가 실행 직후 종료되는 문제가 발생하였다.

#### 원인
Logstash Pipeline 설정 파일(`logstash.conf`)이 존재하지 않아 로그 처리 구성이 없는 상태였다.

#### 해결 내용
- Logstash Pipeline 설정 파일 생성
- Beats 입력 설정 추가
- Elasticsearch 출력 설정 추가

#### 결과
Elasticsearch, Logstash, Kibana 컨테이너가 모두 정상 실행되었다.

---

### 최종 결과
- Docker 기반 ELK Stack 구축 완료
- Elasticsearch, Logstash, Kibana 정상 동작 확인
- Kibana 접속 확인 완료

### 다음 계획
- Ubuntu 서버 Filebeat 설치
- Linux 시스템 로그 수집 설정
- ELK Stack 로그 연동 및 분석 환경 구축

## Day 2 (2026-07-15)

### 목표
- Ubuntu 시스템 로그를 Filebeat를 통해 ELK Stack으로 전송하고 Kibana에서 확인

### 수행 내용
- Filebeat 설치 및 system 모듈 활성화
- filebeat.yml 수정(Logstash 출력 설정)
- Ubuntu와 MacBook(Docker Logstash) 네트워크 연결 확인
- Filebeat → Logstash → Elasticsearch 연동
- Kibana Data View 생성
- SSH 로그인 실패 이벤트 생성
- auth.log에서 이벤트 확인
- Kibana Discover에서 로그 수집 확인

### 문제점
- Filebeat에서 Logstash 연결 시 `dial tcp ... i/o timeout` 오류 발생

### 해결 내용
- MacBook에서 ELK Docker 컨테이너 실행 여부 확인
- `docker compose up -d` 명령어로 Elasticsearch, Logstash, Kibana 재실행
- `sudo filebeat test output`을 통해 Logstash 연결 및 통신 성공 확인
- Kibana에서 SSH 로그인 실패 로그가 정상적으로 수집되는 것을 확인

### 다음 계획
- Kali Linux를 이용한 공격 시나리오 수행
- Nmap 포트 스캔 및 Ping Sweep 테스트
- SSH Brute Force 실습
- Kibana에서 공격 로그 분석

## Day 3 (2026-07-17)

### 목표
- SSH 로그인 실패 이벤트를 생성하고 ELK Stack에서 로그 수집 및 탐지 여부 확인

### 수행 내용
- Kali Linux에서 존재하지 않는 계정으로 SSH 로그인을 시도하여 로그인 실패 이벤트 생성
- Ubuntu auth.log에서 SSH 인증 실패 로그 확인
- Filebeat를 통해 수집된 로그가 Elasticsearch에 정상 저장되는 것을 확인
- Kibana Discover에서 SSH 로그인 실패 이벤트가 정상적으로 탐지되는 것을 확인
- 결과 화면을 스크린샷으로 저장

### 문제점
- Filebeat가 Logstash와 연결되지 않아 Kibana에 로그가 수집되지 않음
- 잘못된 Logstash IP 주소 설정으로 인해 dial tcp ... i/o timeout 오류 발생

### 해결 내용
- Logstash 출력 주소를 UTM 공유 네트워크 게이트웨이(`192.168.64.1:5044`)로 변경
- `sudo filebeat test output` 명령으로 Logstash와 정상적으로 연결되는 것을 확인
- Filebeat 재시작 후 Kibana에서 SSH 로그인 실패 로그가 정상적으로 수집되는 것을 확인

### 다음 계획
- SSH Brute Force 공격 시나리오 수행
- Kibana에서 공격 로그 시각화 구성
- Nmap Port Scan 및 추가 보안 이벤트 생성

## Day 4 (2026-07-21)

### 목표
- SSH Brute Force 공격 이벤트를 생성하고 Kibana를 활용하여 공격 로그 탐지 및 시각화 구성

### 수행 내용
- Hydra를 활용하여 SSH Brute Force 공격 수행
- Ubuntu auth.log에서 반복적인 SSH 로그인 실패 로그 확인
- Filebeat를 통해 수집된 SSH 인증 로그가 Elasticsearch에 정상 저장되는 것을 확인
- Kibana Discover에서 Failed password 검색을 통해 SSH 로그인 실패 이벤트 분석
- Kibana Lens를 활용하여 SSH 로그인 실패 이벤트 추이 시각화 생성
- 생성한 시각화를 Dashboard에 추가하여 보안 이벤트 모니터링 화면 구성
- 추가적으로 Nmap을 이용한 Ubuntu 서버 대상 Port Scan 수행

### 문제점
- Kibana Lens에서 SSH 로그인 실패 로그 검색 시 데이터가 표시되지 않는 문제 발생
- 정상적으로 수집된 로그임에도 시각화 화면에서 이벤트 확인 불가

### 해결 내용
- Kibana의 시간 범위 설정을 확인하고 로그 발생 시간이 포함되도록 범위를 변경
- Lens에서 사용하는 Data View와 Elasticsearch 인덱스(security-logs-*) 설정 확인
- KQL 검색 조건(message : "Failed password")을 적용하여 SSH 로그인 실패 이벤트 정상 조회 확인

### 다음 계획
- Port Scan 결과 분석
- sudo 실행 이벤트 생성 및 분석
- Kibana Dashboard 최종 구성

## Day 5 (2026-07-23)

### 목표
- 추가 보안 이벤트를 생성하고 공격 시나리오 분석 및 Kibana 기반 보안관제 Dashboard 완성

### 수행 내용
- Kali Linux에서 Nmap을 이용하여 Ubuntu 서버 대상 Port Scan 수행
- Nmap 결과를 기반으로 열린 포트 및 서비스 정보 확인
- FTP, Telnet, RPC/NFS 등 외부 노출 서비스에 대한 보안 위험 분석
- Port Scan 공격 시나리오 및 탐지 관점 분석
- Ubuntu에서 sudo 명령을 실행하여 관리자 권한 사용 이벤트 생성
- Kibana Discover에서 sudo 관련 auth.log 이벤트 정상 수집 확인
- Kibana Lens를 활용하여 Sudo Activity 시각화 생성
- 기존 SSH 로그인 실패 시각화와 함께 Dashboard 구성 완료

### 문제점
- 기존에 생성한 Kibana Dashboard가 목록에서 확인되지 않는 문제 발생
- Docker 재실행 이후 Dashboard 및 Visualization 저장 상태 확인 필요

### 해결 내용
- Kibana Dashboard 및 Saved Objects 목록 확인
- 기존 Visualization 존재 여부 확인
- Dashboard를 복구하고 Sudo Activity 시각화를 추가하여 최종 Dashboard 구성 완료

### 다음 계획
- 프로젝트 수행 결과 정리
- README 및 GitHub 저장소 작성
- 포트폴리오용 프로젝트 문서 정리

## Day 6 (2026-07-24)

### 목표
- 프로젝트 결과 정리 및 최종 산출물 작성

### 수행 내용
- Kibana Dashboard 최종 구성 및 시각화 검토
- 주요 공격 시나리오(SSH Brute Force, Port Scan, sudo 이벤트) 분석 결과 정리
- 프로젝트 기술 스택 및 시스템 구성 문서 정리
- GitHub README 작성 및 핵심 스크린샷 정리
- GitHub Repository 최종 점검 및 프로젝트 업로드 완료

### 결과
- ELK Stack 기반 Linux 보안관제 환경 구축 완료
- Linux 시스템 로그 수집부터 공격 시나리오 수행, 탐지 및 시각화까지 전체 보안관제 프로세스 구현
- GitHub를 통한 프로젝트 문서화 및 포트폴리오 공개 완료



