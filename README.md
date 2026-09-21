# Z-Image Turbo API 한국어 가이드: 모델 ID, 장당 가격, 호출 예제

> **장당 $0.01** 종량제. 최소 1달러부터 충전하고 OpenAI 호환 `https://api.apimart.ai/v1` 하나로 끝납니다.

**[Z-Image Turbo 모델 페이지](https://apimart.ai/ko/model/z-image-turbo)** · **[실시간 가격](https://apimart.ai/ko/pricing)** · **[API 키 발급](https://apimart.ai/ko/keys)**

장당 1센트로 카탈로그에서 가장 저렴한 이미지 모델이며 대량 구도 선별에 적합합니다.

## APIMart로 Z-Image Turbo를 호출하는 이유

- **키 하나로 전체 카탈로그.** 같은 base URL과 인증 헤더로 Z-Image Turbo를 포함한 300개 이상의 이미지·영상·언어 모델에 접근하며, 전환은 `model` 필드만 바꾸면 됩니다.
- **최소 1달러, 종량제.** 구독도 선불 요금제도 없고, 먼저 소진해야 하는 무료 한도도 없습니다. 표의 단가가 실제 단가입니다.
- **과금액이 응답에 포함됩니다.** 모든 호출이 `cost` / `credits_cost`를 반환하므로 월말에 추정할 필요가 없습니다.
- **비동기 작업 설계.** 제출 후 `task_id`를 받고 `GET /v1/tasks/{id}`를 폴링합니다. 배치와 재시도는 일반적인 큐 로직입니다.

## 모델 ID와 엔드포인트

| 필드 | 값 |
| --- | --- |
| `model` | `z-image-turbo` |
| endpoint | `POST https://api.apimart.ai/v1/images/generations` |
| task | GET /v1/tasks/{id} |

## 실측 가격

<!-- pricing:model:start -->
| 출력 | 단가 |
| --- | --- |
| `default` | $0.01 |
| `prompt_extend` | $0.02 |
<!-- pricing:model:end -->

## 요청 파라미터

| 필드 | 값 |
| --- | --- |
| `model` | `z-image-turbo` |
| `prompt_extend` | `true / false` |
| `size` | `1:1 / 16:9` |
| `n` | `1-4` |

## 60초 시작하기

```bash
export APIMART_API_KEY="<token>"
curl --request POST \
  --url https://api.apimart.ai/v1/images/generations \
  --header "Authorization: Bearer $APIMART_API_KEY" \
  --header 'Content-Type: application/json' \
  --data '{"model":"z-image-turbo", "prompt":"a cozy reading nook by a rainy window, warm lamp light", "n":1}'
```

```python
import os, time, requests

BASE = "https://api.apimart.ai/v1"
HEADERS = {"Authorization": f"Bearer {os.environ['APIMART_API_KEY']}", "Content-Type": "application/json"}

r = requests.post(f"{BASE}/images/generations", headers=HEADERS, timeout=60, json={
    "model": "z-image-turbo",
    "prompt": "a cozy reading nook by a rainy window, warm lamp light",
    "n": 1,
})
r.raise_for_status()
task_id = (r.json().get("data") or {}).get("id")
while True:
    t = requests.get(f"{BASE}/tasks/{task_id}", headers=HEADERS, timeout=60).json().get("data", {})
    if t.get("status") in ("completed", "failed"):
        print(t.get("status"), t.get("cost"))
        break
    time.sleep(5)
```

## 대량 실행 시 비용

| 사용량 | 비용 |
| --- | --- |
| 1,000 | $10 |
| 10,000 | $100 |

위 단가로 선형 계산한 추정치이며 단계 할인은 반영하지 않았습니다. 예산 책정 전에 실시간 가격을 확인하세요.（snapshot 2026-09-21）

## 첫 호출 문제 해결

| 증상 | 원인 | 해결 |
| --- | --- | --- |
| `401` / invalid api key | 키 누락, 잘림, 헤더에 줄바꿈 혼입 | 콘솔에서 다시 복사하세요. 헤더는 `Authorization: Bearer $APIMART_API_KEY` 형식 |
| 잔액 부족 / credit 오류 | 계정 잔액이 없음 | 콘솔에서 최소 1달러를 충전하세요. 무료 한도는 없습니다 |
| `429` | 동일 키의 동시 요청 과다 | 백오프 후 재시도하고, 재시도 시 같은 `Idempotency-Key`를 사용하세요 |
| `400` / model 없음 | 모델 ID 또는 파라미터 오류 | 위 표의 `model` 값을 그대로 사용하세요. 단계별로 필드명이 다릅니다 |
| 작업 `failed` | 프롬프트 필터 또는 참조 이미지 URL 만료 | 새 `Idempotency-Key`로 다시 제출하고 참조 이미지를 재호스팅하세요 |

## 자주 묻는 질문

**과금 단위는 무엇인가요?**

이미지는 장당, 영상은 초당, 언어 모델은 100만 토큰당입니다. 금액은 작업 응답에 포함되어 건별로 확인할 수 있습니다.

**사용 내역을 볼 수 있나요?**

콘솔의 결제 페이지에서 호출별 소비와 잔액 변동을 확인할 수 있습니다.

**어떤 언어로 호출할 수 있나요?**

HTTP를 보낼 수 있으면 됩니다. OpenAI 호환이라 Python은 openai SDK의 base_url만 바꾸면 됩니다.

**결과 URL이 만료되나요?**

만료됩니다. 작업 완료 후 즉시 자체 스토리지에 저장하세요.

## 고지

이 저장소는 서드파티 중계 서비스 APIMart 사용 가이드이며 모델 제공사와 무관합니다. 가격과 파라미터는 저장소에 표기된 스냅샷 기준이고, 실제 청구는 플랫폼 명세서가 기준입니다.

## 저장소 구조

```
README.md            本文件
data/model.json      模型 ID、价格快照、参数
examples/curl.sh     curl 示例
examples/python.py   Python（提交 + 轮询）
LICENSE, .gitignore
```

## License

MIT
