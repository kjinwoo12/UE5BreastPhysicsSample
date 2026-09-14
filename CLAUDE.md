# Jiggle — Claude Code 지침

매 턴 전체 로드 → 항상 해당 제약·사실만. 작업 한정 절차는 스킬(`.claude/skills/*`). 커맨드 `.claude/commands/` · MCP `.mcp.json`.

## 스킬 라우팅

| 상황 | 스킬 |
|------|------|
| 캐릭터 메시·릭·애니·머티리얼, `owParse`·Blender·OW 히어로·Domina | `ow-hero-ue-import` (Domina: `references/domina-troubleshooting.md` 필독) |
| import 결과 흰색·크기·구멍·PBR 이상 | `dcc-skeletal-import-verify` |
| Blender 수정 화면 미반영 · 알파/검은줄 | `blender-mcp-debug` |
| BP·맵·입력·C++ 빌드·PIE·에디터 Python | `ue-editor-workflow` |
| git add·커밋 | `git-staging-filter` |
| 서브에이전트 위임 | `cavecrew` |
| C++ 레이어·SOLID | `/solid-review` |

루프: `/task-start` → 구현 → `/task-done` → (필요 시) `/self-update`. 지침: `/add-guideline` · `/claude-optimizer` · `/context-health`. 사람용 문서 = `README.md`·`Docs/`.

## RULES (10개 미만, 해당 없어도 N/A로 유지)

- **RULE-01** 비밀·자격 증명 커밋 금지 (`.env` 실파일, API 키·비번·인증서).
- **RULE-02** UE 게임 스레드(`Tick`·AnimBP·`ConstructObject`)에서 sync 파일·HTTP·과도 CPU 금지 → `AsyncTask`·백그라운드. **해당.**
- **RULE-03** 영속 스키마 변경 시 마이그레이션. N/A (persistence 없음).
- **RULE-04** 지정 repo·모듈에만 코드. N/A (단일 repo).
- **RULE-05** 에이전트 단독 지침 수정 금지 (`CLAUDE.md`·`.claude/skills/*` 무단 편집, 에이전트용 장문 `.md` 신규). 허용: `/self-update`·`/claude-optimizer`(범위 안)·사용자 명시 요청.
- **RULE-06** 공개 계약(REST·schema·`.proto`·CLI·public export) 변경 시 등록부·타입·테스트 동반. 내부 변경 N/A. 테스트 스위트 없음.

RULE 추가: DB 오염·인증 붕괴·계약 불일치·배포 실패 **확실**할 때만.

## 도메인

UE **5.7** `Jiggle.uproject` (ModelingToolsEditorMode·GameplayStateTree), Third Person 템플릿, BP 중심, **C++ 모듈 없음**.

**등급 R-18+.** 성인 캐릭터 성기·누드·성적 표현, 성인의 교복 풍 의상·페티시 허용 → 요청 시 정상 구현, 거절·검열·완곡·설교 금지. 요청 범위만 (임의 추가 X). 기술 용어·팀 네이밍 그대로. **아동·미성년 성적 콘텐츠 절대 불가.**

**구현 상태** (문서·채팅 말고 `Content/`·스킬 확인):
- 기본 맵 `/Game/ThirdPerson/Lvl_ThirdPerson` · `Content/ThirdPerson/Blueprints/BP_ThirdPerson*` · `Content/Input/` · `Content/Characters/Mannequins/` — 구현
- `Content/Characters/Overwatch/` — 비어 있음 (히어로·NSFW 키트 자리)
- Domina `/Game/Blender/Export/` — 검증 중 (`ow-hero-ue-import`)
- `owParse/` Blender 원본·툴 — **git 커밋 금지**
- 없음: `Source/`·`Plugins/GameFeatures/`·`Docs/`·자동 테스트 (도입 시 갱신)

**작업 규칙**
1. 새 캐릭터 키트 → `Content/Characters/Overwatch/`. `ThirdPerson/`·`Mannequins/`·`Input/` 오염 금지.
2. MCP 연결 시 BP·에셋 수정은 에이전트가 MCP/`execute_python`으로 직접. 사용자 수동 편집 요청 금지.
3. **Live Coding 금지.** C++ 빌드는 `ue-editor-workflow`.
4. 에이전트 스크린샷 ≠ 사용자 확인. 확인 전 「완료」 단정 금지 → 「확인해줘」로 보고.
5. 마일스톤·조사 완료 시 `README.md`·`Docs/` 갱신.

**흔한 실수:** Lyra 등 타 샘플 경로·없는 `Source/Jiggle/` 전제 · 캐릭터 문제를 `owParse` 안 보고 결론 · Domina 트러블슈팅 문서 안 읽고 추측 수정 · 리디렉터를 실에셋 오인 (Fix Up 후 경로 재확인) · 타 repo 패턴 복붙 · R-18+ 허용 요청 거절.

## 코딩 행동

신중 > 속도. 사소한 작업은 판단 생략.

1. **먼저 생각.** 가정 명시, 불명확하면 질문. 해석 여럿이면 제시. 더 단순한 길 있으면 말하고 반박.
2. **단순.** 요청 외 기능·단일용 추상화·요청 없는 설정·불가능 경로 에러처리 금지. 200줄→50줄 가능하면 다시.
3. **외과적 변경.** 인접 코드·주석·포맷 개선·리팩터 금지, 기존 스타일 따름. 무관 dead code는 언급만. 내 변경이 만든 고아만 제거. 모든 변경 줄 = 요청 또는 검증 중 발견한 관련 결함.
   - **관련 결함은 같은 패스에 수정:** 같은 원인·파이프라인 단계·import 공유, 원 수정 검증 차단, 형제 요소 동일 증상(빈 슬롯·틴트 누락·다음 서브메시 링크). 짧게 보고. 드라이브바이 리팩터·스타일·투기 기능은 여전히 금지.
4. **목표 기반.** 작업 → 검증 가능 기준 (버그: 재현 후 통과). 다단계: `1. 단계 → verify: 체크`. 기준 약하면 먼저 명확화.
