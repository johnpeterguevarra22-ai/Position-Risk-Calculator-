<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Position Size Calculator</title>
<style>
:root{--bg:#f4f5f7;--card:#fff;--ink:#14171c;--mut:#667085;--line:#e1e4ea;--acc:#0a8f5a;--bad:#c4321f;--warn:#a86400;--inp:#f0f2f5;
box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
@media (prefers-color-scheme:dark){:root:not([data-theme="light"]){--bg:#0e1116;--card:#171b22;--ink:#e8ebf0;--mut:#8b94a3;--line:#262c36;--acc:#2ed38a;--bad:#ff6b57;--warn:#f5b042;--inp:#0e1116}}
:root[data-theme="dark"]{--bg:#0e1116;--card:#171b22;--ink:#e8ebf0;--mut:#8b94a3;--line:#262c36;--acc:#2ed38a;--bad:#ff6b57;--warn:#f5b042;--inp:#0e1116}
html{scroll-padding-top:env(safe-area-inset-top,0px)}
*{box-sizing:border-box}
body{margin:0;background:var(--bg);color:var(--ink);font:16px/1.4 system-ui,-apple-system,"Segoe UI",Roboto,sans-serif}
main{max-width:420px;margin:0 auto;padding:16px}
.card{background:var(--card);border:1px solid var(--line);border-radius:16px;padding:16px;margin-bottom:12px}
label{display:block;font-size:12px;color:var(--mut);margin-bottom:4px}
input{width:100%;padding:12px;border:1px solid var(--line);border-radius:10px;background:var(--inp);color:var(--ink);font:inherit;font-size:18px}
input:focus{outline:2px solid var(--acc)}
.two{display:grid;grid-template-columns:1fr 1fr;gap:10px;margin-bottom:12px}
.sd{display:grid;grid-template-columns:1fr 1fr;gap:8px}
.sd button{font:inherit;font-weight:700;padding:13px;border-radius:12px;border:1px solid var(--line);background:var(--inp);color:var(--ink);cursor:pointer}
.sd .on.long{background:var(--acc);border-color:var(--acc);color:#fff}
.sd .on.short{background:var(--bad);border-color:var(--bad);color:#fff}
.big{font-size:42px;font-weight:800;letter-spacing:-.02em;text-align:center;line-height:1.1}
.cap{text-align:center;font-size:12px;color:var(--mut);margin-bottom:2px}
.line{text-align:center;color:var(--mut);margin-top:10px}
.line b{color:var(--ink)}
.st{margin-top:12px;font-size:14px;text-align:center}
.b{color:var(--bad)}.w{color:var(--warn)}.g{color:var(--acc)}
details{margin-top:14px}
summary{cursor:pointer;color:var(--mut);font-size:13px}
.more{display:grid;grid-template-columns:1fr 1fr;gap:10px;margin-top:10px}
.more div:first-child{grid-column:1/3}
.cp{display:flex;justify-content:space-between;align-items:center;margin-bottom:12px;font-size:13px;color:var(--mut)}
.cp input{width:110px;text-align:right;font-size:15px;padding:8px}
</style>
</head>
<body>
<main>
<div class="cp"><span>My capital (USDT)</span><input type="number" id="cap" inputmode="decimal" value="1000"></div>

<div class="card">
  <div class="two">
    <div><label for="risk">Risk $</label><input type="number" id="risk" inputmode="decimal" value="20"></div>
    <div><label for="stop">Stop %</label><input type="number" id="stop" inputmode="decimal" step="any" value="2"></div>
  </div>
  <div class="sd"><button class="long on" data-s="long">Long</button><button class="short" data-s="short">Short</button></div>
  <details><summary>More (entry, fees, margin)</summary>
    <div class="more">
      <div><label for="entry">Entry price</label><input type="number" id="entry" inputmode="decimal" step="any"></div>
      <div><label for="fee">Fee % per side</label><input type="number" id="fee" inputmode="decimal" step="any"></div>
      <div><label for="margin">Margin $</label><input type="number" id="margin" inputmode="decimal"></div>
    </div>
  </details>
</div>

<div class="card" id="res"></div>
</main>

<script>
const $=id=>document.getElementById(id);
const fm=(n,d=2)=>isFinite(n)?n.toLocaleString('en-US',{minimumFractionDigits:d,maximumFractionDigits:d}):'—';
const fp=n=>n>=100?fm(n,2):n>=1?fm(n,4):fm(n,8);
const KEEP=['cap','risk','stop','fee'];
let side='long';

try{const s=JSON.parse(localStorage.getItem('pc')||'{}');KEEP.forEach(i=>{if(s[i]!==undefined)$(i).value=s[i]});if(s.side)side=s.side}catch(e){}
function save(){try{const s={side};KEEP.forEach(i=>s[i]=$(i).value);localStorage.setItem('pc',JSON.stringify(s))}catch(e){}}

function calc(){
  const cap=+$('cap').value,risk=+$('risk').value,stop=+$('stop').value;
  const fee=+$('fee').value||0,entry=+$('entry').value||0,mo=+$('margin').value||0;
  save();
  document.querySelectorAll('.sd button').forEach(b=>b.classList.toggle('on',b.dataset.s===side));
  const res=$('res');
  if(!(cap>0&&risk>0&&stop>0)){res.innerHTML='<div class="line">Enter capital, risk and stop %</div>';return}
  const rt=2*fee,eff=stop+rt,notional=risk/(eff/100),base=mo>0?mo:cap;
  const lev=Math.max(1,Math.ceil(notional/base-1e-9)),margin=notional/lev,liq=100/lev,rp=risk/cap*100;
  const W=[];
  if(lev>50){
    const mr=base*50*eff/100,ms=risk/(base*50)*100-rt;
    W.push(['b',`Not possible: needs ${lev}x. Use risk ≤ $${fm(mr)} or stop ≥ ${fm(Math.max(ms,0.01))}%.`]);
  }
  if(stop>0.8*liq){
    const sf=Math.floor(80/stop);
    W.push(['b',sf>=1?`Could liquidate before stop. Use ≤ ${sf}x with ~$${fm(notional/sf)} margin.`:`Stop too wide for any leverage.`]);
  }
  if(mo>cap)W.push(['w','Margin is more than your capital.']);
  if(rp>2)W.push(['w',`Risk is ${fm(rp)}% of capital (over 2%).`]);

  let ex='';
  if(fee>0)ex+=`<div class="line">Fees <b>$${fm(notional*rt/100)}</b></div>`;
  if(entry>0){
    const sl=side==='long'?entry*(1-stop/100):entry*(1+stop/100);
    ex+=`<div class="line">Stop price <b>${fp(sl)}</b> · Qty <b>${fm(notional/entry,6)}</b></div>`;
  }
  res.innerHTML=`<div class="cap">Position size</div><div class="big ${lev>50?'b':''}">$${fm(notional)}</div>
  <div class="line">Margin <b>$${fm(margin)}</b> · Leverage <b class="${lev>50?'b':''}">${lev}x</b></div>
  <div class="line">Liquidation <b>${fm(liq,1)}%</b> away</div>${ex}
  <div class="st">${W.length?W.map(w=>`<div class="${w[0]}">⚠ ${w[1]}</div>`).join(''):'<span class="g">✓ All good</span>'}</div>`;
}

document.addEventListener('click',e=>{const t=e.target.closest('button');if(t&&t.dataset.s){side=t.dataset.s;calc()}});
document.querySelectorAll('input').forEach(i=>{
  i.addEventListener('input',calc);
  i.addEventListener('focus',()=>i.select());
  i.addEventListener('keydown',e=>{if(e.key==='Enter')i.blur()});
});
calc();
</script>
</body>
</html>
