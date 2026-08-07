# typepack-telegram

Telegram 엔티티 타입팩 — VINEYARD Type Pack (`vineyard:typepack`).

채널/그룹/계정을 **가입하지 않고** 수집하는 tgpeek 게이트웨이 + telegram-peek 플러그인과 짝을 이룹니다.
게이트웨이가 반환하는 `EntityInfo`/`Post`/`Participant`가 이 팩의 타입/프로퍼티에 거의 1:1로 매핑됩니다.

## 타입 (category: `telegram`)

| 타입 | label | icon | 정체성 | 설명 |
|---|---|---|---|---|
| `telegram.user` | `display_name` | user | `telegram_id` | 계정. **봇은 별도 타입이 아니라 `is_bot` 플래그** (같은 peer id 공간, 같은 필드) |
| `telegram.channel` | `title` | megaphone | `telegram_id` | 브로드캐스트 채널 |
| `telegram.group` | `title` | users | `telegram_id` | 그룹/슈퍼그룹. `invite_hash` 프로퍼티로 초대 링크 도달 경로 보존 |
| `telegram.post` | `text` | message-square | `telegram_id`+`message_id` | 게시글. 게시글 내 지표(도메인/해시/주소)를 post에 연결하기 위해 노드로 유지 |

## 엣지 (category: `telegram`)

- `participant_of` / `admin_of` — `telegram.user → telegram.group`(/channel)
- `posted_in` — `telegram.post → telegram.group|channel`
- `forwarded_from` / `replied_to` — `telegram.post → telegram.post` (확산 체인 / 스레드)
- `links_to` — **`web.url → telegram.*` 증거 체인** (크로스팩: infrastructure 팩의 `web.url`; "이 URL이 이 채팅을 가리켰다" 보존)
- `same_as` — `telegram.* → identity.user_account|person` (크로스팩 귀속)

## 설계 메모

- **id 공간**: 텔레그램은 user id와 채널/그룹 id가 같은 숫자 공간을 공유(채널은 `-100...` 접두) → `telegram_id`는 **타입별 고유**로 충분하되 raw peer id(접두 포함)를 보존.
- **봇 = user 플래그**: 분리 시 "봇도 계정"이라는 사실이 타입 체계에서 사라짐. 봇 운영자 추적이 필요해지면 `operated_by` 엣지를 additive minor로 추가.
- **MAJOR 버전**: 타입 제거/정체성 변경 시에만. 프로퍼티·엣지 추가는 minor(additive).

## 검증

```bash
python3 -m pip install jsonschema
python3 -c "import json, jsonschema; jsonschema.validate(json.load(open('typepacks/telegram.json')), json.load(open('../registry/schemas/typepack.schema.json')))"
```

## 배포

`publish-packs.sh typepack-telegram` → 출력된 커밋 SHA를 `registry/registry/community-typepacks.json`에 고정 후 CI 검증.

## 라이선스

Apache-2.0 (다른 VINEYARD 팩과 동일).
