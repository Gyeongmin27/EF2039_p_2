# AI 패션 스타일 평가 시스템

AI 기반 패션 스타일 종합 평가 웹 애플리케이션입니다. 사용자가 상하의를 입고 있는 사진을 업로드하면, AI가 패션 스타일을 분석하여 총점과 세부 점수를 제공하고 개선 방안을 제시합니다.

## 주요 기능

- 📸 실제 사람 사진 업로드 (드래그 앤 드롭 지원)
- 🕐 TPO(Time, Place, Occasion) 선택 기능
- 🤖 AI 기반 패션 스타일 분석 (Gemini 2.5 Flash)
- 📊 7개 항목별 세부 점수 제공
- 🎯 총점 및 등급 (S, A, B, C, D, F) 표시
- 💡 구체적인 개선점 제시
- 📈 시각적 차트 (레이더 차트, 진행 바)
- 👔 패션 비평가 평가 (AI 생성)
- 🛍️ 구매 추천 아이템 (AI 생성, 무신사 검색 연동)
- 💰 예상 가격 정보 제공

## 기술 스택

- **프론트엔드**: Next.js 14, React, TypeScript, Tailwind CSS
- **백엔드**: Next.js API Routes, Vercel Serverless Functions
- **AI/ML**: Google Gemini API (gemini-2.5-flash)
- **데이터베이스**: Supabase (PostgreSQL)
- **스토리지**: Supabase Storage
- **차트**: Recharts
- **이미지 처리**: react-dropzone, sharp
- **배포**: Vercel

## 시작하기

### 1. 의존성 설치

```bash
npm install
```

### 2. 환경 변수 설정

`.env.local` 파일을 생성하고 다음 변수들을 설정하세요:

```env
NEXT_PUBLIC_SUPABASE_URL=your_supabase_project_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
SUPABASE_SERVICE_ROLE_KEY=your_supabase_service_role_key
GEMINI_API_KEY=your_gemini_api_key
```

### 3. Supabase 설정

1. [Supabase](https://supabase.com)에서 새 프로젝트 생성
2. SQL Editor에서 다음 스키마 실행:

```sql
-- fashion_evaluations 테이블
CREATE TABLE fashion_evaluations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  image_id UUID NOT NULL,
  image_url TEXT NOT NULL,
  total_score DECIMAL(5,2) NOT NULL CHECK (total_score >= 0 AND total_score <= 100),
  grade VARCHAR(1) NOT NULL CHECK (grade IN ('S', 'A', 'B', 'C', 'D', 'F')),

  -- 세부 점수
  color_harmony_score DECIMAL(5,2) NOT NULL,
  style_consistency_score DECIMAL(5,2) NOT NULL,
  pattern_combination_score DECIMAL(5,2) NOT NULL,
  proportion_silhouette_score DECIMAL(5,2) NOT NULL,
  texture_harmony_score DECIMAL(5,2) NOT NULL,
  context_appropriateness_score DECIMAL(5,2) NOT NULL,
  overall_harmony_score DECIMAL(5,2) NOT NULL,

  -- 분석 결과 메타데이터
  analysis_metadata JSONB,

  -- 사용자 정보
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  session_id VARCHAR(255),

  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- improvement_suggestions 테이블
CREATE TABLE improvement_suggestions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  evaluation_id UUID NOT NULL REFERENCES fashion_evaluations(id) ON DELETE CASCADE,
  category VARCHAR(50) NOT NULL,
  score DECIMAL(5,2) NOT NULL,
  suggestion_text TEXT NOT NULL,
  priority INTEGER NOT NULL CHECK (priority >= 1 AND priority <= 5),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Storage Bucket 생성 (Supabase Dashboard에서)
-- 이름: fashion-images, Public: true
```

### 4. 개발 서버 실행

```bash
npm run dev
```

브라우저에서 [http://localhost:3000](http://localhost:3000)을 열어 확인하세요.

## Vercel 배포

### 1. GitHub 저장소 준비

프로젝트가 이미 GitHub에 푸시되어 있어야 합니다:

- 저장소: <https://github.com/Gyeongmin27/EF2039_p_2.git>

### 2. Vercel 프로젝트 생성

1. [Vercel](https://vercel.com)에 로그인
2. "Add New Project" 클릭
3. GitHub 저장소 선택: `Gyeongmin27/EF2039_p_2`
4. 프로젝트 설정:
   - **Framework Preset**: Next.js (자동 감지)
   - **Root Directory**: `./` (기본값)
   - **Build Command**: `npm run build` (자동)
   - **Output Directory**: `.next` (자동)

### 3. 환경 변수 설정

Vercel 대시보드에서 다음 환경 변수를 추가하세요:

```
NEXT_PUBLIC_SUPABASE_URL=your_supabase_project_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
SUPABASE_SERVICE_ROLE_KEY=your_supabase_service_role_key
GEMINI_API_KEY=your_gemini_api_key
```

**설정 방법:**

1. 프로젝트 설정 → Environment Variables
2. 각 변수를 추가 (Production, Preview, Development 모두 선택)
3. Save 클릭

### 4. 배포

1. "Deploy" 버튼 클릭
2. 배포가 완료되면 자동으로 URL이 생성됩니다
3. 배포된 사이트에서 테스트하세요

### 5. 자동 배포

GitHub에 푸시하면 자동으로 재배포됩니다:

- `main` 브랜치에 푸시 → Production 배포
- 다른 브랜치에 푸시 → Preview 배포

## 점수 체계

총점은 100점 만점이며, 다음 7개 항목의 합산으로 계산됩니다:

1. **색상 조화도** (18점)
2. **스타일 일관성** (18점)
3. **패턴 조합** (10점)
4. **비율 및 실루엣** (10점)
5. **텍스처 조화** (10점)
6. **상황적합성** (30점) - TPO(Time, Place, Occasion) 기반 평가
7. **전체적 조화** (4점)

등급 체계:

- **S**: 90-100점 (최고 수준)
- **A**: 80-89점 (우수)
- **B**: 70-79점 (양호)
- **C**: 60-69점 (보통)
- **D**: 50-59점 (개선 필요)
- **F**: 0-49점 (전면적 개선 필요)

## 프로젝트 구조

```
project_2/
├── app/
│   ├── api/
│   │   └── analyze-fashion/    # AI 분석 API
│   ├── globals.css
│   ├── layout.tsx
│   └── page.tsx                # 메인 페이지
├── components/
│   ├── ImageUpload.tsx         # 이미지 업로드 컴포넌트
│   ├── ScoreDisplay.tsx        # 점수 표시 컴포넌트
│   ├── ScoreChart.tsx          # 차트 컴포넌트
│   └── ImprovementList.tsx    # 개선점 리스트
├── lib/
│   ├── supabase/               # Supabase 클라이언트
│   └── ai/
│       ├── colorAnalysis.ts    # 색상 분석
│       ├── scoreCalculation.ts # 점수 계산
│       └── improvementSuggestions.ts # 개선점 생성
└── PRD.md                      # 프로젝트 요구사항 문서
```

## 현재 구현 상태

- ✅ Gemini API를 사용한 실제 이미지 분석
- ✅ 색상, 패턴, 스타일, 텍스처 자동 인식
- ✅ 점수 계산 및 개선점 생성
- ✅ Supabase 데이터베이스 스키마 준비

## 향후 개선 사항

- [ ] 인물/의류 자동 분리 기능 (MediaPipe 또는 Remove.bg)
- [ ] 더 정확한 색상 추출 (ColorThief.js 보조 사용)
- [ ] Supabase에 결과 저장 기능
- [ ] 사용자 히스토리 기능
- [ ] 결과 공유 기능
- [ ] 비율 및 실루엣 분석 개선 (이미지 분석 기반)

## 라이선스

MIT
