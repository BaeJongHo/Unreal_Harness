# Unreal_Harness

**언리얼 엔진 5.7 게임 프로젝트를 Claude Code로 개발할 때 쓰는 하네스 엔지니어링 프롬프트 모음.**

이 저장소는 곧바로 빌드되는 게임이 아니라, UE 프로젝트에 얹어 쓰는 **Claude Code 하네스(에이전트·스킬·훅·규약) 템플릿**이다.
핵심은 기능 개발을 **역할이 분리된 4개 서브에이전트의 파이프라인**으로 처리하는 것이다.

## 핵심: 5단계 에이전트 파이프라인

```
/ue-feature "<기능>"
   │
   ▼
 설계 ──▶ 구현 ──▶ 빌드 ──▶ 리뷰 ──▶ 블로그 글
ue-architect → ue-implementer → ue-builder → ue-reviewer → ue-doc-writer
 (읽기전용)      (편집)          (빌드/수정)     (읽기전용)      (devlog 작성)
```

각 단계는 권한이 좁혀진 전용 에이전트가 맡고, 산출물을 다음 단계로 넘긴다. **각 에이전트로 넘어가기 직전에 `gate-pipeline` 훅이 사용자 승인을 요청**하며, 승인해야만 다음 단계로 진행한다. 자세한 설계는 [docs/PIPELINE.md](docs/PIPELINE.md).

## 구성

| 경로 | 내용 |
| --- | --- |
| [`CLAUDE.md`](CLAUDE.md) | UE 5.7 프로젝트 규약 — 빌드 명령, C++ 네이밍, **GC/UPROPERTY 규칙**, 성능(Tick 최소화·풀링), 모듈 구조, **에셋/콘텐츠 규약**(네이밍·폴더·리다이렉터), 금지 경로 |
| [`.claude/agents/`](.claude/agents) | 파이프라인 서브에이전트 5종 (architect / implementer / builder / reviewer / doc-writer) |
| [`.claude/commands/ue-feature.md`](.claude/commands/ue-feature.md) | 파이프라인 진입 슬래시 커맨드 (단계마다 승인 게이트) |
| [`.claude/skills/ue-new-class/`](.claude/skills/ue-new-class) | 규약에 맞는 새 C++ 클래스(Actor/Component/Subsystem 등) 스캐폴딩 |
| [`.claude/skills/ue-crash-triage/`](.claude/skills/ue-crash-triage) | 크래시 로그/콜스택 분석 → 근본 원인 추정 |
| [`.claude/hooks/`](.claude/hooks) | `gate-pipeline`(파이프라인 단계마다 승인 요청), `guard-generated`(생성 산출물 편집 차단), `format-source`(clang-format 자동 적용) |
| [`.claude/settings.json`](.claude/settings.json) | 권한 + 훅 등록 |
| [`docs/`](docs) | [PIPELINE.md](docs/PIPELINE.md)(파이프라인 설계), [INSTALL.md](docs/INSTALL.md)(적용 가이드) |

## 빠른 시작

1. [`docs/INSTALL.md`](docs/INSTALL.md)에 따라 `CLAUDE.md` · `.claude/` · `docs/`를 UE 프로젝트 루트에 복사한다.
2. `CLAUDE.md` 1절의 `<...>` 자리표시자(프로젝트명, 엔진 경로 등)를 채운다.
3. 프로젝트에서 Claude Code를 열고:
   ```
   /ue-feature 플레이어가 'F' 키로 상호작용하는 InteractableComponent 추가
   ```

## 설계 의도

- **권한 분리**: 설계·리뷰는 파일을 못 바꾸게 해 "검토 중 몰래 수정"을 막고, 구현·빌드만 편집 권한을 가진다.
- **UE 특화 안전장치**: GC 추적(`UPROPERTY`+`TObjectPtr`) 누락, 생성 파일 편집 같은 UE 고유 함정을 규약·훅·리뷰어가 삼중으로 잡는다.
- **버전 정합성**: 설계 단계에서 UE 5.7 공식 API를 검증해 버전 차이로 깨지는 코드를 예방한다.
- **이식성**: 특정 게임에 종속되지 않은 자리표시자 기반 템플릿. 어떤 UE 5.7 C++ 프로젝트에도 얹을 수 있다.

> 환경: Windows · PowerShell · UE 5.7 기준. 다른 OS는 [`docs/INSTALL.md`](docs/INSTALL.md)의 포팅 안내 참고.
