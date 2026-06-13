---
name: us-stock-afterhours-brief
description: 평일 KST 05:30 미국 주식 장마감 브리핑 (4:30 PM EDT) 생성 → kkangdago.com/us 배포 + 카카오톡 알림
---

매일 KST 05:30 (미국 EDT 16:30, 장마감 직후), 미국 주식 장마감 브리핑을 생성하고 kkangdago.com/us에 배포합니다.

## GitHub 인증
- Token: GH_TOKEN (환경변수 또는 직접 설정)
- Owner: worascal
- 배포 레포: kkangdago (us/ 폴더)
- 소스 레포: kkangdago-us

> **주의**: 실제 실행 시 GH_TOKEN을 유효한 GitHub Personal Access Token으로 교체하세요.

## STEP 1: 날짜/시간 확인

NaverSearch-get_current_korean_time 으로 현재 한국 시간 확인.
- KST 기준 TODAY (YYYY-MM-DD) 계산
- 미국 EDT 기준 TODAY_US = KST 날짜 - 1일 (KST 05:30 = 전일 EDT 16:30)
- TOMORROW_US = TODAY_US + 1일

예시: KST 2025-06-15 05:30 → TODAY_US = 2025-06-14 (해당 날짜 장마감)

## STEP 2: 뉴스 수집 (병렬)

NaverSearch-search_news로 다음 5개 쿼리 동시 실행 (display:10, sort:date):

1. "나스닥 S&P500 마감 결과 오늘"
2. "미국 주요 종목 장마감 실적"
3. "애프터마켓 시간외 급등 급락"
4. "연준 발표 경제지표 시장 반응"
5. "뉴욕증시 마감 내일 전망"

수집 후 최근 24시간 이내 기사만 선별. 각 쿼리 결과에서 중복 제거.

## STEP 3: 감성분석 및 AI 분석

수집된 뉴스를 바탕으로:
- **오늘 장 결과 요약**: 주요 지수 마감 결과 + 특이사항 (3~5문장)
- **주요 이슈 3~5개**: 오늘 장을 움직인 핵심 이슈 + 한줄 설명
- **주목 종목 6개**:
  - 종목명 + 티커 심볼
  - 오늘 움직임 근거 (뉴스 기반, 1~2문장)
  - 감성 점수 (긍정/중립/부정)
- **내일 전망**: 주요 주목 포인트 1~2개
- **주요 리스크 요인** 1~2개

뉴스 근거 없는 종목 추천 금지. 추측성 분석 금지.

## STEP 4: HTML 생성 (한국어 + 영어 이중언어)

아래 전체 구조로 HTML을 생성한다. [TODAY_US], [SUMMARY_KO], [SUMMARY_EN],
[INDEX_CARDS], [ISSUE_CARDS], [STOCK_CARDS], [RISK_CARDS], [NEWS_ITEMS], [OUTLOOK]
를 실제 내용으로 채운다.

### HTML 템플릿 구조

```
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="google-adsense-account" content="ca-pub-2869718539906266">
  <title>kkangdago US [TODAY_US] 장마감 브리핑</title>
  <meta name="description" content="[TODAY_US] 미국 증시 장마감 AI 브리핑. 오늘 장 결과와 내일 전망.">
  <script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-2869718539906266" crossorigin="anonymous"></script>
  <style>
    :root {
      --bg:#0d0d0d; --surface:#161616; --surface2:#1e1e1e;
      --border:#2a2a2a; --text:#e8e8e8; --text-muted:#888;
      --accent:#00d4aa; --signal:#f0c040;
      --up:#26a69a; --down:#ef5350; --font:'Segoe UI',-apple-system,sans-serif;
    }
    * { box-sizing:border-box; margin:0; padding:0; }
    body { background:var(--bg); color:var(--text); font-family:var(--font); line-height:1.6; }
    header { background:var(--surface); border-bottom:1px solid var(--border); padding:0 1.5rem; position:sticky; top:0; z-index:100; }
    .header-inner { max-width:960px; margin:0 auto; display:flex; align-items:center; justify-content:space-between; height:56px; }
    .logo { font-size:1.4rem; font-weight:800; text-decoration:none; color:var(--text); }
    .logo span { color:var(--accent); }
    .logo .signal { color:var(--signal); font-size:1rem; vertical-align:super; margin-left:2px; }
    .lang-btn { background:var(--surface2); border:1px solid var(--border); color:var(--text-muted); padding:6px 14px; border-radius:20px; cursor:pointer; font-size:0.85rem; }
    .container { max-width:960px; margin:0 auto; padding:2rem 1.5rem; }
    .badge { background:var(--surface2); border:1px solid var(--border); color:var(--accent); font-size:0.78rem; padding:3px 10px; border-radius:20px; letter-spacing:.5px; text-transform:uppercase; }
    .badge.afterhours { color:#b39ddb; border-color:#b39ddb33; }
    .brief-title { font-size:1.6rem; font-weight:800; margin:0.8rem 0 0.3rem; }
    .section { margin-bottom:2.5rem; }
    .section-title { font-size:.9rem; font-weight:700; color:var(--text-muted); text-transform:uppercase; letter-spacing:1px; margin-bottom:1rem; padding-bottom:.5rem; border-bottom:1px solid var(--border); }
    .summary-card { background:var(--surface); border:1px solid var(--border); border-radius:12px; padding:1.5rem; line-height:1.8; }
    .index-grid { display:grid; grid-template-columns:repeat(auto-fit,minmax(150px,1fr)); gap:1rem; margin-bottom:1.5rem; }
    .index-card { background:var(--surface); border:1px solid var(--border); border-radius:12px; padding:1.2rem; }
    .index-name { font-size:.8rem; color:var(--text-muted); margin-bottom:.3rem; }
    .index-val { font-size:1.4rem; font-weight:700; }
    .index-chg { font-size:.85rem; margin-top:.2rem; }
    .up{color:var(--up);} .down{color:var(--down);}
    .issue-grid,.stock-grid { display:grid; grid-template-columns:repeat(auto-fit,minmax(280px,1fr)); gap:1rem; }
    .issue-card { background:var(--surface); border:1px solid var(--border); border-left:3px solid var(--accent2,#ff6b35); border-radius:12px; padding:1.2rem 1.5rem; }
    .issue-name { font-weight:700; margin-bottom:.4rem; }
    .issue-desc { color:var(--text-muted); font-size:.9rem; }
    .stock-card { background:var(--surface); border:1px solid var(--border); border-radius:12px; padding:1.2rem 1.5rem; }
    .stock-header { display:flex; justify-content:space-between; align-items:flex-start; margin-bottom:.6rem; }
    .stock-name { font-weight:700; }
    .stock-ticker { background:var(--surface2); color:var(--accent); font-size:.78rem; padding:2px 8px; border-radius:6px; font-weight:600; }
    .stock-reason { color:var(--text-muted); font-size:.88rem; line-height:1.5; }
    .sentiment { display:inline-block; font-size:.75rem; padding:2px 8px; border-radius:10px; margin-top:.5rem; }
    .sentiment.pos { background:#26a69a22; color:var(--up); }
    .sentiment.neu { background:#f0c04022; color:var(--signal); }
    .sentiment.neg { background:#ef535022; color:var(--down); }
    .news-list { display:flex; flex-direction:column; gap:.8rem; }
    .news-item { background:var(--surface); border:1px solid var(--border); border-radius:10px; padding:1rem 1.2rem; }
    .news-title a { color:var(--text); text-decoration:none; font-size:.95rem; font-weight:600; }
    .news-title a:hover { color:var(--accent); }
    .news-meta { color:var(--text-muted); font-size:.8rem; margin-top:.3rem; }
    .risk-card { background:var(--surface); border:1px solid var(--border); border-left:3px solid var(--down); border-radius:12px; padding:1.2rem 1.5rem; margin-bottom:.8rem; }
    .outlook-card { background:var(--surface); border:1px solid var(--border); border-left:3px solid var(--up); border-radius:12px; padding:1.5rem; line-height:1.8; }
    .nav-bar { background:var(--surface); border:1px solid var(--border); border-radius:12px; padding:1rem 1.5rem; display:flex; gap:.5rem; flex-wrap:wrap; align-items:center; margin-bottom:2rem; }
    .nav-date { background:var(--surface2); border:1px solid var(--border); color:var(--text-muted); padding:4px 12px; border-radius:8px; font-size:.85rem; text-decoration:none; transition:all .2s; }
    .nav-date:hover,.nav-date.active { background:var(--accent); color:#000; border-color:var(--accent); }
    .disclaimer { background:var(--surface2); border:1px solid var(--border); border-radius:10px; padding:1rem 1.5rem; font-size:.82rem; color:var(--text-muted); margin-bottom:2rem; line-height:1.8; }
    footer { border-top:1px solid var(--border); padding:2rem 1.5rem; text-align:center; color:var(--text-muted); font-size:.82rem; }
  </style>
</head>
<body>
<header>
  <div class="header-inner">
    <a href="/us" class="logo">kkang<span>dago</span> <span class="signal">US</span></a>
    <button class="lang-btn" id="langBtn" onclick="toggleLang()">EN</button>
  </div>
</header>
<div class="container">
  <div class="nav-bar" id="historyNav">
    <span data-ko="히스토리:" data-en="History:">히스토리:</span>
  </div>
  <div style="margin-bottom:2rem;">
    <div style="display:flex;gap:.8rem;flex-wrap:wrap;align-items:center;margin-bottom:.8rem;">
      <span class="badge afterhours" data-ko="장마감 브리핑" data-en="After-Hours">장마감 브리핑</span>
      <span class="badge">AI Generated</span>
      <span style="color:var(--text-muted);font-size:.9rem;">[TODAY_US] · EDT 16:30</span>
    </div>
    <h1 class="brief-title" data-ko="[TODAY_US] 미국 증시 장마감 브리핑" data-en="[TODAY_US] US Market After-Hours Briefing">[TODAY_US] 미국 증시 장마감 브리핑</h1>
  </div>

  <div class="section">
    <div class="section-title" data-ko="오늘 마감 결과" data-en="Today's Close">오늘 마감 결과</div>
    <div class="index-grid">[INDEX_CARDS]</div>
    <div class="summary-card">
      <p data-ko="[SUMMARY_KO]" data-en="[SUMMARY_EN]">[SUMMARY_KO]</p>
    </div>
  </div>

  <div class="section">
    <div class="section-title" data-ko="주요 이슈" data-en="Key Issues">주요 이슈</div>
    <div class="issue-grid">[ISSUE_CARDS]</div>
  </div>

  <div class="section">
    <div class="section-title" data-ko="주목 종목" data-en="Notable Stocks">주목 종목</div>
    <div class="stock-grid">[STOCK_CARDS]</div>
  </div>

  <div class="section">
    <div class="section-title" data-ko="내일 전망" data-en="Tomorrow's Outlook">내일 전망</div>
    <div class="outlook-card">
      <p data-ko="[OUTLOOK_KO]" data-en="[OUTLOOK_EN]">[OUTLOOK_KO]</p>
    </div>
  </div>

  <div class="section">
    <div class="section-title" data-ko="주요 리스크" data-en="Key Risks">주요 리스크</div>
    [RISK_CARDS]
  </div>

  <div class="section">
    <div class="section-title" data-ko="주요 뉴스" data-en="Key News">주요 뉴스</div>
    <div class="news-list">[NEWS_ITEMS]</div>
  </div>

  <div class="disclaimer">
    <span data-ko="⚠️ 면책 조항: 본 브리핑은 AI가 뉴스를 기반으로 자동 생성한 정보로, 투자 권유가 아닙니다. 투자 결정은 본인의 판단과 책임 하에 이루어져야 하며, 본 사이트는 투자 결과에 대한 책임을 지지 않습니다."
          data-en="⚠️ Disclaimer: This briefing is automatically generated by AI based on news and does not constitute investment advice. All investment decisions are made at your own risk and discretion. This site is not responsible for any investment outcomes.">
      ⚠️ 면책 조항: 본 브리핑은 AI가 뉴스를 기반으로 자동 생성한 정보로, 투자 권유가 아닙니다.
    </span>
  </div>
</div>
<footer><p>© 2025 kkangdago.com/us — AI-powered US Market Briefing</p></footer>
<script>
  let currentLang = localStorage.getItem('kkangdago-lang') || 'ko';
  function applyLang(lang) {
    currentLang = lang;
    localStorage.setItem('kkangdago-lang', lang);
    document.documentElement.lang = lang;
    document.getElementById('langBtn').textContent = lang === 'ko' ? 'EN' : '한';
    document.querySelectorAll('[data-ko]').forEach(el => {
      const t = el.getAttribute('data-' + lang);
      if (t) el.innerHTML = t;
    });
  }
  function toggleLang() { applyLang(currentLang === 'ko' ? 'en' : 'ko'); }
  (async () => {
    try {
      const r = await fetch('/archive/us-index.json?_=' + Date.now());
      const dates = await r.json();
      if (!dates.length) return;
      const nav = document.getElementById('historyNav');
      nav.innerHTML = '<span data-ko="히스토리:" data-en="History:">히스토리:</span>';
      dates.slice(-10).reverse().forEach(d => {
        const a = document.createElement('a');
        a.href = '/archive/us/' + d + '.html';
        a.className = 'nav-date' + (d === '[TODAY_US]' ? ' active' : '');
        a.textContent = d;
        nav.appendChild(a);
      });
    } catch(e) {}
  })();
  applyLang(currentLang);
</script>
</body>
</html>
```

### HTML 생성 규칙

**[INDEX_CARDS]** — 6개 지수 카드 (S&P500, NASDAQ, DOW, VIX, USD/KRW, WTI):
```html
<div class="index-card">
  <div class="index-name">S&P 500</div>
  <div class="index-val">[종가]</div>
  <div class="index-chg [up|down]">[▲/▼ 등락률%]</div>
</div>
```

**[ISSUE_CARDS]** — 이슈 카드:
```html
<div class="issue-card">
  <div class="issue-name" data-ko="[이슈KO]" data-en="[이슈EN]">[이슈KO]</div>
  <div class="issue-desc" data-ko="[설명KO]" data-en="[설명EN]">[설명KO]</div>
</div>
```

**[STOCK_CARDS]** — 종목 카드 (장전 브리핑과 동일한 구조):
```html
<div class="stock-card">
  <div class="stock-header">
    <div class="stock-name">[회사명]</div>
    <div class="stock-ticker">[TICKER]</div>
  </div>
  <div class="stock-reason" data-ko="[오늘 움직임KO]" data-en="[오늘 움직임EN]">[오늘 움직임KO]</div>
  <span class="sentiment [pos|neu|neg]" data-ko="[감성KO]" data-en="[감성EN]">[감성KO]</span>
</div>
```

**[RISK_CARDS]** — 리스크 카드:
```html
<div class="risk-card">
  <div style="font-weight:700;margin-bottom:.4rem;" data-ko="[리스크명KO]" data-en="[리스크명EN]">[리스크명KO]</div>
  <div style="color:var(--text-muted);font-size:.9rem;" data-ko="[설명KO]" data-en="[설명EN]">[설명KO]</div>
</div>
```

**[NEWS_ITEMS]** — 뉴스 아이템:
```html
<div class="news-item">
  <div class="news-title"><a href="[URL]" target="_blank" rel="noopener">[제목]</a></div>
  <div class="news-meta">[출처] · [시간]</div>
</div>
```

**[OUTLOOK_KO] / [OUTLOOK_EN]** — 내일 전망 텍스트 (2~4문장)

## STEP 5: GitHub 배포 (Chrome MCP javascript_tool)

**반드시 async IIFE + btoa(unescape(encodeURIComponent(...))) 인코딩 사용.**

장전 브리핑 STEP 5와 동일한 방식으로 4단계 배포:
1. archive/us-index.json SHA + 내용 읽기
2. us/index.html SHA 읽기 + 업데이트
3. archive/us/[TODAY_US].html 저장
4. archive/us-index.json 날짜 추가

커밋 메시지는 'feat: [TODAY_US] US afterhours briefing' 사용.

## STEP 6: 카카오톡 알림

KakaotalkChat-MemoChat 으로 전송 (200자 이내):

```
📊 kkangdago US 마감
[TODAY_US] 장마감 브리핑 업로드

🔥 이슈: [이슈1], [이슈2]
⚡ 주목: [종목1]([TICKER1]), [종목2]([TICKER2]), [종목3]([TICKER3])
📅 내일 포인트: [핵심전망1줄]

👉 https://www.kkangdago.com/us
```

## 핵심 규칙

1. **뉴스 근거 없는 종목 언급 금지** — 수집된 뉴스에 등장한 종목만 다룰 것
2. **GitHub API는 Chrome MCP javascript_tool만 사용** — fetch는 IIFE async로 감쌀 것
3. **인코딩 필수**: btoa(unescape(encodeURIComponent(content)))
4. **SHA 충돌 방지**: PUT 전 반드시 GET으로 기존 SHA 획득
5. HTML 내 특수문자 이스케이프 주의
6. TODAY_US 날짜 계산 주의 — KST 05:30 기준이므로 **전일** EDT 날짜 사용
