VM 배포 실행 가이드
작성일: 2026-10-23
대상: 쇼핑몰 Shadow API 프로젝트 — 도커 컨테이너로 VM에서 실행하기

0. 미리 확인할 것
항목	확인
VM 운영체제	레드햇 계열 (RHEL/Rocky/AlmaLinux). 버전은 cat /etc/redhat-release로 확인
VM 인터넷 연결	필요함 (도커 이미지·패키지를 받아야 함)
전달받은 파일	sesac_shop_web_php_*.zip (코드 + deploy/shopdb-dump.sql 데이터 포함)
압축 파일에는 로그 파일이 빠져 있고, deploy/shopdb-dump.sql에 실제 회원 데이터(이름, 이메일, 전화번호, 주소)가 그대로 들어 있다. 압축 파일과 이 문서는 팀 바깥에 공유하지 않는다.

1. 파일 옮기고 풀기
압축 파일을 VM으로 옮긴 뒤(USB, 공유 폴더, scp 등) 원하는 위치에 풀어둔다. 아래 예시는 ~/sesac_shop_web_php에 풀었다고 가정한다.

cd ~
unzip sesac_shop_web_php_*.zip
cd sesac_shop_web_php
ls
이 안에 Dockerfile, docker-compose.yml, deploy/, config/, public/, src/, vendor/가 보여야 한다.

2. Docker 설치 (레드햇 계열)
sudo dnf install -y dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo systemctl enable --now docker
확인:

docker --version
docker compose version
둘 다 버전이 나오면 설치 완료.

3. 네트워크 대역 확인 (v1 접근 제한용)
레거시 v1(의도된 취약점)은 팀 네트워크에서만 열리도록 deploy/apache-vhost.conf에 IP 제한이 걸려 있다. VM의 실제 네트워크 대역과 다르면 이 설정이 무의미해지므로 반드시 확인한다.

ip addr show
VM의 IP(예: 192.168.0.200)를 확인하고, deploy/apache-vhost.conf를 열어 이 부분을 실제 팀 네트워크 대역으로 맞춘다.

<Directory /var/www/html/public/api-v1>
    AllowOverride All
    Require ip 192.168.0.0/24
</Directory>
수정했다면 이후 docker compose up -d --build를 다시 실행해야 반영된다(이미지에 포함되는 파일이라서).

4. 빌드 및 실행
cd ~/sesac_shop_web_php
sudo docker compose up -d --build
db 컨테이너(MySQL 8.0)가 먼저 올라오고, deploy/shopdb-dump.sql로 데이터를 자동 복원한다.
web 컨테이너(Apache + PHP 7.4)가 DB 준비 완료를 기다렸다가 올라온다.
진행 상황 보기:

sudo docker compose logs -f
Ctrl+C로 로그 보기를 멈춰도 컨테이너는 계속 돈다.

5. 정상 동작 확인
curl http://127.0.0.1/api/v2/products
curl -X POST http://127.0.0.1/api/v2/login -H "Content-Type: application/json" -d '{"email":"user1@example.com","password":"Test1234!"}'
curl "http://127.0.0.1/api/v1/users/1"
첫 번째: 상품 목록이 JSON으로 나오면 정상.
두 번째: access_token이 나오면 DB 데이터가 제대로 복원된 것.
세 번째: 토큰 없이도 회원 정보가 나오면 v1(의도된 취약점)이 정상 동작하는 것.
브라우저로도 확인:

http://VM의IP/products.php
http://VM의IP/docs/
6. 데이터 확인 후 정리
데이터가 제대로 들어간 것을 확인했으면, DB를 외부에 직접 열어둘 필요가 없으므로 docker-compose.yml에서 아래 줄을 지운다.

    ports:
      - "3306:3306"   # 이 줄 삭제
수정 후 다시 적용:

sudo docker compose up -d
(이번엔 --build 없이도 된다. 설정만 바뀐 것이므로.)

7. 팀원(해커 역할)에게 공유할 정보
접속 주소: http://VM의IP/ (쇼핑몰 화면), http://VM의IP/docs/ (v2 API 문서)
v1(/api/v1/...)은 공식 문서에 없음 — 의도적으로 숨겨둔 레거시 취약점
v1은 팀 네트워크 안에서만 접근 가능 (3단계에서 설정한 IP 대역)
8. 막혔을 때 확인할 것
증상	확인할 것
docker compose up 중 이미지 다운로드 실패	VM 인터넷 연결 상태
web 컨테이너가 계속 재시작됨	sudo docker compose logs web
DB 접속 안 됨	sudo docker compose logs db, 비밀번호가 docker-compose.yml과 deploy/mysql-init/02-user.sql에서 일치하는지
권한 거부 관련 에러	SELinux 상태 확인 — sudo ausearch -m avc -ts recent
80번 포트 접속 안 됨 (VM 안에서는 되는데 외부에서 안 됨)	방화벽 — sudo firewall-cmd --permanent --add-service=http && sudo firewall-cmd --reload
막히는 에러 메시지는 그대로 복사해서 물어보면 된다.

9. 지금까지 구현된 것 (참고)
구분	내용
v2 (정상)	회원가입·로그인·내 정보, 상품 목록·상세, 주문 생성·내 주문 조회 — 인증·입력검증·IDOR 방지 등 적용
v1 (의도된 취약점)	인증 누락(개인정보 탈취), 입력검증 누락(명령 실행) — 공식 문서에서 제외, IP 제한
로그	앱 로그(JSON), DB 슬로우 쿼리 로그 — 현재는 Windows PC에서만 와주 연동 중 (VM에서는 별도 설정 필요)
아직 안 한 것	ModSecurity(WAF), VM에서의 와주 에이전트 연동, 운영 보강(로그인 시도 제한 등)
