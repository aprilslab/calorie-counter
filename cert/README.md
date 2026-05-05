# cert/ — wildcard 인증서 폴더

## ⚠️ 이 폴더는 강사가 cert 파일을 동봉합니다

학생용 zip(`week10_calorie.zip`)에는 강사가 미리 발급한 wildcard 인증서 2장이 들어 있습니다:

```
cert/
├── fullchain.pem    ← 인증서 본체 + 체인 (공개)
└── privkey.pem      ← 개인키 (비공개) ⚠️⚠️⚠️
```

## 학기 한정 데모용

- 인증서 대상: `*.aiweb2026.site` (모든 학생 서브도메인 커버)
- 만료: 발급일 + 90일 (학기 종료 시점에 맞춰 발급)
- 같은 private key를 26명 학생이 공유 — 학기 한정 데모 목적
- **학기 종료 후 갱신 안 함**. 본인 도메인 + 본인 cert 발급은 학기말 자율 옵션.

## 절대 GitHub에 push 금지

`privkey.pem`이 노출되면 누구나 `*.aiweb2026.site` 인증서를 사칭할 수 있음. `.gitignore`에 `cert/`가 명시되어 있는지 반드시 확인.

```bash
cat .gitignore | grep cert
# → cert/ 가 보여야 함
```

## 서버 배치 (10주차 § 5-1)

```bash
sudo mkdir -p /etc/letsencrypt/live/aiweb2026.site
sudo cp ~/calorie/cert/fullchain.pem /etc/letsencrypt/live/aiweb2026.site/
sudo cp ~/calorie/cert/privkey.pem   /etc/letsencrypt/live/aiweb2026.site/
sudo chmod 600 /etc/letsencrypt/live/aiweb2026.site/privkey.pem
sudo chmod 644 /etc/letsencrypt/live/aiweb2026.site/fullchain.pem
```

## 학기말 본인 cert 발급 (자율)

본인 도메인 구매 후 Let's Encrypt 발급:

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d <본인도메인>
# 또는 wildcard라면 DNS-01 challenge:
sudo certbot certonly --dns-cloudflare \
     --dns-cloudflare-credentials ~/.secrets/cf.ini \
     -d "*.<본인도메인>"
```

자세한 절차: `09_gradio_oracle_deploy.md` 6.3 옵션 A
