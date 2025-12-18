# Repository Guidelines

이 저장소는 간단한 TIL(Today I Learned) 기록용입니다. 구조를 최대한 단순하게 유지하며, 배운 내용 하나를 루트에 있는 Markdown 파일 하나로 남깁니다.

## Project Structure & Module Organization
- 루트에는 TIL 문서와 최소한의 메타 파일만 둡니다.
- 핵심 파일:
  - `README.md`: 작성 규칙과 파일명 형식.
  - `TEMPLATE.md`: 새 문서 시작용 템플릿.
- 디렉토리, 소스코드, 이미지/자산 파일은 만들지 않습니다.

## Build, Test, and Development Commands
빌드/테스트 시스템은 없습니다. 주로 쓰는 명령은 아래뿐입니다.
- 템플릿에서 새 문서 만들기:  
  `cp TEMPLATE.md 2025-12-12-Redis-파이프라인.md`
- 변경사항 커밋하기:  
  `git add . && git commit -m "til: Redis pipeline"`

## Writing Style & Naming Conventions
작성 규칙(파일명/제목/문서 타입/섹션 흐름 등)의 **Single Source of Truth는 `README.md`와 `TEMPLATE.md`**입니다.

- 에이전트는 문서를 작성/수정할 때 항상 먼저 `README.md`와 `TEMPLATE.md`를 읽고 그 규칙을 따른다.
- 이 AGENTS.md에는 중복되는 상세 규칙을 재작성하지 않는다(드리프트 방지).

## Coaching Style
교육/코칭 방식의 Single Source of Truth는 `COACHING.md`입니다. 이 문서와 중복되는 내용을 AGENTS.md에 재작성하지 않습니다.

## Testing Guidelines
테스트 프레임워크/규칙은 적용되지 않습니다.

## Commit & Pull Request Guidelines
- 커밋 메시지는 작고 명확하게 씁니다.
  - 권장 형식: `til: <짧은 제목>`
  - 예시: `til: Redis pipeline`, `til: CSS Grid basics`
- 이 저장소는 PR을 사용하지 않습니다.
- 문서를 작성/수정하면 먼저 사용자에게 검토를 요청합니다.
- 사용자가 승인하면 그때 커밋하고 바로 `push`합니다.
  - 승인 전에는 커밋/푸시하지 않습니다.

## Agent-Specific Instructions
- 사용자가 요청하지 않는 한 디렉토리를 만들지 않습니다.
- 규칙을 바꿀 때는 `README.md`와 `TEMPLATE.md`를 함께 맞춰 수정합니다.

## Communication & Language
- 노트와 질문은 한글로 작성해도 됩니다.
- 사용자가 영어로 질문하면 에이전트는:
  1. 더 자연스러운 영어 문장으로 먼저 교정해 제시하고,
  2. 핵심 수정 포인트(문법/단어/동사)를 짧게 설명한 뒤,
  3. 원 질문에 대한 답변을 이어서 제공합니다.
- 한글 문장 안에서도 기술 용어/스킬은 영어로 유지합니다(예: `Redis`, `CSS Grid`, `Docker`).
