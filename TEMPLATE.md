# Troubleshooting write-up template · 문제 해결 기록 양식

Copy this when you run into another issue. / 다음에 다른 문제를 만나면 복사해서 써 보세요.

| Section | What to write | 뭘 적으면 좋을까요? |
| --- | --- | --- |
| Summary · 요약 | What happened, where, and what works now? | 뭐가 어디에서 안 됐고 지금은 어떤가요? |
| Impact · 영향 | Who or what was affected? How much of that did you actually measure? | 누가 어떤 불편을 겪었나요? 직접 확인한 범위는 어디까지인가요? |
| Timeline · 타임라인 | For each key check, write its time and zone, what you tried, and what you saw. | 중요한 시험마다 시각과 시간대, 해 본 일, 관찰 결과를 적어요. |
| Evidence · 증거 | Add a short log or screenshot and say where and when it came from. | 로그나 사진을 한두 개 붙이고 기록의 출처와 시각을 적어요. |
| Hypotheses · 가설 | What might explain it? What would let you tell the explanations apart? | 뭐가 원인일 수 있나요? 무슨 시험을 하면 가설을 구분할 수 있나요? |
| Recovery · 복구 | What happened before it worked again, and how did you verify recovery? | 다시 되기 직전에 무슨 일이 있었고 정상 작동은 어떻게 확인했나요? |
| Lessons and follow-up · 배운 점과 후속 조치 | What is worth repeating? What should be measured or changed? Add an owner when known. | 다음에도 써먹을 방법은 뭔가요? 더 확인하거나 바꿀 점, 담당자가 정해졌다면 그 사람도 적어요. |

Example timeline row / 타임라인 한 줄 예시:

| Time and zone · 시각·시간대 | Test · 시험 | Observation · 결과 | Source · 근거 |
| --- | --- | --- | --- |
| YYYY-MM-DD HH:MM:SS UTC+09:00 | Reconnect to the affected network | Client sends Discover; no Offer during the sample | Client DHCP capture |

Based on [Google SRE's example postmortem](https://sre.google/sre-book/example-postmortem/) and [Cloudflare's July 2020 outage report](https://blog.cloudflare.com/cloudflare-outage-on-july-17-2020/).
