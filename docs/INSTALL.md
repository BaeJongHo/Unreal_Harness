# 하네스 적용 가이드 (INSTALL)

이 저장소는 **언리얼 엔진 5.7 + Claude Code** 하네스 템플릿이다. 실제 UE 프로젝트에 적용하는 방법.

## 1. 파일 복사
이 저장소의 다음을 대상 UE 프로젝트 루트(`.uproject`가 있는 폴더)에 복사한다.

```
CLAUDE.md
docs/
.claude/
  ├─ agents/        (ue-architect, ue-implementer, ue-builder, ue-reviewer)
  ├─ commands/      (ue-feature)
  ├─ skills/        (ue-new-class, ue-crash-triage)
  ├─ hooks/         (format-source.ps1, guard-generated.ps1)
  └─ settings.json
```

> 이미 `CLAUDE.md`가 있는 프로젝트라면 내용을 병합한다.

## 2. 자리표시자 채우기
`CLAUDE.md` 1절의 표와 본문 `<...>` 자리표시자를 실제 값으로 교체한다. 최소한 다음은 필수:
- `<ProjectName>` — 프로젝트/주 모듈 이름
- `<UE_5.7_ENGINE_PATH>` — 엔진 설치 경로 (예: `C:\Program Files\Epic Games\UE_5.7`)
- `<PROJECT_ROOT>` — `.uproject`가 있는 절대 경로
- `<MODULE_API>` — 모듈 API 매크로(예: `MYGAME_API`). 기존 클래스 헤더에서 확인 가능.

## 3. 동작 확인
1. 프로젝트에서 Claude Code를 실행한다.
2. 서브에이전트 인식: `ue-architect`, `ue-implementer`, `ue-builder`, `ue-reviewer`가 보이는지 확인.
3. 스킬 인식: `ue-new-class`, `ue-crash-triage`.
4. 파이프라인 시험:
   ```
   /ue-feature 테스트용으로 빈 ATestActor 액터 하나 추가
   ```

## 4. 훅 동작 (Windows · PowerShell)
- **guard-generated** (PreToolUse): `*.generated.h`, `Intermediate/`, `Binaries/`, `Saved/`, `.uasset` 편집 시도를 차단.
- **format-source** (PostToolUse): 수정된 `.cpp/.h`에 `clang-format -i` 적용. `clang-format`이 없으면 조용히 통과.
  - clang-format 경로를 직접 지정하려면 환경변수 `CLANG_FORMAT_PATH`에 실행 파일 경로를 설정한다.
  - 프로젝트에 `.clang-format` 규칙 파일을 두면 그 스타일을 따른다(없으면 LLVM 기본).
- 훅 명령은 프로젝트 루트 기준 상대경로(`.claude\hooks\...`)다. 만약 훅이 동작하지 않으면, 사용 중인 OS/셸에 맞게 `settings.json`의 `command`를 절대경로 또는 `$CLAUDE_PROJECT_DIR\.claude\hooks\...` 형태로 조정한다.
- macOS/Linux를 쓴다면 `.ps1` 훅을 `.sh`로 포팅하고 `settings.json`의 `command`를 `bash` 호출로 바꾼다.

## 5. 권한
`settings.json`의 `permissions.allow`에는 안전한 git 읽기 명령만 들어 있다. 빌드/패키징 명령은 처음 실행 시 승인 프롬프트가 뜬다. 자주 쓰는 빌드 명령을 무프롬프트로 돌리려면 본인의 `settings.local.json`에 허용 규칙을 추가한다(개인 설정이라 공유되지 않음).
