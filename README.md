# 10주차 칼로리카운터 — Oracle 배포 + GitHub Actions CI/CD

> 이 zip은 10주차 수업에서 학생이 e-class로 받는 자료입니다.
> 6주차에 만든 칼로리카운터 코드 + Docker + nginx + CI/CD + wildcard cert가 모두 들어있어, **학생은 본인 ID와 HF_TOKEN만 입력하면** 한 차시 안에 본인 칼로리카운터가 `<id>-demo.aiweb2026.site`로 떠야 합니다.

상세 절차는 [`docs/10_week10_lesson.md`](../../docs/10_week10_lesson.md) 본 강의안 참고.

---

## 1. zip 안에 들어있는 것

```
week10_calorie/
├── README.md                        ← 이 파일
├── .gitignore                       ← .env / cert/ 제외 (필수)
│
├── app.py                           ← 6주차 칼로리카운터 코드
├── model_config.py                  ← 모델 상수 + InferenceClient
├── requirements.txt                 ← Python 의존성
│
├── Dockerfile                       ← 컨테이너 이미지 정의
├── docker-compose.yml               ← 운영용 compose (메모리 제한, 헬스체크)
├── .env.example                     ← HF_TOKEN 자리 (.env로 복사 후 입력)
│
├── nginx-calorie.conf               ← nginx reverse proxy + WebSocket + TLS
│
├── cert/                            ← 강사 배포 wildcard cert (*.aiweb2026.site)
│   ├── fullchain.pem                ← 인증서 본체 ⚠️ git push 금지
│   ├── privkey.pem                  ← 개인키 ⚠️⚠️ git push 절대 금지
│   └── README.md                    ← cert 안내
│
└── .github/
    └── workflows/
        └── deploy.yml               ← GitHub Actions CI/CD 정의
```

---

## 2. 학생이 직접 만들 것 — 단 2개

| 파일 | 내용 |
|------|------|
| `.env` | `cp .env.example .env` 후 `HF_TOKEN=hf_xxx` 입력 |
| nginx 설정의 `__STUDENT_ID__` 치환 | `aa-demo`, `bb-demo` … 본인 ID로 |

나머지는 zip 동봉본 그대로 사용.

---

## 3. 빠른 시작 (수업 중 슬롯 매핑)

### 3-1. Oracle 서버에 zip 업로드 (§ 4-2)

```bash
# 학생 PC에서
scp -i ~/.ssh/oracle_e2_micro_key week10_calorie.zip ubuntu@<본인_IP>:~/

# 서버에서
ssh -i ~/.ssh/oracle_e2_micro_key ubuntu@<본인_IP>
unzip ~/week10_calorie.zip -d ~/calorie
cd ~/calorie
```

### 3-2. .env 작성 (§ 4-3)

```bash
cp .env.example .env
nano .env
# HF_TOKEN=hf_xxxxxxxxxxxxxxxxxxxxxxxx
chmod 600 .env
```

### 3-3. 첫 컨테이너 기동 (§ 4-4)

```bash
docker compose up -d --build       # ~3분
docker compose logs -f             # "Running on local URL" 확인
curl -I http://127.0.0.1:7860      # HTTP/1.1 200 OK
```

### 3-4. wildcard cert 서버 배치 (§ 5-1)

```bash
sudo mkdir -p /etc/letsencrypt/live/aiweb2026.site
sudo cp ~/calorie/cert/fullchain.pem /etc/letsencrypt/live/aiweb2026.site/
sudo cp ~/calorie/cert/privkey.pem   /etc/letsencrypt/live/aiweb2026.site/
sudo chmod 600 /etc/letsencrypt/live/aiweb2026.site/privkey.pem
sudo chmod 644 /etc/letsencrypt/live/aiweb2026.site/fullchain.pem
```

### 3-5. nginx 설치 + 본인 ID 치환 (§ 5-2)

```bash
sudo apt update && sudo apt install -y nginx
sudo cp ~/calorie/nginx-calorie.conf /etc/nginx/sites-available/calorie-counter
sudo nano /etc/nginx/sites-available/calorie-counter
# __STUDENT_ID__ → 본인 ID(aa, bb, ...) 치환

sudo ln -s /etc/nginx/sites-available/calorie-counter /etc/nginx/sites-enabled/
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t && sudo systemctl reload nginx
```

브라우저 검증:
```
https://<본인ID>-demo.aiweb2026.site
→ 자물쇠 🔒 + Gradio UI + 음식 사진 업로드 → 칼로리 응답
```

### 3-6. GitHub 리포 생성 + Secret 등록 (§ 7)

```bash
# 학생 PC에서, .env / cert/ 빼고 push
git clone https://github.com/<본인>/my-calorie-counter.git
cd my-calorie-counter
cp -r ~/Downloads/week10_calorie/* .

# .gitignore가 .env / cert/ 차단 — 이 줄 빠뜨리면 사고
cat .gitignore | grep -E "^(\.env|cert/)$"

git add .
git commit -m "init: 10주차 칼로리카운터 + CI/CD"
git push origin main
```

GitHub 리포 → Settings → Secrets → 4개 등록:
- `SSH_HOST` = Oracle Public IP
- `SSH_USER` = `ubuntu`
- `SSH_KEY` = `~/.ssh/oracle_e2_micro_key` 전체 내용 (개행 포함)
- `HF_TOKEN` = `hf_xxx`

### 3-7. 첫 서버 git 연결 (§ 8-2, 1회만)

```bash
# Oracle 서버에서
cd ~/calorie
git init
git remote add origin https://github.com/<본인>/my-calorie-counter.git
git fetch origin main
git reset --hard origin/main
```

### 3-8. 첫 자동 배포 (§ 8-3)

```bash
# 학생 PC에서, my-calorie-counter 리포에 변경 push
nano app.py    # title 한 글자 변경
git commit -am "test: 첫 자동 배포"
git push origin main
```

→ GitHub Actions 탭에서 그린 체크 → `<id>-demo.aiweb2026.site` 새로고침 → 변경 즉시 반영.

### 3-9. 9주차 페이지 "Live Demo" 살리기 (§ 9)

```bash
# 9주차 페이지 리포에서
nano index.html
# <a href="#">Live Demo (Coming Week 10)</a>
# → <a href="https://<본인ID>-demo.aiweb2026.site" target="_blank">▶ Live Demo</a>

git commit -am "feat: Live Demo 링크 활성화"
git push origin main
```

→ 1~2분 후 `<id>.aiweb2026.site` 새로고침 → 버튼이 살아남.

---

## 4. 자주 막히는 함정 8개

| # | 증상 | 처리 |
|---|------|------|
| 1 | `Permissions 0644 for ssh key are too open` | `chmod 600 ~/.ssh/oracle_e2_micro_key` |
| 2 | `Connection timed out` (SSH) | OCI Security List에서 22번 포트 허용 |
| 3 | docker compose 첫 빌드 OOM | swap 2GB 설정 (§ 3-2) |
| 4 | 외부에서 80번 접속 안 됨 | iptables REJECT 위에 ALLOW 추가 (§ 3-3) |
| 5 | 502 Bad Gateway | `docker compose ps` → 죽었으면 `logs`로 OOM/HF_TOKEN 누락 확인 |
| 6 | 자물쇠 X / Common Name mismatch | `server_name`이 본인 도메인이고 cert 경로가 정확한지 |
| 7 | 분석 무한 로딩 (UI는 뜸) | nginx WebSocket 헤더 누락 — `nginx-calorie.conf` 그대로 쓰면 OK |
| 8 | GitHub Actions `Permission denied (publickey)` | SSH_KEY가 CRLF로 들어감. 개행 포함 통째로 다시 등록 |

---

## 5. 학기 종료 후 인프라 유지

| 자산 | 학기 종료 후 |
|------|-------------|
| Oracle Always Free 인스턴스 | ✅ 영구 무료. 본인 OCI 계정에서 관리 |
| 본인 GitHub 리포 + Actions | ✅ 영구 무료 (Public 리포 무제한) |
| `aiweb2026.site` 도메인 | ❌ 강사가 1년 후 갱신 안 함. 학기 한정 |
| wildcard cert | ❌ 학기 종료 후 갱신 안 함 |
| HF_TOKEN | ✅ 본인 계정. 영구 |

→ 본인 도메인 구매(`Cloudflare_Domain_Setup_Guide.md`) + 본인 cert 발급 절차로 학기 종료 후 본인 인프라로 이전 가능.

---

**작성일**: 2026-05-04
**관련 강의**: [10주차 강의안](../../docs/10_week10_lesson.md), [9주차 강의안](../../docs/09_week09_lesson.md)
