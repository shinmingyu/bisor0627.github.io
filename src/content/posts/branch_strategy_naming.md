---
title: "브랜치 전략과 네이밍: 규칙이 흐름이 되기까지"
published: 2025-09-08
description: "팀의 실수는 어디서 시작되고, 어떻게 구조로 환원되는가. 반복된 충돌 끝에 구조화한 브랜치 전략과 네이밍 규칙의 회고."
image: ""
tags: ["Git", "브랜치 전략", "협업 컨벤션"]
category: Guide
draft: false
---

### 브랜치 스크립트, 그냥 자동화 광인이라 만든 건 아니었다

이 문서는 내가 브랜치 전략을 짜는 것에서 문서 배포로, 그리고 스크립트 기반 강제화로 나아가게 된 계기를 설명한다.

## 무작정 적용보다, 맞는 전략을 찾아보기

C1 프로젝트에서는 브랜치 전략이나 리뷰 규칙이 있었지만, 너무 짧은 기간 탓에 전략을 이해시킬 여유가 없었다. 그냥 "이렇게 합시다~" 하고 구두로 약속하고 넘어갔을 뿐이었고, 그에 대한 아쉬움으로 C2는 개인 프로젝트인 겸 본격적으로 팀의 Git 흐름을 설계하고, 브랜치 전략도 직접 정해보기로 했다.

나는 기존 회사에서 사용했던 전략을 변형하여, 다음과 같은 체계를 만들었다:

| 브랜치명      | 설명                          |
| ----------- | --------------------------- |
| `main`      | 실제 배포 브랜치                   |
| `develop`   | 기능 브랜치가 병합되는 기본 브랜치         |
| `feature/*` | 기능 단위 작업 브랜치 (develop에서 분기) |
| `bugfix/*`  | 버그 수정 브랜치 (develop에서 분기)    |
| `hotfix/*`  | 운영 긴급 수정 브랜치 (main에서 분기)    |

참고 링크 👉 [Git Flow Workflow - Atlassian](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)  
나는 릴리즈 브랜치를 따로 두지 않고 `main`에 태그 + 릴리즈 노트를 쓰는 쪽을 선호한다. 또한, 배포할만한 최초 버전이 나오기 전까진 굳이 main과 develop 분기의 필요성도 크게 공감하진 못하고 있으나... 비용이 크지 않기 때문에 적용하는 편이다.

브랜치 전략만큼 중요했던 건 **브랜치명 컨벤션**이었다.  

> '깃 그래프에서 브랜치명만 봐도 티켓 중 어떤 작업이 진행 중인지 알 수 있어햐 해!'  

라는 고집에 따라 아래 규격을 추구했다.

### 추구하는 기본 포맷

```text
<type>/<issue-number>-<간단한설명>
```

### 예시

| 브랜치명 | 설명 |
|----------|------|
| `feature/12-healthkit-setup` | 이슈 12번, 헬스킷 초기 작업 |
| `bugfix/45-missing-data-sync` | 이슈 45번, 누락된 데이터 동기화 수정 |
| `hotfix/73-app-launch-crash` | 이슈 73번, 앱 실행 시 크래시 수정 |

### 구성 요소 설명

- **type**: `feature`, `bugfix`, `hotfix` 중 선택
- **issue-number**: GitHub 이슈 번호 (자동 연결 & 추적 가능)
- **간단한설명**: 소문자+하이픈(-) 조합. PascalCase, 띄어쓰기 불가

## "우리 손가락 걸고 약속하는 거다?"

처음에는 문서를 배포하고, "이해했나요?" 설명한 뒤에, "우리 이렇게 갈겁니다" 하고 땅땅땅.  
그런데 그다음이 문제였다.

### 인생은 실전이다. 협업 중 발생한 문제들

| 잘못된 예시                   | 문제                          |
| ------------------------ | --------------------------- |
| `feature/AddProfileView` | PascalCase 사용으로 일관성 깨짐      |
| `bugfix/#123-fixCrash`   | `#` 포함 → CLI로 해당 브랜치 접근 어려움 |
| `develop`에 직접 push       | PR 없이 코드 유실 가능성 증가          |

어쩌면 문제라기엔... 나에게는 거슬리는 것이지만 팀원들은 아무렇지 않을 수 있는 것들이다.  
나는 팀원들을 잡도리하는 것은 잠시 접어두고, 나의 부족함을 인정하며,  
브랜치에도 보이지 않는 강제화가 필요하다는 걸 절감했다.

## 어둠의 흑마법사 - 자유를 위해 통제하기

브랜치 전략과 브랜치명을 설명하는 문서를 따로 배포했고, 팀원들이 참고할 수 있도록 PR 템플릿에도 명시했다.  
그래도 여전히 실수가 반복되었다.

그래서 다음과 같은 대책들을 도입했다:

- **Branch Protection Rule** 적용: `main`, `develop` 브랜치 보호
- RuleSet 적용하여 PR 없이 push 불가, 삭제 불가, 승인 1인 이상 필수
- 브랜치명 규칙 위반 시 로컬에서 리모트 push 불가 (스크립트 기반 린트)

### 실제 적용한 스크립트 예시

#### `Makefile` + `lefthook` + 브랜치명 검사

```make
setup:
  brew install lefthook
  lefthook install
  brew install swiftlint swiftformat
  git config commit.template .gitmessage.txt
```

- 커밋 메시지 린트: `./scripts/check-commit-msg.sh`
- 브랜치명 린트: `./scripts/check-branch.sh`
- 예외 브랜치, prefix 정규식 기반 검사

```bash
# check-branch.sh 내부 예시
PREFIX_REGEX="^(feature|bugfix|hotfix|refactor|release|test|ci|docs)/[a-z0-9._-]+$"
```

### GitHub Branch Protection Rule 예시 (요약)

```json
"conditions": {
  "ref_name": {
    "include": ["~DEFAULT_BRANCH", "refs/heads/develop", "refs/heads/main"]
  }
},
"rules": [
  {
    "type": "pull_request",
    "parameters": {
      "required_approving_review_count": 1,
      "automatic_copilot_code_review_enabled": true
    }
  },
  { "type": "non_fast_forward" },
  { "type": "deletion" }
]
```

나는 해당 Rule을 만들고, 팀에서는 나만 `bypass` 권한을 갖도록 설정했다.  
급한 일도 있을 거고... 그리고 내 계정으로 처리하는 게 나으니까.

## 마치며 — 브랜치 전략은 흐름을 만드는 도구다

결국 브랜치 전략은 단순히 브랜치를 어떻게 나누느냐의 문제가 아니었다.  
그건 팀의 **작업 흐름**, **합의의 방식**, 그리고 **실수를 줄이기 위한 안전장치**일 것이다.

사랑한다면 자유롭게 두라는 어떤 영상 속 메시지가 떠오른다.
그렇지만 나는 마냥 자유로운 자유는 자유로울 수 없다고 생각한다.
내가 실수해도 구조적으로 방어되는 환경이, 오히려 팀원들을 더 자유롭게 만든다.

은탄환은 없지만, 우리 팀이 안정적으로 협업하려면, 각자의 실수와 경험을 녹여 만든 ‘합의된 흐름’은 반드시 필요하다고 믿는다.
