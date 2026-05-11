<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0">
<title>NEPSE Tracker</title>
<link href="https://fonts.googleapis.com/css2?family=IBM+Plex+Mono:wght@400;600;700&family=Syne:wght@400;600;700;800&display=swap" rel="stylesheet">
<style>
:root {
  --bg:      #080E17;
  --bg2:     #0F1923;
  --bg3:     #162130;
  --border:  rgba(255,255,255,0.07);
  --gold:    #E8B84B;
  --gold2:   #FDD87A;
  --green:   #2ECC71;
  --green2:  #27AE60;
  --red:     #E74C3C;
  --orange:  #F39C12;
  --blue:    #3498DB;
  --grey:    #5D7A8A;
  --light:   #A8BEC9;
  --white:   #E8F0F5;
  --mono:    'IBM Plex Mono', monospace;
  --head:    'Syne', sans-serif;
}
*{margin:0;padding:0;box-sizing:border-box;-webkit-tap-highlight-color:transparent}
html{scroll-behavior:smooth}
body{
  background:var(--bg);
  color:var(--white);
  font-family:var(--mono);
  min-height:100vh;
  overflow-x:hidden;
}

/* ── NOISE TEXTURE OVERLAY ── */
body::after{
  content:'';position:fixed;inset:0;
  background-image:url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)' opacity='0.03'/%3E%3C/svg%3E");
  pointer-events:none;z-index:0;opacity:0.4;
}

/* ── GRID ── */
body::before{
  content:'';position:fixed;inset:0;
  background:
    linear-gradient(rgba(232,184,75,0.03) 1px,transparent 1px),
    linear-gradient(90deg,rgba(232,184,75,0.03) 1px,transparent 1px);
  background-size:32px 32px;
  pointer-events:none;z-index:0;
}

.wrap{position:relative;z-index:1;max-width:480px;margin:0 auto;padding:12px}

/* ── HEADER ── */
.hdr{
  padding:16px;
  background:linear-gradient(135deg,var(--bg2),var(--bg3));
  border:1px solid rgba(232,184,75,0.2);
  border-radius:16px;
  margin-bottom:10px;
  position:relative;overflow:hidden;
}
.hdr::before{
  content:'';position:absolute;top:-30px;right:-30px;
  width:120px;height:120px;
  background:radial-gradient(circle,rgba(232,184,75,0.12),transparent 70%);
  border-radius:50%;
}
.hdr-top{display:flex;align-items:center;justify-content:space-between;margin-bottom:10px}
.hdr-title{font-family:var(--head);font-size:1.1rem;font-weight:800;color:var(--gold);letter-spacing:1px}
.hdr-sub{font-size:0.65rem;color:var(--grey);margin-top:2px}
.hdr-btns{display:flex;gap:8px;align-items:center}

.pill{
  display:inline-flex;align-items:center;gap:5px;
  padding:5px 11px;border-radius:20px;font-size:0.68rem;
  border:1px solid;cursor:pointer;transition:all 0.2s;
  font-family:var(--mono);font-weight:600;
}
.pill-gold{border-color:rgba(232,184,75,0.5);color:var(--gold);background:rgba(232,184,75,0.08)}
.pill-gold:hover{background:rgba(232,184,75,0.2)}
.pill-green{border-color:rgba(46,204,113,0.5);color:var(--green);background:rgba(46,204,113,0.08)}
.pill-red{border-color:rgba(231,76,60,0.5);color:var(--red);background:rgba(231,76,60,0.08)}
.pill-grey{border-color:var(--border);color:var(--grey);background:rgba(255,255,255,0.03)}

.dot{width:6px;height:6px;border-radius:50%;background:currentColor}
.dot.pulse{animation:pulse 1.4s ease-in-out infinite}
@keyframes pulse{0%,100%{opacity:1;transform:scale(1)}50%{opacity:0.3;transform:scale(0.6)}}

/* ── MARKET TIME BAR ── */
.mbar{
  display:flex;align-items:center;justify-content:space-between;
  padding:8px 14px;background:var(--bg2);
  border:1px solid var(--border);border-radius:10px;
  margin-bottom:10px;font-size:0.68rem;flex-wrap:wrap;gap:6px;
}
.mopen{color:var(--green)} .mclosed{color:var(--red)}

/* ── API SOURCE ROW ── */
.src-row{
  display:flex;gap:6px;align-items:center;
  padding:8px 14px;background:var(--bg2);
  border:1px solid var(--border);border-radius:10px;
  margin-bottom:10px;overflow-x:auto;
}
.src-label{font-size:0.65rem;color:var(--grey);white-space:nowrap;margin-right:2px}
.src-btn{
  flex-shrink:0;padding:4px 10px;border-radius:6px;
  border:1px solid var(--border);background:transparent;
  color:var(--grey);cursor:pointer;font-family:var(--mono);
  font-size:0.65rem;transition:all 0.2s;white-space:nowrap;
}
.src-btn.on{border-color:var(--gold);color:var(--gold);background:rgba(232,184,75,0.1)}

/* ── STATUS BANNER ── */
.banner{
  padding:9px 13px;border-radius:10px;margin-bottom:10px;
  font-size:0.72rem;border:1px solid;display:flex;gap:9px;
  align-items:flex-start;line-height:1.5;
}
.bn-ok  {background:rgba(46,204,113,0.07);border-color:rgba(46,204,113,0.3);color:var(--green)}
.bn-err {background:rgba(231,76,60,0.07);border-color:rgba(231,76,60,0.3);color:var(--red)}
.bn-warn{background:rgba(243,156,18,0.07);border-color:rgba(243,156,18,0.3);color:var(--orange)}
.bn-inf {background:rgba(52,152,219,0.07);border-color:rgba(52,152,219,0.3);color:var(--blue)}

/* ── TICKER ── */
.ticker-wrap{overflow:hidden;margin-bottom:10px;border-radius:8px;background:var(--bg2);border:1px solid var(--border);padding:7px 0}
.ticker-track{display:flex;white-space:nowrap;animation:tick 45s linear infinite}
.ticker-track:hover{animation-play-state:paused}
@keyframes tick{0%{transform:translateX(0)}100%{transform:translateX(-50%)}}
.tick-item{display:inline-flex;align-items:center;gap:6px;padding:0 16px;font-size:0.7rem;border-right:1px solid var(--border)}
.tick-sym{color:var(--light);font-weight:600}

/* ── SUMMARY GRID ── */
.sgrid{display:grid;grid-template-columns:1fr 1fr;gap:8px;margin-bottom:10px}
.scard{
  background:var(--bg2);border:1px solid var(--border);
  border-radius:13px;padding:13px 14px;
  transition:border-color 0.2s;position:relative;overflow:hidden;
}
.scard:hover{border-color:rgba(232,184,75,0.2)}
.scard::after{
  content:'';position:absolute;bottom:0;right:0;
  width:40px;height:40px;
  background:radial-gradient(circle,rgba(232,184,75,0.06),transparent);
}
.sc-label{font-size:0.6rem;color:var(--grey);text-transform:uppercase;letter-spacing:1.2px;margin-bottom:6px}
.sc-val{font-size:1rem;font-weight:700;line-height:1.2}
.sc-sub{font-size:0.62rem;color:var(--grey);margin-top:3px}
.cg{color:var(--gold)} .cc{color:var(--green)} .cr{color:var(--red)} .co{color:var(--orange)} .cw{color:var(--white)}

/* ── TABS ── */
.tabs{
  display:grid;grid-template-columns:repeat(4,1fr);gap:3px;
  background:var(--bg2);padding:3px;border-radius:12px;
  border:1px solid var(--border);margin-bottom:10px;
}
.tab{
  padding:9px 4px;text-align:center;cursor:pointer;
  border-radius:9px;font-size:0.68rem;font-weight:600;
  transition:all 0.2s;color:var(--grey);
}
.tab.on{background:var(--bg3);color:var(--gold);border:1px solid rgba(232,184,75,0.25)}
.tab:hover:not(.on){color:var(--white)}

.panel{display:none}
.panel.on{display:block}

/* ── PORTFOLIO TABLE ── */
.tbl-wrap{
  background:var(--bg2);border:1px solid var(--border);
  border-radius:13px;overflow:hidden;margin-bottom:10px;
}
.tbl-hdr{
  display:flex;align-items:center;justify-content:space-between;
  padding:11px 14px;border-bottom:1px solid var(--border);flex-wrap:wrap;gap:6px;
}
.tbl-title{font-family:var(--head);font-weight:700;font-size:0.85rem;color:var(--gold)}
.badge{font-size:0.6rem;padding:2px 9px;border-radius:20px;background:rgba(232,184,75,0.1);color:var(--gold);border:1px solid rgba(232,184,75,0.25)}

/* Mobile-optimised stock cards instead of table */
.stock-list{padding:8px}
.sec-hdr{
  padding:6px 10px;margin:6px 0 4px;
  font-size:0.6rem;font-weight:700;letter-spacing:1.8px;
  color:var(--gold);text-transform:uppercase;
  background:rgba(232,184,75,0.06);border-radius:6px;
}
.srow{
  display:grid;
  grid-template-columns:1fr auto;
  gap:6px;align-items:center;
  padding:10px 10px;
  border-bottom:1px solid var(--border);
  transition:background 0.15s;border-radius:8px;
  margin-bottom:2px;
}
.srow:hover{background:rgba(232,184,75,0.03)}
.srow.r-stop{background:rgba(231,76,60,0.07)}
.srow.r-near{background:rgba(243,156,18,0.05)}
.srow.r-tgt {background:rgba(46,204,113,0.05)}

.sr-left{}
.sr-sym{font-weight:700;font-size:0.85rem;color:var(--white);display:flex;align-items:center;gap:6px}
.sr-name{font-size:0.62rem;color:var(--grey);margin-top:1px}
.sr-meta{display:flex;gap:8px;margin-top:5px;font-size:0.65rem;flex-wrap:wrap}
.sr-meta span{color:var(--grey)}
.sr-meta .cost{color:var(--light)}

.sr-right{text-align:right}
.sr-price{font-size:0.95rem;font-weight:700}
.sr-chg{font-size:0.65rem;margin-top:2px}
.sr-pnl{font-size:0.72rem;font-weight:600;margin-top:3px}
.sr-status{margin-top:4px}

.sbadge{
  display:inline-block;padding:2px 8px;border-radius:20px;
  font-size:0.58rem;font-weight:700;
}
.sb-hold  {background:rgba(46,204,113,0.12);color:var(--green);border:1px solid rgba(46,204,113,0.25)}
.sb-watch {background:rgba(243,156,18,0.12);color:var(--orange);border:1px solid rgba(243,156,18,0.25)}
.sb-free  {background:rgba(232,184,75,0.12);color:var(--gold);border:1px solid rgba(232,184,75,0.25)}
.sb-review{background:rgba(231,76,60,0.1);color:var(--red);border:1px solid rgba(231,76,60,0.2)}
.sb-alert {background:rgba(231,76,60,0.25);color:#fff;border:1px solid var(--red);animation:aPulse 1s infinite}
.sb-target{background:rgba(46,204,113,0.25);color:#fff;border:1px solid var(--green);animation:aPulse 1s infinite}
@keyframes aPulse{0%,100%{opacity:1}50%{opacity:0.55}}

.live-tag{font-size:0.55rem;padding:1px 5px;border-radius:4px;background:rgba(46,204,113,0.15);color:var(--green);border:1px solid rgba(46,204,113,0.3)}
.last-tag{font-size:0.55rem;padding:1px 5px;border-radius:4px;background:rgba(93,122,138,0.15);color:var(--grey);border:1px solid rgba(93,122,138,0.3)}

/* ── ALERTS ── */
.alerts{display:flex;flex-direction:column;gap:8px}
.acard{display:flex;gap:10px;padding:12px 14px;border-radius:12px;border:1px solid;align-items:flex-start}
.ac-danger {background:rgba(231,76,60,0.07);border-color:rgba(231,76,60,0.35)}
.ac-warn   {background:rgba(243,156,18,0.07);border-color:rgba(243,156,18,0.35)}
.ac-ok     {background:rgba(46,204,113,0.07);border-color:rgba(46,204,113,0.35)}
.ac-info   {background:rgba(52,152,219,0.05);border-color:rgba(232,184,75,0.2)}
.ac-ico{font-size:1.2rem;flex-shrink:0}
.ac-body{flex:1}
.ac-title{font-weight:700;font-size:0.78rem;margin-bottom:3px}
.ac-desc{font-size:0.68rem;color:var(--grey);line-height:1.5}
.ac-price{font-size:0.82rem;font-weight:700;text-align:right;flex-shrink:0}

/* ── MARKET GRID ── */
.mgrid{display:grid;grid-template-columns:1fr 1fr;gap:7px;padding:10px}
.mcard{
  background:var(--bg3);border:1px solid var(--border);
  border-radius:10px;padding:11px 12px;transition:all 0.2s;
}
.mcard.owned{border-color:rgba(232,184,75,0.2)}
.mcard:hover{transform:translateY(-1px);border-color:rgba(232,184,75,0.25)}
.mc-sym{font-weight:700;font-size:0.78rem;display:flex;align-items:center;justify-content:space-between}
.mc-price{font-size:1rem;font-weight:700;margin:5px 0}
.mc-chg{font-size:0.68rem}
.mc-sec{font-size:0.6rem;color:var(--grey);margin-top:2px}
.mc-bar{height:2px;border-radius:1px;margin-top:7px;background:rgba(255,255,255,0.07);overflow:hidden}
.mc-fill{height:100%;border-radius:1px;transition:width 0.8s}

/* ── SETTINGS ── */
.sgroup{background:var(--bg2);border:1px solid var(--border);border-radius:13px;margin-bottom:10px;overflow:hidden}
.sg-title{padding:11px 14px;border-bottom:1px solid var(--border);font-family:var(--head);font-size:0.78rem;font-weight:700;color:var(--gold)}
.sg-row{display:flex;align-items:center;justify-content:space-between;padding:10px 14px;border-bottom:1px solid var(--border)}
.sg-row:last-child{border-bottom:none}
.sg-label{font-size:0.75rem;color:var(--light)}
.sg-sub{font-size:0.6rem;color:var(--grey);margin-top:1px}
input[type=number]{
  background:rgba(13,20,30,0.9);border:1px solid rgba(232,184,75,0.3);
  color:var(--gold);border-radius:7px;padding:5px 9px;
  font-family:var(--mono);font-size:0.78rem;width:88px;text-align:right;
  -webkit-appearance:none;
}
input[type=number]:focus{outline:none;border-color:var(--gold)}

.save-btn{
  width:100%;padding:13px;border-radius:12px;
  background:linear-gradient(135deg,rgba(232,184,75,0.2),rgba(232,184,75,0.1));
  border:1px solid rgba(232,184,75,0.4);color:var(--gold);
  font-family:var(--mono);font-size:0.85rem;font-weight:700;
  cursor:pointer;transition:all 0.2s;margin-top:4px;letter-spacing:1px;
}
.save-btn:hover{background:rgba(232,184,75,0.25)}
.save-btn:active{transform:scale(0.98)}

/* ── HOW IT WORKS (CORS explanation) ── */
.info-box{
  background:rgba(52,152,219,0.06);border:1px solid rgba(52,152,219,0.2);
  border-radius:12px;padding:13px;margin-bottom:10px;font-size:0.7rem;
  color:var(--light);line-height:1.7;
}
.info-box h4{color:var(--blue);margin-bottom:7px;font-family:var(--head);font-size:0.82rem}
.info-box code{
  display:block;background:rgba(0,0,0,0.3);border-radius:6px;
  padding:8px 10px;margin:7px 0;font-size:0.65rem;color:var(--gold);
  word-break:break-all;
}

/* ── TOAST ── */
#toasts{position:fixed;bottom:20px;left:50%;transform:translateX(-50%);z-index:9999;display:flex;flex-direction:column;gap:7px;align-items:center;width:90%;max-width:380px;pointer-events:none}
.toast{
  background:var(--bg3);border:1px solid;border-radius:12px;
  padding:11px 15px;width:100%;pointer-events:auto;
  box-shadow:0 8px 32px rgba(0,0,0,0.6);
  animation:toastIn 0.3s ease;
}
@keyframes toastIn{from{transform:translateY(20px);opacity:0}to{transform:translateY(0);opacity:1}}
.toast.ok  {border-color:rgba(46,204,113,0.5)}
.toast.err {border-color:rgba(231,76,60,0.5)}
.toast.inf {border-color:rgba(232,184,75,0.4)}
.t-t{font-weight:700;font-size:0.78rem;margin-bottom:2px}
.t-m{font-size:0.68rem;color:var(--grey)}
.toast.ok .t-t{color:var(--green)} .toast.err .t-t{color:var(--red)} .toast.inf .t-t{color:var(--gold)}

/* ── BOTTOM SAFE AREA ── */
.footer{text-align:center;padding:14px 10px 24px;color:var(--grey);font-size:0.6rem;line-height:1.8;border-top:1px solid var(--border);margin-top:14px}
</style>
</head>
<body>
<div id="toasts"></div>
<div class="wrap">

<!-- HEADER -->
<div class="hdr">
  <div class="hdr-top">
    <div>
      <div class="hdr-title">🇳🇵 NEPSE TRACKER</div>
      <div class="hdr-sub">Personal Portfolio · May 2026</div>
    </div>
    <div class="hdr-btns">
      <span class="pill pill-grey" id="api-pill"><span class="dot pulse" id="api-dot"></span><span id="api-txt">Loading</span></span>
      <span class="pill pill-gold" onclick="doRefresh()" id="ref-btn">⟳ Refresh</span>
    </div>
  </div>
  <div style="display:flex;gap:6px;align-items:center;flex-wrap:wrap">
    <span class="pill pill-grey" id="bell-pill">🔔 <span id="alert-ct">0</span> Alerts</span>
    <span class="pill pill-grey" id="upd-pill" style="font-size:0.6rem">Not updated</span>
  </div>
</div>

<!-- MARKET BAR -->
<div class="mbar">
  <span id="mkt-lbl" class="mclosed">● CLOSED</span>
  <span id="nst-time" style="color:var(--grey)">Loading NST...</span>
  <span id="last-upd" style="color:var(--gold);font-size:0.65rem">—</span>
</div>

<!-- SOURCE ROW -->
<div class="src-row">
  <span class="src-label">DATA:</span>
  <button class="src-btn" id="s0" onclick="setSrc(0)">allorigins</button>
  <button class="src-btn" id="s1" onclick="setSrc(1)">corsproxy</button>
  <button class="src-btn" id="s2" onclick="setSrc(2)">proxy.cors</button>
  <button class="src-btn on" id="s3" onclick="setSrc(3)">Manual ✏️</button>
</div>

<!-- BANNER -->
<div class="banner bn-inf" id="banner">
  <span>ℹ️</span>
  <span id="banner-txt">Tap a proxy source above then REFRESH to try live prices. Or use Manual mode — enter prices from merolagani.com in Settings.</span>
</div>

<!-- TICKER -->
<div class="ticker-wrap"><div class="ticker-track" id="ticker"></div></div>

<!-- SUMMARY -->
<div class="sgrid">
  <div class="scard"><div class="sc-label">Total Invested</div><div class="sc-val cw" id="c-inv">NPR 0</div><div class="sc-sub">Core positions</div></div>
  <div class="scard"><div class="sc-label">Market Value</div><div class="sc-val cg" id="c-val">—</div><div class="sc-sub" id="c-val-sub">0 live prices</div></div>
  <div class="scard"><div class="sc-label">Unrealised P&L</div><div class="sc-val" id="c-pnl">—</div><div class="sc-sub" id="c-pnlp">—</div></div>
  <div class="scard"><div class="sc-label">Locked Profit ✅</div><div class="sc-val cc">NPR 1,17,770</div><div class="sc-sub">RSML+NRN sold</div></div>
  <div class="scard"><div class="sc-label">Cash Available</div><div class="sc-val cg">NPR 1,12,720</div><div class="sc-sub">Ready to deploy</div></div>
  <div class="scard"><div class="sc-label">Cash Reserve</div><div class="sc-val cr">NPR 32,720</div><div class="sc-sub">NEVER touch</div></div>
</div>

<!-- TABS -->
<div class="tabs">
  <div class="tab on" onclick="tab('portfolio',this)">📊 Holdings</div>
  <div class="tab" onclick="tab('alerts',this)">🔔 Alerts</div>
  <div class="tab" onclick="tab('market',this)">📈 Market</div>
  <div class="tab" onclick="tab('settings',this)">⚙️ Settings</div>
</div>

<!-- PORTFOLIO -->
<div class="panel on" id="panel-portfolio">
  <div class="tbl-wrap">
    <div class="tbl-hdr">
      <span class="tbl-title">YOUR HOLDINGS</span>
      <span class="badge" id="ptable-badge">Loading...</span>
    </div>
    <div class="stock-list" id="stock-list"></div>
  </div>
</div>

<!-- ALERTS -->
<div class="panel" id="panel-alerts">
  <div class="alerts" id="alerts-list">
    <div class="acard ac-info"><div class="ac-ico">⏳</div><div class="ac-body"><div class="ac-title">Loading alerts...</div><div class="ac-desc">Refresh to check current prices.</div></div></div>
  </div>
</div>

<!-- MARKET -->
<div class="panel" id="panel-market">
  <div class="tbl-wrap">
    <div class="tbl-hdr"><span class="tbl-title">MARKET WATCHLIST</span><span class="badge">⭐ = Yours</span></div>
    <div class="mgrid" id="mgrid"></div>
  </div>
</div>

<!-- SETTINGS -->
<div class="panel" id="panel-settings">

  <div class="info-box">
    <h4>📡 How Live Prices Work</h4>
    Your browser blocks direct API calls from local files (CORS policy). These proxy servers act as a middleman that forwards the request and bypasses this restriction.
    <code>Your App → Free Proxy → NEPSE API → Back to App</code>
    If one proxy fails, try another. If all fail, use Manual mode — it works 100% of the time.
    <br><br>
    <strong style="color:var(--gold)">Always verify prices on merolagani.com before any trade.</strong>
  </div>

  <div class="sgroup">
    <div class="sg-title">✏️ MANUAL PRICE ENTRY</div>
    <div style="padding:8px 14px 4px;font-size:0.65rem;color:var(--grey)">Enter LTP from merolagani.com. These override all API data.</div>
    <div id="manual-inputs"></div>
  </div>

  <button class="save-btn" onclick="saveManual()">💾 SAVE PRICES</button>

  <div class="sgroup" style="margin-top:10px">
    <div class="sg-title">📋 YOUR RULES</div>
    <div class="sg-row"><div><div class="sg-label" style="color:var(--red)">Stop Loss</div><div class="sg-sub">Sell immediately when hit</div></div><div style="color:var(--red);font-size:0.75rem">10-13% below entry</div></div>
    <div class="sg-row"><div><div class="sg-label" style="color:var(--green)">Take Profit</div><div class="sg-sub">Sell 25% at each level</div></div><div style="color:var(--green);font-size:0.75rem">+10%, +20%, +30%</div></div>
    <div class="sg-row"><div><div class="sg-label">Cash Reserve</div><div class="sg-sub">Sacred — never invest</div></div><div style="color:var(--red);font-size:0.75rem">NPR 32,720</div></div>
    <div class="sg-row"><div><div class="sg-label">Price Check</div><div class="sg-sub">Discipline</div></div><div style="color:var(--grey);font-size:0.75rem">Once/day max</div></div>
    <div class="sg-row"><div><div class="sg-label">Borrowed Money</div><div class="sg-sub">Non-negotiable</div></div><div style="color:var(--red);font-size:0.75rem;font-weight:700">NEVER</div></div>
    <div class="sg-row"><div><div class="sg-label">Monthly Review</div><div class="sg-sub">Track & learn</div></div><div style="color:var(--grey);font-size:0.75rem">Last Sunday</div></div>
  </div>

</div>

<div class="footer">
  🇳🇵 NEPSE Tracker · For personal/educational use only<br>
  Always verify prices on merolagani.com before trading<br>
  Not financial advice · Your decisions, your responsibility
</div>
</div>

<script>
// ════════════════════════════════════════════════════════════
// PORTFOLIO DATA
// ════════════════════════════════════════════════════════════
const PORT = [
  {sec:"CORE HOLDINGS"},
  {sym:"NABIL",  name:"Nabil Bank",          sector:"Commercial Bank",     qty:null,cost:528, stop:468, t1:608, t2:686, st:"hold", note:"Confirm qty"},
  {sym:"KBL",    name:"Kumari Bank",          sector:"Commercial Bank",     qty:70,  cost:214, stop:185, t1:250, t2:285, st:"hold"},
  {sym:"NMB",    name:"NMB Bank",             sector:"Commercial Bank",     qty:20,  cost:240, stop:210, t1:276, t2:312, st:"hold"},
  {sym:"NIFRA",  name:"Nepal Infrastructure", sector:"Infrastructure Bank", qty:54,  cost:261, stop:228, t1:300, t2:340, st:"hold"},
  {sec:"SECTOR HOLDINGS"},
  {sym:"UMRH",   name:"Upper Marsyangdi",     sector:"Hydropower",          qty:11,  cost:532, stop:465, t1:612, t2:692, st:"hold"},
  {sym:"RHPL",   name:"Rasuwagadhi Hydro",    sector:"Hydropower",          qty:20,  cost:275, stop:240, t1:316, t2:358, st:"hold"},
  {sym:"SJLIC",  name:"SuryaJyoti Life",      sector:"Life Insurance",      qty:10,  cost:429, stop:375, t1:493, t2:558, st:"hold"},
  {sym:"SRLI",   name:"Reliable Life",        sector:"Life Insurance",      qty:11,  cost:390, stop:340, t1:448, t2:507, st:"hold"},
  {sym:"SJCL",   name:"Sanjen Jalavidhyut",   sector:"Hydropower",          qty:10,  cost:298, stop:260, t1:343, t2:387, st:"hold"},
  {sym:"SHEL",   name:"Sanima Hydropower",    sector:"Hydropower",          qty:10,  cost:307, stop:268, t1:353, t2:399, st:"hold"},
  {sym:"NICLBSL",name:"NIC Asia Laghubitta",  sector:"Microfinance",        qty:10,  cost:576, stop:490, t1:662, t2:748, st:"watch"},
  {sec:"FREE RIDE"},
  {sym:"RSML",   name:"Reliance Spinning",    sector:"Manufacturing",       qty:15,  cost:820, stop:3000,t1:null,t2:null,st:"free"},
  {sym:"NRN",    name:"NRN Infrastructure",   sector:"Infrastructure",      qty:5,   cost:300, stop:1100,t1:null,t2:null,st:"free"},
  {sec:"NEEDS REVIEW"},
  {sym:"CSY",    name:"CSY",                  sector:"Unknown",             qty:100, cost:null,stop:null,t1:null,t2:null,st:"review"},
  {sym:"LUK",    name:"LUK",                  sector:"Unknown",             qty:100, cost:null,stop:null,t1:null,t2:null,st:"review"},
  {sym:"LEC",    name:"LEC",                  sector:"Unknown",             qty:10,  cost:null,stop:null,t1:null,t2:null,st:"review"},
];

const WATCH = ["NABIL","KBL","NMB","NIFRA","EBL","NICA","SBI","HIDCL","UMRH","RHPL","CHCL","SJLIC","ADBL","NTDC","RSML","NRN"];

const FALLBACK = {
  NABIL:528,KBL:215,NMB:240,NIFRA:261,UMRH:532,RHPL:275,
  SJLIC:430,SRLI:390,SJCL:299,SHEL:307,NICLBSL:577,
  RSML:3850,NRN:1443,CSY:9.32,LUK:9.94,LEC:222,
  EBL:714,NICA:398,SBI:427,HIDCL:301,CHCL:342,ADBL:293,NTDC:880
};

// ════════════════════════════════════════════════════════════
// STATE
// ════════════════════════════════════════════════════════════
let liveData   = {};   // {SYM: {price, change, live:true}}
let manualP    = JSON.parse(localStorage.getItem('np_manual')||'{}');
let srcIdx     = 3;    // Default manual
let fetching   = false;
let shownAlerts= new Set();

// ════════════════════════════════════════════════════════════
// CORS PROXY SOURCES
// ════════════════════════════════════════════════════════════
// Each proxy wraps the ShareBazaar API differently
const SOURCES = [
  {
    name:"allorigins",
    // allorigins.win returns {contents: "json string"}
    url: sym =>
      `https://api.allorigins.win/get?url=${encodeURIComponent('https://nepsetty.kokomo.workers.dev/api?symbol='+sym)}`,
    parse: async resp => {
      const j = await resp.json();
      const inner = JSON.parse(j.contents);
      return {
        price:  parseFloat(inner.lastTradedPrice||inner.ltp||inner.price||0),
        change: parseFloat(inner.percentageChange||inner.change||0),
      };
    }
  },
  {
    name:"corsproxy.io",
    url: sym =>
      `https://corsproxy.io/?${encodeURIComponent('https://nepsetty.kokomo.workers.dev/api?symbol='+sym)}`,
    parse: async resp => {
      const j = await resp.json();
      return {
        price:  parseFloat(j.lastTradedPrice||j.ltp||j.price||0),
        change: parseFloat(j.percentageChange||j.change||0),
      };
    }
  },
  {
    name:"proxy.cors.sh",
    url: sym =>
      `https://proxy.cors.sh/https://nepsetty.kokomo.workers.dev/api?symbol=${sym}`,
    parse: async resp => {
      const j = await resp.json();
      return {
        price:  parseFloat(j.lastTradedPrice||j.ltp||j.price||0),
        change: parseFloat(j.percentageChange||j.change||0),
      };
    }
  },
  { name:"Manual", url:null, parse:null }
];

function setSrc(i) {
  srcIdx = i;
  [0,1,2,3].forEach(n => {
    document.getElementById('s'+n).className = 'src-btn' + (n===i?' on':'');
  });
  if (i===3) {
    setBanner('bn-warn','✏️ Manual mode active. Enter prices in Settings tab and tap Save.');
    setApiPill('grey','Manual');
  } else {
    setBanner('bn-inf',`📡 Using ${SOURCES[i].name} proxy. Tap Refresh to load live prices.`);
    setApiPill('grey',SOURCES[i].name);
  }
}

// ════════════════════════════════════════════════════════════
// FETCH WITH PROXY
// ════════════════════════════════════════════════════════════
async function fetchOne(sym) {
  const src = SOURCES[srcIdx];
  if (!src.url) return null;
  try {
    const resp = await fetch(src.url(sym), {
      signal: AbortSignal.timeout(9000),
      headers: srcIdx===2 ? {'x-cors-api-key':'temp_demo'} : {}
    });
    if (!resp.ok) return null;
    const data = await src.parse(resp);
    if (data.price > 0) return {...data, live:true};
    return null;
  } catch(e) {
    return null;
  }
}

async function doRefresh() {
  if (fetching) return;
  if (srcIdx === 3) {
    renderAll(); toast('Manual mode','Update prices in ⚙️ Settings','inf'); return;
  }
  fetching = true;
  const btn = document.getElementById('ref-btn');
  btn.textContent = '⟳ Loading...';
  setApiPill('orange','Fetching...');
  setBanner('bn-inf','📡 Fetching live prices... This takes ~20 seconds for all stocks.');

  const allSyms = [...new Set([...PORT.filter(r=>r.sym).map(r=>r.sym),...WATCH])];
  let ok=0, fail=0;

  // Sequential with small delay to avoid rate limits
  for (const sym of allSyms) {
    const res = await fetchOne(sym);
    if (res) { liveData[sym]=res; ok++; }
    else fail++;
    await delay(300); // 300ms between requests
  }

  fetching = false;
  btn.textContent = '⟳ Refresh';
  document.getElementById('last-upd').textContent =
    'Updated: ' + new Date().toLocaleTimeString('en-US',{hour:'2-digit',minute:'2-digit'});
  document.getElementById('upd-pill').textContent =
    new Date().toLocaleTimeString('en-US',{hour:'2-digit',minute:'2-digit'});

  if (ok > 0) {
    setApiPill('green', `Live · ${ok} stocks`);
    setBanner('bn-ok', `✅ ${ok} live prices loaded via ${SOURCES[srcIdx].name}. ${fail>0?fail+' used fallback.':''}`);
  } else {
    setApiPill('red','API Failed');
    setBanner('bn-err', `❌ ${SOURCES[srcIdx].name} didn't work. Try another proxy above, or use Manual mode (always works).`);
  }

  renderAll();
  checkAlertSounds();
}

const delay = ms => new Promise(r=>setTimeout(r,ms));

// ════════════════════════════════════════════════════════════
// PRICE GETTERS
// ════════════════════════════════════════════════════════════
function getP(sym)   { return manualP[sym] || liveData[sym]?.price  || FALLBACK[sym] || null; }
function getChg(sym) { return liveData[sym]?.change || 0; }
function isLive(sym) { return !!liveData[sym]; }

// ════════════════════════════════════════════════════════════
// FORMAT
// ════════════════════════════════════════════════════════════
const f  = n => n==null?"—":n>=1000?n.toLocaleString('en-IN',{maximumFractionDigits:1}):n.toFixed(2);
const fp = n => n==null?"—":(n>=0?"+":"")+n.toFixed(1)+"%";
const gc = n => n>0?"cc":n<0?"cr":"";
const ar = n => n>0?"▲":n<0?"▼":"";

// ════════════════════════════════════════════════════════════
// RENDER ALL
// ════════════════════════════════════════════════════════════
function renderAll() {
  buildPortfolio();
  buildAlerts();
  buildMarket();
  buildTicker();
}

// ════════════════════════════════════════════════════════════
// PORTFOLIO
// ════════════════════════════════════════════════════════════
function buildPortfolio() {
  const list = document.getElementById('stock-list');
  list.innerHTML = '';
  let totCost=0, totVal=0, liveCount=0, alertCount=0;

  PORT.forEach(row => {
    if (row.sec) {
      list.innerHTML += `<div class="sec-hdr">${row.sec}</div>`; return;
    }
    const p     = getP(row.sym);
    const chg   = getChg(row.sym);
    const live  = isLive(row.sym);
    const qty   = row.qty;
    const cost  = row.cost;
    const val   = qty&&p ? qty*p : null;
    const cTot  = qty&&cost ? qty*cost : null;
    const pnl   = val&&cTot ? val-cTot : null;
    const pnlP  = pnl&&cTot ? (pnl/cTot)*100 : null;

    if(cTot) totCost+=cTot;
    if(val)  totVal+=val;
    if(live) liveCount++;

    // Status
    let rowCls='srow', stKey=row.st;
    if(row.stop&&p&&p<=row.stop)           {rowCls='srow r-stop';stKey='alert';alertCount++;}
    else if(row.stop&&p&&p<=row.stop*1.05) {rowCls='srow r-near';alertCount++;}
    if(row.t1&&p&&p>=row.t1)               {rowCls='srow r-tgt';stKey='target';alertCount++;}

    const stMap  = {hold:'sb-hold',watch:'sb-watch',free:'sb-free',review:'sb-review',alert:'sb-alert',target:'sb-target'};
    const stLbl  = {hold:'HOLD ✅',watch:'WATCH ⚠️',free:'FREE RIDE 🎯',review:'REVIEW ❓',alert:'🔴 STOP HIT',target:'🎯 TARGET!'};
    const liveTag = live
      ? `<span class="live-tag">LIVE</span>`
      : (manualP[row.sym] ? `<span class="live-tag" style="color:var(--gold);border-color:rgba(232,184,75,0.3)">MANUAL</span>` : `<span class="last-tag">FALLBACK</span>`);

    list.innerHTML += `
    <div class="${rowCls}">
      <div class="sr-left">
        <div class="sr-sym">${row.sym} ${liveTag} ${row.note?`<span style="font-size:0.58rem;color:var(--orange)">${row.note}</span>`:""}
        </div>
        <div class="sr-name">${row.name} · ${row.sector}</div>
        <div class="sr-meta">
          <span>Qty: <strong style="color:var(--white)">${qty??'?'}</strong></span>
          ${cost?`<span class="cost">Cost: NPR ${f(cost)}</span>`:''}
          ${row.stop?`<span style="color:var(--red)">Stop: ${f(row.stop)}</span>`:''}
          ${row.t1?`<span style="color:var(--green)">T1: ${f(row.t1)}</span>`:''}
        </div>
      </div>
      <div class="sr-right">
        <div class="sr-price ${gc(chg)}">${p?"NPR "+f(p):"—"}</div>
        <div class="sr-chg ${gc(chg)}">${chg?(ar(chg)+" "+Math.abs(chg).toFixed(2)+"%"):"—"}</div>
        <div class="sr-pnl ${gc(pnl)}">${pnl!=null?(pnl>=0?"+":"")+f(pnl)+" ("+fp(pnlP)+")":"—"}</div>
        <div class="sr-status"><span class="sbadge ${stMap[stKey]}">${stLbl[stKey]}</span></div>
      </div>
    </div>`;
  });

  // Summary cards
  const pnl  = totVal-totCost;
  const pnlP = totCost?(pnl/totCost)*100:0;
  document.getElementById('c-inv').textContent = "NPR "+f(totCost);
  document.getElementById('c-val').textContent = totVal?"NPR "+f(totVal):"—";
  document.getElementById('c-val-sub').textContent = liveCount+" live prices";
  const pe = document.getElementById('c-pnl');
  pe.textContent = pnl?(pnl>=0?"+":"")+"NPR "+f(Math.abs(pnl)):"—";
  pe.className   = "sc-val "+(pnl>=0?"cc":"cr");
  document.getElementById('c-pnlp').textContent = totCost?fp(pnlP)+" unrealised":"—";
  document.getElementById('alert-ct').textContent = alertCount;
  const bp = document.getElementById('bell-pill');
  bp.className = alertCount>0?"pill pill-red":"pill pill-grey";
  document.getElementById('ptable-badge').textContent =
    `${liveCount} live · ${PORT.filter(r=>r.sym).length-liveCount} fallback`;
}

// ════════════════════════════════════════════════════════════
// ALERTS
// ════════════════════════════════════════════════════════════
function buildAlerts() {
  const list = document.getElementById('alerts-list');
  list.innerHTML = '';
  let found=false;

  PORT.filter(r=>r.sym).forEach(row => {
    const p=getP(row.sym); if(!p) return;
    const chg=getChg(row.sym);

    if(row.stop&&p<=row.stop) {
      found=true;
      list.innerHTML+=aC('danger','🚨',`${row.sym} — STOP LOSS HIT`,
        `Price NPR ${f(p)} has hit your stop loss NPR ${f(row.stop)}. SELL NOW. No waiting, no hoping.`,
        `NPR ${f(p)}`,'var(--red)');
    } else if(row.stop&&p<=row.stop*1.05) {
      found=true;
      const d=(((p-row.stop)/row.stop)*100).toFixed(1);
      list.innerHTML+=aC('warn','⚠️',`${row.sym} — ${d}% from Stop Loss`,
        `Price NPR ${f(p)} is dangerously close to stop NPR ${f(row.stop)}. Monitor closely.`,
        `NPR ${f(p)}`,'var(--orange)');
    }
    if(row.t1&&p>=row.t1) {
      found=true;
      list.innerHTML+=aC('ok','🎯',`${row.sym} — TARGET 1 REACHED!`,
        `Sell 25% of position now. Move stop loss to breakeven (NPR ${f(row.cost)}).`,
        `NPR ${f(p)}`,'var(--green)');
    }
    if(row.cost&&p<row.cost*0.97&&(!row.stop||p>row.stop)) {
      found=true;
      list.innerHTML+=aC('warn','📉',`${row.sym} — Below Your Cost`,
        `Price dipped ${((p-row.cost)/row.cost*100).toFixed(1)}% below entry. Check RSI & support before adding.`,
        `NPR ${f(p)}`,'var(--gold)');
    }
  });

  // Opportunity alerts
  const kb=getP('KBL'); if(kb&&kb<=207){found=true;list.innerHTML+=aC('ok','💰','KBL — Tranche 2 Buy Zone!',`NPR ${f(kb)} is in your buy zone (NPR 200-207). Consider buying ~75 more shares.`,`NPR ${f(kb)}`,'var(--green)');}
  const nb=getP('NABIL'); if(nb&&nb<=515){found=true;list.innerHTML+=aC('ok','💰','NABIL — Add Opportunity',`NPR ${f(nb)} in buy zone NPR 505-515. Consider adding ~78 shares.`,`NPR ${f(nb)}`,'var(--green)');}
  const nm=getP('NMB'); if(nm&&nm<=232){found=true;list.innerHTML+=aC('ok','💰','NMB — Add Opportunity',`NPR ${f(nm)} in buy zone NPR 225-232. Consider ~108 shares.`,`NPR ${f(nm)}`,'var(--green)');}

  if(!found) {
    list.innerHTML=aC('info','✅','All Clear — No Urgent Alerts',
      'All positions within normal ranges. No stop losses triggered. No targets hit yet. Keep monitoring.',
      '','');
    list.innerHTML+=aC('info','📋','How to get live prices',
      Object.keys(liveData).length>0
        ? `${Object.keys(liveData).length} live prices loaded. Refresh every few hours during market hours.`
        : 'Tap a proxy source (allorigins/corsproxy/proxy.cors) then tap Refresh. Or use Settings to enter manually.',
      '','');
  }
}

function aC(type,icon,title,desc,price,color) {
  const cls = {danger:'ac-danger',warn:'ac-warn',ok:'ac-ok',info:'ac-info'};
  return `<div class="acard ${cls[type]}">
    <div class="ac-ico">${icon}</div>
    <div class="ac-body"><div class="ac-title">${title}</div><div class="ac-desc">${desc}</div></div>
    ${price?`<div class="ac-price" style="color:${color}">${price}</div>`:''}
  </div>`;
}

function checkAlertSounds() {
  PORT.filter(r=>r.sym).forEach(row => {
    const p=getP(row.sym);
    if(row.stop&&p&&p<=row.stop&&!shownAlerts.has(row.sym+'_s')) {
      shownAlerts.add(row.sym+'_s');
      toast(`🚨 ${row.sym} STOP LOSS HIT`,`NPR ${f(p)} ≤ Stop NPR ${f(row.stop)} — SELL NOW`,'err');
      beep([440,330,220]);
    }
    if(row.t1&&p&&p>=row.t1&&!shownAlerts.has(row.sym+'_t')) {
      shownAlerts.add(row.sym+'_t');
      toast(`🎯 ${row.sym} TARGET REACHED`,`NPR ${f(p)} ≥ T1 NPR ${f(row.t1)} — Sell 25%`,'ok');
      beep([440,550,660]);
    }
  });
}

// ════════════════════════════════════════════════════════════
// MARKET GRID
// ════════════════════════════════════════════════════════════
function buildMarket() {
  const grid=document.getElementById('mgrid'); grid.innerHTML='';
  const portSyms=new Set(PORT.filter(r=>r.sym).map(r=>r.sym));
  WATCH.forEach(sym=>{
    const p=getP(sym),c=getChg(sym),owned=portSyms.has(sym);
    const cls=c>=0?'cc':'cr'; const fill=Math.min(100,Math.max(5,50+c*8));
    const lv=isLive(sym)?`<span class="live-tag">LIVE</span>`:
             manualP[sym]?`<span class="live-tag" style="color:var(--gold);border-color:rgba(232,184,75,0.3)">MAN</span>`:
             `<span class="last-tag">—</span>`;
    grid.innerHTML+=`<div class="mcard${owned?' owned':''}">
      <div class="mc-sym">${sym}${owned?' ⭐':''} ${lv}</div>
      <div class="mc-price ${cls}">${p?"NPR "+f(p):"—"}</div>
      <div class="mc-chg ${cls}">${c?(ar(c)+" "+Math.abs(c).toFixed(2)+"%"):"—"}</div>
      <div class="mc-sec">${owned?"In your portfolio":"Watchlist"}</div>
      <div class="mc-bar"><div class="mc-fill" style="width:${fill}%;background:${c>=0?"var(--green)":"var(--red)"}"></div></div>
    </div>`;
  });
}

// ════════════════════════════════════════════════════════════
// TICKER
// ════════════════════════════════════════════════════════════
function buildTicker() {
  const syms=WATCH.slice(0,12);
  const items=syms.map(sym=>{
    const p=getP(sym),c=getChg(sym),cls=c>=0?'cc':'cr';
    return `<span class="tick-item"><span class="tick-sym">${sym}</span><span style="color:var(--white)">NPR ${f(p)}</span><span class="${cls}">${c?(ar(c)+" "+Math.abs(c).toFixed(2)+"%"):"—"}</span></span>`;
  }).join('');
  document.getElementById('ticker').innerHTML=items+items;
}

// ════════════════════════════════════════════════════════════
// MANUAL SETTINGS
// ════════════════════════════════════════════════════════════
function buildManualInputs() {
  const cont=document.getElementById('manual-inputs');
  cont.innerHTML='';
  PORT.filter(r=>r.sym).forEach(r=>{
    const v=manualP[r.sym]||'';
    cont.innerHTML+=`<div class="sg-row">
      <div><div class="sg-label">${r.sym}</div><div class="sg-sub">${r.name}</div></div>
      <input type="number" id="mi-${r.sym}" value="${v}" placeholder="${FALLBACK[r.sym]||''}" step="0.1" min="0">
    </div>`;
  });
}

function saveManual() {
  PORT.filter(r=>r.sym).forEach(r=>{
    const el=document.getElementById('mi-'+r.sym);
    if(el&&el.value) manualP[r.sym]=parseFloat(el.value);
    else if(el&&!el.value) delete manualP[r.sym];
  });
  localStorage.setItem('np_manual',JSON.stringify(manualP));
  renderAll();
  toast('✅ Prices Saved','Manual prices applied to all views','ok');
}

// ════════════════════════════════════════════════════════════
// UI HELPERS
// ════════════════════════════════════════════════════════════
function setApiPill(color,text) {
  const pill=document.getElementById('api-pill');
  const dot=document.getElementById('api-dot');
  const txt=document.getElementById('api-txt');
  const colors={green:'pill-green',red:'pill-red',orange:'pill-gold',grey:'pill-grey'};
  pill.className='pill '+(colors[color]||'pill-grey');
  txt.textContent=text;
  if(color==='orange') dot.classList.add('pulse'); else dot.classList.remove('pulse');
}

function setBanner(cls,text) {
  const b=document.getElementById('banner');
  b.className='banner '+cls;
  document.getElementById('banner-txt').textContent=text;
}

function tab(name,el) {
  document.querySelectorAll('.panel').forEach(p=>p.classList.remove('on'));
  document.querySelectorAll('.tab').forEach(t=>t.classList.remove('on'));
  document.getElementById('panel-'+name).classList.add('on');
  el.classList.add('on');
  if(name==='settings') buildManualInputs();
  if(name==='market')   buildMarket();
}

function toast(title,msg,type='inf') {
  const c=document.getElementById('toasts');
  const t=document.createElement('div');
  t.className='toast '+type;
  t.innerHTML=`<div class="t-t">${title}</div><div class="t-m">${msg}</div>`;
  c.appendChild(t);
  setTimeout(()=>{t.style.opacity='0';t.style.transform='translateY(20px)';t.style.transition='all 0.3s';setTimeout(()=>t.remove(),300);},4500);
}

function beep(freqs) {
  try {
    const ctx=new(window.AudioContext||window.webkitAudioContext)();
    freqs.forEach((f,i)=>{
      const o=ctx.createOscillator(),g=ctx.createGain();
      o.connect(g);g.connect(ctx.destination);
      o.frequency.value=f;g.gain.value=0.12;
      o.start(ctx.currentTime+i*0.15);
      o.stop(ctx.currentTime+i*0.15+0.1);
    });
  } catch(e){}
}

// ════════════════════════════════════════════════════════════
// NEPAL TIME
// ════════════════════════════════════════════════════════════
function tick() {
  const now=new Date();
  const nst=new Date(now.toLocaleString('en-US',{timeZone:'Asia/Kathmandu'}));
  const h=nst.getHours(),m=nst.getMinutes(),d=nst.getDay();
  const ts=nst.toLocaleTimeString('en-US',{hour:'2-digit',minute:'2-digit',second:'2-digit'});
  const ds=nst.toLocaleDateString('en-US',{weekday:'short',month:'short',day:'numeric'});
  document.getElementById('nst-time').textContent=`🕐 NST ${ts} · ${ds}`;
  const open=d>=0&&d<=4&&(h>11||(h===11&&m>=0))&&h<15;
  const el=document.getElementById('mkt-lbl');
  el.textContent=open?'● OPEN':'● CLOSED';
  el.className=open?'mopen':'mclosed';
}
setInterval(tick,1000); tick();

// ════════════════════════════════════════════════════════════
// INIT
// ════════════════════════════════════════════════════════════
renderAll();
buildManualInputs();

// Auto-refresh every 5 min if not manual
setInterval(()=>{ if(srcIdx!==3&&!fetching) doRefresh(); }, 5*60*1000);

setTimeout(()=>toast('🇳🇵 NEPSE Tracker Ready',
  Object.keys(manualP).length>0
    ? `${Object.keys(manualP).length} manual prices loaded. Tap Refresh or update in Settings.`
    : 'Tap a proxy source then Refresh for live prices, or enter manually in ⚙️ Settings.',
  'inf'),600);
</script>
</body>
</html>
