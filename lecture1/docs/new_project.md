# 새로운 React 프로젝트 세팅 가이드 (백업 복사 방식)

이 문서는 AI가 백업 템플릿을 사용하여 새로운 React 프로젝트를 빠르게 세팅하는 방법을 설명합니다.

## 사용자 요청 예시
```
{프로젝트명}으로 새로운 프로젝트 하나 세팅해줘
```

## AI가 수행해야 하는 작업 순서

### 1. 백업 템플릿 존재 확인
```bash
# _template_settings 디렉토리 존재 확인
ls -la | grep _template_settings

# 템플릿이 없는 경우, 사용자에게 안내:
# "백업 템플릿이 없습니다. 먼저 기본 프로젝트를 생성하고 백업을 만들어주세요."
```

### 2. 템플릿 복사 및 기본 설정
```bash
# 1. 백업 템플릿을 새 프로젝트명으로 복사 (OS별 명령어)
# Windows (PowerShell):
powershell -Command "Copy-Item -Path '_template_settings\template-setup' -Destination '{프로젝트명}' -Recurse"

# macOS/Linux:
cp -r _template_settings/template-setup {프로젝트명}

# 2. 프로젝트 디렉토리로 이동
cd {프로젝트명}

# 3. package.json의 프로젝트명 수정
# name 필드를 새 프로젝트명으로 변경
```

### 3. 기본 정리 작업
```bash
# 1. node_modules는 유지 (설치 시간 단축을 위해)
# - 백업 템플릿의 node_modules에는 이미 최신 MUI 패키지가 설치되어 있음
# - 복사된 node_modules를 그대로 사용하여 npm install 시간 단축

# 2. package-lock.json 유지 (정확한 버전 고정을 위해)
# - 백업 템플릿과 동일한 패키지 버전 사용

# 3. 불필요한 파일 정리
rm -rf .git  # 기존 git 히스토리 제거 (필요시)
rm -rf template-setup  # 중첩된 디렉토리 제거 (발생 시)
```

### 4. 패키지 확인 및 업데이트 (선택사항)
```bash
# 설치된 패키지 확인
npm ls

# 필요시 특정 패키지만 업데이트
npm update @mui/material @mui/icons-material

# 또는 모든 패키지 최신화 (주의: 호환성 문제 가능)
npm update
```

### 5. 개발 서버 테스트
```bash
# 개발 서버 10초 실행 테스트
timeout 10 npm run dev

# 서버 로그 확인:
# - "Local: http://localhost:xxxx/" 메시지 확인되면 성공
# - 포트 충돌 시 vite.config.js에서 다른 포트 설정

# 개발 서버 종료 처리(주의 ! : 절대 claude code는 종료하면 안됨 - 외부 네트워크에 연결된 것은 종료 x)
# - 개발 서버 확인: netstat -ano | findstr LISTENING | findstr 517
# - 개발 서버 종료: cmd //c "taskkill /PID [개발서버PID] /F"
# - 종료 확인: tasklist | findstr node.exe
```

### 6. 프로젝트 구조 확인
```bash
# 디렉토리 구조 확인
tree src/ -I node_modules

# 예상 구조:
# src/
# ├── components/
# │   ├── common/
# │   ├── ui/
# │   └── landing/
# ├── pages/
# ├── hooks/
# ├── utils/
# ├── theme.js
# ├── main.jsx
# ├── App.jsx
# └── index.css
```

## 완료 후 사용자에게 제공할 정보

1. **생성된 프로젝트 구조**
2. **설치된 패키지 목록**
3. **개발 서버 접속 URL**
4. **사용 가능한 기능들**:
   - MUI 테마 프로바이더 적용 완료
   - React Router 설치 완료
   - 기본 디렉토리 구조 생성 완료
   - CssBaseline 적용 완료

## ⚠️ 중요: 프로젝트 세팅 이후 개발 작업 규칙

**프로젝트 세팅이 완료된 후 추가적인 개발 작업을 진행할 때는 다음 규칙을 준수해야 함:**

1. **AI는 자동으로 `npm run dev`를 실행하지 않음**
   - 프로젝트 세팅 시에만 `timeout 10 npm run dev`로 서버 테스트 진행
   - 세팅 완료 후에는 사용자가 직접 개발 서버를 실행해야 함

2. **개발 서버 실행은 사용자 책임**
   - 컴포넌트 생성, 수정, 추가 기능 개발 시 AI는 코드 작성만 담당
   - 개발 서버 실행 및 테스트는 사용자가 직접 수행

3. **코드 작성 완료 후 안내**
   - AI는 코드 작성 완료 후 "개발 서버를 실행하여 확인해보세요" 형태로 안내
   - 자동으로 서버를 실행하거나 프로세스를 관리하지 않음

이 규칙을 통해 AI가 불필요한 프로세스를 실행하는 것을 방지하고, 사용자가 개발 환경을 직접 제어할 수 있도록 함.

## 주의사항 및 문제 해결

### Windows 환경에서 발생할 수 있는 문제들

1. **node_modules 복사 권한 문제**:
   - node_modules는 삭제하지 않고 유지하는 것이 원칙
   - 복사 과정에서 권한 문제가 발생할 수 있음
   - PowerShell Copy-Item 사용 시 대부분 해결됨

2. **복사 명령어 차이**:
   - Windows: PowerShell Copy-Item 사용
   - macOS/Linux: cp -r 사용

3. **중첩된 디렉토리 문제**:
   - 복사 과정에서 template-setup 디렉토리가 중첩되는 경우
   - 해당 디렉토리를 삭제하여 정리

4. **node_modules 손상 시 대처**:
   - 복사 과정에서 node_modules가 손상된 경우에만 재설치
   - `npm install` 실행하여 재설치

### 오류 발생 시 대응 방법

1. **권한 오류**: 관리자 권한으로 터미널 실행
2. **경로 오류**: 백슬래시(\) 사용 확인 (Windows)
3. **명령어 오류**: OS에 맞는 명령어 사용 확인

---

## 7. GitHub 백업 및 배포 (gh CLI 전용)

새 프로젝트 생성 직후, 사용자가 자연어로 백업/배포를 요청하면 다음 흐름을 따른다.
**전제: setup 단계에서 gh CLI 설치 + OAuth 인증 + `.claude/skills/gh_cli/skill.md` 스킬 등록이 이미 완료되어 있다.**

### 7-1. GitHub 백업 ("백업해줘" 요청 시)

```bash
# 1) 새 저장소 생성 (gh CLI)
gh repo create {프로젝트명} --public --description "{설명}"

# 2) 로컬 git 초기화 + 첫 푸시
git init
git add .
git commit -m "initial setup"
git branch -M main
git remote add origin https://github.com/{사용자명}/{프로젝트명}.git
git push -u origin main
```

**이후 변경사항 백업**:
```bash
git add .
git commit -m "{메시지}"
git push
```

### 7-2. GitHub Pages 자동 배포 ("배포해줘" 요청 시)

**GitHub Actions 워크플로우 방식만 사용** (Legacy 빌드 방식 금지 — "building" 상태 멈춤 / 에러 빈발).

```bash
# 1) .github/workflows/deploy.yml 생성 (아래 템플릿)
# 2) 커밋 + 푸시
# 3) Pages 설정을 workflow 방식으로 변경
gh api repos/{사용자명}/{프로젝트명}/pages -X PUT -f build_type=workflow
```

**워크플로우 파일** (`.github/workflows/deploy.yml`):
```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Setup Pages
        uses: actions/configure-pages@v4
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: '.'
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

**배포 확인**:
- GitHub 저장소 → Actions 탭에서 워크플로우 실행 확인
- 초록색 체크 표시되면 배포 완료
- `https://{사용자명}.github.io/{프로젝트명}` 접속 안내

### 7-3. 금지 사항

- netlify, vercel, surge 등 **외부 정적 호스팅 서비스 사용 금지**
- Personal Access Token 직접 발급 금지 (`gh auth login` OAuth 방식만)
- GitHub Pages **Legacy 빌드 방식 금지** (Actions 워크플로우만)

---

## 8. 데이터베이스 — Supabase MCP (Claude 플러그인 + OAuth)

DB / 백엔드가 필요한 프로젝트는 **Supabase** 를 사용한다.
**전제: lesson2 단계에서 Supabase 계정 생성 + Supabase MCP Claude 플러그인 등록 + OAuth 인증이 이미 완료되어 있다.**

### 8-1. Supabase MCP 설치 방식 (Claude 플러그인 + OAuth)

```bash
# 등록 (Claude 종료 상태에서 터미널 실행)
claude mcp add --transport http supabase https://mcp.supabase.com/mcp
```

**인증** (Claude 안에서):
1. `/mcp` 입력 → MCP 서버 목록 표시
2. `supabase` 선택 → `auth` 선택
3. 브라우저 자동 열림 → Supabase 로그인 (GitHub 계정) → **Authorize** 클릭
4. `/mcp` 에서 `supabase ✔ connected` 확인

토큰이나 프로젝트 ID 를 직접 입력하지 않는다. OAuth 가 자동 처리한다.

### 8-2. DB 작업 표준 도구

DB 작업은 반드시 `mcp__supabase__*` 도구를 통해서만 수행한다:

| 도구 | 용도 |
|---|---|
| `mcp__supabase__list_projects` | 사용 가능한 Supabase 프로젝트 목록 |
| `mcp__supabase__get_project_url` | 프로젝트 URL 확인 |
| `mcp__supabase__list_tables` | 테이블 목록 조회 |
| `mcp__supabase__execute_sql` | 테이블 생성·조회·수정·삭제 |

### 8-3. 환경변수 / 클라이언트 연결

프론트엔드(React)에서 Supabase 사용:
```bash
npm install @supabase/supabase-js
```

`.env` 에 다음 키 저장 (절대 코드에 하드코드 금지, `.gitignore` 에 `.env` 포함 필수):
```
VITE_SUPABASE_URL=https://{프로젝트}.supabase.co
VITE_SUPABASE_ANON_KEY={anon-public-key}
```

`src/lib/supabase.js`:
```javascript
import { createClient } from '@supabase/supabase-js'

export const supabase = createClient(
  import.meta.env.VITE_SUPABASE_URL,
  import.meta.env.VITE_SUPABASE_ANON_KEY
)
```

### 8-4. 금지 사항

- firebase, mongodb, mysql, postgres(직접) 등 **다른 DB / 백엔드 서비스 사용 금지**
- Supabase URL / API 키를 **코드 / 문서 / 깃에 직접 노출 금지** (`.env` + `.gitignore`)
- Supabase service_role key 사용 금지 (anon 키만 사용 — RLS 정책으로 보안)
