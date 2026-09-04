---
description: 
globs: 
alwaysApply: true
---
- ** 중요 ** 모든 기본 컴퍼넌트는 MUI의 가장 최신버전 사용해줘.
- ** 중요 ** 특히 Grid 컴퍼넌트의 경우 아래 코드에 명시된 import 구문과 props를 참고해줘.
- 모든 컴퍼넌트의 스일을 가능한 MUI의 sx 함수를 사용해줘.
- 수정시 의존성을 줄만한 기능들을 독립된 컴퍼넌트로 모듈화를 해줘.
- 새로운 수정, 추가사항이 있을때 지시하지 않은 기존 기능, 형태에 영향을 주지 않도록 조심해줘.
- 특별한 의도가 없다면 구글 메터리얼 디자인의 가이드에 기반한 UX에 충실해줘.

## 반응형 디자인 규칙

### 1. Grid 시스템 사용법
- Grid 컴포넌트는 반드시 size={{ xs: 12, md: 6 }} 형태로 사용
- 주요 breakpoint: xs(0px), sm(600px), md(900px), lg(1200px), xl(1536px)
- 모바일 우선으로 xs부터 설정하고 필요에 따라 md, lg 추가

### 2. Typography 반응형 설정
- fontSize: { xs: '1rem', md: '1.2rem' } 형태로 화면 크기별 폰트 크기 설정
- 제목의 경우 { xs: '2rem', md: '3rem' } 등 더 큰 차이 적용
- lineHeight는 1.2~1.6 사이 값으로 가독성 확보

### 3. 간격 및 패딩 반응형
- py: { xs: 4, md: 8 } 형태로 화면 크기별 간격 조정
- Container maxWidth: 'sm'(600px), 'md'(900px), 'lg'(1200px), 'xl'(1536px)
- 모바일에서는 px: 2, 데스크톱에서는 px: 3 이상 권장

### 4. useMediaQuery 활용
- const isMobile = useMediaQuery(theme.breakpoints.down('md'))
- 복잡한 레이아웃 변경이 필요한 경우에만 사용
- 단순한 크기 조정은 sx 속성의 반응형 객체 사용 권장

## 🚨 페이지 레이아웃 및 중앙 정렬 규칙 [필수 준수]
**⚠️ 중요: 모든 페이지에서 반드시 적용해야 하는 기본 규칙**

### 5.1 CSS 초기화 (index.css에 반드시 포함)
```css
/* CSS 초기화 */
* {
  box-sizing: border-box;
}

/* HTML, Body 초기화 */
html, body {
  margin: 0;
  padding: 0;
  width: 100%;
  min-height: 100vh;
}

/* Root 앱 컨테이너 설정 */
#root {
  width: 100%;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}
```

### 5.2 전체 페이지 중앙정렬 패턴 (App.jsx에 반드시 적용)
```jsx
return (
  <Box sx={{ 
    width: '100%', 
    minHeight: '100vh', 
    display: 'flex', 
    justifyContent: 'center',
    alignItems: 'center',
    py: { xs: 2, md: 4 }
  }}>
    <Container maxWidth="md" sx={{ py: 4 }}>
      {/* 페이지 내용 */}
    </Container>
  </Box>
);
```

### 5.3 반응형 설정 필수 적용
- **Container 최대 너비**: `maxWidth="sm"(600px)`, `"md"(900px)`, `"lg"(1200px)`, `"xl"(1536px)`
- **반응형 패딩**: `py: { xs: 2, md: 4 }` (모바일 2, 데스크톱 4)
- **반응형 간격**: `px: { xs: 2, md: 3 }` (모바일에서는 2, 데스크톱에서는 3 이상)

### 5.4 중앙정렬 필수 지침
**모든 페이지 개발 시 반드시 다음 순서대로 적용해야 함:**

1. **CSS 초기화 적용**: `* { box-sizing: border-box }` 를 index.css에 추가
2. **HTML/Body 초기화**: `html, body { margin: 0, padding: 0 }` 설정
3. **전체 페이지 설정**: `width: 100%, minHeight: 100vh` 로 화면 전체 사용
4. **Flexbox 중앙정렬**: `justifyContent: 'center', alignItems: 'center'` 로 콘텐츠 중앙배치
5. **Container 최대 너비**: `maxWidth` 속성으로 콘텐츠 너비 제한
6. **반응형 패딩**: `py: { xs: 2, md: 4 }` 로 화면 크기별 여백 조정



import * as React from 'react';
import { styled } from '@mui/material/styles';
import Box from '@mui/material/Box';
import Paper from '@mui/material/Paper';
import Grid from '@mui/material/Grid';

const Item = styled(Paper)(({ theme }) => ({
  backgroundColor: '#fff',
  ...theme.typography.body2,
  padding: theme.spacing(1),
  textAlign: 'center',
  color: (theme.vars ?? theme).palette.text.secondary,
  ...theme.applyStyles('dark', {
    backgroundColor: '#1A2027',
  }),
}));

export default function FullWidthGrid() {
  return (
    <Box sx={{ flexGrow: 1 }}>
      <Grid container spacing={2}>
        <Grid size={{ xs: 6, md: 8 }}>
          <Item>xs=6 md=8</Item>
        </Grid>
        <Grid size={{ xs: 6, md: 4 }}>
          <Item>xs=6 md=4</Item>
        </Grid>
        <Grid size={{ xs: 6, md: 4 }}>
          <Item>xs=6 md=4</Item>
        </Grid>
        <Grid size={{ xs: 6, md: 8 }}>
          <Item>xs=6 md=8</Item>
        </Grid>
      </Grid>
    </Box>
  );
}