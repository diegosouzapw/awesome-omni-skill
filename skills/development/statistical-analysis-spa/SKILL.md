---
name: statistical-analysis-spa
description: 웹 기반 통계 분석 SPA 개발 스킬. 이상치 탐지(Outlier Detection)와 행별 통계 분석(Row Statistics)을 수행하는 React 애플리케이션 구현. Z-Score, IQR, MAD, Grubbs, Winsorize 이상치 탐지와 T-test, ANOVA 통계 분석 지원. Copy & Paste 또는 CSV/TXT 파일 드래그 앤 드롭으로 데이터 입력, Recharts를 활용한 시각화 기능 포함. 모든 데이터는 로컬에서만 처리되며 네트워크 전송 없음.
license: MIT
---

# Statistical Analysis SPA Skill

React + TypeScript로 이상치 탐지 및 통계 분석 SPA를 구현합니다.

## ⚠️ 보안 필수 요구사항

```
🔒 LOCAL-ONLY DATA PROCESSING
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
• 모든 데이터는 브라우저 내에서만 처리
• 네트워크 요청 절대 금지 (fetch, XHR 사용 불가)
• 외부 CDN/API 호출 금지
• localStorage/sessionStorage 사용 금지
• 오프라인 실행 가능해야 함
```

### 금지 패턴
```typescript
// ❌ FORBIDDEN
fetch(), axios, XMLHttpRequest
localStorage, sessionStorage, IndexedDB
외부 CDN 스크립트, Google Fonts
Analytics, 트래킹 코드
```

### 허용 패턴
```typescript
// ✅ ALLOWED
FileReader API (로컬 파일 읽기)
Clipboard API (붙여넣기)
Blob + download (결과 저장)
React state (임시 데이터)
```

## 프로젝트 초기화

```bash
# Vite + React + TypeScript 프로젝트 생성
npm create vite@latest statistical-analysis-spa -- --template react-ts
cd statistical-analysis-spa

# 핵심 의존성
npm install recharts jstat lodash papaparse
npm install -D @types/lodash @types/papaparse

# Tailwind CSS
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

## 핵심 구현 가이드

### 1. 데이터 파서

```typescript
// lib/utils/parser.ts
export function parseText(text: string, delimiter = '\t'): number[][] {
  return text
    .trim()
    .split('\n')
    .map(row => row.split(delimiter).map(Number));
}

export function detectDelimiter(text: string): string {
  const counts = { '\t': 0, ',': 0, ' ': 0 };
  for (const char of text) {
    if (char in counts) counts[char as keyof typeof counts]++;
  }
  return Object.entries(counts)
    .sort((a, b) => b[1] - a[1])[0][0];
}
```

### 2. 이상치 탐지 알고리즘

```typescript
// lib/outlier/zscore.ts
export function detectZScore(data: number[], threshold = 3.0) {
  const clean = data.filter(x => !isNaN(x));
  const mean = clean.reduce((a, b) => a + b, 0) / clean.length;
  const std = Math.sqrt(
    clean.reduce((sum, x) => sum + (x - mean) ** 2, 0) / clean.length
  );
  
  const mask = data.map(x => Math.abs((x - mean) / std) > threshold);
  return {
    mask,
    bounds: { lower: mean - threshold * std, upper: mean + threshold * std },
    stats: { mean, std, threshold }
  };
}

// lib/outlier/iqr.ts
export function detectIQR(data: number[], k = 1.5) {
  const sorted = [...data].filter(x => !isNaN(x)).sort((a, b) => a - b);
  const q1 = sorted[Math.floor(sorted.length * 0.25)];
  const q3 = sorted[Math.floor(sorted.length * 0.75)];
  const iqr = q3 - q1;
  const lower = q1 - k * iqr;
  const upper = q3 + k * iqr;
  
  return {
    mask: data.map(x => x < lower || x > upper),
    bounds: { lower, upper },
    stats: { q1, q3, iqr, k }
  };
}

// lib/outlier/mad.ts
export function detectMAD(data: number[], threshold = 3.5) {
  const clean = data.filter(x => !isNaN(x));
  const sorted = [...clean].sort((a, b) => a - b);
  const median = sorted[Math.floor(sorted.length / 2)];
  const mad = sorted
    .map(x => Math.abs(x - median))
    .sort((a, b) => a - b)[Math.floor(sorted.length / 2)];
  
  const mask = data.map(x => 
    Math.abs(0.6745 * (x - median) / mad) > threshold
  );
  
  return { mask, bounds: { lower: median - threshold * mad / 0.6745, 
                           upper: median + threshold * mad / 0.6745 },
           stats: { median, mad, threshold } };
}
```

### 3. 통계 분석

```typescript
// lib/statistics/ttest.ts
import jstat from 'jstat';

export function ttest(g1: number[], g2: number[], equalVar = true) {
  const n1 = g1.length, n2 = g2.length;
  const m1 = mean(g1), m2 = mean(g2);
  const v1 = variance(g1), v2 = variance(g2);
  
  if (equalVar) {
    const pooled = ((n1-1)*v1 + (n2-1)*v2) / (n1+n2-2);
    const se = Math.sqrt(pooled * (1/n1 + 1/n2));
    const t = (m1 - m2) / se;
    const df = n1 + n2 - 2;
    const p = 2 * (1 - jstat.studentt.cdf(Math.abs(t), df));
    return { t, df, p, type: 'T-test' };
  } else {
    const se = Math.sqrt(v1/n1 + v2/n2);
    const t = (m1 - m2) / se;
    const df = ((v1/n1 + v2/n2)**2) / 
      ((v1/n1)**2/(n1-1) + (v2/n2)**2/(n2-1));
    const p = 2 * (1 - jstat.studentt.cdf(Math.abs(t), df));
    return { t, df, p, type: 'Welch T-test' };
  }
}

// lib/statistics/levene.ts
export function levene(g1: number[], g2: number[]) {
  const m1 = mean(g1), m2 = mean(g2);
  const z1 = g1.map(x => Math.abs(x - m1));
  const z2 = g2.map(x => Math.abs(x - m2));
  return ttest(z1, z2, true);
}
```

### 4. 핵심 컴포넌트 구조

```typescript
// components/DataInput.tsx
function DataInput({ onDataLoad }) {
  const [text, setText] = useState('');
  
  const handleParse = () => {
    const delimiter = detectDelimiter(text);
    const data = parseText(text, delimiter);
    onDataLoad(data);
  };
  
  return (
    <div>
      <textarea value={text} onChange={e => setText(e.target.value)} />
      <DropZone onDrop={handleFileDrop} />
      <button onClick={handleParse}>Parse Data</button>
    </div>
  );
}

// components/OutlierPanel.tsx
function OutlierPanel({ data, onResult }) {
  const [method, setMethod] = useState('iqr');
  const [config, setConfig] = useState({ k: 1.5 });
  
  const handleDetect = () => {
    const result = detectOutliers(data, method, config);
    onResult(result);
  };
  
  return (
    <div>
      <Select value={method} onChange={setMethod}>
        <Option value="zscore">Z-Score</Option>
        <Option value="iqr">IQR</Option>
        <Option value="mad">MAD</Option>
      </Select>
      <ConfigPanel method={method} config={config} onChange={setConfig} />
      <button onClick={handleDetect}>Detect Outliers</button>
    </div>
  );
}
```

### 5. 시각화 (Recharts)

```typescript
// components/visualization/BoxPlot.tsx
import { ComposedChart, Bar, Scatter, XAxis, YAxis, Tooltip } from 'recharts';

function BoxPlot({ data, outliers }) {
  const stats = computeBoxStats(data);
  
  return (
    <ComposedChart data={stats}>
      <Bar dataKey="box" />
      <Scatter data={outliers} fill="red" />
      <XAxis /><YAxis />
      <Tooltip />
    </ComposedChart>
  );
}
```

## 파일 구조

```
src/
├── components/
│   ├── DataInput.tsx
│   ├── OutlierPanel.tsx
│   ├── StatsPanel.tsx
│   └── visualization/
│       ├── BoxPlot.tsx
│       ├── DistChart.tsx
│       └── ResultTable.tsx
├── lib/
│   ├── outlier/
│   │   ├── zscore.ts
│   │   ├── iqr.ts
│   │   └── mad.ts
│   ├── statistics/
│   │   ├── ttest.ts
│   │   ├── anova.ts
│   │   └── levene.ts
│   └── utils/
│       ├── parser.ts
│       └── math.ts
├── types/
│   └── index.ts
└── App.tsx
```

## 품질 체크리스트

### 기능
- [ ] TypeScript strict mode 활성화
- [ ] 모든 함수에 JSDoc 주석
- [ ] 에러 바운더리 구현
- [ ] 반응형 레이아웃
- [ ] 키보드 접근성
- [ ] 로딩 상태 표시
- [ ] 빈 데이터 처리

### ⚠️ 보안 (필수)
- [ ] fetch/axios/XHR 사용 없음
- [ ] 외부 CDN 스크립트 없음
- [ ] Google Fonts 등 외부 폰트 없음
- [ ] localStorage/sessionStorage 없음
- [ ] Analytics/트래킹 없음
- [ ] 오프라인 실행 가능

## 참고 자료

- `docs/PROJECT_SPECIFICATION.md` - 상세 명세서
- `CLAUDE.md` - Claude Code 가이드
