# VM웨어 우분투에서 처음부터 끝까지 — 완전 초보용 가이드

작성일: 2026-10-23
전제: VM웨어 안에 우분투만 설치되어 있고, 다른 건 아무것도 안 깔려 있는 상태

---

## 0. 먼저 알아둘 것 — "터미널"이 뭔데?

Windows의 **cmd(명령 프롬프트)**와 같은 역할을 하는 게 우분투(리눅스)에서는 **터미널(Terminal)**이라고 불러. "cmd 창을 연다"는 말 대신 "터미널을 연다"고 생각하면 돼. 안에 입력하는 명령어도 Windows cmd와는 다른 리눅스 명령어를 써.

### 터미널 여는 법

우분투 VM 화면에 데스크톱(바탕화면)이 보인다면:

- 키보드로 **`Ctrl` + `Alt` + `T`**를 누르면 터미널이 바로 열려.
- 또는 화면 왼쪽 아래(또는 왼쪽 세로줄)의 **"프로그램 표시(Show Applications)"** 아이콘을 눌러서 `Terminal`을 검색해서 눌러도 돼.

검은 화면에 글자가 깜빡이는 창이 열리면 성공이야. 앞으로 "터미널에서 실행"이라고 하면 이 창에 입력하면 된다는 뜻이야.

### 명령어 입력하는 법 (중요한 습관)

- 한 줄 치고 **Enter**를 눌러야 실행돼. (Windows cmd와 같음)
- `sudo`로 시작하는 명령은 **관리자 비밀번호**를 물어봐. 비밀번호를 칠 때 **화면에 글자나 점이 하나도 안 보여도 정상이야.** 리눅스는 비밀번호 입력할 때 아예 표시를 안 해줘. 그냥 다 치고 Enter 누르면 돼.
- 명령어는 대소문자를 구분해. `Sudo`와 `sudo`는 다른 걸로 인식해서 오류가 나.
- 긴 명령을 복사해서 붙여넣을 땐, 터미널에서는 보통 **마우스 오른쪽 클릭 → 붙여넣기**를 쓰거나, `Ctrl+Shift+V`를 써. (`Ctrl+V`만 누르면 안 될 수 있어)

---

## 1. 압축 파일을 VM 안으로 옮기기

지금 네 압축 파일(`sesac_shop_web_php_*.zip`)은 **Windows PC**에 있고, 작업은 **VM 안(우분투)**에서 해야 해. 그러니 이 파일을 VM 안으로 옮기는 게 첫 단계야. VM웨어에서 가장 쉬운 방법은 **공유 폴더** 기능이야.

### 1-1. VM웨어에서 공유 폴더 켜기

1. 우분투 VM을 **끈 상태** 또는 **켠 상태**에서 (VM웨어 버전에 따라 다름) VM 설정 화면을 열어. VM웨어 창 위쪽 메뉴에서 **VM(가상 머신) → Settings(설정)**을 눌러.
2. **Options(옵션)** 탭 → **Shared Folders(공유 폴더)**를 찾아.
3. **Always enabled(항상 사용)**을 선택.
4. **Add(추가)** 버튼을 눌러서, Windows의 `D:\sesac_shop_web_php`가 들어있는 폴더(또는 압축 파일이 있는 폴더, 예: `D:\`나 바탕화면)를 선택해.
5. 저장하고 VM을 켠다(또는 이미 켜져 있으면 재부팅).

### 1-2. 우분투 안에서 공유 폴더 찾기

터미널에서 실행해봐.

```bash
ls /mnt/hgfs/
```

Windows에서 공유로 지정한 폴더 이름이 보이면 성공이야. 안 보이면 VM웨어 도구(VMware Tools 또는 open-vm-tools)가 설치되어 있어야 하는데, 터미널에서 이걸로 설치할 수 있어.

```bash
sudo apt update
sudo apt install -y open-vm-tools open-vm-tools-desktop
```

설치 후 VM을 재부팅하고 다시 `ls /mnt/hgfs/`를 확인해.

### 1-3. 압축 파일을 내 계정 폴더로 복사

공유 폴더가 보이면, 거기서 내 작업 폴더로 복사해.

```bash
cp /mnt/hgfs/공유폴더이름/sesac_shop_web_php_*.zip ~/
```

`공유폴더이름`은 1-1에서 지정한 폴더 이름으로 바꿔 넣어. `~/`는 "내 계정의 홈 폴더"라는 뜻이야.

### 1-4. (공유 폴더가 안 되면) 대안: USB 사용

공유 폴더 설정이 복잡하면, Windows에서 USB에 압축 파일을 넣고, VM웨어 메뉴에서 **VM → Removable Devices(이동식 장치) → USB 장치 이름 → Connect(연결)**를 선택해서 우분투에 USB를 인식시킨 다음, 우분투의 파일 관리자(또는 터미널의 `cp` 명령)로 복사해도 돼.

---

## 2. 압축 풀기

터미널에서 실행.

```bash
cd ~
```
→ 내 홈 폴더로 이동한다는 뜻.

```bash
unzip sesac_shop_web_php_*.zip
```
→ 압축을 푼다. 만약 `unzip: command not found`(명령을 찾을 수 없음)라고 나오면, 아직 그 프로그램이 안 깔려 있는 거니까 이걸 먼저 실행해.

```bash
sudo apt update
```
→ 설치 가능한 프로그램 목록을 최신으로 갱신한다. 시간이 10~30초 걸릴 수 있어.

```bash
sudo apt install -y unzip
```
→ 압축 풀기 프로그램을 설치한다. 설치 끝나면 위의 `unzip sesac_shop_web_php_*.zip`을 다시 실행해.

```bash
cd sesac_shop_web_php
```
→ 방금 압축을 푼 폴더 안으로 들어간다.

```bash
ls
```
→ 지금 폴더 안에 뭐가 있는지 보여준다. `Dockerfile`, `docker-compose.yml`, `deploy`, `config`, `public`, `src`, `vendor` 같은 이름들이 보여야 해.

---

## 3. Docker 설치

**Docker**는 "컨테이너"라는 걸 만들고 실행해주는 프로그램이야. 지금 우분투에는 전혀 안 깔려 있으니 하나씩 설치해야 해. 아래 명령을 **한 줄씩** 순서대로 입력하고 Enter를 눌러.

```bash
sudo apt update
```
→ 역시 목록 갱신. 비밀번호를 물어보면 입력.

```bash
sudo apt install -y ca-certificates curl gnupg
```
→ Docker를 설치하기 위해 먼저 필요한 기본 도구들을 설치한다.

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```
→ Docker의 보안 열쇠(key)를 저장할 폴더를 만든다.

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
→ Docker 공식 사이트에서 보안 열쇠를 받아서 저장한다. 이 열쇠가 있어야 "진짜 Docker 회사가 만든 프로그램"이라는 걸 확인하고 설치할 수 있어.

```bash
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```
→ 방금 받은 열쇠 파일을 누구나 읽을 수 있게 권한을 바꾼다.

다음 명령은 **길어 보이지만 한 번에 통째로 복사해서 붙여넣고** Enter를 누르면 돼. 줄바꿈이 있어도 하나의 명령이야.

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
→ "Docker 프로그램을 어디서 받을지"를 우분투에 알려주는 설정을 추가한다.

```bash
sudo apt update
```
→ 방금 추가한 곳에서 받을 수 있는 목록을 다시 갱신한다.

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```
→ 드디어 Docker 본체를 설치한다. 시간이 1~3분 정도 걸릴 수 있어.

```bash
sudo systemctl enable --now docker
```
→ Docker를 지금 실행시키고, 앞으로 VM을 켤 때마다 자동으로 켜지게 설정한다.

### 설치됐는지 확인

```bash
docker --version
```
→ `Docker version 2x.x.x, build ...` 같은 글자가 나오면 성공.

```bash
docker compose version
```
→ 이것도 버전이 나오면 성공.

둘 중 하나라도 "command not found"가 나오면, 위 단계 중 하나가 실패한 거니까 그 단계의 출력 내용을 그대로 복사해서 물어보면 돼.

### (선택) 매번 sudo 안 치고 싶으면

```bash
sudo usermod -aG docker $USER
```
→ 내 계정을 docker 그룹에 넣어서, 앞으로 `sudo` 없이 `docker` 명령을 쓸 수 있게 한다. **이 설정은 터미널을 새로 열거나 재부팅해야 적용돼.**

```bash
reboot
```
→ VM을 재부팅한다. (재부팅하라는 뜻. `sudo`가 필요할 수도 있어: `sudo reboot`)

이후 이 문서의 모든 `sudo docker ...` 명령에서 `sudo`를 빼고 써도 되는데, 헷갈리지 않게 이 문서는 계속 `sudo`를 붙여서 적어둘게. (`sudo`를 붙여도 항상 똑같이 동작하니 그냥 그대로 따라 해도 문제없어.)

---

## 4. 네트워크 주소(IP) 확인하기

나중에 다른 사람이 이 서버에 접속하려면 VM의 주소를 알아야 하고, 설정 파일 한 곳도 이 주소에 맞게 고쳐야 해.

```bash
ip addr show
```

결과에서 `inet 192.168.0.XXX` 같은 줄을 찾아. `inet 127.0.0.1`은 VM 자기 자신을 가리키는 특수 주소라서 무시하고, 그 외의 `192.168.x.x` 같은 숫자가 VM의 진짜 주소야. 이 숫자를 메모해둬.

---

## 5. 접근 제한 설정 파일 고치기

이 프로젝트에는 "레거시 v1"이라는, 일부러 취약하게 만든 기능이 있는데, 이게 팀 네트워크 밖(인터넷)에 노출되면 안 돼. 그래서 설정 파일 하나를 고쳐야 해.

```bash
nano deploy/apache-vhost.conf
```

→ `nano`라는 쉬운 글자 편집기로 파일을 연다. 화면 아래쪽에 `^O Write Out`(저장), `^X Exit`(나가기) 같은 안내가 항상 보여.

파일 안에서 이 부분을 찾아봐. 방향키로 움직여서 찾으면 돼.

```apache
<Directory /var/www/html/public/api-v1>
    AllowOverride All
    Require ip 192.168.0.0/24
</Directory>
```

`192.168.0.0/24` 부분을, 4단계에서 확인한 VM 주소에 맞게 바꿔. 예를 들어 VM 주소가 `192.168.1.50`이었다면 `192.168.1.0/24`로 바꾸면 돼. (맨 끝 숫자만 `0`으로 바꾸고 `/24`는 그대로 둔다고 생각하면 쉬워.)

고친 뒤:

- **`Ctrl` + `O`** 를 눌러서 저장 (그다음 Enter를 한 번 더 눌러야 확정돼)
- **`Ctrl` + `X`** 를 눌러서 나가기

---

## 6. 빌드하고 실행하기

드디어 메인 단계야. 지금 있는 폴더(`sesac_shop_web_php`) 안에서 실행해야 해. 지금 터미널이 그 폴더 안에 있는지 확인하려면:

```bash
pwd
```
→ 현재 위치를 보여준다. `/home/계정이름/sesac_shop_web_php`처럼 끝나면 맞는 위치야. 아니라면:

```bash
cd ~/sesac_shop_web_php
```
→ 다시 그 폴더로 이동.

이제 실제로 실행.

```bash
sudo docker compose up -d --build
```

이 명령이 하는 일:
- `docker-compose.yml`이라는 설계도 파일을 읽는다.
- 그 안에 적힌 대로 DB(MySQL)와 웹서버(Apache+PHP) 두 개를 새로 만든다 (`--build`가 "새로 만들어라"는 뜻).
- `-d`는 "백그라운드로 돌려라"는 뜻 — 터미널을 계속 띄워놓지 않아도 계속 돌아간다는 뜻이야.

**이 명령은 1~5분 정도 걸릴 수 있어.** 인터넷에서 필요한 프로그램들을 받는 과정이 있어서 그래. 화면에 글자가 쭉 올라가다가, 맨 마지막에 `Started` 또는 `done` 같은 글자가 나오고 다시 입력할 수 있는 상태(`$` 표시)로 돌아오면 끝난 거야.

### 중간에 멈춘 것 같을 때

화면이 몇 분째 그대로면 인터넷에서 큰 파일을 받는 중일 수 있어. 조금 더 기다려봐. 만약 빨간 글자로 `error`가 뜨고 멈추면, 그 화면 내용을 전체 복사해서 물어보면 돼.

---

## 7. 잘 됐는지 확인하기

```bash
sudo docker compose ps
```
→ 지금 돌고 있는 것들을 보여줘. `db`와 `web` 두 개가 보이고, 상태가 `Up`이라고 나오면 정상이야.

```bash
curl http://127.0.0.1/api/v2/products
```
→ 화면에 `[{"id":1,"name":...` 처럼 상품 목록이 글자로 쭉 나오면 성공이야. `curl: command not found`가 나오면 `sudo apt install -y curl`로 설치하고 다시 해봐.

```bash
curl "http://127.0.0.1/api/v1/users/1"
```
→ 이것도 `{"id":1,"name":...}` 같은 회원 정보가 나오면, 레거시 v1(의도된 취약점)도 정상이라는 뜻이야.

---

## 8. 다른 사람이 접속할 수 있는 주소

4단계에서 확인한 VM 주소를 써서, 같은 네트워크에 있는 다른 컴퓨터의 브라우저에서 이렇게 열면 돼.

```
http://VM주소/products.php      ← 쇼핑몰 화면
http://VM주소/docs/             ← API 문서
http://VM주소/api/v1/users/1    ← 레거시 취약점 (팀 네트워크에서만 열림)
```

---

## 막혔을 때 체크리스트

| 상황 | 해볼 것 |
| --- | --- |
| `sudo`쳤는데 비밀번호가 안 먹히는 것 같음 | 화면에 아무것도 안 보여도 정상이야. 그냥 끝까지 치고 Enter |
| `command not found` | 그 프로그램이 아직 설치 안 된 것. `sudo apt install -y 프로그램이름`으로 설치 |
| `docker compose up` 중 멈춤 | 인터넷 연결 확인. VM 설정에서 네트워크가 "NAT" 또는 "Bridged"로 되어 있는지 확인 |
| 터미널에 긴 글씨가 안 보이거나 깨짐 | 터미널 창 크기를 키워봐 |
| 붙여넣기가 안 됨 | `Ctrl+V` 대신 마우스 오른쪽 클릭 → 붙여넣기, 또는 `Ctrl+Shift+V` |

막히는 화면은 전체 내용을 그대로 복사해서 보여주면 같이 봐줄 수 있어.
