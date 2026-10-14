# 아랑재 게스트 가이드 챗봇 — Cloudflare Worker

게스트가 /guide 페이지에서 묻는 질문에 가이드 내용을 근거로 답합니다.

```
브라우저(arangjae.com/guide) → 이 Worker → OpenRouter → Claude
```

API 키는 Cloudflare에만 저장되고 브라우저로는 절대 내려가지 않습니다.


## 처음 배포할 때 (한 번만)

프로젝트 폴더에서 아래 순서대로 실행하세요.

```bash
cd chatbot
```

### 1. Cloudflare 로그인

```bash
npx wrangler login
```

브라우저가 열리면 허용을 누르세요.

### 2. API 키 등록

```bash
npx wrangler secret put OPENROUTER_API_KEY
```

입력창이 뜨면 OpenRouter 키(`sk-or-...`)를 붙여넣고 엔터.
한 번 넣으면 Cloudflare에 암호화되어 저장되고, 다시 넣을 일은 없습니다.

### 3. 배포

```bash
npx wrangler deploy
```

끝나면 주소가 출력됩니다. 이렇게 생겼습니다.

```
https://arangjae-chat.<계정이름>.workers.dev
```

**이 주소를 알려주세요.** 가이드 페이지의 채팅창이 이 주소를 보도록 연결해야 합니다.


## 요청 제한 켜기 (권장, 선택)

누군가 장난으로 질문을 쏟아부어 요금이 나가는 걸 막습니다.

```bash
npx wrangler kv namespace create RATE
```

출력된 `id = "..."` 를 복사해서 `wrangler.toml` 의 주석을 풀고 붙여넣으세요.

```toml
[[kv_namespaces]]
binding = "RATE"
id = "복사한_id"
```

다시 배포하면 적용됩니다.

```bash
npx wrangler deploy
```

설정하지 않아도 챗봇은 동작합니다. 다만 그때는 OpenRouter 쪽 지출 한도만이
유일한 안전장치이니, **OpenRouter에서 월 지출 한도를 꼭 걸어두세요.**

기본값은 IP당 분당 6회, 하루 60회입니다. `src/worker.js` 위쪽에서 바꿀 수 있습니다.


## 고칠 일이 생기면

| 하고 싶은 것 | 방법 |
|---|---|
| 모델 바꾸기 | `src/worker.js` 의 `MODEL` 한 줄 |
| 답변 말투·규칙 바꾸기 | `src/worker.js` 의 `systemPrompt()` |
| 답변 길이 | `MAX_TOKENS` |
| 요청 제한 | `LIMIT_PER_MINUTE`, `LIMIT_PER_DAY` |

고친 뒤에는 `npx wrangler deploy` 로 다시 올리면 됩니다.

### 안내 내용은 따로 고칠 필요 없습니다

챗봇은 `https://arangjae.com/guide/` 를 직접 읽어서 답합니다.
가이드 페이지를 고치면 **한 시간 안에 챗봇도 자동으로 최신 내용을 따라갑니다.**
같은 내용을 두 군데 관리할 필요가 없습니다.


## 비용

**Cloudflare** 무료 (하루 10만 요청까지)

**OpenRouter** 질문 1건당 — 모델에 따라 다릅니다

| 모델 | 질문당 | 월 200건 |
|---|---:|---:|
| `anthropic/claude-opus-5` | 약 100원 | 1만 5천 ~ 2만 원 |
| `anthropic/claude-sonnet-5` | 약 63원 | 1만 ~ 1만 3천 원 |
| `anthropic/claude-haiku-4.5` | 약 21원 | 3천 ~ 4천 원 |

한 게스트가 5분 안에 연달아 물으면 캐시가 걸려 더 저렴해집니다.


## 동작 확인

배포 후 터미널에서 직접 찔러볼 수 있습니다.

```bash
curl -N https://arangjae-chat.<계정이름>.workers.dev \
  -H 'Content-Type: application/json' \
  -H 'Origin: https://arangjae.com' \
  -d '{"messages":[{"role":"user","content":"짐을 미리 맡길 수 있나요?"}]}'
```

답변이 조각조각 흘러나오면 정상입니다.
