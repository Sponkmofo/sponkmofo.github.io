# deepfield.github.io
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>DEEPFIELD-9 // Procedural Observatory</title>
<link href="https://fonts.googleapis.com/css2?family=VT323&display=swap" rel="stylesheet">
<style>
:root{
  --amber:#d8a24a; --amber-hi:#f0d9a0; --amber-dim:#8a6d35;
  --line:#3a3226; --bg-panel:rgba(12,9,5,.9); --ink:#cbb387;
}
*{box-sizing:border-box}
html,body{margin:0;height:100%;overflow:hidden;background:#070503}
body{font-family:'VT323',monospace;color:var(--ink);-webkit-user-select:none;user-select:none}
#view{position:fixed;left:0;top:0;cursor:grab;image-rendering:pixelated}
#view.drag{cursor:grabbing}

#hud{position:fixed;left:14px;top:14px;background:var(--bg-panel);border:1px solid var(--line);
  box-shadow:0 0 0 1px #000;padding:9px 13px 8px;min-width:216px;pointer-events:none;z-index:4}
#hud .t{font-size:23px;letter-spacing:2px;color:var(--amber);line-height:1}
#hud .t .cur{display:inline-block;width:9px;height:15px;background:var(--amber);margin-left:5px;
  animation:blink 1.1s steps(1) infinite;vertical-align:-2px}
@keyframes blink{50%{opacity:0}}
#hud .s{font-size:11px;letter-spacing:3px;color:var(--amber-dim);margin:1px 0 7px}
#hud .rows{display:grid;grid-template-columns:auto 1fr;gap:1px 12px;font-size:16px}
#hud .rows em{font-style:normal;color:#6f5c39}
#hud .rows b{font-weight:normal;color:var(--amber-hi);text-align:right}

#panel{position:fixed;right:14px;top:14px;bottom:14px;width:268px;background:var(--bg-panel);
  border:1px solid var(--line);box-shadow:0 0 0 1px #000;overflow-y:auto;overflow-x:hidden;
  padding:12px 14px 22px;transition:transform .28s ease,opacity .28s;z-index:5}
#panel.hidden{transform:translateX(calc(100% + 34px));opacity:0}
#panel::-webkit-scrollbar{width:8px}
#panel::-webkit-scrollbar-track{background:#0d0a06}
#panel::-webkit-scrollbar-thumb{background:var(--line)}
#panel h3{font-size:14px;letter-spacing:3px;color:var(--amber-dim);margin:15px 0 9px;
  border-bottom:1px dashed var(--line);padding-bottom:4px;font-weight:normal}
#panel h3:first-child{margin-top:2px}
.ctl{margin-bottom:9px}
.lr{display:flex;justify-content:space-between;font-size:16px;line-height:1.1}
.lr b{color:var(--amber);font-weight:normal}
input[type=range]{-webkit-appearance:none;appearance:none;width:100%;height:15px;background:transparent;margin:3px 0 0;cursor:ew-resize}
input[type=range]::-webkit-slider-runnable-track{height:6px;background:#1c1710;border:1px solid var(--line)}
input[type=range]::-webkit-slider-thumb{-webkit-appearance:none;width:12px;height:15px;background:var(--amber);border:1px solid #000;margin-top:-6px}
input[type=range]::-moz-range-track{height:6px;background:#1c1710;border:1px solid var(--line)}
input[type=range]::-moz-range-thumb{width:10px;height:13px;background:var(--amber);border:1px solid #000;border-radius:0}
.btn{border:1px solid var(--amber-dim);background:#181209;color:var(--amber);font-family:inherit;
  font-size:16px;letter-spacing:1px;padding:6px 10px;cursor:pointer;box-shadow:2px 2px 0 #000;
  display:inline-flex;align-items:center;gap:7px}
.btn:hover{background:#241a0c}
.btn:active{transform:translate(1px,1px);box-shadow:1px 1px 0 #000}
.btn.on{background:var(--amber);color:#140e05}
.btn svg{display:block}
.btnrow{display:flex;gap:8px;margin-bottom:4px}
.btnrow .btn{flex:1;justify-content:center}
.seedrow{display:flex;gap:8px}
.seedrow input{flex:1;min-width:0;border:1px solid var(--line);background:#0d0a06;color:var(--amber-hi);
  font-family:inherit;font-size:17px;padding:5px 8px;letter-spacing:2px}
.chips{display:grid;grid-template-columns:1fr 1fr;gap:6px}
.chip{font-size:13px;padding:5px 3px;text-align:center;cursor:pointer;color:#7a663c;background:#100b06;
  border:1px solid var(--line);letter-spacing:1px;line-height:1.15}
.chip:hover{color:var(--amber)}
.chip.on{color:#140e05;background:var(--amber);border-color:var(--amber)}
.mini{font-size:12px;padding:2px 7px;box-shadow:none}
.pals{display:flex;flex-direction:column;gap:6px}
.pal{display:flex;align-items:center;gap:9px;cursor:pointer;padding:5px 8px;border:1px solid var(--line);
  background:#100b06;font-size:15px;letter-spacing:1px;color:#9c8352}
.pal:hover{color:var(--amber-hi)}
.pal.on{border-color:var(--amber);color:var(--amber)}
.pal .sw{display:flex;flex-shrink:0}
.pal .sw i{width:9px;height:13px;display:block}

#panelToggle{position:fixed;top:14px;right:296px;z-index:6;width:34px;height:34px;justify-content:center;
  padding:0;transition:right .28s ease}
body.panelClosed #panelToggle{right:14px}

#inspector{position:fixed;left:14px;bottom:14px;background:var(--bg-panel);border:1px solid var(--line);
  box-shadow:0 0 0 1px #000;padding:10px 13px 9px;min-width:236px;max-width:300px;z-index:4;display:none}
#inspector.open{display:block}
#inspector .n{font-size:21px;color:var(--amber);letter-spacing:1px;line-height:1}
#inspector .c{font-size:12px;letter-spacing:3px;color:var(--amber-dim);margin:2px 0 7px}
#inspector .rows{display:grid;grid-template-columns:auto 1fr;gap:1px 12px;font-size:15px}
#inspector .rows em{font-style:normal;color:#6f5c39}
#inspector .rows b{font-weight:normal;color:var(--amber-hi);text-align:right}
#inspector .f{font-size:11px;letter-spacing:2px;color:#55431f;margin-top:7px}

#hint{position:fixed;left:50%;bottom:12px;transform:translateX(-50%);font-size:14px;color:#6b5a3a;
  letter-spacing:2px;pointer-events:none;z-index:3;white-space:nowrap}

#fault{position:fixed;top:14px;left:50%;transform:translateX(-50%);z-index:20;display:none;
  background:#2a0906;border:1px solid #a83a20;color:#ffb9a0;font-size:15px;letter-spacing:1px;
  padding:7px 14px;max-width:70vw;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}

/* ---- idle auto-hide (fault readout stays visible) ---- */
#hud,#panel,#panelToggle,#inspector,#hint{transition:opacity .35s ease}
body.uiHidden #hud,body.uiHidden #panel,body.uiHidden #panelToggle,
body.uiHidden #inspector,body.uiHidden #hint{opacity:0;pointer-events:none}
body.uiHidden,body.uiHidden #view{cursor:none}

@media(max-width:760px){
  #panel{width:236px}#panelToggle{right:264px}#hud{min-width:0;transform:scale(.85);transform-origin:top left}
  #hint{display:none}
}
</style>
</head>
<body>
<canvas id="view"></canvas>

<div id="hud">
  <div class="t">DEEPFIELD-9<span class="cur"></span></div>
  <div class="s">PROCEDURAL OBSERVATORY CONSOLE</div>
  <div class="rows">
    <em>SECTOR</em><b id="hSector">0000/0000</b>
    <em>HEADING</em><b id="hHdg">000</b>
    <em>VELOCITY</em><b id="hVel">0.0</b>
    <em>MAGNIFICATION</em><b id="hZoom">1.00x</b>
    <em>BODIES</em><b id="hBodies">0</b>
    <em>NEXT JUMP</em><b id="hJump">—</b>
    <em>FPS</em><b id="hFps">--</b>
  </div>
</div>

<div id="fault"></div>

<button class="btn" id="panelToggle" title="Toggle console">
  <svg width="14" height="14" viewBox="0 0 14 14"><path d="M2 1h3v12H2zM9 1h3v12H9z" fill="currentColor"/></svg>
</button>

<aside id="panel"></aside>

<div id="inspector">
  <div class="n" id="insName">-</div>
  <div class="c" id="insClass">-</div>
  <div class="rows" id="insRows"></div>
  <div class="f">CLICK EMPTY SPACE TO RELEASE TARGET</div>
</div>

<div id="hint">DRAG: PAN &nbsp;/&nbsp; WHEEL: ZOOM &nbsp;/&nbsp; CLICK: SCAN &nbsp;/&nbsp; [S] LOCK UI</div>

<script>
'use strict';
/* ================================================================
   DEEPFIELD-9 v5 — infinite procedural universe
   + snacks (pizza/burger/fries/cherry/banana)
   + neon signs (PIZZA / SPONK / JUNGLE / DnB / WAVE) w/ 5x7 font
   ================================================================ */

/* ---------------- fault readout (never fail silently) ------------- */
const faultEl=document.getElementById('fault');
const faultSeen=new Map();let faultTimer=0;
function fault(msg){
  const now=performance.now(),last=faultSeen.get(msg);
  if(last!==undefined&&now-last<60000)return;     /* same message: at most once a minute */
  faultSeen.set(msg,now);console.error(msg);
  if(window._noFault)return;                      /* ?fault=0 hides the banner entirely */
  faultEl.textContent='CONSOLE FAULT: '+msg;faultEl.style.display='block';
  clearTimeout(faultTimer);
  faultTimer=setTimeout(()=>{faultEl.style.display='none';},8000);
}
addEventListener('error',e=>fault(e.message||'unknown error'));

/* ---------------- utilities ---------------- */
const TAU=Math.PI*2;
const clamp=(v,a,b)=>v<a?a:v>b?b:v;
function ihash(x,y,s){
  let h=Math.imul(x|0,374761393)^Math.imul(y|0,668265263)^Math.imul(s|0,974634721);
  h=Math.imul(h^(h>>>13),1274126177); h^=h>>>16; return h>>>0;
}
function mulberry(a){return function(){a|=0;a=a+0x6D2B79F5|0;let t=Math.imul(a^a>>>15,1|a);
  t=t+Math.imul(t^t>>>7,61|t)^t;return((t^t>>>14)>>>0)/4294967296;};}
function vnoise(x,y,s){
  const xi=Math.floor(x),yi=Math.floor(y),xf=x-xi,yf=y-yi;
  const u=xf*xf*(3-2*xf),v=yf*yf*(3-2*yf);
  const a=ihash(xi,yi,s)/4294967296,b=ihash(xi+1,yi,s)/4294967296,
        c=ihash(xi,yi+1,s)/4294967296,d=ihash(xi+1,yi+1,s)/4294967296;
  return (a+(b-a)*u)*(1-v)+(c+(d-c)*u)*v;
}
const fbm=(x,y,s)=>vnoise(x,y,s)*.65+vnoise(x*2.7,y*2.7,s^0x9e37)*.35;
function poisson(R,l){if(l<=0)return 0;const L=Math.exp(-l);let k=0,p=1;do{k++;p*=R();}while(p>=L);return k-1;}
const pick=(R,a)=>a.length?a[(R()*a.length)|0]:0;
function strHash(s){let h=2166136261;for(let i=0;i<s.length;i++){h^=s.charCodeAt(i);h=Math.imul(h,16777619);}return h>>>0;}
function el(tag,cls){const e=document.createElement(tag);if(cls)e.className=cls;return e;}

/* ---- color conversion ---- */
function rgb2hsl(r,g,b){
  r/=255;g/=255;b/=255;
  const mx=Math.max(r,g,b),mn=Math.min(r,g,b);
  let h=0,s=0;const l=(mx+mn)/2;
  if(mx!==mn){
    const d=mx-mn;
    s=l>.5?d/(2-mx-mn):d/(mx+mn);
    if(mx===r)h=(g-b)/d+(g<b?6:0);
    else if(mx===g)h=(b-r)/d+2;
    else h=(r-g)/d+4;
    h*=60;
  }
  return [h,s,l];
}
function hsl2rgb(h,s,l){
  h=((h%360)+360)%360;
  const c=(1-Math.abs(2*l-1))*s,x=c*(1-Math.abs((h/60)%2-1)),m=l-c/2;
  let r,g,b;
  if(h<60){r=c;g=x;b=0;}else if(h<120){r=x;g=c;b=0;}
  else if(h<180){r=0;g=c;b=x;}else if(h<240){r=0;g=x;b=c;}
  else if(h<300){r=x;g=0;b=c;}else{r=c;g=0;b=x;}
  return [clamp((r+m)*255,0,255)|0,clamp((g+m)*255,0,255)|0,clamp((b+m)*255,0,255)|0];
}

/* ---------------- palettes ---------------- */
const PALETTES=[
 {n:'VOID CLASSIC', c:['#04060a','#0a0e1c','#141a33','#1f2851','#2f3d78','#48569e','#6d7ac4','#9ba7e2','#c6cef5','#f0f3ff','#fffaea','#ffe9a6','#ffc65c','#ff9a3c','#e86a2e','#a83a20']},
 {n:'NEBULA ROSE',  c:['#0a0409','#1b0a17','#2e1032','#491750','#682070','#8b2f86','#b448a4','#d96cc2','#f298d8','#ffd2e8','#fff2f8','#ffd9b0','#ffb06a','#f2833c','#c9ebff','#8fd8ff']},
 {n:'PHOSPHOR',     c:['#020802','#051808','#082e12','#0b4a1c','#0f6626','#158230','#1d9e3a','#2cba44','#4bd65e','#7fee74','#b4faa2','#e2fdd6','#f9ffe9','#c9ffbf','#7df5b0','#40e08c']},
 {n:'AMBER DECK',   c:['#0a0602','#1a0f04','#301d08','#4b3310','#6b4a15','#8f681d','#b38726','#d3a833','#eec255','#ffd97d','#ffeda8','#fff8d6','#c86b1e','#e89a3c','#a35416','#5c3a10']},
 {n:'ICEFIELD',     c:['#030608','#071018','#0c1d29','#123041','#1a4b5e','#256c81','#3594ac','#52bdcf','#7fdce8','#b3f0f4','#e6fbfb','#ffffff','#cfe8ff','#9cc8e8','#f0d8b0','#d8a86a']},
 {n:'EMBER SECTOR', c:['#080304','#170609','#2b0b0f','#461015','#6a171b','#90231f','#b53a24','#d35c2c','#ec8340','#f7ac63','#fdc78e','#ffe6c2','#ffd9d0','#fff0ea','#5c1a22','#8f5a2a']},
 {n:'OBSIDIAN',     c:['#060607','#0f1013','#1a1c22','#282b33','#3a3f4a','#515866','#6e7686','#9099a8','#b4bac4','#d8dbe0','#f2f3f5','#ffffff','#c2b49a','#a3875c','#d4c7a8','#6b5a3f']},
 {n:'SPECTRUM',     c:['#050308','#140a1e','#241238','#3b1d52','#572a6b','#763a80','#96508f','#b56f9a','#d094a5','#e8bcae','#f8ddcd','#fff6e0','#ffd23f','#ff8f3f','#ff4f6a','#3fe0c8']},
 {n:'GAMEBOY',      c:['#0f380f','#306230','#8bac0f','#9bbc0f']},
 {n:'MONOCHROME',   c:['#050505','#141414','#262626','#3d3d3d','#5a5a5a','#7d7d7d','#a3a3a3','#cccccc','#f0f0f0','#ffffff']}
];
/* fixed role colors for characters / snacks / special objects */
const ROLE_T={
  R:[228,65,58],  B:[64,92,206], S:[243,199,153], K:[110,70,38], G:[84,172,66],
  Y:[246,214,74], W:[252,252,252],P:[246,152,172],D:[30,28,36],  Q:[206,86,114],
  O:[246,152,64], T:[214,176,124],A:[138,255,152], L:[150,205,255]
};
const P={i:0,n:1,baseN:1,R:null,G:null,B:null,str:[],lum:[],hot:[],top:[],hi:[],lo:[],deep:[],fams:[],ROLE:{},cache:null};

function setPalette(i){
  P.i=i;
  const base=PALETTES[i].c.slice();
  const baseN=base.length;
  const seen=new Set(base);
  /* extended palette: hue-rotations of saturated mid-bright base colors */
  for(let k=0;k<baseN;k++){
    const r=parseInt(base[k].slice(1,3),16),g=parseInt(base[k].slice(3,5),16),b=parseInt(base[k].slice(5,7),16);
    const [h,s,l]=rgb2hsl(r,g,b);
    if(s<.14||l<.18||l>.93)continue;
    for(let rot=1;rot<8;rot++){
      const [nr,ng,nb]=hsl2rgb(h+rot*45,s,l);
      const hex='#'+[nr,ng,nb].map(v=>v.toString(16).padStart(2,'0')).join('');
      if(!seen.has(hex)){seen.add(hex);base.push(hex);}
    }
  }
  while(base.length>96)base.pop();
  P.baseN=baseN;P.n=base.length;
  P.R=new Uint8Array(P.n);P.G=new Uint8Array(P.n);P.B=new Uint8Array(P.n);
  P.lum=[];P.str=[];
  base.forEach((hex,k)=>{
    const r=parseInt(hex.slice(1,3),16),g=parseInt(hex.slice(3,5),16),b=parseInt(hex.slice(5,7),16);
    P.R[k]=r;P.G[k]=g;P.B[k]=b;
    P.lum.push((r*.2126+g*.7152+b*.0722)/255);
    P.str.push('rgb('+r+','+g+','+b+')');
  });
  const byBase=[...Array(baseN).keys()].sort((a,b)=>P.lum[b]-P.lum[a]);
  const bin=(lo,hi)=>byBase.filter(k=>P.lum[k]>lo&&P.lum[k]<=hi);
  P.top=bin(.8,1.01); P.hot=bin(.62,1.01);
  P.hi=bin(.34,.72);  P.lo=bin(.13,.34); P.deep=bin(-.01,.13);
  if(!P.hot.length)P.hot=byBase.slice(0,2);
  if(!P.top.length)P.top=[P.hot[0]];
  if(!P.hi.length)P.hi=P.hot.slice(0,2);
  if(!P.lo.length)P.lo=P.hi.slice();
  if(!P.deep.length)P.deep=[byBase[byBase.length-1]];
  /* hue families over the FULL extended palette */
  P.fams=[];
  const buckets=Array.from({length:12},()=>({bright:[],mid:[],dim:[]}));
  for(let k=0;k<P.n;k++){
    const [h,s]=rgb2hsl(P.R[k],P.G[k],P.B[k]);
    if(s<.15)continue;
    const fb=buckets[Math.floor((((h%360)+360)%360)/30)%12];
    if(P.lum[k]>.55)fb.bright.push(k);
    else if(P.lum[k]>.3)fb.mid.push(k);
    else fb.dim.push(k);
  }
  for(const fb of buckets){
    if(fb.bright.length+fb.mid.length+fb.dim.length<3)continue;
    const br=fb.bright.length?fb.bright:(fb.mid.length?fb.mid:fb.dim);
    const md=fb.mid.length?fb.mid:(fb.bright.length?fb.bright:fb.dim);
    const dm=fb.dim.length?fb.dim:(fb.mid.length?fb.mid:br);
    P.fams.push({bright:br,mid:md,dim:dm});
  }
  if(!P.fams.length)P.fams.push({bright:P.hot.slice(),mid:P.hi.slice(),dim:P.lo.slice()});
  P.ROLE={};
  for(const k in ROLE_T)P.ROLE[k]=nearest(ROLE_T[k][0],ROLE_T[k][1],ROLE_T[k][2]);
  P.cache=new Uint8Array(32768);
}
const CS=ci=>P.str[((ci%P.n)+P.n)%P.n];
function nearest(r,g,b){
  let bi=0,bd=1e9;
  for(let k=0;k<P.n;k++){
    const dr=r-P.R[k],dg=g-P.G[k],db=b-P.B[k],d=dr*dr+dg*dg+db*db;
    if(d<bd){bd=d;bi=k;}
  }
  return bi;
}
const nearT=(r,g,b)=>nearest(clamp(r,0,255),clamp(g,0,255),clamp(b,0,255));
function famCol(R,f,tier){
  if(f<0||!P.fams.length)
    return tier===0?pick(R,P.hot):tier===1?pick(R,P.hi):pick(R,P.lo);
  const F=P.fams[clamp(f,0,P.fams.length-1)];
  const arr=tier===0?F.bright:tier===1?F.mid:F.dim;
  return pick(R,arr);
}

/* ---------------- settings ---------------- */
const S={ps:4,dither:.65,glow:.9,twinkle:.7,dots:.35,scan:.12,drift:16,ztzoom:1,tint:1,fps:60,crisp:true,sprSize:1,
         autoReroll:false,rerollMin:60,
         stars:1,neb:1,deep:1,planets:1,small:1,traffic:1,events:1,paused:false};
let zoom=1;
const ON={};const TYPELIST=[
 ['suns','SUNS'],['binary','BINARIES'],['planets','PLANETS'],['gas','GAS GIANTS'],
 ['ringed','RINGED WORLDS'],['moons','MOONS'],['pulsar','PULSARS'],['bhole','BLACK HOLES'],
 ['emission','EMISSION NEBULAE'],['snr','SN REMNANTS'],['pn','PLANETARY NEB'],
 ['dark','DARK NEBULAE'],['galaxy','GALAXIES'],['aster','ASTEROIDS'],['lone','BIG ASTEROIDS'],
 ['comet','COMETS'],['meteor','METEORS'],['ship','SHIPS'],['station','STATIONS'],
 ['battle','BATTLE MOONS'],['shattered','SHATTERED WORLDS'],['mega','MEGA STATIONS'],
 ['ring','RINGWORLDS'],['dyson','DYSON SPHERES'],['rift','REALITY TEARS'],
 ['probe','PROBES'],['chars','FUN BODIES'],
 ['earth','EARTHS'],['ocean','OCEAN WORLDS'],['storm','HYPERCANES'],
 ['sat','SATELLITES'],['ufo','UFOS'],['astro','ASTRONAUTS'],
 ['food','SPACE SNACKS'],['sign','NEON SIGNS'],['icon','ICONS'],['worm','WORMHOLES'],['jelly','VOID JELLIES'],['beacon','LIGHTHOUSES'],['nova','SUPERNOVAE'],['rock','ROCKS'],['junk','SPACE JUNK'],['field','DEBRIS FIELDS'],['xtra','ANOMALIES']];
TYPELIST.forEach(t=>ON[t[0]]=true);

/* ---------------- URL parameters (read on load, kept in sync) ---------------- */
const S_DEF=Object.assign({},S);
const NUMR={ps:[2,8],dither:[0,1],glow:[0,1.5],twinkle:[0,1],dots:[0,1],scan:[0,1],drift:[0,30],
  ztzoom:[.35,3],fps:[10,300],stars:[0,2],neb:[0,2],deep:[0,2],planets:[0,2],small:[0,2],
  traffic:[0,2],events:[0,2],tint:[0,1],sprSize:[.5,2],rerollMin:[60,1440]};
const BOOLK=['crisp','autoReroll'];
const URLC={seed:null,pal:null,panel:null,ui:null};
(function(){
  let q;try{q=new URLSearchParams(location.search);}catch(e){return;}
  const no=v=>/^(0|false|off|no)$/i.test(v);
  for(const k in NUMR){
    const v=parseFloat(q.get(k));
    if(q.has(k)&&isFinite(v))S[k]=clamp(k==='ps'?Math.round(v):v,NUMR[k][0],NUMR[k][1]);
  }
  for(const k of BOOLK)if(q.has(k))S[k]=!no(q.get(k));
  const list=k=>(q.get(k)||'').split(',').map(s=>s.trim().toLowerCase()).filter(Boolean);
  if(q.has('only')){const on=list('only');TYPELIST.forEach(t=>ON[t[0]]=on.includes(t[0]));}
  if(q.has('off')){const off=list('off');TYPELIST.forEach(t=>{if(off.includes(t[0]))ON[t[0]]=false;});}
  if(q.get('seed'))URLC.seed=q.get('seed').slice(0,12);
  const pv=q.get('pal');
  if(pv!==null){
    let i=/^\d+$/.test(pv)?parseInt(pv,10):-1;
    if(i<0||i>=PALETTES.length){
      const norm=s=>s.toLowerCase().replace(/[\s_-]+/g,'');
      i=PALETTES.findIndex(p=>norm(p.n)===norm(pv));
    }
    if(i>=0)URLC.pal=i;
  }
  if(q.has('panel'))URLC.panel=!no(q.get('panel'));
  if(q.has('ui'))URLC.ui=!no(q.get('ui'));
  if(q.has('fault')&&no(q.get('fault')))window._noFault=true;
})();

/* ---------------- universe state ---------------- */
let seedStr='',seed=1,bandC=1,bandS=0,STAMP='';
const BANDW=1500;
const cam={x:0,y:0};let hdg=0;
let wt=0,rt=0;
function bandFactor(x,y){
  const d=Math.abs(-bandS*x+bandC*y),f=1-d/BANDW;
  return f<=0?0:f*f*(3-2*f);
}
function contentStamp(){
  return P.i+'|'+seed+'|'+(+S.tint).toFixed(2)+'|'+
   [S.stars,S.neb,S.deep,S.planets,S.small,S.traffic].map(v=>(+v).toFixed(2)).join(',')+
   '|'+TYPELIST.map(t=>ON[t[0]]?1:0).join('');
}
function rebuildStamp(){STAMP=contentStamp();}
function chunkFam(cx,cy,salt){
  if(S.tint<=0)return -1;
  if(vnoise(cx*.09,cy*.09,seed^0x5EED)>=.62*S.tint)return -1;
  const n=vnoise(cx*.09+9.3,cy*.09,seed^(salt||0xA0B));
  return Math.min(P.fams.length-1,(n*P.fams.length*1.35)|0);
}

/* ---------------- canvases ---------------- */
const cv=document.getElementById('view'),mctx=cv.getContext('2d');
const low=document.createElement('canvas'),lctx=low.getContext('2d',{willReadFrequently:true});
/* crossfade snapshot canvas — allocated once, reused for every reroll */
const fadeC=document.createElement('canvas'),fctx=fadeC.getContext('2d');
const FADE_DUR=2200;
const fade={active:false,t0:0};
let PX=4,lw=0,lh=0,dotPat=null,scanPat=null;
function calcPX(){return Math.max(S.ps,Math.ceil(Math.sqrt(innerWidth*innerHeight/340000)));}
function buildMasks(){
  try{
    if(S.dots>0.02){
      const c=document.createElement('canvas');c.width=c.height=PX;
      const x=c.getContext('2d');x.fillStyle='#000';x.fillRect(0,0,PX,PX);
      x.globalCompositeOperation='destination-out';
      x.beginPath();x.arc(PX/2,PX/2,Math.max(.6,PX/2*(1-S.dots*.45)),0,TAU);x.fill();
      dotPat=mctx.createPattern(c,'repeat');
    }else dotPat=null;
    if(S.scan>0.02){
      const c=document.createElement('canvas');c.width=1;c.height=PX;
      const x=c.getContext('2d');x.fillStyle='#000';
      x.fillRect(0,0,1,Math.max(1,Math.round(PX*.28)));
      scanPat=mctx.createPattern(c,'repeat');
    }else scanPat=null;
  }catch(err){fault('mask: '+err.message);dotPat=null;scanPat=null;}
}
function resize(){
  PX=calcPX();
  lw=Math.max(1,Math.ceil(innerWidth/PX));lh=Math.max(1,Math.ceil(innerHeight/PX));
  low.width=lw;low.height=lh;
  cv.width=lw*PX;cv.height=lh*PX;
  cv.style.width=cv.width+'px';cv.style.height=cv.height+'px';
  fade.active=false;  /* dimension change invalidates the frozen frame */
  buildMasks();
}
addEventListener('resize',resize);

/* ---------------- bayer dither post-process ---------------- */
const B8=[0,32,8,40,2,34,10,42,48,16,56,24,50,18,58,26,12,44,4,36,14,46,6,38,
 60,28,52,20,62,30,54,22,3,35,11,43,1,33,9,41,51,19,59,27,49,17,57,25,
 15,47,7,39,13,45,5,37,63,31,55,23,61,29,53,21];
const LUT=new Int16Array(64);
function rebuildLUT(){const a=S.dither*72;for(let i=0;i<64;i++)LUT[i]=(((B8[i]+.5)/64)-.5)*a;}
function post(){
  const img=lctx.getImageData(0,0,lw,lh),d=img.data;
  const cache=P.cache,PR=P.R,PG=P.G,PB=P.B;
  let i=0;
  for(let y=0;y<lh;y++){
    const row=(y&7)<<3;
    for(let x=0;x<lw;x++,i+=4){
      const o=LUT[row|(x&7)];
      let r=d[i]+o,g=d[i+1]+o,b=d[i+2]+o;
      r=r<0?0:r>255?255:r;g=g<0?0:g>255?255:g;b=b<0?0:b>255?255:b;
      const key=((r>>3)<<10)|((g>>3)<<5)|(b>>3);
      let ci=cache[key];
      if(ci===0){ci=nearest(r,g,b)+1;cache[key]=ci;}
      ci--;
      d[i]=PR[ci];d[i+1]=PG[ci];d[i+2]=PB[ci];
    }
  }
  lctx.putImageData(img,0,0);
}

/* ---------------- names & catalog flavour ---------------- */
const SYL=['ka','ve','lor','an','tau','ry','xa','mo','bel','cy','dra','el','fa','gor','hel','ix',
'ju','kor','ly','mar','nor','oc','pel','qui','ras','sol','tar','ul','vor','wex','zen','ith','ara','cei','ob'];
function genName(R){let n='';const c=2+((R()*2)|0);for(let i=0;i<c;i++)n+=pick(R,SYL);
  return n[0].toUpperCase()+n.slice(1);}
const cat=(R,pool)=>pick(R,pool)+' '+((1000+R()*8999)|0);
const GAL=['MESSIER','NGC','IC','UGC','PGC'];
const NPLAN=['KEPLER','TRAPPIST','TOI','K2','WASP','HAT-P','LHS','TEEGARDEN'];
const NBUL=['NGC','IC','M','Sh2','RCW','LBN','CED'];
const NBH=['SGR A*','CYGNUS X-1','V404 MON','GRS 1915','M87*','GX 339-4','IGR J17091','A0620-00'];
const NSTA=['WAYPOINT','RELAY','OUTPOST','DRY DOCK','BEACON','ANCHORAGE'];
const NSUN=['HD','GLIESE','WOLF','LALANDE','ROSS','GJ','HIP','TZ'];
const NSHIP=['ISV','SSV','NV','CSV','MSV'];
const NSAT=['SAT-COM','ORBITAL-EYE','RELAY','GEO-EYE','SKYNET','TIANGONG','IRIDIUM'];
const CHARSPEC={plumber:'HOMINID PLUMBER',puffball:'SPHEROID BIOFORM',mount:'SAURIAN MOUNT',
  ape:'SIMIAN HEAVYWEIGHT',invader:'CANOPIC INVADER'};
const FOODSPEC={pizza:'ITALIAN-STYLE SLICE ANOMALY',burger:'TRIPLE-DECK PROTEIN STACK',
  fries:'SOLANUM BATON ARRAY',cherry:'DUAL-LOBE DRUPE ANOMALY',banana:'CURVED STARCH CRESCENT'};

/* ---------------- 5x7 bitmap font for neon signs ---------------- */
const FONT5={
A:['01110','10001','10001','11111','10001','10001','10001'],B:['11110','10001','10001','11110','10001','10001','11110'],
C:['01110','10001','10000','10000','10000','10001','01110'],D:['11110','10001','10001','10001','10001','10001','11110'],
E:['11111','10000','10000','11110','10000','10000','11111'],F:['11111','10000','10000','11110','10000','10000','10000'],
G:['01111','10000','10000','10111','10001','10001','01110'],H:['10001','10001','10001','11111','10001','10001','10001'],
I:['11111','00100','00100','00100','00100','00100','11111'],J:['00111','00010','00010','00010','00010','10010','01100'],
K:['10001','10010','10100','11000','10100','10010','10001'],L:['10000','10000','10000','10000','10000','10000','11111'],
M:['10001','11011','10101','10101','10001','10001','10001'],N:['10001','11001','10101','10011','10001','10001','10001'],
O:['01110','10001','10001','10001','10001','10001','01110'],P:['11110','10001','10001','11110','10000','10000','10000'],
Q:['01110','10001','10001','10001','10101','10010','01101'],R:['11110','10001','10001','11110','10100','10010','10001'],
S:['01111','10000','10000','01110','00001','00001','11110'],T:['11111','00100','00100','00100','00100','00100','00100'],
U:['10001','10001','10001','10001','10001','10001','01110'],V:['10001','10001','10001','10001','10001','01010','00100'],
W:['10001','10001','10001','10001','10101','10101','01010'],X:['10001','10001','01010','00100','01010','10001','10001'],
Y:['10001','10001','01010','00100','00100','00100','00100'],Z:['11111','00001','00010','00100','01000','10000','11111'],
'0':['01110','10001','10011','10101','11001','10001','01110'],'1':['00100','01100','00100','00100','00100','00100','01110'],
'2':['01110','10001','00001','00010','00100','01000','11111'],'3':['11110','00001','00001','01110','00001','00001','11110'],
'4':['00010','00110','01010','10010','11111','00010','00010'],'5':['11111','10000','11110','00001','00001','10001','01110'],
'6':['00110','01000','10000','11110','10001','10001','01110'],'7':['11111','00001','00010','00100','01000','01000','01000'],
'8':['01110','10001','10001','01110','10001','10001','01110'],'9':['01110','10001','10001','01111','00001','00010','01100'],
'/':['00001','00010','00010','00100','01000','01000','10000'],'-':['00000','00000','00000','11111','00000','00000','00000'],
'!':['00100','00100','00100','00100','00100','00000','00100'],'>':['00000','00100','00010','11111','00010','00100','00000'],
'<':['00000','00100','01000','11111','01000','00100','00000'],'+':['00000','00100','00100','11111','00100','00100','00000'],
'&':['01100','10010','10100','01000','10101','10010','01101'],'*':['00100','10101','01110','11111','01110','10101','00100'],
'\u2665':['01010','11111','11111','11111','01110','00100','00000']
};
const SIGNS=['PIZZA','SPONK','JUNGLE','DNB','WAVE','24/7','OPEN','EXIT>','\u2665LOVE\u2665','RETRO','METRO','BAR','MOTEL',
 'DINER','ARCADE','CAFE','RAMEN','TACOS','DISCO','JAZZ','TAXI','LIVE','ON AIR','SYNTH','TOKYO','VIDEO','DONUTS','NEON',
 'NO|VACANCY','LOFI|RADIO','HOT|DOGS','FREE|WIFI','COSMIC|DINER','LAST|CHANCE','SPACE|BAR','GAME|OVER','INSERT|COIN',
 'PRESS|START','HIGH|SCORE','OPEN|24/7'];
const SIGNDISP={DNB:'DnB','\u2665LOVE\u2665':'LOVE','EXIT>':'EXIT'};
const NEONS=[[255,70,170],[60,235,255],[150,255,80],[255,176,48],[255,64,70],[176,96,255],[255,238,200],[255,120,40],
 [80,140,255],[90,255,190],[255,60,255],[255,240,70]].map(on=>({on,fr:on.map(v=>Math.min(255,Math.round(v*.55+115)))}));

/* ---------------- pixel character / snack sprites ---------------- */
const CHARS={
 plumber:{spr:[
  '....RRRRR.......','...RRRRRRRRR....','..KKKRRRRRRRRR..','..KSKSSSSDSSS...',
  '..KSSSSSSSSSSS..','..KSSKKKKKSS....','...SSSSSSSS.....','...RRRBRRBRR....',
  '..RRRBBBBBBRR...','.WWRBBYBBYBRWW..','.WWWBBBBBBBBWWW.','....BBBBBBBB....',
  '....BBBBBBBB....','....BBBB.BBBB...','...KKKK...KKKK..','..KKKKK...KKKKK.']},
 puffball:{spr:[
  '.....PPPPPP.....','...PPPPPPPPPP...','..PPPPPPPPPPPP..','.PPPPPPPPPPPPPP.',
  '.PPPPDWPPDWPPPP.','.PPPPDDPPDDPPPP.','.PPPPDBPPDBPPPP.','.PQQPPPPPPPPQQP.',
  'PPPPPPPRRPPPPPPP','.PPPPPPPPPPPPPP.','..PPPPPPPPPPPP..','...PPPPPPPPPP...',
  '..RRRRR..RRRRR..','.RRRRRR..RRRRRR.']},
 mount:{spr:[
  '......GGG.......','....GGWWGGGG....','....GGWDGGGGGG..','...GGGGGGGWWWWW.',
  '...GGGGGGWWWWWWW','...GGGGGGGWWWWW.','....GGGGGGGGGG..','..GGRRRRRRGGG...',
  '.GGGRRRRRRGWWW..','GGGGGGGGGGGWWW..','GGGGGGGGGGGWWW..','.GGGGGGGGGGWW...',
  '..GGGGGGGGGG....','...GG....GG.....','..OOOO...OOOO...','..OOOOO..OOOOO..']},
 ape:{spr:[
  '....KKKKKKK.....','...KKKKKKKKK....','...KDDTTTDDK....','...KTWDTWDTK....',
  '..KKTTTTTTTKK...','..KKTTDTDTTKK...','...KKTDDDDTKK...','.KKKKKKRRKKKKKK.',
  'KKKKKTTRRTTKKKKK','KKKKTTTRRTTTKKKK','KKKKTTTRRTTTKKKK','KKKK.TTRRTT.KKKK',
  'KKKKK.KKKK.KKKKK','KTTTK.KKKK.KTTTK','....KKKK..KKKK..','...KKKKK..KKKKK.']},
 invader:{spr:[
  '..A.....A..','...A...A...','..AAAAAAA..','.AA.AAA.AA.','AAAAAAAAAAA',
  'A.AAAAAAA.A','A.A.....A.A','...AA.AA...']}
};
const FOOD={
 pizza:['.OOOOOOOOOOO.','OTTTTTTTTTTTO','OTTTTTTTTTTTO','.YYYYYYYYYYY.','.YYRRYYYYYYY.',
        '..YRRYYYRRY..','..YYYYYYRRY..','...YYYYYYY...','...YYRRYYY...','....YRRYY....',
        '.....YYY.....','......O......'],
 burger:['....TTTTT....','..TTTTTTTTT..','.TTWTTTTWTTT.','.TTTTTWTTTTT.','TTTWTTTTTWTTT',
         'GGAGGGAGGGAGG','RRRRRRRRRRRRR','YYYYYYYYYYYYY','.KKKKKKKKKKK.','.KDKKKKKDKKK.',
         'TTTTTTTTTTTTT','.OOOOOOOOOOO.'],
 fries:['..Y...Y.Y..','.YY.Y.YYYY.','.YYYYYYYYY.','RRRRRRRRRRR','RRRRWWWRRRR',
        '.RRWWWWWRR.','.RRRWWWRRR.','..RRRRRRR..','..RRRRRRR..'],
 cherry:['.....GG.....','....G..G....','...G....G...','..G......G..','.RRR....RRR.',
         'RRRRR..RRRRR','RWRRR..RWRRR','RRRRR..RRRRR','.RRR....RRR.'],
 banana:['KY.........GG','YYY.......YYG','YYYY.....YYYY','.YYYYYYYYYYY.','..YYYYYYYYY..',
         '...OOOOOOO...']
};
const ICONS={
 heart:['..RRR...RRR..','.RWRRRRRRRRR.','RRWRRRRRRRRRR','RRRRRRRRRRRRR','RRRRRRRRRRRRR',
        '.RRRRRRRRRRR.','..RRRRRRRRR..','...RRRRRRR...','....RRRRR....','.....RRR.....','......R......'],
 star:['......Y......','.....YYY.....','.....YYY.....','YYYYYYYYYYYYY','.YYYWYYYYYYY.',
       '..YYYYYYYYY..','...YYYYYYO...','...YYYYYYO...','..YYYYYYYOO..','..YYYY.YYYY..',
       '.YYYY...YYYY.','.YYY.....YYY.','.YY.......YY.'],
 fire:['.......R....','......RR....','.....RRR.R..','....RRRRRR..','...RRRORRRR.','...RROOORRR.',
       '..RRROOOORRR','..RROOYOOORR','.RRROOYYOORR','.RROOYYYYOOR','.RROOYYWYYOR',
       '.RROYYWWYYOR','..RROYYYYOR.','...RROOOORR.','....RRRRRR..'],
 skull:['...WWWWWWWW...','..WWWWWWWWWW..','.WWWWWWWWWWWW.','.WWWWWWWWWWWW.','.WDDDWWWWDDDW.',
        '.WDDDWWWWDDDW.','.WWDWWWWWWDWW.','..WWWWDDWWWW..','...WWWWWWWW...','....WWWWWW....',
        '....WDWDWDW...','....WWWWWWW...'],
 coin:['...OOOOOO...','..OYYYYYYO..','.OYYWYYYYYO.','OYYWYYOYYYYO','OYWYYYOYYYYO',
       'OYYYYYOYYYYO','OYYYYYOYYYYO','OYYYYYOYYYYO','OYYYYYOYYYOO','.OYYYYYYYYO.',
       '..OYYYYYYO..','...OOOOOO...'],
 bolt:['....WYYYY.','...YYYYY..','..YYYYY...','.YYYYY....','YYYYYYYY..','..YYYYYYY.',
       '.....YYY..','....YYY...','...YYY....','..YYY.....','..YY......','.Y........'],
 ghost:['....LLLLLL....','..LLLLLLLLLL..','.LWWLLLLLLLLL.','.LLDDLLLLDDLL.','.LLDDLLLLDDLL.',
        '.LLLLLDDLLLLL.','LLLLLLLLLLLLLL','LLLLLLLLLLLLLL','LLLLLLLLLLLLLL','LLLLLLLLLLLLLL',
        'LLLLLLLLLLLLLL','LL.LLL.LLL.LLL'],
 gem:['...LLLLLL...','..LWWLLLLB..','.LWLLLLLLLB.','LLLLLLLLLLLL','.LLLLLLLLLB.',
      '..LLLLLLLB..','...LLLLLB...','....LLLB....','.....LB.....','......B.....']
};

/* ---- sprite registry: legacy letter sprites + PNG-derived sprites (frames = animation) ---- */
const PIXCH='0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
const PIX={
 block:{"pal":["000000","e8f0f8","d8a038","885818","f8d820"],"f":[["..000000000000..",".01112222222230.","0114000000042230","0140111111104230","0101111111110230","0201110001110230","0140000001110230","0244401111104230","0244401110044230","0244440004444230","0244401110444230","0244401110444230","0224440004442230","0322222222222330",".03333333333330.","..000000000000.."],["..000000000000..",".01112222222230.","0114444440000030","0144444401111130","0144444011111130","0244404011100030","0144404400000030","0244404444011130","0244404444011130","0244444444400030","0244444444011130","0244444444011130","0224444444400030","0322222222222330",".03333333333330.","..000000000000.."],["..000000000000..",".01112222222230.","0110444444440030","0111044444401130","0111104444011130","0211104444011130","0111104444000030","0211004444044030","0200404444044030","0244444444444430","0204444444444030","0204444444444030","0324444444444430","0322222222222330",".03333333333330.","..000000000000.."],["..000000000000..",".01112222222230.","0110004444442230","0111110444444230","0111111044444230","0200111044044230","0100111044044230","0211110444044230","0211004444044230","0200444444444230","0211044444444230","0211044444444230","0230444444442230","0322222222222330",".03333333333330.","..000000000000.."]],"k":1,"fps":6,"seq":null},
 coin:{"pal":["000000","e8f0f8","f8d820","d8a038","f8f800"],"f":[["......0000......","....00111100....","...0112222230...","...0124114230...","..012414204230..","..012414204230..","..012414204230..","..012414204230..","..012414204230..","..012414204230..","..012412204230..","..012412204230..","...0244004230...","...0222222330...","....00333300....","......0000......"],[".......00.......","......0110......",".....012230.....",".....014430.....","....01210230....","....01210230....","....01210230....","....01210230....","....01210230....","....01210230....","....01210230....","....01210230....",".....024430.....",".....022230.....","......0330......",".......00......."],[".......00.......","......0110......","......0220......","......0440......",".....044230.....",".....014230.....",".....014230.....",".....014230.....",".....014230.....",".....014230.....",".....014230.....",".....014230.....","......0440......","......0220......","......0330......",".......00......."],[".......00.......","......0110......",".....014120.....",".....011420.....","....01141420....","....01414120....","....01141420....","....01414120....","....01141420....","....01414120....","....01141420....","....01414120....",".....014120.....",".....041420.....","......0220......",".......00......."]],"k":1,"fps":8,"seq":null},
 evilflame:{"pal":["b82800","f87800","f8f800","f8c000","f8f8f8","f88800"],"f":[["........0.......","..0..0.0....0...",".0..0..00...00..","...00..01.0010..","0.010..11.0110..","00010.11.1002...","00110113111000..","011111.33331100.","01111...33..1100","01133...22..3110","013.44.44232.310","333..422..42.331","3322.........331",".2222.44..44.33.","..224444444222..","...224.4444....."],["....0..0....0...","...0..00...000..","..00..000.0.0...",".000.1050.0.....","..0.0111..0..0..","...001311300.0..","...01.123122010.",".001...23..31100",".003...23..3110.","..3.2.242222100.","..3..22..24.30..","..3.........3...","...2........3...","...2........3...","...2.24..442....","....44444443...."]],"k":1,"fps":6,"seq":null},
 flame:{"pal":["b82800","f88800","f8c000"],"f":[[".....000000.....","...0011111100...","..011222222110..",".0122.2222.2210.",".012...22...210.","0122...22...2210","0122...22...2210","01222.2222.22210","0122222222222210","0122222222222210","0122212222122210","0012112222112100",".01110122101110.",".00100011000100.","..00..0000..00..",".......00......."],[".....222222.....","...2211111122...","..211000000112..",".2100.0000.0012.",".210...00...012.","2100...00...0012","2100...00...0012","21000.0000.00012","2100000000000012","2100000000000012","2100010000100012","2210110000110122",".21112100121112.",".22122211222122.","..22..2222..22..",".......22......."]],"k":1,"fps":4,"seq":null},
 goomba:{"pal":["000000","b82800","f88800","f8f8f8","f87800","f8f800","f8c000"],"f":[["......0000......","....00111100....",".000100011120...",".0010011112320..",".30003111112110.",".33033311111110.","0301033111111110","0331331111111110","0222222111111110",".00000322111110.",".02222002211110.","..022222222110..","...0024444200...",".....455664.....","....4366664.....","....0000000....."],["................","......0000......","....00111100....","0000111000020...","..000100012320..",".01300031112110.",".03330333111110.","0133010331111110","0113313311111110","0112222221111110",".02300003221110.",".02022220022444.",".402222222445664","0350022224355660",".00660000455600.","...0000..0000..."]],"k":1,"fps":5,"seq":null},
 kirby:{"pal":["784104","f765a7","fbb6d7","f10771"],"f":[["..000.000000.00.",".012101222210110","0122212222221020","0222222222222120","0122222220202220",".012222220202210",".00122222020220.",".01222112222210.",".01222222222220.","..0222222202210.","..012222222220..",".0301222222210..",".033001222110...",".03330000000....",".0330.0330......","..00...00......."],[".......0000.....","..00.00122100.0.",".011012222221020","0122222222222020","0122222222222110","0122222222222200",".01222222020220.",".00222222020220.","..0122112020210.","..01200222222200","..01033022222030","...0333022020330","...0333302203330","...033330103330.","....0330000330..",".....000..000..."]],"k":1,"fps":3,"seq":null},
 luigi:{"pal":["fffeff","388700","ea9e22"],"f":[[".....00000......","....000000000...","....1112212.....","...1212221222...","...12112221222..","...1122221111...",".....2222222....","....110111......","...1110110111...","..111100001111..","..221020020122..","..222000000222..","..220000000022..","....000..000....","...111....111...","..1111....1111.."]],"k":1,"fps":0,"seq":null},
 mario:{"pal":["b53120","6b6d00","ea9e22"],"f":[[".....00000......","....000000000...","....1112212.....","...1212221222...","...12112221222..","...1122221111...",".....2222222....","....110111......","...1110110111...","..111100001111..","..221020020122..","..222000000222..","..220000000022..","....000..000....","...111....111...","..1111....1111.."]],"k":1,"fps":0,"seq":null},
 mushroom:{"pal":["000000","f8f8f8","880000","b80000","f80000"],"f":[[".....000000.....","...0011222200...","..011113333110..",".01111444443110.",".01114411114310.","0234441111114320","0211441111114320","0111141111114310","0111133111133110","0211222222222110","0222000000002210",".00011011011000.","..011101101110..","..011111111110..","...0111111110...","....00000000...."]],"k":1,"fps":0,"seq":null},
 pirahna:{"pal":["000000","f8f8f8","880000","b80000","f80000"],"f":[[".....00..00.....","....01100110....","...0111001110...","...0111001110...","...0111001110...","..001110011100..",".02301100110220.",".04301100110340.","0414011001104130","0243301111033420","0223330000333220",".04434114344220.",".01144114411420.","..014244241140..","...0022222400...",".....000000....."],["................",".0001......1000.","01110......01110","011101....101110","011110....011110",".011101..101110.","..011101101110..",".02011100111040.",".04301111110410.","0414301111033420","0243330000333220","0223341143344220",".04434114341140.",".01142442241140.","..004222222400..","....00000000...."]],"k":1,"fps":3,"seq":null},
 pizza:{"pal":["222034","d9a066","eec39a","fbf236","df7126","d95763","ac3232"],"f":[["................",".........00.....","........0110....","........02210...",".......0344210..","......056334210.",".....03653334210","....033333333410","...0563335633110","..0365333653110.",".0333333111300..","0333313300030...","011110030..0....","00000.030.......",".......0........","................"]],"k":1,"fps":0,"seq":null},
 pow:{"pal":["1f1f1f","ffffff","4242ff"],"f":[["0000000000000000","0111111111111110","0122222222222210","0000000000000000","1111000110010101","1100101101010101","1100101101010101","1100101101010101","1100101101010101","1111001101010101","1100001101011111","1100000110011011","0000000000000000","0122222222222210","0111111111111110","0000000000000000"]],"k":1,"fps":0,"seq":null},
 soda:{"pal":["222034","cbdbfc","9badb7","ac3232","d95763","ffffff"],"f":[["......0000......","....00111100....","...0211221220...","...0002112000...","...0340000330...","...0344433330...","...0344433330...","...0114433130...","...0551111130...","...0551111130...","...0115555130...","...0341111330...","...0344433330...","...0344433330...","....00443300....","......0000......"]],"k":1,"fps":0,"seq":null},
 strawberry:{"pal":["170625","505401","353401","4c4e01","686f0a","a00227","d3042b","ef1339","620527","838c1a","ff2a51","ca042a","640426","c1d542","ff7d8f","ff5e77","fffee8","6c1a37","76334b","120c1a"],"f":[["......00......",".....0120.00..","......0320410.","...000020310..","..0567583900..",".057ababc29d0.",".06a7e7ebc0240","0677f7g7a50210","077a7e7e5bc10.","06a7f7f565h0..","076a7a7a5ci0..","0676a5a5hi0...",".057575c00....",".j000000j.....","....jj........"]],"k":1,"fps":0,"seq":null},
 toad:{"pal":["ed1f1f","ffffff","ffd6bd","7f5b4d","2552ba","ffc600","5c4136"],"f":[["...0111110...","..011111110..",".00110011100.",".01100001110.",".11100001111.",".1111001111..",".001111232...","..00122232...",".....22222...","....445254...","...24452542..","..2255515522.","....111111...","..666111666..","..666...666.."]],"k":1,"fps":0,"seq":null},
 watermelon:{"pal":["170625","ff426c","baae89","ff7592","fc0e3d","ff7591","ff426d","ff416c","ced0a0","722238","ff2b59","9e9415","fc0f3e","5c4b0e","591228","ff2c59","766d25","9e9313","460219","5b4a0e","bccb1b","f6fed2","756c25","45421b","45411b","120c1a"],"f":[[".............00.","...........00120",".........0033420",".......005366420",".....00337731480",".00003311679a480","0b8431171367c8d0","0dd8c7636e1fc8g0","0dh8ca1i1aa48j0.","0hkjlcaaf4c8g0..","0gdhdl44c88d0...",".00gmnl88dg0....","...00gogj00.....","...pp0000p......",".....ppp........"]],"k":1,"fps":0,"seq":null},
 yoshi:{"pal":["00dd15","fdfffe","42404b","fb6f4f","ffffff","fcffff","f81d1d"],"f":[["......00.....",".....001.....",".....012.000.","....001200020","...3000000000","..30044400000","..30444400000","...344445000.","...334440....","....30004....","0.664000400..","00440000000..","400000044....",".4000044.....","..44000......","...333.......","...3333......"]],"k":1,"fps":0,"seq":null},
 boo:{"pal":["000000","c0c0c0","e0e0e0","f8f8f8","f81058"],"f":[["......00000.....","....001111100...","...01122222110..","..0122333332210.",".01233333303010.","0120003333030210","0120330333030210",".020333333333210","0123033334343410","0112333334444410","0123333334444410","012333334444410.",".01223334342410.","..011222222110..","...0011111100...",".....000000....."]],"k":1,"fps":0,"seq":null},
 flower:{"pal":["000000","f88800","b82800","00f800","00b800"],"f":[[".......00.......","..0...0110...0..",".010.011110.020.",".01101111110110.","0111111111111120","0111001110011120","0110110101101120","0111111111111220",".01111111111120.","..021111112220..","...0022222200...",".....000000.....","..000.0340000...",".0334003403340..","033334030333340.","044444000444440."]],"k":1,"fps":0,"seq":null},
 kong:{"pal":["714d35","eac8a3","fe1322","8d8d8d","000000","ffffff","53301d","eccba0"],"f":[["........0000.......",".......0000........","......0000000....11","......00011111..111",".....21103404...111","....0211054141.0111","...0002111101010000","..00000141111111000","..0000021444444000.","..0000000111111000.",".0000600001111000..",".0000600002220000..",".0006600001227.....","0000060001122......","0000060000.22......","0100606100..22.....",".1111.1111..22....."]],"k":1,"fps":0,"seq":null},
 pokeball:{"pal":["484039","403830","ffe4d9","f04028","a7102e","000000","e1e1f0","888078","424033"],"f":[["...0001...","..122341..",".12233441.","0333344440","5334114445","5141675455","5685775575",".56655775.","..577775..","...5555..."]],"k":2,"fps":0,"seq":null}
};
const SDEF={chars:{},food:{},icons:{}};
function regLegacy(type,tbl,isObj){for(const k in tbl)SDEF[type][k]={frames:[isObj?tbl[k].spr:tbl[k]],pal:ROLE_T,ol:true,fps:0,seq:null,k:1};}
regLegacy('chars',CHARS,true);regLegacy('food',FOOD);regLegacy('icons',ICONS);
function regPix(type,kind,name){
  const p=PIX[name],pal={};
  p.pal.forEach((h,i)=>{pal[PIXCH[i]]=[parseInt(h.slice(0,2),16),parseInt(h.slice(2,4),16),parseInt(h.slice(4,6),16)];});
  SDEF[type][kind]={frames:p.f,pal,ol:false,fps:p.fps||0,seq:p.seq||null,k:p.k||1};
}
regPix('chars','plumber','mario');
regPix('chars','luigi','luigi');
regPix('chars','puffball','kirby');
regPix('chars','mount','yoshi');
regPix('chars','ape','kong');
regPix('chars','goomba','goomba');
regPix('chars','toad','toad');
regPix('chars','piranha','pirahna');
regPix('chars','boo','boo');
regPix('food','pizza','pizza');
regPix('food','soda','soda');
regPix('food','strawberry','strawberry');
regPix('food','watermelon','watermelon');
regPix('icons','coin','coin');
regPix('icons','block','block');
regPix('icons','pow','pow');
regPix('icons','mushroom','mushroom');
regPix('icons','flame','flame');
regPix('icons','evilflame','evilflame');
regPix('icons','flower','flower');
regPix('icons','pokeball','pokeball');
Object.assign(CHARSPEC,{luigi:'HOMINID PLUMBER (VERDANT)',goomba:'FUNGOID WALKER',toad:'SPORE-CAP ATTENDANT',
  piranha:'CARNIVOROUS BULB',boo:'SHY SPECTER'});
Object.assign(FOODSPEC,{soda:'ALUMINIUM FIZZ CAN',strawberry:'ACHENE-BEARING BERRY',watermelon:'STRIPED MELON WEDGE'});
function sprFields(type,kind){
  const d=SDEF[type][kind],cols={},tcols={};
  for(const ch of new Set(d.frames.flat().join('')))if(ch!=='.'&&d.pal[ch]){
    const c=d.pal[ch];cols[ch]=nearT(c[0],c[1],c[2]);tcols[ch]='rgb('+c[0]+','+c[1]+','+c[2]+')';
  }
  return{frames:d.frames,cols,tcols,ol:d.ol,fps:d.fps,seq:d.seq};
}

/* ---------------- ship / entity sprites ---------------- */
const SPRITES=[
 ['.......DD...','..D..DHHHD..','.DDHDHHHHHD.','EHHDWHHHAAHD','.DDHDHHHHHD.','..D..DHHHD..','.......DD...'],
 ['...DD.....','.DHWHHD...','EDHHHHAADD','.DHWHHD...','...DD.....'],
 ['......DDDD........','...DDHHHHHDDD.....','..DHHHHHHHHHDDDD..','.DHHWWHHHHHHHHHHD.','EDHHHHHHHHHHHHHHHD','.DHHWWHHHHHAAHHHD.','..DHHHHHHHHHDDDD..','...DDHHHHHDDD.....'],
 ['......DDDD..........','..DDDHHHHHDDD.......','.DHHHHHHHHHHHHHDDD..','EDHWWHHHHHHHHHHHHAAD','.DHHHHHHHHHHHHHDDD..'],
 ['..DD...','.DHHD..','EDHWAHD','..DD...'],
 ['....HHHH...','...HWWWWH..','..DHHHHHHD.','.DHHHHHHHD.','DHDHDHDHDHD'],
 ['....D..','..DDD..','.DWWWD.','..DDD..','..E....'],
 ['......DDDD........','...DDHHHHHHDD.....','..DHHHWWHHHHHDD...','.DHHHHHHHHHHHHHDD.','EDHHHHHHHHHHHHHD..']
];
const SHIPCLASS=['INTERCEPTOR','SURVEY COURIER','HEAVY FREIGHTER','LIQUID TANKER','AUTONOMOUS DRONE',
  'SCOUT SAUCER','STRIKE FIGHTER','CAPITAL SHIP'];
const UFO_SPR=[
 '...LLLL...',
 '..LLLLLL..',
 '.WWWWWWWW.',
 'GGRRGGRRGG',
 '.GG....GG.'];
const AST_SPR=[
 '..LLL..',
 '..LLL..',
 '.WWWWW.',
 'RWWWWWB',
 '.WWWWW.',
 '.W...W.',
 '.W...W.'];
const SAT_SPR=[
 'PPP..BB..PPP',
 'PPP.BBBB.PPP',
 'PPP..BB..PPP'];

/* ================================================================
   CHUNK SYSTEM
   ================================================================ */
const LAYERS=[
  {p:.30,ch:640,map:new Map(),tag:0x51},
  {p:.55,ch:480,map:new Map(),tag:0xB2},
  {p:1.0,ch:320,map:new Map(),tag:0xC3}
];
let genBudget=0,bodiesDrawn=0;
function getChunk(L,cx,cy){
  const k=cx+','+cy;
  let c=L.map.get(k);
  if(c){
    if(c.stamp!==STAMP){L.map.delete(k);c=null;}
    else return c;
  }
  if(genBudget<=0)return null;
  genBudget--;
  try{c=GEN[L.tag](cx,cy);}
  catch(err){fault('gen: '+err.message);c={stars:[],items:[]};}
  c.stamp=STAMP;
  L.map.set(k,c);
  return c;
}
function chunkRange(L){
  const z=zoom,camX=cam.x*L.p,camY=cam.y*L.p,ch=L.ch;
  return{
    x0:Math.floor((camX-lw/2/z)/ch)-1,
    x1:Math.floor((camX+lw/2/z)/ch)+1,
    y0:Math.floor((camY-lh/2/z)/ch)-1,
    y1:Math.floor((camY+lh/2/z)/ch)+1
  };
}
function warm(){
  genBudget=160;
  for(let li=LAYERS.length-1;li>=0;li--){
    const L=LAYERS[li],r=chunkRange(L);
    for(let cy=r.y0;cy<=r.y1;cy++)for(let cx=r.x0;cx<=r.x1;cx++)getChunk(L,cx,cy);
  }
  genBudget=0;
}
function eachChunk(L,cb){
  const z=zoom,camX=cam.x*L.p,camY=cam.y*L.p,r=chunkRange(L);
  for(let cy=r.y0;cy<=r.y1;cy++)for(let cx=r.x0;cx<=r.x1;cx++){
    const c=getChunk(L,cx,cy);
    if(c){
      try{cb(c,camX,camY,z);}
      catch(err){fault('draw: '+err.message);L.map.delete(cx+','+cy);}
    }
  }
}
function pruneChunks(){
  for(const L of LAYERS)if(L.map.size>520){
    let kill=L.map.size-480;
    for(const k of L.map.keys()){if(kill<=0)break;L.map.delete(k);kill--;}
  }
}

/* ---------------- layer generators ---------------- */
function genL0(cx,cy){
  const R=mulberry(ihash(cx,cy,seed^0xA1));
  const items=[],stars=[];
  const px=cx*640+320,py=cy*640+320;
  const bi=bandFactor(px,py);
  const fam=chunkFam(cx,cy,0x11D);
  let n=(26*S.stars*(0.5+R()*(0.4+bi)))|0;
  for(let i=0;i<n;i++)stars.push({x:cx*640+R()*640,y:cy*640+R()*640,
    ci:P.hot[P.hot.length-1],a:.25+R()*.3,sz:1,ph:0,sp:0,tw:0,g:0});
  if(R()<.30*S.deep&&ON.galaxy){
    const g={t:'galaxy',x:cx*640+80+R()*480,y:cy*640+80+R()*480,
      kind:R()<.7?'sp':(R()<.8?'el':'irr'),size:26+R()*46,rot:R()*TAU,
      dots:[],coreC:P.top[0],rs:(R()*1e9)|0,name:cat(R,GAL)};
    const arms=2+((R()<.4)?1:0),S0=g.size;
    const dimC=R()<.3?famCol(R,fam,1):pick(R,P.lo.concat(P.hi));
    for(let a=0;a<arms;a++){
      for(let k=0;k<80;k++){
        const th=(k/80)*2.2*Math.PI,rr=S0*(.1+.9*Math.pow(k/80,.8));
        const ang=th+a*TAU/arms+(R()-.5)*.5,ja=(R()-.5)*S0*.1;
        g.dots.push({dx:Math.cos(ang)*rr+ja,dy:Math.sin(ang)*rr+ja,
          a:.55-.4*(k/80),ci:R()<.85?dimC:g.coreC});
      }
    }
    for(let k=0;k<40;k++){
      const ang=R()*TAU,rr=Math.pow(R(),.6)*S0*.16;
      g.dots.push({dx:Math.cos(ang)*rr,dy:Math.sin(ang)*rr,a:.8,ci:g.coreC});
    }
    if(g.kind==='el'){
      g.dots.length=0;
      for(let k=0;k<140;k++){
        const ang=R()*TAU,rr=Math.pow(R(),.55)*S0*.8;
        g.dots.push({dx:Math.cos(ang)*rr,dy:Math.sin(ang)*rr*.55,
          a:.6-.4*(rr/(S0*.8)),ci:R()<.9?dimC:g.coreC});
      }
    }
    g.dots.sort((a,b)=>a.ci-b.ci);
    items.push(g);
  }
  return{stamp:STAMP,stars,items};
}

function genL1(cx,cy){
  const R=mulberry(ihash(cx,cy,seed^0xB2));
  const items=[],stars=[];
  const CH=480,px=cx*CH+CH/2,py=cy*CH+CH/2;
  const bi=bandFactor(px,py);
  const dn=fbm(cx*.13,cy*.13,seed^0x77);
  const fam=chunkFam(cx,cy,0xBEEF);

  let n=Math.min(420,(46*(.3+dn*1.7)*(1+bi*2.6)*S.stars)|0);
  for(let i=0;i<n;i++){
    const bright=R();
    let ci;
    if(bright>.72)ci=P.hot[P.hot.length-1];
    else if(fam>=0&&R()<.12)ci=famCol(R,fam,0);
    else ci=pick(R,P.hot);
    const sz=R()<bi*.4?2:(R()<.82?1:2);
    stars.push({x:cx*CH+R()*CH,y:cy*CH+R()*CH,ci,a:.45+R()*.55,sz,
      ph:R()*TAU,sp:.5+R()*2.5,tw:R()<.55?1:0,g:(sz===2&&R()<.3)?1:0});
  }
  stars.sort((a,b)=>a.ci-b.ci);

  if(bi>.12){
    const cnt=1+((R()*3)|0);
    for(let i=0;i<cnt;i++){
      const t=(px+py)*.5+(R()-.5)*CH;
      items.push({t:'haze',x:bandC*t+(R()-.5)*BANDW*bi,y:bandS*t+(R()-.5)*BANDW*bi,
        r:170+R()*230,ci:(fam>=0&&R()<.6)?famCol(R,fam,2):(R()<.7?1:2),a:.045+R()*.03});
    }
  }
  const nebP=(.34+bi*.4)*S.neb;
  if(ON.emission&&R()<nebP){
    const r=70+R()*150;
    const ciA=famCol(R,fam,0),ciB=famCol(R,fam,1);
    const nb=[];
    const cnt=26+((R()*22)|0);
    for(let i=0;i<cnt;i++){
      const gx=(R()+R()+R()-1.5)/1.5,gy=(R()+R()+R()-1.5)/1.5;
      nb.push({dx:gx*r*.6,dy:gy*r*.55,r:r*(.18+R()*.4),a:.05+R()*.07,ci:R()<.7?ciA:ciB});
    }
    const knots=[];
    for(let i=0;i<3+((R()*4)|0);i++)
      knots.push({dx:(R()-.5)*r*.7,dy:(R()-.5)*r*.6,r:r*(.04+R()*.06),a:.3+R()*.25,
        ci:R()<.6?ciA:P.top[0]});
    nb.sort((a,b)=>a.ci-b.ci);
    items.push({t:'neb',kind:'em',x:cx*CH+R()*CH,y:cy*CH+R()*CH,r,blobs:nb,knots,
      rs:(R()*1e9)|0,name:cat(R,NBUL)});
  }
  if(ON.snr&&R()<.09*S.deep){
    const r=26+R()*40,ciA=famCol(R,fam,0),ciB=famCol(R,fam,2);
    const nb=[],fil=[];
    for(let i=0;i<26;i++){
      const a=i/26*TAU+(R()-.5)*.3,rad=r*(.8+R()*.3);
      nb.push({dx:Math.cos(a)*rad,dy:Math.sin(a)*rad,r:4+R()*9,a:.06+R()*.06,ci:R()<.6?ciA:ciB});
    }
    for(let i=0;i<5;i++){
      const a1=R()*TAU,a2=a1+.6+R()*1.6;
      fil.push([Math.cos(a1)*r,Math.sin(a1)*r,Math.cos(a2)*r,Math.sin(a2)*r]);
    }
    items.push({t:'neb',kind:'snr',x:cx*CH+R()*CH,y:cy*CH+R()*CH,r,blobs:nb,fil,
      ciB,rs:(R()*1e9)|0,name:cat(R,NBUL)});
  }
  if(ON.pn&&R()<.07*S.deep){
    const r=9+R()*14;
    items.push({t:'neb',kind:'pn',x:cx*CH+R()*CH,y:cy*CH+R()*CH,r,
      ciA:famCol(R,fam,1),ciB:famCol(R,fam,0),rs:(R()*1e9)|0,name:cat(R,NBUL)});
  }
  if(ON.dark&&R()<(bi>.2?.5:.08)*S.neb){
    const r=60+R()*140,db=[];
    for(let i=0;i<10+((R()*7)|0);i++)
      db.push({dx:(R()-.5)*r,dy:(R()-.5)*r*.7,r:r*(.15+R()*.3),a:.4+R()*.25});
    items.push({t:'dark',x:cx*CH+R()*CH,y:cy*CH+R()*CH,r,blobs:db});
  }
  return{stamp:STAMP,stars,items};
}

function genL2(cx,cy){
  const R=mulberry(ihash(cx,cy,seed^0xC3));
  const items=[],CH=320;
  const x=()=>cx*CH+R()*CH,y=()=>cy*CH+R()*CH;
  const fam=chunkFam(cx,cy,0xC0F);

  /* suns & binaries */
  const nSun=poisson(R,.5*((ON.suns||ON.binary)?1:0));
  for(let i=0;i<nSun;i++){
    if(!ON.suns&&!ON.binary)break;
    const sf=R()<.28?fam:-1;
    const mk=()=>({r:3.5+R()*9,
      c1:famCol(R,sf,2),c2:famCol(R,sf,0),c3:famCol(R,sf,0),c4:P.top[0],
      ph:R()*TAU,sp:.5+R()*1.5,nrays:5+((R()*5)|0),
      rayl:Array.from({length:10},()=>R()),
      rsp:(R()-.5)*.4,rs:(R()*1e9)|0,name:cat(R,NSUN)});
    if(ON.binary&&R()<.22){
      const s1=mk(),s2=mk();
      items.push({t:'binary',x:x(),y:y(),orb:9+R()*10,hitR:26,w:.15+R()*.35,ph:R()*TAU,
        s1,s2,rs:(R()*1e9)|0,name:s1.name+' + '+s2.name});
    }else{
      const s=mk();s.t='sun';s.x=x();s.y=y();items.push(s);
    }
  }
  /* planets (incl. earths & ocean worlds) */
  const nPla=poisson(R,1.15*S.planets);
  for(let i=0;i<nPla;i++){
    if(!ON.planets&&!ON.gas)continue;
    const kindPool=['rocky','rocky','rocky','gas','gas','ice','lava','terra','terra'];
    if(ON.earth)kindPool.push('earth','earth');
    if(ON.ocean)kindPool.push('ocean','ocean','ocean');
    let kind=pick(R,kindPool);
    if(!ON.planets&&kind!=='gas')continue;
    if(!ON.gas&&kind==='gas')kind='rocky';
    if(!ON.earth&&kind==='earth')kind='terra';
    if(!ON.ocean&&kind==='ocean')kind='terra';
    const r=kind==='gas'?6+R()*8:2.5+R()*5.5;
    const pf=R()<.42?fam:-1;
    const cols={};
    if(kind==='gas')cols.bands=[famCol(R,pf,1),famCol(R,pf,2),famCol(R,pf,0)];
    else if(kind==='ice'){cols.base=famCol(R,pf,1);cols.crack=famCol(R,pf,0);}
    else if(kind==='lava'){cols.base=famCol(R,pf,2);cols.crack=famCol(R,pf,0);}
    else if(kind==='terra'){cols.base=famCol(R,pf,1);cols.patch=famCol(R,pf,2);cols.polar=P.top[0];}
    else if(kind==='earth'){
      cols.base=nearT(38,84,190);cols.land=nearT(96,168,84);
      cols.land2=nearT(70,140,60);cols.polar=P.ROLE.W;
    }
    else if(kind==='ocean'){
      cols.base=nearT(28,64,170);cols.lite=nearT(96,150,235);cols.crest=P.ROLE.W;
    }
    else{cols.base=famCol(R,pf,1);cols.dark=famCol(R,pf,2);cols.lite=famCol(R,pf,0);}
    const o={t:'planet',kind,x:x(),y:y(),r,cols,dir:R()*TAU,rs:(R()*1e9)|0};
    if(kind==='gas'){
      const bands=[];let yy=-1;const nb=3+((R()*4)|0);
      for(let b=0;b<nb&&yy<1;b++){
        const h=(0.25+R()*.5)*(2/nb)*1.7;
        bands.push({y:yy,h,ci:R()<.75?b%2:2});yy+=h*(.75+R()*.5);
      }
      o.bands=bands;
      if(R()<.5)o.storm={y:(R()-.5)*1.2,rx:.1+R()*.1,ry:.05+R()*.05,
        sp:.1+R()*.2,ph:R()*TAU,c:cols.bands[2]};
    }
    if(kind==='rocky')
      o.craters=Array.from({length:2+((r/2)|0)},()=>({dx:(R()-.5)*1.3,dy:(R()-.5)*1.3,r:.06+R()*.13}));
    if(kind==='ice'||kind==='lava'){
      o.cracks=Array.from({length:3+((R()*3)|0)},()=>{const a=R()*TAU,l=.4+R()*.7;
        return{x1:-Math.cos(a)*l,y1:-Math.sin(a)*l,x2:Math.cos(a)*l,y2:Math.sin(a)*l};});
      if(kind==='lava')o.spots=Array.from({length:2+((R()*3)|0)},()=>({dx:(R()-.5),dy:(R()-.5),ph:R()*TAU}));
    }
    if(kind==='terra'||kind==='earth'){
      o.patches=Array.from({length:3+((R()*4)|0)},()=>({dx:(R()-.5)*1.2,dy:(R()-.5)*1.2,r:.14+R()*.22}));
      o.clouds=Array.from({length:2+((R()*3)|0)},()=>({dy:(R()-.5)*1.3,ph:R()*TAU,r:.15+R()*.15}));
    }
    if(kind==='ocean'){
      o.bandsOcean=Array.from({length:4+((R()*3)|0)},()=>({y:(R()-.5)*1.6,h:.12+R()*.15}));
      o.oceanSp=.3+R()*.5;
      o.crests=Array.from({length:2+((R()*3)|0)},()=>({dx:(R()-.5),dy:(R()-.5),ph:R()*TAU}));
    }
    o.ring=ON.ringed&&R()<.32;
    if(o.ring){
      o.ringRot=(R()-.5)*.7;o.ringSq=.25+R()*.15;o.ringRx=1.7+R()*.5;
      o.rings=Array.from({length:2+((R()*2)|0)},(_,k)=>({d:1.15+k*.28,w:.09,a:.4+R()*.3,
        c:famCol(R,pf,1)}));
    }
    if(ON.moons&&r>5){
      const nm=Math.min(3,poisson(R,.7));
      o.moons=Array.from({length:nm},(_,k)=>({d:1.7+k*.65,sp:.25+R()*.5,ph:R()*TAU,
        r:.8+R()*1.4,ci:famCol(R,pf,2)}));
    }
    o.atmo=R()<.5;
    items.push(o);
  }
  /* shattered worlds */
  if(ON.shattered&&R()<.05*S.planets){
    const sf=R()<.6?fam:-1;
    const cracks=[];const nc=3+((R()*2)|0);
    for(let k=0;k<nc;k++){
      const a=R()*TAU,l=.7+R()*.5;
      const mx=Math.cos(a+1.2)*l*.6,my=Math.sin(a+1.2)*l*.6;
      cracks.push([-Math.cos(a)*l,-Math.sin(a)*l,mx,my,Math.cos(a)*l,Math.sin(a)*l]);
    }
    const frags=[];const nf=3+((R()*3)|0);
    for(let k=0;k<nf;k++){
      const n=4+((R()*3)|0),pts=[];
      for(let q=0;q<n;q++)pts.push([Math.cos(q/n*TAU)*(.6+R()*.5),Math.sin(q/n*TAU)*(.6+R()*.5)]);
      frags.push({a0:R()*TAU,w:.1+R()*.3,d:1.6+R()*.9,r:.25+R()*.2,
        rot0:R()*TAU,rsp:(R()-.5),c:famCol(R,sf,1),pts});
    }
    items.push({t:'shattered',x:x(),y:y(),r:4+R()*6,hitR:14,rs:(R()*1e9)|0,
      cBase:famCol(R,sf,2),cCrack:famCol(R,sf,0),cracks,frags,name:cat(R,NPLAN)});
  }
  /* hypercanes */
  if(ON.storm&&R()<.045*S.neb)
    items.push({t:'storm',x:x(),y:y(),R:11+R()*13,hitR:17,
      arms:2+((R()*2)|0),w:.5+R()*.9,ph:R()*TAU,rs:(R()*1e9)|0,
      cHi:P.ROLE.W,cMid:nearT(90,140,220),cGlow:nearT(40,80,160),cEye:nearT(12,22,44),
      dotsPer:11});
  /* satellites */
  if(ON.sat&&R()<.07*S.traffic)
    items.push({t:'sat',x:x(),y:y(),ph:R()*TAU,hitR:9,rs:(R()*1e9)|0,
      cPan:nearT(70,120,210),cBody:nearT(160,165,175)});
  /* battle moons */
  if(ON.battle&&R()<.02*S.deep)
    items.push({t:'battle',x:x(),y:y(),r:9+R()*6,hitR:14,rs:(R()*1e9)|0,
      c1:nearT(150,155,165),c2:nearT(95,100,112),c3:nearT(38,42,50),cL:nearT(140,255,160),
      dots:Array.from({length:14},()=>[R()*1.6-.8,R()*1.6-.8])});
  /* mega stations */
  if(ON.mega&&R()<.03*S.traffic)
    items.push({t:'mega',x:x(),y:y(),r:20+R()*12,hitR:30,ph:R()*TAU,rs:(R()*1e9)|0,
      arms:3+((R()*3)|0),cDim:pick(R,P.lo),cMid:pick(R,P.hi),cHi:pick(R,P.hi),cHot:P.top[0]});
  /* ringworlds & orbitals */
  if(ON.ring&&R()<.025*S.deep){
    const fHi=famCol(R,fam,0),fMid=famCol(R,fam,1);
    const kind=R()<.55?'ring':'orb';
    const it={t:'ring',kind,x:x(),y:y(),R:45+R()*45,hitR:60,tilt:(R()-.5)*.9,
      cHi:fHi,cMid:fMid,cGlow:famCol(R,fam,2),cDark:P.deep[0],
      rs:(R()*1e9)|0,name:cat(R,GAL)};
    if(kind==='ring'){
      const ns=5+((R()*4)|0);
      it.squares=Array.from({length:ns},(_,k)=>Math.PI+.4+(k/(ns-1))*(TAU-.8));
      it.sun={r:5+R()*4,c1:famCol(R,fam,2),c2:fHi,c3:fHi,c4:P.top[0],
        ph:R()*TAU,sp:1,nrays:6,rayl:[.4,.4,.4,.4,.4,.4,.4,.4,.4,.4],rsp:0};
    }else{
      it.stripes=Array.from({length:8+((R()*8)|0)},()=>({x:-1+R()*2,w:.03+R()*.06,
        c:pick(R,[fMid,fHi,P.top[0],P.deep[0]])}));
    }
    items.push(it);
  }
  /* dyson swarms */
  if(ON.dyson&&R()<.02*S.deep){
    const fHi=famCol(R,fam,0);
    items.push({t:'dyson',x:x(),y:y(),r:14+R()*10,hitR:22,rs:(R()*1e9)|0,
      rsp:.04+R()*.05,name:'DS-'+((1000+R()*8999)|0),
      panels:Array.from({length:46},()=>({a:R()*TAU,w:.05+R()*.1,
        cl:pick(R,P.lo.concat(P.hi)),cd:P.deep[0]})),
      sun:{r:5+R()*3.5,c1:famCol(R,fam,2),c2:fHi,c3:fHi,c4:P.top[0],
        ph:R()*TAU,sp:1.2,nrays:6,rayl:[.4,.4,.4,.4,.4,.4,.4,.4,.4,.4],rsp:0}});
  }
  /* reality tears */
  if(ON.rift&&R()<.03*S.deep){
    const pts=[[0,0]];let ang=R()*TAU,px=0,py=0;
    const n=6+((R()*4)|0);
    for(let i=0;i<n;i++){
      const l=6+R()*10;
      px+=Math.cos(ang)*l;py+=Math.sin(ang)*l;
      pts.push([px,py]);
      ang+=(R()<.5?-1:1)*(0.4+R()*.7);
    }
    items.push({t:'rift',x:x(),y:y(),pts,hitR:30,rs:(R()*1e9)|0,
      cHi:famCol(R,fam,0),cMid:famCol(R,fam,1),cGlow:famCol(R,fam,2)});
  }
  /* deep space probes */
  if(ON.probe&&R()<.05*S.traffic)
    items.push({t:'probe',x:x(),y:y(),ph:R()*TAU,hitR:8,rs:(R()*1e9)|0,
      cBody:pick(R,P.hi),cHot:P.top[0]});
  /* fun bodies (characters) */
  if(ON.chars&&R()<.03*S.events){
    const kind=pick(R,Object.keys(SDEF.chars));
    items.push({t:'char',kind,x:x(),y:y(),hitR:14,ph:R()*TAU,rs:(R()*1e9)|0,
      ...sprFields('chars',kind),scale:((kind==='invader'?2:1)+((R()*2)|0))*SDEF.chars[kind].k});
  }
  /* space snacks */
  if(ON.food&&R()<.035*S.events){
    const kind=pick(R,Object.keys(SDEF.food));
    items.push({t:'food',kind,x:x(),y:y(),hitR:12,ph:R()*TAU,rs:(R()*1e9)|0,
      ...sprFields('food',kind),scale:(1+((R()*2)|0))*SDEF.food[kind].k});
  }
  /* neon signs — large, rare, family-colored */
  if(ON.sign&&R()<.018*S.events){
    const Rs=mulberry(ihash(cx,cy,seed^0x5167));
    const spec=pick(Rs,SIGNS),lines=spec.split('|');
    const h1=pick(Rs,NEONS),h2=Rs()<.5?pick(Rs,NEONS):h1;
    const mk=c=>'rgb('+c[0]+','+c[1]+','+c[2]+')',dm=c=>c.map(v=>(v*.35)|0);
    const css={on:mk(h1.on),dim:mk(dm(h1.on)),fr:mk(h1.fr),on2:mk(h2.on),dim2:mk(dm(h2.on))};
    const idx={on:nearT(...h1.on),dim:nearT(...dm(h1.on)),fr:nearT(...h1.fr),on2:nearT(...h2.on),dim2:nearT(...dm(h2.on))};
    const lineCol=lines.map((_,i)=>i&&Rs()<.6?1:0);
    const letters=[];
    const lineL=lines.map(l=>l.split('').map(ch=>{
      if(ch===' ')return{ch};
      const mr=Rs(),L={ch,mode:mr<.55?'on':mr<.73?'blink':mr<.91?'flicker':'dead',ph:Rs()*TAU,sp:.6+Rs()*2.2};
      letters.push(L);return L;}));
    const fr=Rs()<.15?0:1+((Rs()*3)|0),anim=Rs()<.18?'blink':(Rs()<.18?'chase':'normal');
    const wMax=Math.max.apply(null,lines.map(l=>l.length));
    items.push({t:'sign',word:spec,disp:(SIGNDISP[spec]||spec).replace(/\|/g,' / '),lines,lineL,letters,
      x:x(),y:y(),hitR:18+wMax*6+(lines.length-1)*8,ph:R()*TAU,rs:(R()*1e9)|0,css,idx,lineCol,fr,anim});
  }
  /* rare heavyweights */
  if(ON.pulsar&&R()<.05*S.deep)
    items.push({t:'pulsar',x:x(),y:y(),w:.6+R()*1.8,L:18+R()*16,hitR:24,ph:R()*TAU,
      rs:(R()*1e9)|0});
  if(ON.bhole&&R()<.045*S.deep)
    items.push({t:'bhole',x:x(),y:y(),r:6+R()*7,rs:(R()*1e9)|0,
      pts:Array.from({length:70},()=>({a:R()*TAU,w:.4+R()*1.4,d:.55+R()*.55}))});
  if(ON.station&&R()<.05*S.traffic)
    items.push({t:'station',x:x(),y:y(),r:7+R()*6,rs:(R()*1e9)|0});
  if(ON.aster&&R()<.10*S.small){
    const rocks=[],rx=40+R()*60;
    const cnt=14+((R()*24)|0);
    for(let i=0;i<cnt;i++){
      const n=4+((R()*4)|0),pts=[];
      for(let k=0;k<n;k++)pts.push([Math.cos(k/n*TAU)*(.6+R()*.5),Math.sin(k/n*TAU)*(.6+R()*.5)]);
      rocks.push({dx:(R()-.5)*rx*2,dy:(R()-.5)*rx*1.2,r:.8+R()*3,
        rot0:R()*TAU,rsp:(R()-.5)*1.2,c:pick(R,P.lo.concat(P.hi)),pts});
    }
    items.push({t:'belt',x:x(),y:y(),rx,hitR:rx,rocks,rs:(R()*1e9)|0});
  }
  if(ON.lone&&R()<.08*S.small){
    const pts=[];const n=6+((R()*3)|0);
    for(let k=0;k<n;k++)pts.push([Math.cos(k/n*TAU)*(.7+R()*.4),Math.sin(k/n*TAU)*(.7+R()*.4)]);
    items.push({t:'lone',x:x(),y:y(),r:3+R()*4,rot0:R()*TAU,rsp:(R()-.5)*.5,
      c:pick(R,P.lo.concat(P.hi)),pts,rs:(R()*1e9)|0});
  }
  {
    const R4=mulberry(ihash(cx,cy,seed^0x4E11));
    const X4=()=>cx*CH+R4()*CH,Y4=()=>cy*CH+R4()*CH;
    if(ON.worm&&R4()<.014*S.deep)
      items.push({t:'worm',x:X4(),y:Y4(),r:9+R4()*9,hitR:24,rs:(R4()*1e9)|0,dir:R4()<.5?1:-1,arms:3+((R4()*3)|0),
        sp:.5+R4()*.9,cA:famCol(R4,fam,0),cB:famCol(R4,fam,1),cC:famCol(R4,fam,2)});
    if(ON.jelly&&R4()<.03*S.deep)
      items.push({t:'jelly',x:X4(),y:Y4(),r:5+R4()*7,hitR:16,rs:(R4()*1e9)|0,n:4+((R4()*4)|0),sp:.6+R4()*.9,ph:R4()*TAU,
        cA:famCol(R4,fam,1),cB:famCol(R4,fam,0),cC:famCol(R4,fam,2)});
    if(ON.beacon&&R4()<.02*S.small){
      const pts=[];for(let k=0;k<7;k++)pts.push([Math.cos(k/7*TAU)*(.75+R4()*.35),Math.sin(k/7*TAU)*(.75+R4()*.35)]);
      items.push({t:'beacon',x:X4(),y:Y4(),r:7+R4()*5,hitR:20,rs:(R4()*1e9)|0,pts,w:.5+R4()*.8,ph:R4()*TAU,L:34+R4()*22,
        cRock:pick(R4,P.lo.concat(P.hi)),cBeam:pick(R4,P.hot)});
    }
  }
  {
    const R5=mulberry(ihash(cx,cy,seed^0x70C5));
    const X5=()=>cx*CH+R5()*CH,Y5=()=>cy*CH+R5()*CH;
    const mk=(t,k,x_,y_,extra)=>{const sp=IMGS[k];
      return Object.assign({t,img:k,x:x_,y:y_,scale:1,hue:0,rs:(R5()*1e9)|0,ph:R5()*TAU,hitR:Math.max(sp.w,sp.h)/2+3},extra);};
    const hueQ=()=>R5()<.35?(1+((R5()*5)|0))*60:0;
    /* small drifting rocks (always slowly spinning) */
    if(ON.rock&&R5()<.09*S.small)
      items.push(mk('rock',R5()<.06?'big186':pick(R5,ROCKKEYS),X5(),Y5(),{rk:'rock',hue:hueQ()}));
    /* single pieces of junk (tumbling) */
    if(ON.junk&&R5()<.05*S.small)
      items.push(mk('junk',pick(R5,JUNKKEYS),X5(),Y5(),{rk:'junk',hue:R5()<.15?(1+((R5()*5)|0))*60:0}));
    /* rare debris field: a loose cluster that slowly turns as a whole */
    if(ON.field&&R5()<.012*S.small){
      const n=5+((R5()*6)|0),spread=18+R5()*22,parts=[];
      for(let i=0;i<n;i++){
        const k=R5()<.7?pick(R5,ROCKKEYS):pick(R5,JUNKKEYS),a=R5()*TAU,d=Math.sqrt(R5())*spread;
        const p=mk('part',k,0,0,{rk:k[0]==='a'&&JUNKKEYS.indexOf(k)<0?'rock':'junk',hue:hueQ()});
        p.dx=Math.cos(a)*d;p.dy=Math.sin(a)*d*.7;parts.push(p);
      }
      items.push({t:'field',x:X5(),y:Y5(),hitR:spread+12,rs:(R5()*1e9)|0,w:(R5()-.5)*.06,ph:R5()*TAU,parts});
    }
    /* anomalies: crates, monoliths, crystals, eggs, mines... weighted by category */
    if(ON.xtra&&R5()<.045*S.events){
      const cat=pickW(R5,XCAT),k=pick(R5,XCAT[cat].keys),sp=IMGS[k];
      items.push(mk('xtra',k,X5(),Y5(),{cat,glow:XCAT[cat].glow&&R5()<.8,scale:(Math.max(sp.w,sp.h)<=22&&R5()<.35)?2:1}));
    }
  }
  if(ON.icon){
    const R2=mulberry(ihash(cx,cy,seed^0x1C0));
    if(R2()<.04*S.events){
      const kind=pick(R2,Object.keys(SDEF.icons));
      items.push({t:'icon',kind,x:cx*CH+R2()*CH,y:cy*CH+R2()*CH,hitR:11,ph:R2()*TAU,
        rs:(R2()*1e9)|0,...sprFields('icons',kind),scale:(1+((R2()*2)|0))*SDEF.icons[kind].k});
    }
  }
  items.forEach(it=>{if(!it.name&&it.t!=='rift'&&it.t!=='probe'&&it.t!=='char'&&
    it.t!=='dyson'&&it.t!=='battle'&&it.t!=='mega'&&it.t!=='storm'&&it.t!=='sat'&&
    it.t!=='food'&&it.t!=='sign'&&it.t!=='worm'&&it.t!=='jelly'&&it.t!=='beacon'&&it.t!=='rock'&&it.t!=='junk'&&it.t!=='xtra'&&it.t!=='field')it.name=cat(R,NPLAN);});
  return{stamp:STAMP,stars:[],items};
}
const GEN={0x51:genL0,0xB2:genL1,0xC3:genL2};

/* ================================================================
   DRAWING
   ================================================================ */
function circ(x,y,r){lctx.beginPath();lctx.arc(x,y,r,0,TAU);lctx.fill();}

function drawSun(o,sx,sy,s){
  const r=Math.max(1.6,o.r*s*(1+.05*Math.sin(wt*o.sp+o.ph))),g=S.glow;
  lctx.fillStyle=CS(o.c1);
  if(g>0){
    lctx.globalAlpha=.05*g;circ(sx,sy,r*2.7);
    lctx.globalAlpha=.09*g;circ(sx,sy,r*1.95);
    lctx.globalAlpha=.15*g;circ(sx,sy,r*1.45);
  }
  lctx.fillStyle=CS(o.c2);lctx.globalAlpha=Math.min(1,.2+g*.12);
  for(let k=0;k<o.nrays;k++){
    const a=o.ph+k*TAU/o.nrays+wt*o.rsp,len=r*(1.9+o.rayl[k]);
    lctx.beginPath();lctx.moveTo(sx,sy);
    lctx.lineTo(sx+Math.cos(a-.06)*len,sy+Math.sin(a-.06)*len);
    lctx.lineTo(sx+Math.cos(a+.06)*len,sy+Math.sin(a+.06)*len);
    lctx.closePath();lctx.fill();
  }
  lctx.globalAlpha=1;
  lctx.fillStyle=CS(o.c2);circ(sx,sy,r);
  lctx.fillStyle=CS(o.c3);circ(sx-r*.15,sy-r*.15,r*.72);
  lctx.fillStyle=CS(o.c4);circ(sx-r*.3,sy-r*.3,r*.34);
}

function drawRingHalf(sx,sy,r,o,back){
  const B=o.ringRx*r*2;
  lctx.save();lctx.translate(sx,sy);lctx.rotate(o.ringRot);
  lctx.beginPath();
  if(back)lctx.rect(-B,-B,2*B,B);else lctx.rect(-B,0,2*B,B);
  lctx.clip();
  for(const rg of o.rings){
    lctx.strokeStyle=CS(rg.c);lctx.globalAlpha=rg.a;
    lctx.lineWidth=Math.max(1,rg.w*r);
    lctx.beginPath();lctx.ellipse(0,0,rg.d*r*o.ringRx,rg.d*r*o.ringSq*o.ringRx,0,0,TAU);lctx.stroke();
  }
  lctx.restore();lctx.globalAlpha=1;
}

function drawPlanet(o,sx,sy,s,sunPts){
  const r=Math.max(2,o.r*s),c=o.cols;
  if(r<2.2){
    lctx.globalAlpha=1;
    lctx.fillStyle=CS(c.base!==undefined?c.base:(c.bands?c.bands[0]:P.lo[0]));
    lctx.fillRect(sx|0,sy|0,2,2);return;
  }
  let sd=o.dir;
  if(sunPts.length){
    let bd=1e18;
    for(const p of sunPts){const dx=p.x-sx,dy=p.y-sy,dd=dx*dx+dy*dy;if(dd<bd){bd=dd;sd=Math.atan2(-dy,-dx);}}
  }
  const ca=Math.cos(sd),sa=Math.sin(sd);
  if(o.ring)drawRingHalf(sx,sy,r,o,true);
  lctx.save();lctx.beginPath();lctx.arc(sx,sy,r,0,TAU);lctx.clip();
  const k=o.kind;
  if(k==='gas'){
    for(const b of o.bands){
      lctx.fillStyle=CS(c.bands[b.ci]);
      lctx.fillRect(sx-r-2,sy+b.y*r,2*r+4,Math.max(1,b.h*r));
    }
    if(o.storm){
      const stx=sx+Math.sin(wt*o.storm.sp+o.storm.ph)*r*.55;
      lctx.fillStyle=CS(o.storm.c);lctx.globalAlpha=.9;
      lctx.beginPath();lctx.ellipse(stx,sy+o.storm.y*r,o.storm.rx*r,o.storm.ry*r,0,0,TAU);lctx.fill();
      lctx.globalAlpha=1;
    }
  }else if(k==='terra'||k==='earth'){
    lctx.fillStyle=CS(c.base);lctx.fillRect(sx-r-2,sy-r-2,2*r+4,2*r+4);
    if(k==='earth'){
      /* blue marble: two-tone continents */
      for(const p of o.patches){
        lctx.fillStyle=CS(p.r>.2?c.land:c.land2);lctx.globalAlpha=.9;
        circ(sx+p.dx*r,sy+p.dy*r,p.r*r);
      }
      lctx.globalAlpha=.85;lctx.fillStyle=CS(c.polar);
      circ(sx,sy-r*.82,r*.38);circ(sx,sy+r*.85,r*.28);
    }else{
      lctx.fillStyle=CS(c.patch);lctx.globalAlpha=.85;
      for(const p of o.patches)circ(sx+p.dx*r,sy+p.dy*r,p.r*r);
      lctx.globalAlpha=.8;lctx.fillStyle=CS(c.polar);circ(sx,sy-r*.8,r*.42);
    }
    lctx.globalAlpha=.5;lctx.fillStyle=CS(c.polar);
    for(const cl of o.clouds){
      const cx2=sx+Math.sin(wt*.12+cl.ph)*r*.6;
      lctx.beginPath();
      lctx.ellipse(cx2,sy+cl.dy*r,cl.r*r*1.8,cl.r*r*.7,0,0,TAU);lctx.fill();
    }
    lctx.globalAlpha=1;
  }else if(k==='ocean'){
    /* fluid world: oscillating bright bands + glinting crests */
    lctx.fillStyle=CS(c.base);lctx.fillRect(sx-r-2,sy-r-2,2*r+4,2*r+4);
    for(let b=0;b<o.bandsOcean.length;b++){
      const bd=o.bandsOcean[b];
      const off=Math.sin(wt*o.oceanSp+b*1.7)*r*.18;
      lctx.fillStyle=CS(c.lite);lctx.globalAlpha=.45;
      lctx.beginPath();lctx.ellipse(sx+off,sy+bd.y*r,r*.8,bd.h*r,0,0,TAU);lctx.fill();
    }
    lctx.fillStyle=CS(c.crest);
    for(const cr of o.crests){
      const a=.3+.7*Math.abs(Math.sin(wt*.8+cr.ph));
      lctx.globalAlpha=a*.8;
      lctx.fillRect((sx+cr.dx*r)|0,(sy+cr.dy*r)|0,2,1);
    }
    lctx.globalAlpha=1;
  }else{
    lctx.fillStyle=CS(c.base);lctx.fillRect(sx-r-2,sy-r-2,2*r+4,2*r+4);
    if(k==='rocky'){
      for(const cr of o.craters){
        lctx.fillStyle=CS(c.dark);lctx.globalAlpha=.7;
        circ(sx+cr.dx*r,sy+cr.dy*r,cr.r*r);
        lctx.fillStyle=CS(c.lite);lctx.globalAlpha=.4;
        circ(sx+cr.dx*r-ca*cr.r*r*.4,sy+cr.dy*r-sa*cr.r*r*.4,cr.r*r*.35);
      }
      lctx.globalAlpha=1;
    }
    if(o.cracks){
      lctx.strokeStyle=CS(c.crack);lctx.lineWidth=1;
      lctx.globalAlpha=k==='ice'?.55:.8;
      lctx.beginPath();
      for(const cr of o.cracks){
        lctx.moveTo(sx+cr.x1*r,sy+cr.y1*r);lctx.lineTo(sx+cr.x2*r,sy+cr.y2*r);
      }
      lctx.stroke();
      if(o.spots){
        for(const sp of o.spots){
          lctx.globalAlpha=.35+.3*Math.sin(wt*2+sp.ph);
          circ(sx+sp.dx*r,sy+sp.dy*r,r*.14);
        }
      }
      lctx.globalAlpha=1;
    }
  }
  lctx.fillStyle=P.str[0];
  lctx.globalAlpha=.55;circ(sx+ca*r*.9,sy+sa*r*.9,r*1.05);
  lctx.globalAlpha=.3;circ(sx+ca*r*.5,sy+sa*r*.5,r*.9);
  planetExtras(o,sx,sy,r,ca,sa);
  lctx.globalAlpha=1;lctx.restore();
  if(o.atmo){
    lctx.strokeStyle=CS(P.top[0]);
    lctx.globalAlpha=.5;lctx.lineWidth=Math.max(1,r*.12);
    lctx.beginPath();lctx.arc(sx,sy,r*.96,sd+Math.PI-1.9,sd+Math.PI+1.9);lctx.stroke();
    lctx.globalAlpha=1;
  }
  if(o.ring)drawRingHalf(sx,sy,r,o,false);
  if(o.moons)for(const m of o.moons){
    const a=wt*m.sp+m.ph,mx=sx+Math.cos(a)*m.d*r,my=sy+Math.sin(a)*m.d*r*.38;
    if(Math.sin(a)>-.55){
      lctx.fillStyle=CS(m.ci);circ(mx,my,Math.max(.8,m.r*s));
      lctx.fillStyle=P.str[0];lctx.globalAlpha=.45;
      circ(mx+ca*m.r*s*.6,my+sa*m.r*s*.6,Math.max(.6,m.r*s*.9));
      lctx.globalAlpha=1;
    }
  }
}

function drawBhole(o,sx,sy,s){
  const r=Math.max(3,o.r*s);
  const hotI=P.hot;
  const half=back=>{
    lctx.save();lctx.beginPath();
    lctx.rect(sx-2*r,back?-2*r:0,4*r,2*r);lctx.clip();
    lctx.strokeStyle=CS(P.hi[0]);lctx.globalAlpha=.22;lctx.lineWidth=1;
    lctx.beginPath();lctx.ellipse(sx,sy,r*1.15,r*.36,0,0,TAU);lctx.stroke();
    for(const p of o.pts){
      const a=p.a+wt*p.w,rad=p.d*r;
      const px=sx+Math.cos(a)*rad,py=sy+Math.sin(a)*rad*.32;
      const ci=rad<r*.7?hotI[hotI.length-1]:(rad<r*.95?hotI[Math.min(1,hotI.length-1)]:P.hi[0]);
      lctx.fillStyle=CS(ci);lctx.globalAlpha=.85;
      const sz=s>1?2:1;lctx.fillRect(px|0,py|0,sz,sz);
    }
    lctx.restore();lctx.globalAlpha=1;
  };
  half(true);
  lctx.fillStyle=P.str[0];circ(sx,sy,r*.52);
  lctx.strokeStyle=CS(hotI[hotI.length-1]);lctx.globalAlpha=.9;
  lctx.lineWidth=Math.max(1,r*.08);
  lctx.beginPath();lctx.arc(sx,sy,r*.6,0,TAU);lctx.stroke();
  lctx.globalAlpha=.5;
  lctx.beginPath();lctx.arc(sx,sy,r*.68,-Math.PI*.85,-Math.PI*.15);lctx.stroke();
  lctx.globalAlpha=1;
  half(false);
}

function drawPulsar(o,sx,sy,s){
  const a0=wt*o.w+o.ph,L=o.L*s;
  const hotI=P.hot,blink=.7+.3*Math.sin(wt*6+o.ph);
  lctx.fillStyle=CS(hotI[hotI.length-1]);
  for(let q=0;q<2;q++){
    const a=a0+q*Math.PI;
    lctx.globalAlpha=.26*blink;
    lctx.beginPath();lctx.moveTo(sx,sy);
    lctx.lineTo(sx+Math.cos(a-.15)*L,sy+Math.sin(a-.15)*L);
    lctx.lineTo(sx+Math.cos(a+.15)*L,sy+Math.sin(a+.15)*L);
    lctx.closePath();lctx.fill();
    lctx.globalAlpha=.45*blink;
    lctx.beginPath();lctx.moveTo(sx,sy);
    lctx.lineTo(sx+Math.cos(a-.06)*L*.65,sy+Math.sin(a-.06)*L*.65);
    lctx.lineTo(sx+Math.cos(a+.06)*L*.65,sy+Math.sin(a+.06)*L*.65);
    lctx.closePath();lctx.fill();
  }
  lctx.globalAlpha=.5*S.glow+.15;circ(sx,sy,3.5);
  lctx.globalAlpha=1;lctx.fillRect((sx|0)-1,(sy|0)-1,2,2);
}

function drawStation(o,sx,sy,s){
  const r=Math.max(4,o.r*s);
  lctx.save();lctx.translate(sx,sy);lctx.rotate(wt*.15);
  lctx.strokeStyle=CS(P.hi[0]);lctx.lineWidth=Math.max(1,r*.14);
  lctx.beginPath();lctx.arc(0,0,r,0,TAU);lctx.stroke();
  lctx.fillStyle=CS(P.lo[0]);
  lctx.fillRect(-r,-r*.09,2*r,r*.18);
  lctx.fillRect(-r*.09,-r,r*.18,2*r);
  lctx.fillStyle=CS(P.hi[Math.min(1,P.hi.length-1)]);
  lctx.fillRect(-r*.3,-r*.3,r*.6,r*.6);
  lctx.fillStyle=CS(P.hot[P.hot.length-1]);
  for(let k=0;k<8;k++){
    const a=k*TAU/8;
    if(Math.sin(wt*3+k)>0)
      lctx.fillRect(Math.cos(a)*r-1,Math.sin(a)*r-1,2,2);
  }
  lctx.restore();
}

function drawBattle(o,sx,sy,s){
  const r=Math.max(5,o.r*s);
  lctx.fillStyle=CS(o.c1);circ(sx,sy,r);
  lctx.save();lctx.beginPath();lctx.arc(sx,sy,r,0,TAU);lctx.clip();
  lctx.fillStyle=CS(o.c2);lctx.globalAlpha=.4;
  circ(sx-r*.3,sy-r*.25,r*.75);
  lctx.globalAlpha=1;
  lctx.fillStyle=CS(o.c2);circ(sx-r*.28,sy-r*.22,r*.34);
  lctx.fillStyle=CS(o.c3);circ(sx-r*.28,sy-r*.22,r*.18);
  lctx.fillStyle=CS(o.c3);lctx.fillRect(sx-r,sy-r*.07,r*2,Math.max(1,r*.14));
  lctx.fillStyle=CS(o.c2);lctx.globalAlpha=.5;
  for(const d of o.dots)lctx.fillRect((sx+d[0]*r)|0,(sy+d[1]*r)|0,1,1);
  lctx.globalAlpha=1;lctx.restore();
  lctx.fillStyle=P.str[0];
  lctx.globalAlpha=.4;circ(sx+r*.55,sy+r*.55,r*.95);
  lctx.globalAlpha=.2;circ(sx+r*.3,sy+r*.3,r*.8);
  lctx.globalAlpha=1;
  const b=.5+.5*Math.sin(rt*2.5+o.rs%10);
  lctx.fillStyle=CS(o.cL);lctx.globalAlpha=.4+.6*b;
  const lwid=Math.max(1,r*.1)|0;
  lctx.fillRect((sx-r*.28)|0,(sy-r*.22)|0,lwid,lwid);
  lctx.globalAlpha=1;
}

function drawShattered(o,sx,sy,s){
  const r=Math.max(3,o.r*s);
  lctx.fillStyle=CS(o.cBase);circ(sx,sy,r);
  lctx.fillStyle=P.str[0];lctx.globalAlpha=.4;circ(sx+r*.4,sy+r*.4,r*.9);
  lctx.globalAlpha=1;
  lctx.fillStyle=CS(o.cCrack);
  lctx.globalAlpha=.35+.2*Math.sin(rt*1.5+o.rs%10);
  circ(sx,sy,r*.5);
  lctx.globalAlpha=1;
  lctx.strokeStyle=CS(o.cCrack);lctx.lineWidth=Math.max(1,r*.08);
  lctx.beginPath();
  for(const cr of o.cracks){
    lctx.moveTo(sx+cr[0]*r,sy+cr[1]*r);
    lctx.lineTo(sx+cr[2]*r,sy+cr[3]*r);
    lctx.lineTo(sx+cr[4]*r,sy+cr[5]*r);
  }
  lctx.stroke();
  for(const f of o.frags){
    const a=f.a0+wt*f.w;
    const fx=sx+Math.cos(a)*f.d*r,fy=sy+Math.sin(a)*f.d*r*.5;
    lctx.save();lctx.translate(fx,fy);lctx.rotate(f.rot0+wt*f.rsp);
    lctx.fillStyle=CS(f.c);const rr=Math.max(1,f.r*s);
    lctx.beginPath();lctx.moveTo(f.pts[0][0]*rr,f.pts[0][1]*rr);
    for(let k=1;k<f.pts.length;k++)lctx.lineTo(f.pts[k][0]*rr,f.pts[k][1]*rr);
    lctx.closePath();lctx.fill();lctx.restore();
  }
}

function drawMega(o,sx,sy,s){
  const r=Math.max(10,o.r*s);
  lctx.save();lctx.translate(sx,sy);lctx.rotate(wt*.05+o.ph);
  lctx.strokeStyle=CS(o.cDim);lctx.lineWidth=Math.max(1,r*.06);
  lctx.globalAlpha=.7;
  lctx.beginPath();lctx.arc(0,0,r,0,TAU);lctx.stroke();
  lctx.globalAlpha=1;
  lctx.strokeStyle=CS(o.cMid);lctx.lineWidth=Math.max(1,r*.08);
  for(let k=0;k<o.arms;k++){
    const a=k*TAU/o.arms;
    lctx.beginPath();lctx.moveTo(0,0);lctx.lineTo(Math.cos(a)*r,Math.sin(a)*r);lctx.stroke();
    lctx.fillStyle=CS(o.cHi);
    circ(Math.cos(a)*r,Math.sin(a)*r,Math.max(1.5,r*.16));
  }
  lctx.fillStyle=CS(o.cHi);lctx.fillRect(-r*.22,-r*.22,r*.44,r*.44);
  lctx.fillStyle=CS(o.cHot);lctx.fillRect(-r*.08,-r*.08,r*.16,r*.16);
  lctx.fillStyle=CS(o.cHot);
  for(let k=0;k<o.arms;k++){
    if(Math.sin(rt*2.5+k*1.7)>.4){
      const a=k*TAU/o.arms;
      lctx.fillRect(Math.cos(a)*r-1,Math.sin(a)*r-1,2,2);
    }
  }
  lctx.restore();lctx.globalAlpha=1;
}

function drawRing(o,sx,sy,s){
  const R=Math.max(20,o.R*s);
  if(o.kind==='ring'){
    lctx.save();lctx.translate(sx,sy);lctx.rotate(o.tilt);
    lctx.strokeStyle=CS(o.cGlow);lctx.globalAlpha=.18*(S.glow*.75+.25);
    lctx.lineWidth=Math.max(3,R*.09);
    lctx.beginPath();lctx.ellipse(0,0,R,R*.26,0,0,TAU);lctx.stroke();
    lctx.strokeStyle=CS(o.cMid);lctx.globalAlpha=.9;lctx.lineWidth=Math.max(2,R*.055);
    lctx.beginPath();lctx.ellipse(0,0,R,R*.26,0,0,TAU);lctx.stroke();
    lctx.strokeStyle=CS(o.cHi);lctx.globalAlpha=.8;lctx.lineWidth=Math.max(1,R*.02);
    lctx.beginPath();lctx.ellipse(0,0,R,R*.26,0,0,TAU);lctx.stroke();
    lctx.fillStyle=CS(o.cDark);lctx.globalAlpha=.85;
    for(const t of o.squares){
      const px=Math.cos(t)*R,py=Math.sin(t)*R*.26;
      lctx.fillRect((px-2)|0,(py-1)|0,4,3);
    }
    lctx.restore();lctx.globalAlpha=1;
    drawSun(o.sun,sx,sy,s);
  }else{
    const H=Math.max(3,R*.08);
    lctx.save();lctx.translate(sx,sy);lctx.rotate(o.tilt);
    lctx.beginPath();lctx.rect(-R-2,-H-2,R*2+4,H*2+4);lctx.clip();
    lctx.fillStyle=CS(o.cMid);lctx.fillRect(-R,-H,R*2,H*2);
    for(const st of o.stripes){
      lctx.fillStyle=CS(st.c);
      lctx.fillRect(st.x*R,-H,st.w*R,H*2);
    }
    lctx.restore();
    lctx.fillStyle=CS(o.cHi);
    lctx.fillRect((sx+Math.cos(o.tilt)*R-1)|0,(sy+Math.sin(o.tilt)*R-H)|0,2,Math.max(2,H*2)|0);
    lctx.fillRect((sx-Math.cos(o.tilt)*R-1)|0,(sy-Math.sin(o.tilt)*R-H)|0,2,Math.max(2,H*2)|0);
    lctx.globalAlpha=1;
  }
}

function drawDyson(o,sx,sy,s){
  const R=Math.max(8,o.r*s),w=o.rsp;
  const half=front=>{
    for(const p of o.panels){
      const a=p.a+wt*w,yy=Math.sin(a);
      if(front?yy<=0:yy>0)continue;
      const px=sx+Math.cos(a)*R,py=sy+yy*R*.9;
      lctx.fillStyle=CS(front?p.cl:p.cd);
      lctx.globalAlpha=front?.9:.5;
      const pw=Math.max(2,(p.w*R)|0);
      lctx.fillRect((px-pw/2)|0,(py-1)|0,pw,2);
    }
    lctx.globalAlpha=1;
  };
  half(false);
  drawSun(o.sun,sx,sy,s);
  half(true);
}

function strokePts(pts,sx,sy,s){
  lctx.beginPath();
  lctx.moveTo(sx+pts[0][0]*s,sy+pts[0][1]*s);
  for(let k=1;k<pts.length;k++)lctx.lineTo(sx+pts[k][0]*s,sy+pts[k][1]*s);
  lctx.stroke();
}
function drawRift(o,sx,sy,s){
  const fl=.7+.3*Math.sin(rt*6+o.rs%10);
  lctx.strokeStyle=CS(o.cGlow);lctx.globalAlpha=.12*fl*(S.glow*.75+.25)+.04;
  lctx.lineWidth=5;strokePts(o.pts,sx,sy,s);
  lctx.strokeStyle=CS(o.cMid);lctx.globalAlpha=.3*fl;lctx.lineWidth=2.5;
  strokePts(o.pts,sx,sy,s);
  lctx.strokeStyle=CS(o.cHi);lctx.globalAlpha=.95*fl;lctx.lineWidth=1;
  strokePts(o.pts,sx,sy,s);
  lctx.fillStyle=CS(o.cHi);
  for(const p of o.pts)lctx.fillRect((sx+p[0]*s-1)|0,(sy+p[1]*s-1)|0,2,2);
  lctx.globalAlpha=1;
}

function drawProbe(o,sx,sy,s){
  const sc=Math.max(1,s*.9);
  const bob=Math.sin(wt*.8+o.ph)*1.5;
  const x=sx,y=sy+bob;
  lctx.strokeStyle=CS(o.cBody);lctx.lineWidth=1;
  lctx.fillStyle=CS(o.cBody);
  lctx.fillRect((x-sc)|0,(y-sc)|0,Math.max(2,2*sc)|0,Math.max(2,2*sc)|0);
  lctx.beginPath();lctx.moveTo(x+sc,y);lctx.lineTo(x+4*sc,y);lctx.stroke();
  lctx.beginPath();lctx.arc(x+4.5*sc,y,1.5*sc,-1.2,1.2);lctx.stroke();
  lctx.beginPath();lctx.moveTo(x,y-sc);lctx.lineTo(x,y-4*sc);lctx.stroke();
  if(Math.sin(rt*4+o.ph)>0){
    lctx.fillStyle=CS(o.cHot);
    lctx.fillRect((x-1)|0,(y-4*sc-2)|0,2,2);
  }
}

function drawStorm(o,sx,sy,s){
  const R=Math.max(6,o.R*s);
  lctx.fillStyle=CS(o.cGlow);
  lctx.globalAlpha=.08*(S.glow*.75+.25);circ(sx,sy,R*1.35);
  lctx.globalAlpha=1;
  for(let arm=0;arm<o.arms;arm++){
    for(let k=0;k<o.dotsPer;k++){
      const t=k/o.dotsPer;
      const a=arm*TAU/o.arms+t*3.6+wt*o.w+o.ph;
      const rr=R*(.25+.75*t);
      const px=sx+Math.cos(a)*rr,py=sy+Math.sin(a)*rr*.9;
      lctx.fillStyle=CS(k<o.dotsPer*.4?o.cHi:o.cMid);
      lctx.globalAlpha=(1-t*.5)*.9;
      const sz=rr>R*.6?2:1;
      lctx.fillRect(px|0,py|0,sz,sz);
    }
  }
  lctx.globalAlpha=1;
  lctx.fillStyle=CS(o.cEye);circ(sx,sy,R*.18);
  lctx.fillStyle=CS(o.cHi);circ(sx,sy,R*.05);
}

function drawSatLegacy(o,sx,sy,s){
  const sc=Math.max(1,Math.round(Math.max(1,s*1.2)));
  lctx.save();lctx.translate(sx,sy);lctx.rotate(Math.sin(wt*.12+o.ph)*.2);
  drawSpriteIdx(SAT_SPR,0,0,sc,{P:o.cPan,B:o.cBody},0);
  lctx.strokeStyle=CS(o.cBody);lctx.lineWidth=1;
  lctx.beginPath();lctx.moveTo(0,-1.5*sc);lctx.lineTo(0,-5*sc);lctx.stroke();
  if(Math.sin(rt*3+o.ph)>0){
    lctx.fillStyle=CS(P.ROLE.R);
    lctx.fillRect(-1,(-5*sc-2)|0,2,2);
  }
  lctx.restore();
}

/* ---- pixel characters / snacks / generic sprite draw ---- */
function drawSpriteIdx(spr,sx,sy,sc,cols,bob,tcols,ol){
  const w=spr[0].length,h=spr.length;
  const ox=Math.round(sx-w*sc/2),oy=Math.round(sy-h*sc/2+(bob||0));
  if(tcols&&ol){
    lctx.fillStyle='#0b0908';
    for(let r=0;r<h;r++)for(let c=0;c<spr[r].length;c++)
      if(spr[r][c]!=='.')lctx.fillRect(ox+c*sc-1,oy+r*sc-1,sc+2,sc+2);
  }
  for(let r=0;r<h;r++){
    const row=spr[r];
    for(let c=0;c<row.length;c++){
      const ch=row[c];
      if(ch==='.')continue;
      if(tcols){const t=tcols[ch];if(!t)continue;lctx.fillStyle=t;}
      else{const ci=cols[ch];if(ci===undefined)continue;lctx.fillStyle=CS(ci);}
      lctx.fillRect(ox+c*sc,oy+r*sc,sc,sc);
    }
  }
}
/* crisp sprites are queued and drawn AFTER the dither/palette pass */
const spriteQ=[];
const drawChar=(o,sx,sy,s)=>drawSpr(o,sx,sy,s,7,.06,.9);
const drawFood=(o,sx,sy,s)=>drawSpr(o,sx,sy,s,8,.05,.8);
const drawIcon=(o,sx,sy,s)=>drawSpr(o,sx,sy,s,7,.07,.9);

/* ---- neon sign ---- */
function drawSign(o,sx,sy,s){
  if(S.crisp)spriteQ.push(()=>renderSign(o,sx,sy,s,true));
  else renderSign(o,sx,sy,s,false);
}

function drawBelt(o,sx,sy,s){
  for(const rk of o.rocks){
    const rx=sx+rk.dx*s,ry=sy+rk.dy*s;
    if(rx<-8||ry<-8||rx>lw+8||ry>lh+8)continue;
    lctx.save();lctx.translate(rx,ry);lctx.rotate(rk.rot0+wt*rk.rsp);
    lctx.fillStyle=CS(rk.c);
    const rr=Math.max(.8,rk.r*s);
    lctx.beginPath();
    lctx.moveTo(rk.pts[0][0]*rr,rk.pts[0][1]*rr);
    for(let k=1;k<rk.pts.length;k++)lctx.lineTo(rk.pts[k][0]*rr,rk.pts[k][1]*rr);
    lctx.closePath();lctx.fill();lctx.restore();
  }
}

function drawGalaxy(g,sx,sy,s){
  lctx.fillStyle=CS(P.hi[0]);
  lctx.globalAlpha=.06;circ(sx,sy,g.size*s);
  lctx.globalAlpha=.1;circ(sx,sy,g.size*s*.4);
  let last=-1;
  const zf=s<.6?.55:1;
  for(const d of g.dots){
    if(zf<1&&d.a<.3)continue;
    if(d.ci!==last){lctx.fillStyle=CS(d.ci);last=d.ci;}
    lctx.globalAlpha=d.a*.6*zf;
    lctx.fillRect((sx+d.dx*s)|0,(sy+d.dy*s)|0,1,1);
  }
  lctx.globalAlpha=1;
}

function drawNebula(n,sx,sy,s){
  if(n.kind==='pn'){
    lctx.strokeStyle=CS(n.ciA);lctx.globalAlpha=.5;
    lctx.lineWidth=Math.max(1.5,n.r*s*.12);
    lctx.beginPath();lctx.arc(sx,sy,n.r*s,0,TAU);lctx.stroke();
    lctx.globalAlpha=.14;lctx.fillStyle=CS(n.ciA);circ(sx,sy,n.r*s*.8);
    lctx.globalAlpha=1;lctx.fillStyle=CS(n.ciB);
    lctx.fillRect((sx|0)-1,(sy|0)-1,2,2);
    return;
  }
  for(const b of n.blobs){
    lctx.globalAlpha=b.a*(S.glow*.75+.35);
    lctx.fillStyle=CS(b.ci);
    circ(sx+b.dx*s,sy+b.dy*s,b.r*s);
  }
  if(n.knots)for(const b of n.knots){
    lctx.globalAlpha=b.a;lctx.fillStyle=CS(b.ci);
    circ(sx+b.dx*s,sy+b.dy*s,Math.max(1,b.r*s));
  }
  if(n.fil){
    lctx.strokeStyle=CS(n.ciB);lctx.globalAlpha=.25;lctx.lineWidth=1;
    lctx.beginPath();
    for(const f of n.fil){lctx.moveTo(sx+f[0]*s,sy+f[1]*s);lctx.lineTo(sx+f[2]*s,sy+f[3]*s);}
    lctx.stroke();
    lctx.fillStyle=CS(P.hot[P.hot.length-1]);lctx.globalAlpha=.8;
    lctx.fillRect((sx|0)-1,(sy|0)-1,1,1);
  }
  lctx.globalAlpha=1;
}

/* ---------------- entities ---------------- */
function drawSprite(spr,x,y,sc,flip,cols){
  x=Math.round(x-spr[0].length*sc/2);y=Math.round(y-spr.length*sc/2);
  const eng=[];
  for(let r=0;r<spr.length;r++){
    const row=spr[r];
    for(let c=0;c<row.length;c++){
      const ch=row[flip?row.length-1-c:c];
      if(ch==='.')continue;
      if(ch==='E'){eng.push([x+c*sc,y+r*sc]);continue;}
      const ci=cols[ch];
      if(ci===undefined)continue;
      lctx.fillStyle=CS(ci);
      lctx.fillRect(x+c*sc,y+r*sc,sc,sc);
    }
  }
  return eng;
}
const ships=[],comets=[],meteors=[],fx=[],ufos=[],astro=[],novas=[];
let shipTimer=1.2,cometTimer=3,ufoTimer=12,astroTimer=20,novaTimer=20;
function viewRect(){const z=zoom;return{x0:cam.x-lw/2/z,y0:cam.y-lh/2/z,x1:cam.x+lw/2/z,y1:cam.y+lh/2/z};}
const SIPOOL=[0,1,2,2,3,4,5,6,7,7,0,1,2,8,9,10,11,11,12,13,14,15,15,16,17,18,19,20,20];
function spawnComet(){
  const vr=viewRect(),M=120;
  const side=(Math.random()*2)|0;
  const x=side?vr.x1+M:vr.x0-M,y=vr.y0+Math.random()*(vr.y1-vr.y0);
  const tx=side?vr.x0-M*.5:vr.x1+M*.5,ty=vr.y0+Math.random()*(vr.y1-vr.y0);
  const d=Math.hypot(tx-x,ty-y)||1,sp=6+Math.random()*9;
  comets.push({cls:'comet',x,y,vx:(tx-x)/d*sp,vy:(ty-y)/d*sp,len:30+Math.random()*35,
    ci:pick(Math.random,P.hot),rs:(Math.random()*1e9)|0});
}
function spawnUfo(){
  const vr=viewRect(),M=60;
  const side=Math.random()<.5;
  const x=side?vr.x0-M:vr.x1+M;
  const y=vr.y0+Math.random()*(vr.y1-vr.y0);
  const sp=5+Math.random()*8;
  ufos.push({cls:'ufo',x,y,vx:(side?1:-1)*sp,vy:(Math.random()-.5)*3,
    ph:Math.random()*TAU,rs:(Math.random()*1e9)|0,img:Math.random()<.65?pick(Math.random,['s389','s400']):null});
}
function spawnAstro(){
  const vr=viewRect(),M=50;
  const side=(Math.random()*4)|0;
  let x,y;
  if(side===0){x=vr.x0-M;y=vr.y0+Math.random()*(vr.y1-vr.y0);}
  else if(side===1){x=vr.x1+M;y=vr.y0+Math.random()*(vr.y1-vr.y0);}
  else if(side===2){y=vr.y0-M;x=vr.x0+Math.random()*(vr.x1-vr.x0);}
  else{y=vr.y1+M;x=vr.x0+Math.random()*(vr.x1-vr.x0);}
  const sp=2+Math.random()*4,a=Math.random()*TAU;
  astro.push({cls:'astro',x,y,vx:Math.cos(a)*sp,vy:Math.sin(a)*sp*.5,
    rot:Math.random()*TAU,rsp:(Math.random()-.5)*.6,ph:Math.random()*TAU,
    rs:(Math.random()*1e9)|0});
}
function updateEntities(dt){
  shipTimer-=dt*S.traffic;
  if(shipTimer<0&&ships.length<10&&ON.ship){spawnShip();shipTimer=3+Math.random()*6;}
  cometTimer-=dt*S.traffic;
  if(cometTimer<0&&comets.length<2&&ON.comet){spawnComet();cometTimer=8+Math.random()*10;}
  ufoTimer-=dt*S.traffic;
  if(ufoTimer<0&&ufos.length<2&&ON.ufo){spawnUfo();ufoTimer=25+Math.random()*45;}
  astroTimer-=dt*S.traffic;
  if(astroTimer<0&&astro.length<2&&ON.astro){spawnAstro();astroTimer=30+Math.random()*70;}
  novaTimer-=dt*S.events;
  if(novaTimer<0&&novas.length<2&&ON.nova){spawnNova();novaTimer=45+Math.random()*90;}
  if(ON.meteor&&Math.random()<dt*S.events*.7&&meteors.length<4){
    meteors.push({x:Math.random()*lw,y:Math.random()*lh*.7,
      vx:(Math.random()-.5)*220,vy:140+Math.random()*260,t:0,life:.4+Math.random()*.3});
  }
  const vr=viewRect(),M=500;
  for(let i=ships.length-1;i>=0;i--){const s=ships[i];
    if(s.turn){const sp=Math.hypot(s.vx,s.vy);s.ang+=s.turn*dt;s.vx=Math.cos(s.ang)*sp;s.vy=Math.sin(s.ang)*sp;}
    s.x+=s.vx*dt;s.y+=s.vy*dt;
    if(s.x<vr.x0-M||s.x>vr.x1+M||s.y<vr.y0-M||s.y>vr.y1+M)ships.splice(i,1);}
  for(let i=comets.length-1;i>=0;i--){const c=comets[i];
    c.x+=c.vx*dt;c.y+=c.vy*dt;
    if(c.x<vr.x0-M||c.x>vr.x1+M||c.y<vr.y0-M||c.y>vr.y1+M)comets.splice(i,1);}
  for(let i=ufos.length-1;i>=0;i--){const u=ufos[i];
    u.x+=u.vx*dt;u.y+=u.vy*dt;
    if(u.x<vr.x0-M||u.x>vr.x1+M||u.y<vr.y0-M||u.y>vr.y1+M)ufos.splice(i,1);}
  for(let i=astro.length-1;i>=0;i--){const a=astro[i];
    a.x+=a.vx*dt;a.y+=a.vy*dt;a.rot+=a.rsp*dt;
    if(a.x<vr.x0-M||a.x>vr.x1+M||a.y<vr.y0-M||a.y>vr.y1+M)astro.splice(i,1);}
  for(let i=novas.length-1;i>=0;i--){novas[i].t+=dt;if(novas[i].t>novas[i].life)novas.splice(i,1);}
  for(let i=meteors.length-1;i>=0;i--){const m=meteors[i];
    m.t+=dt;m.x+=m.vx*dt;m.y+=m.vy*dt;
    if(m.t>m.life)meteors.splice(i,1);}
}
function drawUfo(u,sx,sy){
  const bob=Math.sin(wt*1.4+u.ph)*1.5;
  const y=sy+bob;
  const beamOn=Math.sin(rt*.7+u.ph)>.2;
  if(beamOn){
    const fl=.5+.5*Math.sin(rt*17);
    lctx.fillStyle=CS(nearT(255,225,110));
    lctx.globalAlpha=.14+.2*fl;
    lctx.beginPath();
    lctx.moveTo(sx-1.5,y+3);lctx.lineTo(sx+1.5,y+3);
    lctx.lineTo(sx+5,y+16);lctx.lineTo(sx-5,y+16);
    lctx.closePath();lctx.fill();
    lctx.globalAlpha=.7*fl;
    lctx.fillRect((sx-1)|0,(y+14-((rt*8)%12))|0,2,2);
    lctx.globalAlpha=1;
  }
  drawSpriteIdx(UFO_SPR,sx,y,1,
    {L:P.ROLE.L,W:P.ROLE.W,G:nearT(150,155,165),R:P.ROLE.R},0);
}
function drawAstro(a,sx,sy){
  lctx.save();lctx.translate(sx,sy);lctx.rotate(a.rot);
  drawSpriteIdx(AST_SPR,0,0,1,{L:P.ROLE.L,W:P.ROLE.W,R:P.ROLE.R,B:P.ROLE.B},0);
  lctx.restore();
}
function drawEntities(){
  const z=zoom,cx0=lw/2,cy0=lh/2;
  drawNovas(z,cx0,cy0);
  for(const c of comets){
    const sx=(c.x-cam.x)*z+cx0,sy=(c.y-cam.y)*z+cy0;
    if(sx<-70||sy<-70||sx>lw+70||sy>lh+70)continue;
    const vl=Math.hypot(c.vx,c.vy)||1,nx=-c.vx/vl,ny=-c.vy/vl;
    for(let k=0;k<16;k++){
      const tt=k/16,d=3+tt*c.len*z;
      const wob=Math.sin(tt*6+wt*2+(c.rs%10))*tt*3;
      const sz=Math.max(1,((1-tt)*2.2*z)|0);
      lctx.globalAlpha=(1-tt)*.5;
      lctx.fillStyle=CS(tt<.35?P.hot[P.hot.length-1]:c.ci);
      lctx.fillRect((sx+nx*d-ny*wob)|0,(sy+ny*d+nx*wob)|0,sz,sz);
    }
    lctx.globalAlpha=.3;circ(sx,sy,4*z+2);
    lctx.globalAlpha=1;lctx.fillStyle=CS(P.hot[P.hot.length-1]);
    lctx.fillRect((sx|0)-1,(sy|0)-1,2,2);
  }
  for(const u of ufos){
    const sx=(u.x-cam.x)*z+cx0,sy=(u.y-cam.y)*z+cy0;
    if(sx<-60||sy<-60||sx>lw+60||sy>lh+60)continue;
    (u.img?drawUfoImg:drawUfo)(u,sx,sy);
  }
  for(const a of astro){
    const sx=(a.x-cam.x)*z+cx0,sy=(a.y-cam.y)*z+cy0;
    if(sx<-40||sy<-40||sx>lw+40||sy>lh+40)continue;
    drawAstro(a,sx,sy);
  }
  if(!ON.ship)return;
  for(const sh of ships){
    const sx=(sh.x-cam.x)*z+cx0,sy=(sh.y-cam.y)*z+cy0,mg=60+(sh.hr||0)*z*1.4;
    if(sx<-mg||sy<-mg||sx>lw+mg||sy>lh+mg)continue;
    const sc=Math.max(1,Math.round(z*1.25));
    if(sh.warp){
      lctx.strokeStyle=CS(sh.cols.E);lctx.globalAlpha=.35;lctx.lineWidth=Math.max(1,sc);
      lctx.beginPath();lctx.moveTo(sx,sy);
      lctx.lineTo(sx-sh.vx*.09,sy-sh.vy*.09);lctx.stroke();lctx.globalAlpha=1;
    }
    if(sh.si>=IMGSHIP0){drawImgShip(sh,sx,sy,sc);continue;}
    if(!sh.frames){sh.frames=[SPRITES[sh.si]];sh.eng=true;}
    const cv2=sprCanvas(sh,0,sc,false),spr=sh.frames[0],w=spr[0].length,h=spr.length;
    lctx.save();lctx.translate(Math.round(sx),Math.round(sy));lctx.rotate(sh.ang);
    if(Math.cos(sh.ang)<0)lctx.scale(1,-1);        /* keep the hull's top side up when flying left */
    lctx.imageSmoothingEnabled=false;
    lctx.drawImage(cv2,-(cv2.width>>1),-(cv2.height>>1));
    lctx.fillStyle=CS(sh.cols.E);
    for(const e of ENG[sh.si]){
      const len=sc*(1+Math.random()*2.5);
      lctx.globalAlpha=.4+Math.random()*.5;
      lctx.fillRect(Math.floor((e[0]-w/2)*sc-len),Math.floor((e[1]-h/2)*sc),len,Math.max(1,sc*.8));
    }
    lctx.restore();lctx.globalAlpha=1;
  }
}
function drawMeteorsFx(){
  for(const m of meteors){
    const a=Math.sin(Math.PI*m.t/m.life);
    for(let k=0;k<6;k++){
      lctx.globalAlpha=a*(1-k/6)*.85;
      lctx.fillStyle=CS(k<2?P.hot[P.hot.length-1]:P.hot[Math.min(1,P.hot.length-1)]);
      lctx.fillRect((m.x-m.vx*k*.016)|0,(m.y-m.vy*k*.016)|0,k<1?2:1,k<1?2:1);
    }
  }
  for(let i=fx.length-1;i>=0;i--){
    const f=fx[i],age=rt-f.t0;
    if(age>.5){fx.splice(i,1);continue;}
    lctx.strokeStyle=CS(P.hot[P.hot.length-1]);
    lctx.globalAlpha=.7*(1-age*2);lctx.lineWidth=1;
    lctx.beginPath();lctx.arc(f.x,f.y,2+age*34,0,TAU);lctx.stroke();
  }
  lctx.globalAlpha=1;
}

/* ---------------- star list drawing ---------------- */
function drawStars(stars,camX,camY){
  const z=zoom,cx0=lw/2,cy0=lh/2;
  let last=-1;
  for(const st of stars){
    const sx=((st.x-camX)*z+cx0)|0,sy=((st.y-camY)*z+cy0)|0;
    if(sx<-3||sy<-3||sx>lw+3||sy>lh+3)continue;
    let a=st.a;
    if(S.twinkle>0&&st.tw){
      a*=1+S.twinkle*Math.sin(wt*st.sp+st.ph)*.8;
      if(a<=.05)continue;if(a>1)a=1;
    }
    if(st.ci!==last){lctx.fillStyle=CS(st.ci);last=st.ci;}
    lctx.globalAlpha=a;
    lctx.fillRect(sx,sy,st.sz,st.sz);
    if(st.g){
      lctx.globalAlpha=a*.4;
      lctx.fillRect(sx-2,sy,st.sz+4,1);
      lctx.fillRect(sx,sy-2,1,st.sz+4);
    }
  }
  lctx.globalAlpha=1;
}

/* ---------------- master draw ---------------- */
let selected=null,sunPts=[];
function draw(){
  spriteQ.length=0;
  bodiesDrawn=0;
  lctx.globalAlpha=1;
  lctx.setTransform(1,0,0,1,0,0);
  lctx.fillStyle=P.str[0];lctx.fillRect(0,0,lw,lh);
  const cx0=lw/2,cy0=lh/2;

  eachChunk(LAYERS[0],(c,camX,camY,z)=>{
    for(const it of c.items){
      const sx=(it.x-camX)*z+cx0,sy=(it.y-camY)*z+cy0;
      if(sx<-120||sy<-120||sx>lw+120||sy>lh+120)continue;
      drawGalaxy(it,sx,sy,z);bodiesDrawn++;
    }
    drawStars(c.stars,camX,camY);
  });
  eachChunk(LAYERS[1],(c,camX,camY,z)=>{
    for(const it of c.items){
      if(it.t==='haze'){
        const sx=(it.x-camX)*z+cx0,sy=(it.y-camY)*z+cy0;
        lctx.globalAlpha=it.a;lctx.fillStyle=CS(it.ci);
        circ(sx,sy,it.r*z);
      }
    }
    lctx.globalAlpha=1;
    for(const it of c.items)if(it.t==='neb'){
      const sx=(it.x-camX)*z+cx0,sy=(it.y-camY)*z+cy0;
      if(sx<-260||sy<-260||sx>lw+260||sy>lh+260)continue;
      drawNebula(it,sx,sy,z);bodiesDrawn++;
    }
    drawStars(c.stars,camX,camY);
    for(const it of c.items)if(it.t==='dark'){
      const sx=(it.x-camX)*z+cx0,sy=(it.y-camY)*z+cy0;
      if(sx<-180||sy<-180||sx>lw+180||sy>lh+180)continue;
      lctx.fillStyle=P.str[0];
      for(const b of it.blobs){
        lctx.globalAlpha=b.a*.6;
        circ(sx+b.dx*z,sy+b.dy*z,b.r*z);
      }
      lctx.globalAlpha=1;
    }
  });
  /* near layer — pass 1: light sources */
  sunPts=[];
  eachChunk(LAYERS[2],(c,camX,camY,z)=>{
    for(const it of c.items){
      const sx=(it.x-camX)*z+cx0,sy=(it.y-camY)*z+cy0;
      if(sx<-90||sy<-90||sx>lw+90||sy>lh+90)continue;
      if(it.t==='sun'){drawSun(it,sx,sy,z);sunPts.push({x:sx,y:sy});bodiesDrawn++;}
      else if(it.t==='binary'){
        const a=wt*it.w+it.ph,ox=Math.cos(a)*it.orb*z,oy=Math.sin(a)*it.orb*z*.5;
        drawSun(it.s1,sx+ox,sy+oy,z);
        drawSun(it.s2,sx-ox,sy-oy,z);
        sunPts.push({x:sx+ox,y:sy+oy},{x:sx-ox,y:sy-oy});bodiesDrawn++;
      }
      else if(it.t==='bhole'){drawBhole(it,sx,sy,z);bodiesDrawn++;}
      else if(it.t==='dyson'){drawDyson(it,sx,sy,z);sunPts.push({x:sx,y:sy});bodiesDrawn++;}
    }
  });
  /* near layer — pass 2: everything else */
  eachChunk(LAYERS[2],(c,camX,camY,z)=>{
    for(const it of c.items){
      const sx=(it.x-camX)*z+cx0,sy=(it.y-camY)*z+cy0;
      let m=60;
      if(it.t==='belt')m=(it.rx+10)*z;
      else if(it.t==='ring')m=(it.R+12)*z;
      else if(it.t==='dyson'||it.t==='mega')m=(it.r+14)*z;
      else if(it.t==='rift'||it.t==='storm')m=90;
      else if(it.t==='sign')m=it.hitR*z*1.6+60;else if(it.t==='worm')m=(it.r*2+10)*z;else if(it.t==='beacon')m=(it.L+10)*z;else if(it.t==='field')m=(it.hitR+24)*z;
      if(sx<-m||sy<-m||sx>lw+m||sy>lh+m)continue;
      if(it.t==='planet'){drawPlanet(it,sx,sy,z,sunPts);bodiesDrawn++;}
      else if(it.t==='station'){drawStation(it,sx,sy,z);bodiesDrawn++;}
      else if(it.t==='pulsar'){drawPulsar(it,sx,sy,z);bodiesDrawn++;}
      else if(it.t==='battle'){drawBattle(it,sx,sy,z);bodiesDrawn++;}
      else if(it.t==='shattered'){drawShattered(it,sx,sy,z);bodiesDrawn++;}
      else if(it.t==='mega'){drawMega(it,sx,sy,z);bodiesDrawn++;}
      else if(it.t==='ring'){drawRing(it,sx,sy,z);bodiesDrawn++;}
      else if(it.t==='rift'){drawRift(it,sx,sy,z);bodiesDrawn++;}
      else if(it.t==='probe'){drawProbe(it,sx,sy,z);bodiesDrawn++;}
      else if(it.t==='char'){drawChar(it,sx,sy,z);bodiesDrawn++;}
      else if(it.t==='food'){drawFood(it,sx,sy,z);bodiesDrawn++;}
      else if(it.t==='sign'){drawSign(it,sx,sy,z);bodiesDrawn++;}
      else if(it.t==='icon'){drawIcon(it,sx,sy,z);bodiesDrawn++;}
      else if(it.t==='worm'){drawWorm(it,sx,sy,z);bodiesDrawn++;}
      else if(it.t==='jelly'){drawJelly(it,sx,sy,z);bodiesDrawn++;}
      else if(it.t==='beacon'){drawBeacon(it,sx,sy,z);bodiesDrawn++;}
      else if(it.t==='rock'||it.t==='junk'||it.t==='xtra'){drawImgItem(it,sx,sy,z);bodiesDrawn++;}
      else if(it.t==='field'){drawField(it,sx,sy,z);bodiesDrawn++;}
      else if(it.t==='storm'){drawStorm(it,sx,sy,z);bodiesDrawn++;}
      else if(it.t==='sat'){drawSat(it,sx,sy,z);bodiesDrawn++;}
      else if(it.t==='belt'){drawBelt(it,sx,sy,z);bodiesDrawn++;}
      else if(it.t==='lone'){
        lctx.save();lctx.translate(sx,sy);lctx.rotate(it.rot0+wt*it.rsp);
        lctx.fillStyle=CS(it.c);const rr=Math.max(1,it.r*z);
        lctx.beginPath();lctx.moveTo(it.pts[0][0]*rr,it.pts[0][1]*rr);
        for(let k=1;k<it.pts.length;k++)lctx.lineTo(it.pts[k][0]*rr,it.pts[k][1]*rr);
        lctx.closePath();lctx.fill();lctx.restore();
        bodiesDrawn++;
      }
    }
  });
  drawEntities();
  drawMeteorsFx();
  if(selected){
    let sx,sy,r=12;
    if(selected.ent){sx=(selected.ent.x-cam.x)*zoom+cx0;sy=(selected.ent.y-cam.y)*zoom+cy0;}
    else{
      const p=selected.p;
      const fo=(selected.it.frames||selected.it.img)?floatOff(selected.it):[0,0];
      sx=(selected.it.x-cam.x*p)*zoom+cx0+fo[0]*zoom;sy=(selected.it.y-cam.y*p)*zoom+cy0+fo[1]*zoom;
      r=clamp((selected.it.hitR||selected.it.r||8)*zoom,9,34);
    }
    const b=.55+.35*Math.sin(rt*4);
    lctx.strokeStyle=CS(P.hot[P.hot.length-1]);
    lctx.globalAlpha=b;lctx.lineWidth=1;
    const CORN=[[-1,-1],[1,-1],[1,1],[-1,1]];
    for(let q=0;q<4;q++){
      const dx=CORN[q][0],dy=CORN[q][1];
      lctx.beginPath();
      lctx.moveTo(sx+dx*r,sy+dy*r-dy*4.5);
      lctx.lineTo(sx+dx*r,sy+dy*r);
      lctx.lineTo(sx+dx*r-dx*4.5,sy+dy*r);
      lctx.stroke();
    }
    lctx.globalAlpha=1;
  }
  post();
  lctx.globalAlpha=1;
  for(const f of spriteQ)f();
  spriteQ.length=0;
  mctx.imageSmoothingEnabled=false;
  mctx.drawImage(low,0,0,lw*PX,lh*PX);
  if(dotPat){mctx.globalAlpha=Math.min(1,S.dots*1.7)*.85;
    mctx.fillStyle=dotPat;mctx.fillRect(0,0,cv.width,cv.height);}
  if(scanPat){mctx.globalAlpha=S.scan*.55;
    mctx.fillStyle=scanPat;mctx.fillRect(0,0,cv.width,cv.height);}
  /* reroll crossfade: frozen old frame blends out over the new universe */
  if(fade.active){
    const k=(performance.now()-fade.t0)/FADE_DUR;
    if(k>=1||fadeC.width!==cv.width||fadeC.height!==cv.height)fade.active=false;
    else{
      const e=k*k*(3-2*k);
      mctx.globalAlpha=1-e;
      mctx.drawImage(fadeC,0,0);
      mctx.globalAlpha=1;
    }
  }
  mctx.globalAlpha=1;
}

/* ================================================================
   INSPECTOR
   ================================================================ */
const insEl=document.getElementById('inspector');
function describe(it,ent){
  const R=mulberry(((ent?ent.rs:(it&&it.rs))||1)>>>0);
  if(ent){
    if(ent.cls==='nova')return{n:'SN '+(2020+(R()*9|0))+String.fromCharCode(65+(R()*26|0)),c:'SUPERNOVA / STELLAR DEATH',
      rows:[['TYPE',pick(R,['IA','II-P','IB','IC'])],['PEAK','10^'+(42+(R()*3|0))+' J'],
            ['REMNANT',pick(R,['NEUTRON STAR','BLACK HOLE','NOTHING'])],['AGE',(ent.t||0).toFixed(1)+' S']]};
    if(ent.cls==='ufo')return{n:'UFO-'+(1000+R()*8999|0),c:'UNIDENTIFIED CRAFT / BEAM ACTIVE',
      rows:[['ORIGIN',pick(R,['ZETA RET','UNKNOWN','CLASSIFIED','BEYOND'])],
            ['BEAM',pick(R,['TRACTOR','EXAMINATION','HARVEST'])],
            ['HULL','NON-EUCLIDEAN'],['INTENT',pick(R,['CURIOSITY','MISCHIEF','UNCLEAR'])]]};
    if(ent.cls==='astro')return{n:'ASTR-'+(100+R()*899|0)+' '+genName(R).toUpperCase(),
      c:'ADRIFT ASTRONAUT / EVA MISHAP',
      rows:[['SUIT',pick(R,['MK-III','STEEL','ORION','DEEP'])],
            ['O2',(2+R()*40|0)+'%'],['HEART',(60+R()*80|0)+' BPM'],
            ['STATUS',pick(R,['CALM','PANICKED','PHILOSOPHICAL','SIGNALING'])]]};
    if(ent.cls==='comet')return{n:'C/20'+(10+((R()*89)|0))+' '+genName(R),c:'LONG-PERIOD COMET',
      rows:[['PERIOD',(60+R()*900|0)+' YR'],['NUCLEUS',(1+R()*18).toFixed(1)+' KM'],
            ['TAIL','0.'+(2+((R()*8)|0))+' AU'],['ECC',(.5+R()*.49).toFixed(3)]]};
    return{n:pick(R,NSHIP)+' '+genName(R),c:SHIPCLASS[ent.si]+(ent.warp?' / WARP TRANSIT':''),
      rows:[['REGISTRY',pick(R,NSHIP)],['HULL','GRADE-'+(2+((R()*4)|0))],
            ['HEADING',((Math.atan2(ent.vy,ent.vx)*180/Math.PI+360)%360|0)+''],
            ['VELOCITY',(Math.hypot(ent.vx,ent.vy)|0)+' U/S'],['FORMATION',(ent.form||'solo').toUpperCase()]]};
  }
  switch(it.t){
    case'sun':return{n:it.name,c:pick(R,['MAIN SEQUENCE STAR','RED DWARF','ORANGE DWARF','BLUE GIANT','WHITE STAR','SUBGIANT']),
      rows:[['SPECTRAL',pick(R,['G2-V','K4-V','M3-V','F8-V','B2-III','A0-V'])],
            ['TEMP',(2800+R()*22000|0)+' K'],['MASS',(.2+R()*8).toFixed(2)+' SOLAR'],
            ['AGE',(.4+R()*9).toFixed(1)+' GYR']]};
    case'binary':return{n:it.name,c:'BINARY SYSTEM / BARYCENTRIC ORBIT',
      rows:[['PERIOD',(2+R()*40|0)+' D'],['SEPARATION',(4+R()*120|0)+' AU'],
            ['PRIMARY',it.s1.name],['COMPANION',it.s2.name]]};
    case'planet':{
      const cls={rocky:'TERRESTRIAL WORLD',gas:'CLASS-'+pick(R,['I','II','III','IV','V'])+' GAS GIANT',
        ice:'CRYOGENIC WORLD',lava:'VOLCANIC WORLD',terra:'TEMPERATE WORLD',
        earth:'BLUE MARBLE / TEMPERATE',ocean:'PELAGIC FLUID WORLD'}[it.kind]+(it.ring?' / RINGED':'');
      const rows=[['RADIUS',((it.r*2979)|0)+' KM'],['ORBIT',(12+R()*900|0)+' D'],
        ['MOONS',(it.moons?it.moons.length:0)+'']];
      if(it.kind==='gas')rows.push(['WINDS',(80+R()*540|0)+' M/S'],['STORMS',it.storm?'ACTIVE':'DORMANT']);
      if(it.kind==='lava')rows.push(['SURFACE',(900+R()*1400|0)+' K']);
      if(it.kind==='ice')rows.push(['ALBEDO',(.5+R()*.4).toFixed(2)]);
      if(it.kind==='terra')rows.push(['ATMOSPHERE',pick(R,['THIN','DENSE','STANDARD','TRACE'])]);
      if(it.kind==='earth')rows.push(['BIOSIGN','POSITIVE'],['POPULATION',(R()*9|0)+'.'+(R()*9|0)+'B']);
      if(it.kind==='ocean')rows.push(['DEPTH',(2+R()*40|0)+' KM'],['TIDES','EXTREME'],['BIOSIGN','PROBABLE']);
      return{n:it.name,c:cls,rows};}
    case'storm':return{n:'HYC-'+(1000+R()*8999|0)+' '+genName(R).toUpperCase(),
      c:'HYPERCANE / MESOSCALE VORTEX',
      rows:[['WINDS',(300+R()*600|0)+' KM/H'],['EYE',(50+R()*300|0)+' KM'],
            ['PRESSURE',(850+R()*120|0)+' HPA'],['SPIN','CORIOLIS']]};
    case'sat':return{n:pick(R,NSAT)+'-'+(1+R()*99|0),c:'ARTIFICIAL SATELLITE',
      rows:[['ORBIT',pick(R,['LEO','MEO','GEO','HEO','GRAVEYARD'])],
            ['POWER','SOLAR ARRAY'],['SIGNAL',pick(R,['STRONG','WEAK','INTERMITTENT','DEAD'])],
            ['MASS',(80+R()*4000|0)+' KG']]};
    case'battle':return{n:'STATION DS-'+(100+R()*899|0),c:'MOON-CLASS BATTLE STATION',
      rows:[['CREW',(1+R()*2).toFixed(1)+'M'],['PRIMARY','SUPERLASER'],
            ['POWER','HYPERMATTER'],['STATUS',pick(R,['OPERATIONAL','TRACKING','STANDBY'])]]};
    case'shattered':return{n:it.name,c:'SHATTERED WORLD',
      rows:[['CAUSE',pick(R,['COLLISION','TIDAL DISRUPTION','CORE BREACH','ORBITAL BOMBARDMENT'])],
            ['DEBRIS',it.frags.length*212+' BODIES'],['AGE',(1+R()*900|0)+' KYR']]};
    case'mega':return{n:pick(R,['STARFORGE','HEAVENLY','ORBITAL','MASS DRIVER'])+' '+genName(R).toUpperCase()+'-'+(1+R()*99|0),
      c:'MEGASTRUCTURE / ORBITAL HABITAT',
      rows:[['POPULATION',(2+R()*180|0)+'M'],['MODULES',it.arms*4+''],
            ['GRAVITY','0.'+(7+R()*3|0)+' G'],['POWER',(9+R()*90|0)+' PW']]};
    case'ring':return{n:it.name,c:it.kind==='ring'?'RINGWORLD / NIVEN-CLASS':'BANKS ORBITAL',
      rows:[['SPAN',(1+R()*9|0)+' AU'],['HABITABLE',(1+R()*3|0)+'M EARTHS'],
            ['SPIN',(1+R()*9|0)+'X G'],
            ['SHADOW SQ',it.kind==='ring'?it.squares.length:'N/A']]};
    case'dyson':return{n:it.name,c:'DYSON SWARM / STELLAR ENGINE',
      rows:[['COLLECTORS',it.panels.length*312+''],['OUTPUT',(1+R()*9|0)+'.'+(R()*9|0)+'E26 W'],
            ['COMPLETION',(10+R()*70|0)+'%'],['STAR','CAPTURED']]};
    case'rift':return{n:'RIFT-'+(1000+R()*8999|0)+' '+genName(R).toUpperCase(),c:'REALITY TEAR / ANOMALY',
      rows:[['WIDTH',(2+R()*40|0)+' M'],['FLUX',(1+R()*9).toFixed(1)+' E9 T'],
            ['ENTROPY','IRREVERSIBLE'],['CONTAINMENT','IMPOSSIBLE']]};
    case'probe':return{n:pick(R,['PIONEER','VOYAGER','NEW HORIZONS','DEEP FIELD'])+'-'+(1+R()*12|0),
      c:'DEEP SPACE PROBE',
      rows:[['LAUNCHED','19'+(58+R()*40|0)],['SIGNAL',pick(R,['WEAK','FAINT','LOST','STRONG'])],
            ['POWER','RTG'],['VELOCITY',(11+R()*30|0)+' KM/S']]};
    case'char':return{n:'SPECIMEN '+genName(R).toUpperCase(),c:CHARSPEC[it.kind]||'ANOMALOUS BIOFORM',
      rows:[['SPECIES',CHARSPEC[it.kind]||'UNKNOWN'],['MASS',(1+R()*99|0)+' KG'],
            ['THREAT',pick(R,['NEGLIGIBLE','MINIMAL','CUTE','SEVERE','EXTREME'])],
            ['ORIGIN','NON-SEQUITUR']]};
    case'food':return{n:'SNACK-'+(1000+R()*8999|0)+' '+genName(R).toUpperCase(),
      c:FOODSPEC[it.kind]||'CULINARY ANOMALY',
      rows:[['CALORIES',(120+R()*900|0)+' KCAL'],['TEMPERATURE',(R()<.5?'COLD':'HOT')],
            ['PRESERVATION','VACUUM ETERNAL'],['ORIGIN',pick(R,['DEEP FRYER ANOMALY','MATTER PRINTER','CARGO SPILL','UNKNOWN'])]]};
    case'sign':{
      const dead=it.letters.filter(L=>L.mode==='dead').length;
      return{n:'SIGN-'+(1000+R()*8999|0)+' '+genName(R).toUpperCase(),
        c:'NEON SIGNAGE / ADVERTISING ANOMALY',
        rows:[['TEXT','"'+it.disp+'"'],['LETTERS',it.letters.length+''],
              ['DEAD TUBES',dead+''],['DRAW',(1+R()*40|0)+' KW'],
              ['MESSAGE',pick(R,['EAT','LISTEN','UNCLEAR','WELCOME','OPEN 24H'])]]};}
    case'rock':{const sp=IMGS[it.img];return{n:genName(R)+'-'+(100+R()*899|0),c:(sp.lab||'ASTEROID')+' / '+pick(R,['C-TYPE','S-TYPE','M-TYPE']),
      rows:[['DIAMETER',(3+R()*180).toFixed(1)+' KM'],['ROTATION',(.5+R()*30).toFixed(1)+' H'],
            ['ALBEDO',(.05+R()*.4).toFixed(2)],['TRAJECTORY',pick(R,['STABLE','DECAYING','UNCERTAIN'])]]};}
    case'junk':{const sp=IMGS[it.img];return{n:(sp.lab||'DEBRIS')+' '+(1000+R()*8999|0),c:'ORBITAL DEBRIS / TRACKED',
      rows:[['ORIGIN',pick(R,['UNKNOWN','LOST CONVOY','OLD WAR','CARGO SPILL','SOMEONE\'S BAD DAY'])],
            ['AGE',(2+R()*400|0)+' YR'],['STATUS',pick(R,['INERT','DEAD','LEAKING','TUMBLING'])],
            ['HAZARD',pick(R,['NEGLIGIBLE','LOW','CURIOUS'])]]};}
    case'field':return{n:'FIELD '+genName(R).toUpperCase(),c:'DEBRIS FIELD / MIXED',
      rows:[['PIECES',it.parts.length*23+''],['SPAN',(it.hitR*31|0)+' KM'],
            ['ORIGIN',pick(R,['SHATTERED MOON','OLD BATTLE','COLLISION','UNKNOWN'])],['SALVAGE',pick(R,['POOR','FAIR','PROMISING'])]]};
    case'xtra':{
      const T={cargo:['DRIFTING CARGO CONTAINER','CONTENTS',['MACHINE PARTS','UNKNOWN','RATIONS','CONTRABAND','SOMEONE\'S LUNCH'],'SEAL',['INTACT','BREACHED','WELDED SHUT']],
        cryo:['CRYOGENIC CAPSULE','OCCUPANT',['UNKNOWN','HUMANOID','DO NOT OPEN','NO ANSWER'],'POWER',['RESERVE','FAILING','NOMINAL']],
        monolith:['MONOLITH / ORIGIN UNKNOWN','INSCRIPTION',['UNREADABLE','A WARNING','A RECIPE','DIFFERENT DAILY'],'AGE',[(1+R()*90|0)+' MYR']],
        crystal:['RESONANT CRYSTAL FORMATION','HARMONIC',['LOW HUM','HIGH SINGING','SILENT'],'VALUE',['PRICELESS','POWER SOURCE','JUST PRETTY']],
        bio:['XENO-BIOFORM / DORMANT','DIET',['STARLIGHT','UNKNOWN','VISITORS'],'TEMPERAMENT',['SHY','CURIOUS','HUNGRY']],
        mine:['ORBITAL MINE / ARMED?','FUSE',['LIVE','DUD','TIMER UNKNOWN'],'YIELD',[(1+R()*90|0)+' KT']]}[it.cat]||['ANOMALY','NOTE',['NONE'],'STATUS',['UNKNOWN']];
      return{n:it.cat.toUpperCase()+'-'+(100+R()*899|0)+' '+genName(R).toUpperCase(),c:T[0],
        rows:[[T[1],pick(R,T[2])],[T[3],pick(R,T[4])],['MASS',(1+R()*900|0)+' T'],['SIGNAL',pick(R,['NONE','FAINT','PULSING'])]]};}

    case'worm':return{n:'WH-'+(1000+R()*8999|0)+' '+genName(R).toUpperCase(),c:'TRAVERSABLE WORMHOLE / EINSTEIN-ROSEN',
      rows:[['THROAT',(1+R()*90|0)+' M'],['STABILITY',pick(R,['EXOTIC-HELD','FLICKERING','COLLAPSING','ETERNAL'])],
            ['TRANSIT',(1+R()*9).toFixed(1)+' S'],['DESTINATION',pick(R,['UNKNOWN','ANDROMEDA','2 LY AWAY','THE PAST','LUNCH'])]]};
    case'jelly':return{n:genName(R).toUpperCase()+'-'+(10+R()*89|0),c:'VOID JELLYFISH / SPACEBORNE ORGANISM',
      rows:[['SPAN',(30+R()*900|0)+' M'],['GLOW',pick(R,['SLOW PULSE','FAST PULSE','STROBING'])],
            ['DIET','COSMIC DUST'],['MOOD',pick(R,['SERENE','CURIOUS','DRIFTING','HUNGRY'])]]};
    case'beacon':return{n:pick(R,['POINT','CAPE','ROCK','REEF'])+' '+genName(R).toUpperCase()+' LIGHT',c:'LIGHTHOUSE ROCK / NAV BEACON',
      rows:[['PERIOD',(2+R()*9).toFixed(1)+' S'],['KEEPER',pick(R,['AUTOMATED','ONE HERMIT','VACANT','A VERY OLD ROBOT'])],
            ['RANGE',(2+R()*40|0)+' LY'],['STATUS',pick(R,['OPERATIONAL','FLICKERING','LOW OIL'])]]};
    case'icon':return{n:it.kind.toUpperCase()+'-'+(1000+R()*8999|0),c:'SPECIAL OBJECT / '+it.kind.toUpperCase(),
      rows:[['ORIGIN',pick(R,['UNKNOWN','CARGO SPILL','NON-SEQUITUR','DREAM'])],
            ['STATUS',pick(R,['DRIFTING','GLOWING','DORMANT'])],['VALUE',(R()*999|0)+' CR']]};
    case'pulsar':return{n:'PSR J'+(1000+R()*8999|0)+(R()<.5?'+':'-')+(10+R()*89|0),
      c:'NEUTRON STAR / PULSAR',
      rows:[['PERIOD','0.'+(String((16+R()*1300)|0)).padStart(4,'0')+' S'],
            ['B FIELD',(1+R()*9).toFixed(1)+'E12 G'],['MASS',(1.2+R()*.8).toFixed(2)+' SOLAR'],
            ['DISCOVERED','19'+(68+R()*30|0)]]};
    case'bhole':return{n:pick(R,NBH),c:'BLACK HOLE / ACCRETION DISC',
      rows:[['MASS',(3+R()*40).toFixed(1)+' SOLAR'],['SPIN','0.'+(60+R()*38|0)+' A/M'],
            ['DISC TEMP',(1+R()*9).toFixed(1)+' MK'],['HORIZON','3.'+(R()*9|0)+'E1 KM']]};
    case'station':return{n:pick(R,NSTA)+' '+genName(R)+'-'+(1+R()*99|0),c:'ORBITAL STATION',
      rows:[['CREW',(40+R()*900|0)+''],['DOCKS',1+((R()*8)|0)],
            ['STATUS',pick(R,['NOMINAL','STANDBY','LOW POWER','QUARANTINE'])],
            ['POWER',(2+R()*9).toFixed(1)+' GW']]};
    case'belt':return{n:'FIELD '+genName(R),c:'ASTEROID CLUSTER',
      rows:[['BODIES',it.rocks.length*37+''],['SPAN',(it.rx*29|0)+' KM'],
            ['CLASS',pick(R,['C-TYPE','S-TYPE','M-TYPE'])]]};
    case'lone':return{n:genName(R)+'-'+(100+R()*899|0),c:pick(R,['C-TYPE','S-TYPE','M-TYPE'])+' ASTEROID',
      rows:[['DIAMETER',((it.r*112)|0)+' KM'],['ROTATION',(.5+R()*30).toFixed(1)+' H'],
            ['ALBEDO',(.05+R()*.3).toFixed(2)]]};
    case'galaxy':return{n:it.name,c:{sp:'SBc SPIRAL',el:'E3 ELLIPTICAL',irr:'IRR DWARF'}[it.kind],
      rows:[['SPAN',(30+R()*90|0)+' KLY'],['STARS',(20+R()*300|0)+' B'],
            ['REDSHIFT','0.00'+(10+R()*89|0)]]};
    case'neb':return{n:it.name,
      c:{em:'EMISSION NEBULA',snr:'SUPERNOVA REMNANT',pn:'PLANETARY NEBULA'}[it.kind],
      rows:[['SPAN',(2+R()*40).toFixed(1)+' LY'],['AGE',it.kind==='snr'?(1+R()*40|0)+' KYR':(1+R()*9|0)+' MYR'],
            ['BRIGHTNESS',it.kind==='pn'?'HIGH':'DIFFUSE']]};
  }
  return{n:'UNKNOWN CONTACT',c:'UNCLASSIFIED',rows:[]};
}
function openInspector(sel){
  selected=sel;
  const d=describe(sel.it,sel.ent);
  document.getElementById('insName').textContent=d.n;
  document.getElementById('insClass').textContent=d.c;
  const rows=document.getElementById('insRows');
  rows.innerHTML='';
  const sx=sel.it?sel.it.x:sel.ent.x,sy=sel.it?sel.it.y:sel.ent.y;
  const sec=el('em');sec.textContent='SEC';
  const b=el('b');b.textContent=String(Math.floor(sx/320)).padStart(4,'0')+'/'+String(Math.floor(sy/320)).padStart(4,'0');
  rows.appendChild(sec);rows.appendChild(b);
  for(const kv of d.rows){
    const e1=el('em');e1.textContent=kv[0];const e2=el('b');e2.textContent=kv[1];
    rows.appendChild(e1);rows.appendChild(e2);
  }
  insEl.classList.add('open');
}
function closeInspector(){selected=null;insEl.classList.remove('open');}
function scanAt(mx,my){
  try{
    const lx=mx/PX,ly=my/PX,z=zoom,cx0=lw/2,cy0=lh/2;
    let best=null,bd=1e9;
    for(const L of LAYERS)eachChunk(L,(c,camX,camY)=>{
      for(const it of c.items){
        if(it.t==='haze')continue;
        const sx=(it.x-camX)*z+cx0,sy=(it.y-camY)*z+cy0;
        const hr=Math.max(10,(it.hitR||it.r||8)*z);
        const fo=(it.frames||it.img)?floatOff(it):null;
        const d=Math.hypot(lx-sx-(fo?fo[0]*z:0),ly-sy-(fo?fo[1]*z:0))-hr;
        if(d<bd){bd=d;best={it,p:L.p};}
      }
    });
    const entPools=[ships,comets,ufos,astro,novas];
    for(const pool of entPools)for(const e of pool){
      const sx=(e.x-cam.x)*z+cx0,sy=(e.y-cam.y)*z+cy0;
      const d=Math.hypot(lx-sx,ly-sy)-(e.hr||0)*z;
      if(d<bd&&d<16){bd=d;best={ent:e};}
    }
    if(best&&bd<14)openInspector(best);else closeInspector();
  }catch(err){fault('scan: '+err.message);closeInspector();}
}

/* ================================================================
   UI PANEL
   ================================================================ */
const panel=document.getElementById('panel');
const inputs={},updaters={};
function section(title){
  const s=el('div');const h=el('h3');h.textContent=title;
  s.appendChild(h);panel.appendChild(s);return s;
}
function slider(sec,label,key,min,max,step,fmt,cb){
  const w=el('div','ctl');
  const lr=el('div','lr');const sp=el('span');sp.textContent=label;
  const b=el('b');lr.appendChild(sp);lr.appendChild(b);w.appendChild(lr);
  const inp=document.createElement('input');
  inp.type='range';inp.min=min;inp.max=max;inp.step=step;inp.value=S[key];
  inp.addEventListener('input',()=>{S[key]=parseFloat(inp.value);upd();if(cb)cb();});
  w.appendChild(inp);sec.appendChild(w);
  const upd=()=>{b.textContent=fmt?fmt(S[key]):S[key];};
  upd();inputs[key]=inp;updaters[key]=upd;
}
function applySeed(v){
  seedStr=v;seed=strHash(v)||1;
  for(const L of LAYERS)L.map.clear();
  /* flush all transient state — nothing survives a universe jump */
  ships.length=0;comets.length=0;ufos.length=0;astro.length=0;
  meteors.length=0;fx.length=0;novas.length=0;
  selected=null;insEl.classList.remove('open');
  bandC=Math.cos((seed%628)/100);bandS=Math.sin((seed%628)/100);
  hdg=Math.atan2(bandS,bandC)+.5;
  rebuildStamp();
  warm();
}
/* ---- reroll with crossfade ---- */
let nextReroll=Infinity;
function doReroll(customSeed){
  try{
    fadeC.width=cv.width;fadeC.height=cv.height;
    fctx.imageSmoothingEnabled=false;
    fctx.drawImage(cv,0,0);
    fade.active=true;fade.t0=performance.now();
  }catch(err){fade.active=false;}
  const v=customSeed!==undefined?customSeed:
    Math.random().toString(36).slice(2,8).toUpperCase();
  const si=panel.querySelector('.seedrow input');
  if(si)si.value=v;
  applySeed(v);
  if(S.autoReroll)nextReroll=Date.now()+S.rerollMin*60000;
}
/* --- console --- */
{
  const sec=section('CONSOLE');
  const row=el('div','btnrow');
  const pause=el('button','btn');
  pause.innerHTML='<svg width="12" height="12" viewBox="0 0 12 12"><path d="M1 0h3v12H1zM8 0h3v12H8z" fill="currentColor"/></svg><span>PAUSE</span>';
  pause.addEventListener('click',()=>{
    S.paused=!S.paused;
    pause.classList.toggle('on',S.paused);
    pause.querySelector('span').textContent=S.paused?'RESUME':'PAUSE';
  });
  /* SNAPSHOT: synchronous toDataURL — the click stays within the
     user gesture, so no browser can silently swallow the download */
  const snap=el('button','btn');
  snap.innerHTML='<svg width="12" height="12" viewBox="0 0 12 12"><path d="M1 2h2V1h6v1h2v9H1zM3 4h6v5H3z" fill="currentColor"/></svg><span>SNAPSHOT</span>';
  snap.addEventListener('click',()=>{
    try{
      const a=document.createElement('a');
      a.href=cv.toDataURL('image/png');
      a.download='deepfield-'+seedStr+'-'+(Date.now()%100000)+'.png';
      document.body.appendChild(a);
      a.click();
      a.remove();
    }catch(err){fault('snapshot: '+err.message);}
  });
  row.appendChild(pause);row.appendChild(snap);sec.appendChild(row);
  const cp=el('button','btn');cp.style.cssText='width:100%;justify-content:center;margin-top:6px';
  cp.textContent='COPY CONFIG URL';
  cp.addEventListener('click',()=>{
    const url=location.href.split(/[?#]/)[0]+'?'+buildQuery();
    const done=()=>{cp.textContent='COPIED';setTimeout(()=>{cp.textContent='COPY CONFIG URL';},1200);};
    (navigator.clipboard?navigator.clipboard.writeText(url):Promise.reject()).then(done)
      .catch(()=>{window.prompt('Copy this URL:',url);});
  });
  sec.appendChild(cp);
}
/* --- universe --- */
{
  const sec=section('UNIVERSE');
  const sr=el('div','seedrow');
  const inp=document.createElement('input');inp.value='';inp.maxLength=12;inp.spellcheck=false;
  inp.title='universe seed';
  const rr=el('button','btn');
  rr.innerHTML='<svg width="12" height="12" viewBox="0 0 12 12"><path d="M1 1h10v10H1z" fill="none" stroke="currentColor"/><circle cx="4" cy="4" r="1.2" fill="currentColor"/><circle cx="8" cy="8" r="1.2" fill="currentColor"/><circle cx="8" cy="4" r="1.2" fill="currentColor"/><circle cx="4" cy="8" r="1.2" fill="currentColor"/></svg><span>REROLL</span>';
  inp.addEventListener('change',()=>doReroll(inp.value.trim()||'VOID'));
  rr.addEventListener('click',()=>doReroll());
  sr.appendChild(inp);sr.appendChild(rr);sec.appendChild(sr);
  /* auto reroll toggle + interval */
  const ar=el('button','btn');ar.style.width='100%';ar.style.justifyContent='center';
  ar.style.marginTop='8px';
  const arLbl=()=>{ar.classList.toggle('on',S.autoReroll);
    ar.textContent='AUTO REROLL: '+(S.autoReroll?'ON':'OFF');};
  ar.addEventListener('click',()=>{
    S.autoReroll=!S.autoReroll;
    nextReroll=S.autoReroll?Date.now()+S.rerollMin*60000:Infinity;
    arLbl();
  });
  arLbl();sec.appendChild(ar);
  slider(sec,'REROLL EVERY','rerollMin',60,1440,15,
    v=>{const h=Math.floor(v/60),m=v%60;return h+(m?'h '+m+'m':'h');},
    ()=>{if(S.autoReroll)nextReroll=Date.now()+S.rerollMin*60000;});
  const palWrap=el('div','pals');palWrap.style.marginTop='10px';
  PALETTES.forEach((p,i)=>{
    const b=el('button','pal'+(i===P.i?' on':''));
    const sw=el('span','sw');
    const step=Math.max(1,Math.floor(p.c.length/6));
    for(let k=0;k<p.c.length;k+=step){const s=el('i');s.style.background=p.c[k];sw.appendChild(s);}
    const nm=el('span');nm.textContent=p.n;
    b.appendChild(sw);b.appendChild(nm);
    b.addEventListener('click',()=>{
      setPalette(i);rebuildLUT();rebuildStamp();warm();
      palWrap.querySelectorAll('.pal').forEach(x=>x.classList.remove('on'));
      b.classList.add('on');
    });
    palWrap.appendChild(b);
  });
  sec.appendChild(palWrap);
}
/* --- display --- */
{
  const sec=section('DISPLAY / HALFTONE');
  const f2=v=>(+v).toFixed(2);
  slider(sec,'PIXEL SIZE','ps',2,8,1,v=>v+'px',()=>{resize();});
  slider(sec,'DITHER','dither',0,1,.05,f2,rebuildLUT);
  slider(sec,'GLOW','glow',0,1.5,.05,f2);
  slider(sec,'TWINKLE','twinkle',0,1,.05,f2);
  slider(sec,'DOT MATRIX','dots',0,1,.05,f2,buildMasks);
  slider(sec,'SCANLINES','scan',0,1,.05,f2,buildMasks);
  slider(sec,'SPRITE SIZE','sprSize',.5,2,.25,v=>(+v).toFixed(2)+'x');
  const cb=el('button','btn');cb.style.cssText='width:100%;justify-content:center;margin-top:6px';
  const cbL=()=>{cb.classList.toggle('on',S.crisp);cb.textContent='CRISP SPRITES: '+(S.crisp?'ON':'OFF');};
  cb.addEventListener('click',()=>{S.crisp=!S.crisp;cbL();});
  cbL();sec.appendChild(cb);
}
/* --- motion --- */
{
  const sec=section('MOTION');
  slider(sec,'DRIFT SPEED','drift',0,30,.5,v=>(+v).toFixed(1));
  slider(sec,'ZOOM','ztzoom',.35,3,.05,v=>(+v).toFixed(2));
  slider(sec,'FPS TARGET','fps',10,300,5,v=>v+' FPS');
}
/* --- density & color --- */
{
  const sec=section('DENSITY & COLOR');
  const f2=v=>(+v).toFixed(2)+'x';
  slider(sec,'STARS','stars',0,2,.05,f2,rebuildStamp);
  slider(sec,'NEBULAE','neb',0,2,.05,f2,rebuildStamp);
  slider(sec,'DEEP SKY','deep',0,2,.05,f2,rebuildStamp);
  slider(sec,'PLANETS','planets',0,2,.05,f2,rebuildStamp);
  slider(sec,'SMALL BODIES','small',0,2,.05,f2,rebuildStamp);
  slider(sec,'TRAFFIC','traffic',0,2,.05,f2,rebuildStamp);
  slider(sec,'EVENTS','events',0,2,.05,f2);
  slider(sec,'COLOR VARIETY','tint',0,1,.05,v=>Math.round(v*100)+'%',rebuildStamp);
}
/* --- object picker --- */
{
  const sec=section('OBJECT PICKER');
  const bar=el('div','btnrow');bar.style.marginBottom='8px';
  const all=el('button','btn mini');all.textContent='ALL';
  const none=el('button','btn mini');none.textContent='NONE';
  bar.appendChild(all);bar.appendChild(none);sec.appendChild(bar);
  const grid=el('div','chips');
  const rebuild=()=>{rebuildStamp();warm();refresh();};
  const refresh=()=>grid.querySelectorAll('.chip').forEach(c=>{
    c.classList.toggle('on',!!ON[c.dataset.k]);});
  TYPELIST.forEach(t=>{
    const c=el('button','chip'+(ON[t[0]]?' on':''));c.textContent=t[1];c.dataset.k=t[0];
    c.addEventListener('click',()=>{ON[t[0]]=!ON[t[0]];rebuild();});
    grid.appendChild(c);
  });
  all.addEventListener('click',()=>{TYPELIST.forEach(t=>ON[t[0]]=true);rebuild();});
  none.addEventListener('click',()=>{TYPELIST.forEach(t=>ON[t[0]]=false);rebuild();});
  sec.appendChild(grid);
}

/* panel toggle */
const panelToggle=document.getElementById('panelToggle');
const ICON_OPEN='<svg width="14" height="14" viewBox="0 0 14 14"><path d="M2 1h3v12H2zM9 1h3v12H9z" fill="currentColor"/></svg>';
const ICON_CLOSED='<svg width="14" height="14" viewBox="0 0 14 14"><path d="M3 1l8 6-8 6z" fill="currentColor"/></svg>';
function setPanel(open){
  panel.classList.toggle('hidden',!open);
  document.body.classList.toggle('panelClosed',!open);
  panelToggle.innerHTML=open?ICON_OPEN:ICON_CLOSED;
}
panelToggle.addEventListener('click',()=>setPanel(panel.classList.contains('hidden')));

/* ================================================================
   UI AUTO-HIDE + [S] LOCK
   ================================================================ */
let uiLocked=false,lastAct=performance.now();
const IDLE_MS=3000;
function showUI(){document.body.classList.remove('uiHidden');}
function hideUI(){document.body.classList.add('uiHidden');}
function pokeUI(){
  lastAct=performance.now();
  if(!uiLocked)showUI();
}
['pointermove','pointerdown','wheel','touchstart','keydown'].forEach(ev=>
  addEventListener(ev,pokeUI,{passive:true}));
addEventListener('keydown',e=>{
  if(e.key.toLowerCase()!=='s')return;
  const t=e.target;
  if(t&&t.tagName==='INPUT')return;   /* don't hijack the seed field */
  uiLocked=!uiLocked;
  if(uiLocked)hideUI();
  else pokeUI();
});

/* ================================================================
   INPUT
   ================================================================ */
let dragging=false,dragMoved=0,lastPX=0,lastPY=0;
cv.addEventListener('pointerdown',e=>{
  dragging=true;dragMoved=0;lastPX=e.clientX;lastPY=e.clientY;
  cv.classList.add('drag');
  try{cv.setPointerCapture(e.pointerId);}catch(err){}
});
cv.addEventListener('pointermove',e=>{
  if(!dragging)return;
  const dx=e.clientX-lastPX,dy=e.clientY-lastPY;
  dragMoved+=Math.abs(dx)+Math.abs(dy);
  lastPX=e.clientX;lastPY=e.clientY;
  cam.x-=dx/PX/zoom;cam.y-=dy/PX/zoom;
});
cv.addEventListener('pointerup',e=>{
  dragging=false;cv.classList.remove('drag');
  if(dragMoved<5){
    if(fx.length<24)fx.push({x:e.clientX/PX,y:e.clientY/PX,t0:rt});
    scanAt(e.clientX,e.clientY);
  }
});
cv.addEventListener('wheel',e=>{
  e.preventDefault();
  S.ztzoom=clamp(S.ztzoom*Math.exp(-e.deltaY*.0012),.35,3);
  if(inputs.ztzoom)inputs.ztzoom.value=S.ztzoom;
},{passive:false});

/* ================================================================
   MAIN LOOP
   ================================================================ */
const hSector=document.getElementById('hSector'),hHdg=document.getElementById('hHdg'),
      hVel=document.getElementById('hVel'),hZoom=document.getElementById('hZoom'),
      hBodies=document.getElementById('hBodies'),hFps=document.getElementById('hFps'),
      hJump=document.getElementById('hJump');
let last=performance.now(),lastRender=-1e9,simAcc=0;
let hudT=0,fpsC=0,fpsT=0,fpsV=60,pruneT=0;
function fmtRem(ms){
  if(ms<=0)return 'NOW';
  const s=Math.floor(ms/1000);
  return String(Math.floor(s/3600)).padStart(2,'0')+':'+
         String(Math.floor(s/60)%60).padStart(2,'0')+':'+
         String(s%60).padStart(2,'0');
}
function frame(now){
  requestAnimationFrame(frame);
  const dt=Math.min(.1,(now-last)/1000);last=now;rt+=dt;
  simAcc+=dt;
  /* wall-clock reroll schedule (immune to RAF throttling) */
  if(S.autoReroll&&Date.now()>=nextReroll)doReroll();
  /* idle UI auto-hide */
  if(!uiLocked&&performance.now()-lastAct>IDLE_MS)hideUI();
  /* FPS limiter */
  if(now-lastRender<1000/S.fps-.5)return;
  const step=Math.min(.25,simAcc);simAcc=0;lastRender=now;
  fpsC++;fpsT+=step;if(fpsT>=.5){fpsV=Math.round(fpsC/fpsT);fpsC=0;fpsT=0;}
  try{
    if(!S.paused){
      wt+=step;
      hdg+=(vnoise(wt*.05,3.7,seed)-.5)*.25*step;
      cam.x+=Math.cos(hdg)*S.drift*step;
      cam.y+=Math.sin(hdg)*S.drift*step;
      updateEntities(step);
    }
    zoom+=(S.ztzoom-zoom)*Math.min(1,step*8);
    warm();
    draw();
  }catch(err){fault(err.message);}
  hudT+=step;
  if(hudT>.25){
    hudT=0;
    hSector.textContent=String(Math.floor(cam.x/320)).padStart(4,'0')+'/'+String(Math.floor(-cam.y/320)).padStart(4,'0');
    hHdg.textContent=String(Math.round((hdg*180/Math.PI+360)%360)).padStart(3,'0');
    hVel.textContent=S.paused?'HOLD':S.drift.toFixed(1);
    hZoom.textContent=zoom.toFixed(2)+'x';
    hBodies.textContent=bodiesDrawn;
    hFps.textContent=fpsV;
    hJump.textContent=fade.active?'JUMPING…':
      (S.autoReroll?fmtRem(nextReroll-Date.now()):'—');
    if(updaters.ztzoom&&document.activeElement!==inputs.ztzoom){
      inputs.ztzoom.value=S.ztzoom;updaters.ztzoom();
    }
  }
  pruneT+=step;if(pruneT>1){pruneT=0;pruneChunks();}
}

/* ================================================================
   v8: rotation, floating, sprite blitting, signs, formations, new bodies
   ================================================================ */
/* ---- v9: 13 more ship hulls (indices 8-20) ---- */
const NEWSHIPS=[
 /* 8  needle racer   */ ['..DDHHHHHHHDD...','EEHHHHHHHHWAAHD.','..DDHHHHHHHDD...'],
 /* 9  strategic bomber */ ['DD............','.DDD..........','..DHHHDD......','EEHHHHHHHWAAD.','..DHHHDD......','.DDD..........','DD............'],
 /* 10 twin-boom raider */ ['EDHHHHDD....','..DDHHD.....','...DHWAADDD.','..DDHHD.....','EDHHHHDD....'],
 /* 11 container hauler */ ['.DDDDDDDDDDDDDD.','EDHHAHHAHHAHHWD.','EDHHAHHAHHAHHHDD','EDHHAHHAHHAHHHDD','EDHHAHHAHHAHHWD.','.DDDDDDDDDDDDDD.'],
 /* 12 escape pod     */ ['..DDDD..','.DHHHHD.','DHHWWHHD','EHHWWHHD','DHHHHHHD','.DHHHHD.','..DDDD..'],
 /* 13 salvage tug    */ ['..DD......','EDHHDDDDD.','EDHWHHHAAD','EDHHDDDDD.','..DD......'],
 /* 14 line cruiser   */ ['....DDDDDD..........','..DDHHHHHHDDDD......','.DHHHHHHHHHHHHDDDD..','EDHHAAHHHHWWHHHHHAAD','.DHHHHHHHHHHHHDDDD..','..DDHHHHHHDDDD......','....DDDDDD..........'],
 /* 15 stinger dart   */ ['.D.....','EDHD...','EHHWAAD','EDHD...','.D.....'],
 /* 16 twin-tank tanker */ ['.DDDDDDDDDDDD.','EDHHHHHHHHHHAD','EDDDDDDDDDDDWD','EDHHHHHHHHHHAD','.DDDDDDDDDDDD.'],
 /* 17 solar sailer   */ ['AA..........','AAA.........','AAAA........','AAAHHWHDDD..','AAAA........','AAA.........','AA..........'],
 /* 18 fleet carrier  */ ['......DDDDDDDDDDDD......','....DDHHHHHHHHHHHHDD....','..DDHHHWWHHHHHHWWHHHDD..','EEHHHHAAHHHHHHHHHHAAHHHD',
                          'EEHHHHHHHHHHHHHHHHHHHHDD','EEHHHHAAHHHHHHHHHHAAHHHD','..DDHHHWWHHHHHHWWHHHDD..','....DDHHHHHHHHHHHHDD....','......DDDDDDDDDDDD......'],
 /* 19 survey scout   */ ['..A.......','..A.......','.DHHD.....','EDHWHHDAAA','.DHHD.....','..D.......'],
 /* 20 pirate raider  */ ['D...DD......','DD.DHHDD....','EDDHHHHHHDD.','EDHWHHAHHHDD','EDDHHHHHHDD.','DD.DHHDD....','D...DD......']
];
NEWSHIPS.forEach(s=>SPRITES.push(s));
SHIPCLASS.push('NEEDLE RACER','STRATEGIC BOMBER','TWIN-BOOM RAIDER','CONTAINER HAULER','ESCAPE POD','SALVAGE TUG',
  'LINE CRUISER','STINGER DART','TWIN-TANK TANKER','SOLAR SAILER','FLEET CARRIER','SURVEY SCOUT','PIRATE RAIDER');
const SHIPKIND={fighter:[0,1,4,6,8,9,10,15,20],heavy:[2,3,7,11,14,16,18]};
const SHIPSPD={7:[9,8],18:[7,6],14:[12,14],17:[6,8],12:[8,8],5:[7,8],8:[36,34],15:[30,30],16:[10,10],11:[10,10],13:[12,10]};
/* ================================================================
   v10: PNG sprite registry (rocks, junk, satellites, anomalies, ships)
   ================================================================ */
const IMGS={};
function regImg(key,w,h,b64,o){
  const s=Object.assign({key,w,h,ready:false,cv:null,v:new Map(),eng:[],fc:'rgb(255,190,90)',glowC:[200,200,255]},o||{});
  IMGS[key]=s;
  const im=new Image();
  im.onload=()=>{const c=document.createElement('canvas');c.width=w;c.height=h;const g=c.getContext('2d');
    g.drawImage(im,0,0);s.cv=c;analyseImg(s,g);s.ready=true;};
  im.src='data:image/png;base64,'+b64;
}
/* engine anchors + flame colour from the sprite's left edge, glow colour from its saturated pixels */
function analyseImg(s,g){
  const w=s.w,h=s.h,d=g.getImageData(0,0,w,h).data;
  let x0=w;
  for(let y=0;y<h;y++)for(let x=0;x<w;x++)if(d[(y*w+x)*4+3]>=128&&x<x0)x0=x;
  const rows=[];
  for(let y=0;y<h;y++){let on=false;for(let x=x0;x<Math.min(w,x0+2);x++)if(d[(y*w+x)*4+3]>=128)on=true;rows.push(on);}
  const runs=[];let st=-1;
  for(let y=0;y<=h;y++){const on=y<h&&rows[y];if(on&&st<0)st=y;else if(!on&&st>=0){runs.push([st,y-1]);st=-1;}}
  runs.sort((a,b)=>(b[1]-b[0])-(a[1]-a[0]));
  s.eng=runs.slice(0,3).map(r=>[x0-w/2,(r[0]+r[1]+1)/2-h/2,r[1]-r[0]+1]);
  let best=-1,fc=[255,190,90];
  for(let y=0;y<h;y++)for(let x=x0;x<Math.min(w,x0+4);x++){
    const i=(y*w+x)*4;if(d[i+3]<128)continue;
    const mx=Math.max(d[i],d[i+1],d[i+2]),mn=Math.min(d[i],d[i+1],d[i+2]),q=(mx-mn)*mx;
    if(q>best){best=q;fc=[d[i],d[i+1],d[i+2]];}
  }
  s.fc='rgb('+fc.join(',')+')';
  let wr=0,wg=0,wb=0,ws=0;
  for(let i=0;i<d.length;i+=4){
    if(d[i+3]<128)continue;
    const mx=Math.max(d[i],d[i+1],d[i+2]),mn=Math.min(d[i],d[i+1],d[i+2]),k=(mx-mn+10)*mx/255;
    wr+=d[i]*k;wg+=d[i+1]*k;wb+=d[i+2]*k;ws+=k;
  }
  if(ws)s.glowC=[(wr/ws)|0,(wg/ws)|0,(wb/ws)|0];
}
regImg('a1',12,12,'iVBORw0KGgoAAAANSUhEUgAAAAwAAAAMCAYAAABWdVznAAABwklEQVR42m2LTWjScRyHn9/fv27O0pk7WGbowCYtWowIPAVWdFiH6BTEerttdIzOderQMTqNoNNuMejkithsLZEkNpxLHG1pL87l0v98mzr9dh8+x8/zeRQ98AwHJHznAe2uBbNVYzP2ieW3b9Thnzj9Y3Lj0TN5lS7Ii6zI7fc1mS2LvPzRkeGr9wRAAVy+NSmhu1McP32BEZ+ZlXSDz2tFJi66KFXbLG4K5kaRuclxTDenHsrE0xncXi8+l4n4ukFso0oo6EAEFlJ76HRRSnHSVkNTdi+dgyZau8lSsszCmkHQY8Vodoh+q6A0M1bVxu/qo4YDUyaTf9Iw+xhwHmOr1CEUHMRhsxBLV7DbdM74jqLK26TTOyQ/RtEtrW3swwGWv/wk4HdSyNUQ6TIyaGZ3x2ApkadarWMYLVStgNYwdpWrFMft82HUhPWiYn5+hbnZRX7ny/wxFOHrl3D2t/m7GkEDONj7x1Y0guRijJ7z4D47it5/hFPjY5wYarKRylLKZYCO0gGS8QTZ1a/80gzuh69QLbep51LkYn1kP7zmuz7EfsWgJ9ceP5fz0zOi1ID08vrhIRF5R6e1j0hd9Qr+A/iHyIdmOWqzAAAAAElFTkSuQmCC',{"lab":"ICE CHUNK"});
regImg('a10',14,14,'iVBORw0KGgoAAAANSUhEUgAAAA4AAAAOCAYAAAAfSC3RAAACBElEQVR42nWS3U/TYBjFf+/abnTduo5VxuhwODAIQUEJF2JijCF66x/qhRfemUhMjFeEhJAIBhgOWQZsfMx9dO3avl6YzMxk5/I8v4vnnBwYIys1JTOGLcfdYwCTZmEEUNW4XC2/xsk+GoFtqygN3ZIAqqYk5LP5LcIolPuVrzTbNXLpWQrZOVp3LXKpWSlFRLmwgm067B5t03XviA1CT+wcfEIEKhsP37I0/YJFZ51IBviBi65alPJLrJQ2qddrNFs1AMTwDb0sX669w0ybBGHAdavO6cV3slaWn7UjiuYy1et9rjoVMcwI4EyXkIrPt8OPVK8OyGVm2Fh8g9sZQKjgem063u2/HsrFNflgaplCdo5O/xbfDZGJGH7gISNIxtM8WdgkCCLsXIHz5rFsts+Ied0evXaf6q9TVGGwVFrHC/oIIYjHNZz8PJ7Xx0gaRJFE19JoSuJvRoEi86ky95L3iRnwuPycdreFkTRRFY36zTGHJ3uEMqDRqTAIPREDkISi6Z4xW1ygcXfB5533VGs/GPgeX3Y/0O7dgBpx3T1nEHqj5TxdeIUQCul4Dl2zsK08tcsTdMWk0WiS04vYKWd0OaY+KVOJSfYq21Rv9vCCDlEoUcUELfeS3/4VPb9F1pghoRn/z1AZMRanNuWqsyXVWHzoG5o1drtDZSZsOaGlx4J/AI1m1wvIJu4cAAAAAElFTkSuQmCC',{"lab":"SPINED BODY"});
regImg('a11',14,12,'iVBORw0KGgoAAAANSUhEUgAAAA4AAAAMCAYAAABSgIzaAAAB3ElEQVR42m2SS28SYRSGn5lvgJmhDBRooaUtRIuxFUmsiUnVuNGNO134V91rXLhw4SVeQhQCpUIplqHAfMNl5nNBqqn2XZ2cPCc5yftoXJFHewWVTdkEQcj72k+artT+ZS4tXty/oQpZh91cAkOApgva3QGvPtRxvTkfO+M/vH4xPD8sq6cHRXx3wNib0uq6TKTP+UiSEDp3immeHeyoNcdSAAbAvXJePbm9zcvXn/CnAbGoyfZWliAM6fRdSltZru3kmfk+9ZMhp+dyefi4usN0scA2LYrrCTRdoaHQhODB3ZsoFJoGjdYZTnT5rfHw1rbaSMcJger+Fr6cE7ejCMNA06HV7mFZUXQjwniqqJRydIa+0nNJm/l0xtdak4E7IpO2aRyd8ObdN6Sck0klSDsrBLMFld1NpCeJRQRGrdnDmnuYZoT8WprReMaXepdiPo0ZEaiIwefvx4wnHpu5FJ6UCCGWdVRycXV4fZVSscAigFCFJCyT496A7umA4Viy6tiEoaLtSup9+bfH6saKKmVsbENnr1xg5M2Imxb1oy6e73MmF/zoS1qDpQyXBEjGhNrfSLC+EsOJx1AqZCIX9CZT3jZcAqW0K825iBMTajNpEhEak1lI/Zf3H/cbw2fKuMKdpK4AAAAASUVORK5CYII=',{"lab":"BASALT ROCK"});
regImg('a12',22,13,'iVBORw0KGgoAAAANSUhEUgAAABYAAAANCAYAAACtpZ5jAAADHElEQVR42qWUT2gcdRzFP7+Z2ZlZN7vJ7mab7G6arrslmrTWmGCVYJSCYKWgd/9UpEehnsSjNw8FDx4VehVR8CQIUgXRi61NE0mbLmmav7vZdv8km2SyM7Oz8/UQKFZv+m6P7+MdHu994T8jJsOWLqMjOfngXEH+eTX+Tsy+lAS+i2n2IaqHt99Uo+NnxbZNWg+qNKr3FcD7M8dlZkJnphThyz+LfDjjEwt35covu+ox48nnX5DBXInxZ89gWxFMK8rcjT9YXLgt8USatfI82eMFGtX7AJS3mlx5K4sV7ZGwoBX4fPruIM0OcvX6kbkBkBstMJgvUCyMcurUafyuz93F21RWFqiaFlOlNKsbCwwVihKPWtTqG2zeC8lHfZQbEFZcOsMdProwxDfXd2UflAGwsl5nvxvjZCGFNH00I4Yf+DyZzzM8NERRHJ6Z6GfZTBNqBr+vVbn49UO+u5jB0EGzDA4aAb7n8OLsNJVdTwyAdGaYRDqH1lknUf8J1ywiukU8cOh3q7SyZ6is32N+6RpgMHWin8tnLXR8ghCUCIap0XN6vPbqOe7Udo6iKC/dotDt0TpxmquLGnZ8AOlWqKokveRT7NQP2dt1ARQEvDlhy3uvxFjddBBABCQEy7Ior9S4s1HFONaflHh7k6CRwmESLdIhbth4KsKsV+OCbfDFcoWt7ZYqjT0tiXiMT364heMl+fjlPjSl0JROJALbGz0eOhFcsTCSfTalgUGqPQ8tVIifwHVtIr0G11yDHSdLcUgnFrFkP5Umkx1h6e4qGiHogtcN8MIezXaE6gOLWmORtnOAUa5sq003IzMvTWPqEQ7UAFEzQSLRJvT3+PnXHx91k7XtR0PIRPNsbdnsdXTmliDrmXT0KIs3b+CA0gBS8X4ODjwa9RaxqI2hhFq9xXTmCS49lxWAz96ZlaOMUZcmR3HbMcorCi8QDg+FdNLk23IF50iDemyGdkKCIIAwROmCdD01NTwg0/kUc402N9ebCuD18Zy8UbQoHVN8VRnh7TH4fn6Zz3/bVvxfmDFTTuYsyWezcvn82L9+xV+X4lTHmVz8JwAAAABJRU5ErkJggg==',{"lab":"SALVAGED MODULE"});
regImg('a154',20,17,'iVBORw0KGgoAAAANSUhEUgAAABQAAAARCAYAAADdRIy+AAADiklEQVR42o2UW28aVxSFvzPAzIC5eQzY2BCIb7XsKmmaJlFSVZWqPFbKX+pPax9SRXGrpmqayhdsjLGN8cDAXBlg5vQhVZsqadT1trW1Pu29HpbgI1rf3JG3tzZwbAdrMOT44AgpQ/ExzweXjea2vHv/c7b3dlF1DcMoctXtcdY+49X+S87bR0gpmM/nxPH8X4zku0O11pAPHz9hY3uHdEbH812CoY8zGlOtLSNlzCSYU1qus1yt4I5t3rz+TV51j8V7F97e3pXfPnuG54e82v+VerPGxnYToSioaoo4lvz04hXTcIoiEhilApXKEubA5sXz77loHwoA5S+e/ObpU2aR5OXzn1lrrBHFEZNJiG2NUTWVXG6BQi7Lam2FB4/v0O1ccnXVR1UVKpUayaQu/wZWV+tM5xHu2MEoLaJn0qzWqvhewMC08ByPy/NLlpcNFKDd7pJMKLiOi+N4CJlisbT8T4afffEQRSaYyinNjTrl8hKBH3BxPaRUNhjcDDk/u0DVVCZ+SK6QpbnR4PS4Q783QEYCId+mpywsFGTRKLO1s47v+ITBlN5lj/bJObqaQkslOO9cYpQMmuu30DMaruNxfW0ysmwymQz5xSzlah0hUlJZazRJplJ4QYCUMB6NsUcOqp4indWxrDHFYoGd3U3WNxvcubdHOq1zcthmZa3Mk6/vIxTQtQylSpVESs1+d6vZ5PDNEZqmUWuscvf+p9gjh5Fl8+irB+i6hut6BMGE1sEppjlg0SigqiqxlJwenZHW03iujZJQEpj9ESPLpVpbIY5ijg9PSaczRPOITquD53gsZDMMzSHdzgX5QhZN02i3uvz+ywEpVcOxHcLJhGQY+NjWiHwhx48/7NPcrJNMJGi3ujz68h5+MGE6m9G/vsF3A0qVMv3rG8IgRAAraxWCIMS2LEbjHknT7AoEcvfO29fOTrpE8whFCIbmkE/2tjj8o8Wb1y1UNYGMIQynGEtFwnCI2TcpLhYJJj7RbCqSAOZNV5ydGHKpsko0i0gkBLGUdDtXRHFM/9okX1hASSgIIDlJYtseipKkaBjYI5vLbuv9cigursl6c4u0nmE8ttE0jVwhQxzHhOGMXH6B6WyO70wYWX0Cf4yq6jjOiLHVEx9sm3yxLAsFA9d1ECiUK3Vy+SK+56MkFSaBjz3u0++div9dX+8qnS7IbK6IEIIojnHtAWHo/afvT21wtRm5x4vqAAAAAElFTkSuQmCC',{"lab":"GREY CRATERED ROCK"});
regImg('a159',19,16,'iVBORw0KGgoAAAANSUhEUgAAABMAAAAQCAYAAAD0xERiAAADjUlEQVR42n2U3U9bBRyGn9NTWmih32tpoWUr3xufyhwIYWKYom6ZGo1LlhhvTPTK/8Bw4x9gvDHZ1Vyym5kt6vxY3IbJdAgbYAbCWhhdaYWW0g9LSzk9Ped4pdmy4Hv35pfnvXt+AgfEKFZpgz2ttAX9yJUKofUNHqzFKZZKwkHMM4eujoD2XHsLI73tyFKFG7eW6O8NEmyysraW5K9UmmQ+z1Yqy3w48hSvf7J8cu605rE6aPA6sNRVkdwpUu0UMZtFDHo916ZmONbSxAdnXmY7med454b2y9wSoXhcAND9O/Te+IjmNtn44tJ1vrp2m3K5gtNl5p3xIe7Or+BwWnjz1BAt/nqyu3k+v3wdpaRwdnSAWlO19t9YW6BeC7jdfHbxCjtyjrfODLIRT7MajiNoKkd8Pu5Mhzja6sff4EZRdURzSS7+PEUut8dLfV0AiABnx16YDPo8LD+O8fG5CWYXH2Ewigx0N3Pp61+Zmn/A3T8ekkkVcTtrsZiNtAYaWY7GkcsKbX4/98Ork7oXe9q17mY/qk7ly08/opDZ5+rNu2yn8qyub/PjvTl28rsYDHr0oohoMHDlh1lkSWK07yjvvjKILCn4XR7E4d72yTZ/A+VyheWVGBoaXU1+Rp7v4OrN30lkswy1duC1OOntbEIQVA45a1kIRYhuZHDYzOzulxDQIXrtzslcep9wLE5dbQ0Oi5mRE51UFJV4IkMhs4/H4eDEQCfdxxrZ3EojVRSspjr625uZ+TOEQRTJlyQEs9GovT08TCSxxemT/cwtRZB0Mn3Bw3S2BYjF0uh0GsbqKiqyyoVvfiKd2+f9iTGW1xKoqkz7YS8Xvr+FvihJQmwnpb023EehIKEqIt/emWZ7Zxen1cJiJIqqSBzx+Lg89RvxzQznT41it5hJ5jIEfS6ml8MUpaKgA1hcf0QsnufG9BJmm54PX30dUdURT2SIbOxQ2FMolGS8tQ7Oj5/EZbNwfymKva6GKtHE4vrG0zpNHB/QvLZD3F5Y4I2RAVx2E9nMHqoqACKFUgm3w8y+JFHYK/PdzD06GwJIisxseEV4xs2xvh7NYbIjyyrFcgGP3YmxSs9KLIrZZMBnc/M4kSTo85LI/k22kGM29FA4UHRTtUELuOppcHlwWa3oBIFQLIaqaLQ2NrKdzxBPpsgVc6QLBeF/v8aTMRtrNJfVjtfpoqJUiCY2SeVzBzL/AJ9niOX4fsr3AAAAAElFTkSuQmCC',{"lab":"ASTEROID"});
regImg('a166',14,13,'iVBORw0KGgoAAAANSUhEUgAAAA4AAAANCAYAAACZ3F9/AAACDElEQVR42mWSXW/SYACFn7f0k9KClBDA6eZcJBgz92GM0Rg1XviXjUbdH3ALc2HEMYZOmI61TSwIbYHXC29Ez91JnnN1HsE/ESIjV2+us7JSZZpM+X4x5Fu/B1KKJe7vsnG7ITc3t7BMHc/zODxqUfJK6IZGu31Ms3nAfB6LpWGjsS13trf48OEts4XCk8ePuHF9hZPOKWdnXVbX1gnCiOPWJ0ajoVAA8vmS3N3ZRSAxrSJ3G5vY2RwIuByGfDm/5OioTa1aoeiVAVABHuw+JAwDrvyAly+eEscTVF3n13hCvb5BNIoZjcYEYYCdzeLkPJmpVFbl+q01FCVDNmuxWMxwXJdxFNHp9qiUSxS9PK6bwysWOD8fkMwSFCHg/d4ejpMjCENev3nHx/0DdMOgWi0Tpymt9gm2rdPt9jBMEyEUFE2zqNXWGA59FEXFNK/hOgWiaEwcp3w+6ZARKqNoQuv4AMPQMQwL4bhlWb9zD01TKBTyGLoGAsbjCc3mPr5/ybPnr1jM5wwuBiBV+v3TP3fUKhvStLOYZhbbMomTFFXTmM+maJqJ4zr0+wOSJAUJvd6hUAGG/oB66T6IDJ3TLtNpSLm8hmGZzOYTVFVHSsgocOX/WDYnZxek7eQplWpIuUBRNC4GX9FNDdsuEP0MCIMhSTIS/ykHYBiGNA0X3bBYSNBUBd/vk6bpEvsb873U0ND9HAMAAAAASUVORK5CYII=',{"lab":"DARK CRATERED ROCK"});
regImg('a167',17,12,'iVBORw0KGgoAAAANSUhEUgAAABEAAAAMCAYAAACEJVa/AAACWklEQVR42nWSS08TcQDEf/toy+62PGqhLigWEgKCBLwIJEYT4yt60YtXE+PX8OC38WS8GDUx4OOCMQFBi0oiSilNu6Vv2O12H38PRBNF5ziZmUwmI/EfHEsYYnZ6Ck9PUC9ViEZUfGefD5+/SH9rjxBjQ0Ni5NQJTpqD0HA43bFQcfEDiaxusuc5rGa/ki+XpX+GXJ87Jy5dmCeIRNGWnnBz1GLUlJDHuvC/tVne0nhqXMUNQx4vvmJ7Ny8BKL8CbizMiWtXLhLt68GxbVqBTBjX0P02CaNDOKgxkoT2ZpFdI4MMbOZyD383mcxkxL07t4j19xJ4AYoi4wRg2wFK3UKzCtyfWWOjnia73Y2zX2DZG+HN2hp5qyCpAPNnp6lXqsQ8Dy2hk13NYqa6GZgcJ+wxsfoGebGRw9RKLIVTJEOX29o2tpkkbxWQh/vTIqHFUQ0dt9nk2fN3FDe/I1f38EIZ4QvibpW3wQSlssKC85F89CQrjSiX+yUGkqZQ45qObbdRWipdepzxWI275wNeGhk826FdreHZNmpU43VzhIkwT71UZMVXySkKPVoUte26NGpN/CBA1jzMVC/vlW4OOhArlzlo2TidkL2GjbrvMj4UEORbuIpCGRXPcw+HPTM8KobTg+hGhEiiDyEEPV0yqizTOHCoVhrYjo3VjvBgtorbknlUSlGoFFj/sXU47KfcluR6HZFOpsCq0tudINANHMfB7/gEYUhfJMKYanO8y6Fea1LcsVi3qtKRs6mqKhRJwkymEAIkSUKWZTqhxIwRkorYLBbq7LTEH76fMMMVqMYZZUAAAAAASUVORK5CYII=',{"lab":"VOLCANIC ROCK"});
regImg('a168',12,10,'iVBORw0KGgoAAAANSUhEUgAAAAwAAAAKCAYAAACALL/6AAABeklEQVR42l2RXU/TABiFn34staEVMRiC65ZsUZkTEyFDQgARI8H4FY36f/g/3nrhrZKgyS7ExGVjWQpbQ7GOUoazZrr19WoKPlfnJOfcnAOnWJkvyeLcrAx9LpuWuwu35XRGH4pXj9blwdoKTbcFfVUUDV4+uUcYxXSiWD7Vvih/CyndlAv2GPueT7Ppk9JNfqPwobyDu3eAmrLIZ/Lieq6ipScnZG1pic9Vjyjs0PDa2Octnj9e5mO5wmVnnBdP79D/+YvGXmtDm5kubjx7uMqgF+N96zA1lUUXIe7GHIYnJAnE3S6H0Q8GyQDV99tUKrt8DU5YnLvOcqkAwNZ2nauFDJZlslmukSSCAigAC7dK0h+ojI1aXLRNmkHE+v15DFWo11vUdgOOvx9RbVQVZbhSdnJCdN3CSTsYuoZzaZSqG2DbJr1en/fld/9WKuRzcvPaDV6/fUP7KCSfcTinaxiGRtD2CaOIM+QyWSleKZ45yDBMsUdGhP/4A+Iul9a7aEUKAAAAAElFTkSuQmCC',{"lab":"RUBBLE CLUSTER"});
regImg('a169',22,14,'iVBORw0KGgoAAAANSUhEUgAAABYAAAAOCAYAAAArMezNAAADBElEQVR42p2U2W8bdRRGz3g8tsceO+MtjhuSOntJA6FBhJaqb4gn+gB/KVKfK4GQkBAICWhIszlJ7XhLYo/Hs28/HhCVKnbu65GOPl3d+0n8zykWCmKttUpzfgHTMHlxcszUmki/c+m/Ct/d3BFLi3d4/PhDHNvDNlyqdR3Pczk4PObw7IiX7WMp/U+iuWJR+G7IemuVJx89Yn1lBdfzuOmPmYyn5DWVSCRcdIZoWY0ne48YjW/En4rLpbLYf28XNVdgubGIkk1T1ucolUscvTwnLcvUGxUqdR3Lcvji2XN8N+Dh+zvIskyzuvDmKrZbW+Lh/ge07i4xMx3O2x1yGYW9/R0kWeJ6cAtC4Dge5sxmvlbl629+QM0obGy08IMA1/E5eXXC68Sfffyp2L3/NvVmFd+P+OrL78lnFYp5lfGtgRAC1/JIhMC2XNKyzOnpBY2aztp6i+P2JYQJpVKBMIp/S/z5J0/F/t4eM8ciLUtcda+pVkokAvSyjjGeYFsuk4mJSMG9rVUOfzlDCEESR8QSkAjiOGYytThoH5BerDbFztYWkQhxbY8XB23UjMI7u5uY0xndiyumpk0cxyQI1jeW6fWHyEh4UcjYmKHPaYRxRByA7TgY1ph0s3EHY2xSTDSsmcXCvM697TU6rwZcnHZJKSmq9TJZWSYlJDrnfXzPp9sfYdoOBTVHNpshqygMJyN6tz2SJJHSIBgbM8I4ol6rYM0cvvv2Z2zbZXPjLrW6ThjH+K5P6EWkUymOrvoEQYSm5klEgmGadIcdBrcDYhFKANJSY0k82HpAtVrC9wKCIEQt5CgV82iayo8/neDaHotvNTBMi25vQLGgoao5hjcjznrnhKFHGPtvXFi6M+xImjonEtEiCELiKCIIQw6P20hI3N9ep1op0ekOGU+mbKwsczUYcXR+ROf68i8/9zXQtYpYqDRp1OZxXQ8SCa2Yx3FdspkMxnRGrTZHbzTg9PKEUAR/Wwd/gIqSFRklh54vkZZT+H6AJMlIKejfDhGE/6pffgU41YKR+ZSNbgAAAABJRU5ErkJggg==',{"lab":"ELONGATED CHONDRITE"});
regImg('a188',10,9,'iVBORw0KGgoAAAANSUhEUgAAAAoAAAAJCAYAAAALpr0TAAABNElEQVR42l3Q2ytDcQDA8e/ZTnlgl0aYsSK3UHKLbC9TTIYSXrx59Af4g7xIPClhYQ9zzaWMmjDWNsO25rI255xmfp5Ivs+fp6/Ev3SyTbgmxojFUigfWWIXXglA/ovc03PC4Rnh7jqCqo/icDpYWzKJ4N6yJP2goZl5MTw5hSDPysIiRrMFnR4e4ymyiRB6AOforLA2tLOzuYXZXIp73IV/+4BkMklTazN5LYeurc8jTJV1vCQSdHR3Yq+vIXQbofCZp8JahaaoBA53kUss1fQ4+1EVjexrGt/qOucnZ5RbyzCbjOx7N8hnopIcD9+g5DIc+fykkilURcVgMNLY2kYoGCT79gCABFBc0SVs9lp6Bwe4PD7lOXyP9vHO61MA+JJ+IYCpvEUIuYiCppJLX0n//34Dea18Cjq2ya8AAAAASUVORK5CYII=',{"lab":"AZURITE SHARD"});
regImg('a189',16,15,'iVBORw0KGgoAAAANSUhEUgAAABAAAAAPCAYAAADtc08vAAACmElEQVR42n2TzW4bZQBFz8x8nonH4xk7sdM4qWOCElrSRqUgsQAkXoA1D8AbsmOFVJYVEmpJQ2KTJsGuk/HPeP4//38sygIR4C6vrs7qHo3/ycmjD9X+XoP+7YCr7i1hmmr/3NwrXMdRrcYDDpo7PP3ogFZrl19P2/QHAcNgwsVVj0Ew0e4Bjo8O1OH+HrWKx3K1YiqnfPr8Y07fdDCEwYZVwB+E1DY9Otc9Xvz8SgMQANubFfXdt9/Q2Knzy+szQCPLJe3ODaZlkWc5o3GEWise1KtsVcvc+iPV7r7T3gOqHhcXl5ydv6VeqyAKAgDT0IlSiS4Mms0dZD6lfdlFsUat1xQMQwmAUtHm9W9vccs2SZYSTBK+/uITrC2PRc/HWpnEUUoUJzQbdfp+gCkKlItFjGePj9RXn5+QySlhmJDEKUkqmS9WnLevKTtF3LKNZVqUSjbnnRsur/t4ZZs4yxCeY9Pt3lF1yxx+0GR3t8aPL14ihOD5ySN0Q2e7XuPqps+r0w53/piK5yB0ndV6jRiOA/a3t8ik5PfLkMV8xoZpEkYRxsEenuvy/Q8/0evdIgyDx4cPKVoW/miCKAjEYByQ5TlyumA0iXDsDXIp0XSNN2cdrrs+4yDiy8+OmcQZVz0fz7FJM0mSZ+jjONNGUUKYZLiuwzhMqHouTslGLddkmcQtl5DzOcvVkr16ldbDbWaLJbPZEh3gD39EmuUYKOpbHlGUEkxi7FIRu7jBfL5gNltwNwpJ8xlDPyRMUlZKvf/BTd/X3GJRpVJyXCgwDEIMQydOM5JcUnFLWAWBKQwG4wlxLgn+8uKeC1XbVgrY9Mo8OWoxHEXk0xmplHTvBhgFg+l8qf2nTH9PY7OqLNMkiGJiKf91+ycChUeWgfuF8gAAAABJRU5ErkJggg==',{"lab":"SANDSTONE ROCK"});
regImg('a194',14,14,'iVBORw0KGgoAAAANSUhEUgAAAA4AAAAOCAYAAAAfSC3RAAACSklEQVR42n2S3U9SAQDFf5d7AQFBwDADhpKA0JyKplZa8WClzh5aW3+P/TM99OBDrbZWc1k5Yzb7lAnmkJABmsiHaJePe3twurlc5+ls55yn84P/qD8cVsdvRdTzMunETM3OqIn1Ter1Gt1eL+l0kgePHpJIpPm08kWVj4rCP0Nja6t6b3qWYF+alehXJqcjpFIpCoUK26ksVkcXvf6ranY7zUY8LgCIAPVabc5sc3Bn8iZNRWH5wyqWtjasFgMWmwW9Tkt/uI/RsetkMrm5wu+dxxLAQHiYCw47a7GfPJt/TbFUwuXuRFEEdJIGq9XEUVVmKxljK5kDQNPl7VF9wRCSKNJoNJB0OiYiIxhNLeR2d3F5nARCPSgorK9t0ul0AajSwNAggYCPWk2mVCwzNRNBI2pIpdIEgz08n3+D0aSnN+TD0KrD0+1HJ9aRokvLaHUmmk2BZrOO3qAlm9nj7tQES4urLC1GcXldDA0PIIoiiiJQPZQRWk0W1ekN0WzAzP3bmC1mctk8NruVVy/eI0oaro0PkoglSf/KU65U2MnE0RxUy8JhZR+v30293uT75zgut5O3C8vY2i2ErgRIp/LEYhtIehGtKAN1QTq+o8To2BDvFqJIkoanT17S0WmjXCpTk2WUJlz2e6hUZETpmINTGsIjN9TEehKL1YE/2M1BpYq5zYyxRUfsxwYXL3VwUC4R+/ZRODM8ka3drVrtHXh9Hnbz+xT2yiiKQrmYpVrZOe0L5wGs1RpUo9lOQ27QUGrU/hyiqvKZ7l8e1vCEKrTCoQAAAABJRU5ErkJggg==',{"lab":"CRATERED IRON"});
regImg('a2',14,13,'iVBORw0KGgoAAAANSUhEUgAAAA4AAAANCAYAAACZ3F9/AAACOElEQVR42nWSW0iTAQCFv//fP+fc3CWl1DLbzFvmyFBThAgqErTCIp8Ue+glKOihx6jnnnuICoIeCjV8kZoFGpWRXexiXsqZuU23fzed011Yc/49CGKF5+1wznk4hwNb4Fz9GaXR0qJspUubSQaFiqAJcLb2PBebDzM/pyP31S5lLDjIr8C0sNm7QZqOmZSrJ64w+CyP2spq4qkJ6lrsPLpbgd+rx3ywi5/yMj19M8JGUJeNMnS/jYX3HTjcKQIrfgqNe1E0HjzBRfSrNZxuH8Fqm+JkZxj7aI8gAZhUVl4+bMVQ4KH9Wh9pWaH7QSvE9lBVnENm/mN6uxspf1dNJHoTABGg3lpDciWLlDiCQRwmHnLiXQwgZqyRjGXinP1NSlhAXkiQiKvWOzaU1CodDW1kSxVMOifYtrOfdMyGEj2OxZogkdIQX84kGPHhi81iNsbo/fwEUbWWy/gPNd7YFypLLMizFxBTRyizLXHq0i10piHcXi91TV+5fsOOXxYwqguR3sz0Y5aK0AUthOQQB8rMJKIG8koHWHGFGf6wilMOUOSK4HcUEIom8S2711dtLK5SBm5vx/46n/Fvu9EopQSXwshhH7osLbZyF73Ps9FqTTjjT/ke+CSIAONeB/e6YgQDaRwzRibnHChqGaQIqbSMpEpSs0/N5c63BJOjfx8ARGWHZj+5Bh052mKaD2nQqCVefNQx5pniaIMLlWaOO/aI8E9w0/VUeiU/qxJthp5o2s380vR/vj+jpuceZ4xDCQAAAABJRU5ErkJggg==',{"lab":"AMETHYST GEODE"});
regImg('a3',14,13,'iVBORw0KGgoAAAANSUhEUgAAAA4AAAANCAYAAACZ3F9/AAACIUlEQVR42oWSTUgUcRjGf7Pzn1l3XXXNylp31U1WI7IP0ECCCMxCskOIx4JuYdCls9El6BRBduvQrYQwIuws1GpQWZnhR2X50ZbJ7rqtq+PszLwdhCgo+t0eHnjged8H/kEkUibX+hJyumuH/M1Xv4vLZ4JimDq7QkLn2SYqD9QguTUe3SrI6DTcGLCwikXtj4S+3loROSUy1yKSrBB5oIk10yr2ZLPIs7iIfVQe9teIqRAADaCrGRno30n+vUKtpyiPengFQWusw2jeR+7VEmtD4wSDimPXi7xMb2g+gEu9MfRJl9zTBcqiLq6mUHsTqFgcd3KKkL6AFnZx5z3a45uVte5DIbn/uA3XVeRHFimNWqhYBAHsZBJdc/DVNuCkc6y+zlCoNmi5WEQdOWjDpzdYoQRGtY3RmMD7voQ7PYGvrh69PoZvo8D6dIpgk0FVAM61maiJKR3vcw5rZRSzqgq7shRvbAw93oDZEMMbf8vsYJq8o0h0h7Gfr+P74mwep/9kiVzosXH2VKMVbLxMhtUVk+x8EWsWAn4/sRMOqWEhNWdyfsRFB9B/6Fc6TAMjnCX7wSY74WfrboeSckGv8OM6LqaCbSrIzRcOg7MbmgIYWi5y72MpPVJGKKxhf4Vvw7D9sENIPLS8zlzS5MmycOedza8/buKTzqhi/xaNxkqDjoiPQMAjnYG7c3B7xmKx4Gj8j/YaQ662lsjxmPrrVn8Ch9Hn6bWloecAAAAASUVORK5CYII=',{"lab":"AMBER GEODE"});
regImg('a4',14,13,'iVBORw0KGgoAAAANSUhEUgAAAA4AAAANCAYAAACZ3F9/AAAB0UlEQVR42nWSy07bQAAAZ9frR7DjOKQhKbQpTSWkSr0h9fv6bxX/ACoHUENEiB0Sv2JnbW9vVK1gTnOY48D/CMucnc5NEBwbUIY3EH9VmtlsThAMCMMhbdtxf3fL03oJgHIUzaF66RWAELa5vPzO+GTCzfUtPU9TlDldC8fDKUYYzs8/87R6MMvlAmO0UCDNxcU3/CDi6ucVVVXR81w2SUJrWmazT2RZTl1pvsy/IoRisfhllOv0iKJjbq6v0YcD76enuJ5HWewZT8fYyiFer5HCwpKCYTRitVpg9bzBD0tajEbv6IchYTSkKAp6Rx5FVhCGIUHQ5/k5wXVdOgzpbodwndA4tsf4ZMJkMqVpG5pG4zguWmuklHSmQx9qXNfl/u6ePN+i6kMqOqNNmh0RDgbsdlueNxt83+fsw0eEgOXvB0zXoZSi0Q2daZEAWlcY07Ddblg9PqIsmyzNSZKEstxTZAVSKoqiJC821HUuFICUNm3TEgQhAkkSxwgpsKQkjtd4noNtK+J4h2724p8BbPvIhP0RUTSka1t6gU+eZqRpimVZlGXKvtqKV84BpTzjun0CPwAjOGiNbirKIqUztXhjuReMZTm4jk/TNRzq7LWGP8zt4N9QG6y/AAAAAElFTkSuQmCC',{"lab":"CARBONACEOUS ROCK"});
regImg('a437',21,20,'iVBORw0KGgoAAAANSUhEUgAAABUAAAAUCAYAAABiS3YzAAAEU0lEQVR42o2VyW8bBRSHv1ns8czES5w4TtzGdpOSNCQtBaUg2qKqSPSChCoQEjf+O05cuFecikho0xZMmrZOnKVpFu+e8WyejUMlKDRI/R2fnj69d3jfE3iH6MWleKoyR3pygu7RMZ36Gr5jEUSRcFa/cDZGjEVZRh2vUlq+wcrtOwiSjB+EeIZBv9tBDwb8ee8HWicHbzHk/xYqq1/Fl27fRZZ8hgOXUZTAMDyiUZ/2boMgFskWp5lL7uKVK5jWKHbNE+F/oXq+Gn9693u04zX8/Q3OL92iZhao/76JKossfLiCEg0w13/C9E+IwxyF6iqh3Yk7pzt4w6YAIL4JnVm+idZ7xljjZ87nVVL5GdTOJjelB1y/OsH0pMIlY42PywnM1DlMvYxMxGJlhhsr1bcnvXjjm3isME/bhPzlLxhkKtRrLyh11vnoSpWUdkDQ3yVUIZmdRjVdomCGXPsZC5pCZibP1k4+Pm51BRlg6cq1+M7X39E6esWTBzUOZsqo1gZz9h98cHWBiJiOI1FUAo4sn/7BCSU1w4uei5DKICclZktF9FSSv9e/9v4ier+Bf7LF6rxCvvWQUtJA1xUaL/Z5ygJb47dodgxqtacMStdh/jMy2FiTy+x3QwzLRVQmXkNlQYjd8QvUTSiHB1Q45NLnXxJf/ZbNXpad1DLPg1n83il77RBtbhW3/AnKyQYL4j7h6S6ZdJoff22SzJbJl5ZjsVop03Cy9MUcQ0EjQsZHZO/RE/bDWZr6AikFOkevaMsFbClDyawRtF9y2BwgChGPDkNajkigTiIIMWIuJbEwmyF0bVr9EaHvIOw/RpRjUprCVDGPpqUQJi9QKaj0ttYZbf+GH8b0fYWhG+AEEbnSHIKYIApGyNagg7l5Hy9ZIh33MewkRbVJ7AhoxVlCQaJnhVTdOoPt+yxV8jT7PrutBMmkiJZMo09XcdtdQuMQz/eQ99oG5w6fI6U6oEHHlnEij9FwSKBHvHrykILi0bf3yKRyuKaHELi4ZBHGz3Hx8jUa67+QNusghWwPusiOHwuW0YoTsUzNUREFOBn6LE01OT20yTKg4A7ZHIxhhiKLWWjF5/EyJRKJBEf1Blq/zrQ+onZkQWgJIsDDvRYFdYQUe1gBpKSQ+UmJz8smAiHbdo7lGYX5rEdeGiIGLoHdx2lsoDTuMZv2MaIUzV7vn4vyXVM47Fnxxakk+wMLfSxguyMzkRwhhR4rJY3FSYm1nYDtnkTXHyInAuaXFnGtIcfWiN5JDdc8Ff51po9329hinrQUEYchjtGjHiYojYWkowGPnnsYnoKTKJKRXNRkwEjS8Y197CDJUfP4bJ8q+mT83oV5fJLkVIVzqokqOpiuwOlIw0bHjnUIbCZSAYHTp2sMafZM7Df0d4ak5VjPFdDHZ9FTGjN6hCto9EYCw+5LRo6JO+ziuR6yDEEwEt7R/K8jJdKxlp1CTshEgcOgdSC8y/v5Cy/YHdwfH6k8AAAAAElFTkSuQmCC',{"lab":"OCEANIC PLANETOID"});
regImg('a441',20,20,'iVBORw0KGgoAAAANSUhEUgAAABQAAAAUCAYAAACNiR0NAAAEcklEQVR42oWUS2wbZRSFvxmPx/bM2B47tuM0ie0kbdOmkKZKRSBqqVoJqoqyQGLFhg0LNrBlgYRSlkgItixgwxpaISSQqoJaBK1IaJqkD4jTPOrYeTh+P2f8+FlUrShQOKt7dXWOdKSrT+K/Jc4cHSYSMKjUm6xv5bmT2gOQnmb414OmKmL6SJzzJ8dBltnLV/F6ZHr8OtcXVllYySCrTubupej+LeOJZSRqitdOHWV/fw/hoI/dYo33Pr3EuenDvDCRYK9Qp91qsS/kR8gO0tkCP84lmU2mH+c4Hg0DIZ94/82XuLu+x/L6DmFTQ3OrBHQ3L08fpliqcXV2hcmxGLPJDJ9/cwNJCM4eP0CpWpvZKtQuAMiPAs8/dxC/7iJbKDN1JE40EkA33Lxyepx8voRAxqu7GO7vIV+s4lWdhL0ekps5zhzb/7ilDBDx6aIv5Of2cooP3jpLrtrknU8u8u5HX/H1lUXCYZNqvUmj2eaXhfu8/fqLxAbDtNtt+kIGzZbAcKniceVzUwdnTk7up9OVWFzd5uMvr9CjexjpC1Kv1IgPRBiImNi2RSZXxk2b5ydGiIRM8oUyS8kMVatFpWFfkAeDhnj1xBjfXl1Cc6ts7BTxaW5GB0MkoiaTY/20WjbZQomRoSi5QgOf38/3127z3dVFNrYK7FYaDEUDDyuPxiNIkkwylaNQqjISDdAf8mO4nfh0ld5IkFrDBgHNhs0b5ya5mdzixr0UzY5gYW2XkajJ2GAIp4xwxCLmTCzk5dTkAYqVGnQFA9EAmupganyIVkfwx0qGfKWBqipIQnA3mWJ6fBinSyHq12haHdxOB41WB0WRJUpVix9+m2cg7KMjwKerDMcjLK1kkCSJA7EIl366TdCvU6xbKA6FWsNCbtn0BQ0u31pCU52YhhsltVtCdsgYXjc9fh1F89AX9DG/tEILCd3QaJVrJKImmtvBaKIXVXaQ3Nim24WljV0UBGMDPaxlyzhylcaFYqk+c2oiQSpf5bOL10ll8pyeGsUpd/l1cZ1qrckzI1H6IiaW3eHnOxt0Ox1kBJt7VXoDXqJBg8W17Ydvo6nKTDxsUq5ZGIqDoOFibjnNsdFBAoaHO6s7mF4PBxN9fPjFZa4trWG6nCiKQr7cQHU6WU7nyBQqkgxgdTrM3ttEQTBxaABTV0n0BlhJ5bj5e5rdSpNOu8v1W6vsFmscH+7jUDxCsWYhKTLpfJnV3cKTcDgxFhdd22biUIz+sB8h4Nr8CvOrWxwd6mUoYpLOltE0Nz6fh81siUKphu5SKNRsFjZ2pCcCxxNR4fWotCwLQ3PTFfAgWyLe6+dILMxyOk+5btOyOxgeJ2XLJujVkYTgxh8PqFpt6R/4Gt3XIw7HwrTtNtlyg8S+AF6XyuVbK9StNqau4VQcxEIGskOiXm8zd3+LqmVJTwVs1KeL3qDBdqFKj9dDPBKgabdxOSTuZfI8G49g2S3upvZottrkqg3pf4n9V4V9HmG4XJTrFg5FxinLZAoVxFO8fwJJ5OIfgtUztwAAAABJRU5ErkJggg==',{"lab":"CRATERED BROWN ROCK"});
regImg('a442',18,20,'iVBORw0KGgoAAAANSUhEUgAAABIAAAAUCAYAAACAl21KAAADqElEQVR42p2U60+bdRTHP8/TFlpupRRooUK5lVu5ZJARTNCNYLJ53dT4ZosvZtT4xr/HxGhMjInG6MzIoji5OGQ6bilsjMKQUmgpA3tvn97bn6/EdPoCPS9P8v2c70nO90j8vxLWpgZGh/tIptJMTs1L6v+itlrrhb2jg76eNvrsnaTiMXRlFSytOcWZQe9cf0X0dfdSrS/HYjEzPXOPRnMl7sMIx8d+5LNALl84L5obG3E4NolEAtyavENVhQaDoZ7puSVkWf4n6MP3r4urly6KU8jFETE2Osjdu6sEg2FWVraRpTyNFhO/Lm6Qz2fRaksoWq212SyG+tuJBMK89dK4MJlrsLVZWV7eQKvVUl6uw2w2MjzUjccbZGt7D6QCipKUihwN9nRRyOcxGvUcePxUlJfhcGzi8flBknj9yjjDw3YisQx3ZhaxmGqIxZMAfzt6+YXnxasvjhGLK3R3tZNMC6ZnHZSUqInEE4z3tRFXFAQqvvjyR0o1gqiSJK6kAJAALk+MiWtXJ1ASMdwHx9QYKmlvb+EPf4KPPvmGwQErttYGHjkP2XUfoZKg1qAnFFeoM1Ywv7SGyt7ZLt57+w3qaqtJpHIo8TQdXZ14PCfc/mGeUo3Mc6M9lGq1CFEgkcigK9VgaTDiD0ZobTaTSWdQS0Jwb2GZnFCjK9cxNGTnq6+/JxQIkysUuDA2QDYLIpuh0/YM6hIdrp19XPs+6uv1yCo1J4Eo6o0dFyq1ipbGeh5thXnw0ImMRFVlGWZzDXXGKmKRJOlchuYmC2aTGsfqBi1WM2qNivsrm+wf+iQZkNadOzwJhOi1NaMt0VAoCJR4ikPvCbW1RtTaMvoHh5iadeDa8xCMxkgoSdLJHKmMAnB6kNLi2ibbrgMKQuI4GMJz8oSR83by2Rx2u4252fuUkGPjoZPhgW68vhCxWByP76gIBCBt7R6w5/XR3mLmgxuvIVFAliV2dg74eWGFQDDMuzeu0GBpIJHJ8eCxCyWZlZ4GEUsoPDti59qbE0SjUVo7WjgJpfn4s5sUJMHe0Qm/uwN8e2sOfaWWfe/RqbYIZDToOddv45ff1qkxVOM9DPLp55PYrI3oK0qJRBWmflrgXG8r0XiUbC4n/aUtylo2m2Vl1UlfrxUlKbg5OUNbUx3JdBJ/KIT3yE+5VsP6VoTt3YOisBeBwlEFk6mGdCbPd7en0enUuH1eNh+7Tyc7d93/+mqkpxsNJpOQZZlUOk0gGJTO+vj+BK0UnM3pFXUVAAAAAElFTkSuQmCC',{"lab":"ANGULAR SHARD"});
regImg('a443',20,20,'iVBORw0KGgoAAAANSUhEUgAAABQAAAAUCAYAAACNiR0NAAAEkklEQVR42o3U20+bBRiA8efrmZYeaIFCS6GUQ2GjHMaAQSBb3MlkiXrhhZoYvdDs3kwvDfsD9MIrb7zRLIszMW6LODenc9kQZBuMDkbLgK7lWNpCD3wttF8/L+ZpUaLv9fv+kvfmEfgfY7VY5a6+AVKZDFlRJBmPEwkvCP+2K+yHlJSUyHZHLa76Jo6eOEVbRyfbmSy3Rq6g1uhIRFfx3x8jHF4S/guUD3b24vG24XA4sDucaPUGtFodQwPdfPLxR+xk9zBX1CBIeR6O/cTkxOifjurvUltHt9x39CRWu50ztgDZxdt8PV2H2dtDq8+Hp8mD2WLl6dNZyotrqCu9dPUfZ2MlIq+uRgQAxR/YoZ4B+dPPL9LY0k55dBTl4q+Mx0yUHRigv6+L/h4fU/4ABUmmICtotgk05v1sJndwuDyoVCr5ObC1vZvpR0HCgQfkJTUXUkMUfK/S23eIF08Mko4nGPnmKqKYQ0eOO+s2/DkPemWeolREo9H99XJltUtu7+llcuIX8nklKfcx2lwOTh87gtVUysj121y6cIntrRgtBw9w4OVXuHHlMgZzHdFwCJVahclchihmnoFGk4m1cAiDQcfB/k7qal1UOauY9fu5OzZJ8PEckpSnweulwunGd+QIwZlHTN+bIJ/L4fC0IOayrK9FnoHuplYWl56i06hwuWqZXF8hsBAhGHyCxWSgtFSP01lPVU0tSLs413+krcnF+LgfnSoPggKXu4FHk2Oysr7FJ587dw5PhYYHsyFmpyYJrcbwmeO81bbHwp4dd1Mr1dVVVNgsHC7cwWo0MjK9xfxcAJtZh7VUi77Gy/bKPAqLxcZWbJPYvat0dXVQWWVHq5BIyyYmYhb6hwYxl+qodjowFRLk1eWk688wPvoLRl0BcVfmjcY1qosR9BX1KE1m67DBaOF0ZYTx8C4VDR2IqQTpgoaYUIZSoSCTzWE1GWhta+PJjpGvvryI3z/DS141FmWOpKTEXExzOxBDtbG2hiTG2W1oJRbMUZpY4r0P3ufm99e4/u13rIee8Pq7Z/G11HH56nV+uHWX0Pxj6px2vK4iHZW7vHYhh9FiQZnbRrmbzZzflRTD0dJ23jkEu6sBfl6UOHXyOOoSPZHQEs1uF3OBJS5euoJTm+KFJi2haI7eJhNfTBVQWJtJR5dYWAigBFBrSoYddhtiKsE1fwbkIvNTo2zn1GQzKSKRCNHNLVAq8BmTLMUlBAHuBRKsC05qnZXMTE+QSacEJUBW3BlWKLWs7pnI7MnkMwnebIzjYI0bD9coSAIqjRp5T2QssElGMBJN5pAMFXR2dzF6c4Tl0DwynFcAFAp7wkp4ASG5zNvN2+j0Oj67D/7wNgqhSDy6QTQcYm4uyDF3kQ8Hc1htVnoHhwjPTRMOBSn+Xq7n8uX2eOU+TxmhbQHJYENW6jFplRQlEYpFsjtpzh6GqdU9RjcsaBUSi4EZ1jeWhX17qNJb5TKTgQprOeX2GnRGGxp9CdlkFK1KYH0lwnxoGfJp0unUP4x9i61SqGSj2UyJvpSCVEQhCIjiDql0EoqFfe9+A3AT9mzNN79GAAAAAElFTkSuQmCC',{"lab":"METALLIC ORE ROCK"});
regImg('a445',18,18,'iVBORw0KGgoAAAANSUhEUgAAABIAAAASCAYAAABWzo5XAAAD2klEQVR42nWUW2yTBQCFv//S/9J2vbF2rmt13XCOu9MNVCLeiJEpER2RF40EHzQ+IYbwgsKLD5IQEyUaX3QhMTiISoIxYw6YMCKTMG6jY2yzk7aTTTp6Wbfe/v/3CUgEz+NJzpfzco7A/fTiNqtl3WoiXp0q0wRV5rohceXcMNM9x2H4sPDfyD2Gc9s+a9PWzbwXdgAQA2LJWXIOjZmyQWo6Q+70AMcP9VDo3SfcF/TsZ4es9i0beNkl46sYfGuJHBic5mZ0khqvHX+VxuaVdSRSJQbnTbr3fEmmc4cAIN6GPPlJp7Vz60bkisFgIk1GlrgYTTF9+QbrXmrmlWcaqfMq7PxmgE8PD+F1KDQ+vfpOiTug19/uoGsiw+6uKHvOphhLFzk98Bdr10TYUqvyrlcmfDPNxNgk2nweMVug+bGl8NwH1h3Qoo87raBLp697lGCkBmV5mF9yZTpeaGB7k4v9qQqdRZMHloVYUFdNdnaeg8eiOO0i9c0RAGSAx1uaODGeIV0b4OHmAM0+EbcostHu5GjOYCAjUJYMqmur+PDNVq6c+ZOEw4EtVEXoqUeZ+AoElm6ytn73BUnNjq6K+BwSEZ9EyICDtyyulgRWOC3e90h0ZyvEZRvBUplobIayXadeU9j3zkfIy9evpTrgxqmY+CUR0a3QaJikBRg3BHQsXnOKTJUMLhQEYoks8eMXQdZYvSyAooLqDyAWPQuQnArtkkV/fBbzRg63CYMFAz8m650WOVFg17V5clN5sr8P49J1Ag1BBJvFDz+fQRVF5PxkiqEbeQ6cGcXUHfQe+4cVYQ8daxeiaxbD6TJ9aYFCIo87pCJaJeY0D21tNXhiMS6JIopkIM0qkd3epsVMjk/hUmVaG3ycOD/JnKyRUhWmBBHxVomZqRyeWjfhxhrkuTk6wjK935+i6K2hOHIe0Tz1uTDzd4rgqiUopQLNASdup86FkZvcmi4QnC9xfTRFy0Mu3vAZVM5dI3lpnF079hOvqOguB1NH9goiQPLoMaxsnhlNo2xXeaLBxaqQgzapQH9XP7X2CktI8fXen+jrOYu32o1piHgfWUiy98jdrcmLN1j+Na/iXrGYuWyOuqZ6WtQ8vWfjlCww4jGKRZN8ZpZFz7fhWOAkMVnAKRW4uL397tYq0cNC5sJJyhNJXLqd1GicQ92XyYyMwFwO0V8LNpXIyqU0LatnbOAqik0k/mvP/W9Ea91s2YNhZFXD7q3GstmwbAp6XRjfgzX4Qx6SJ//g2m/9FBNjlIZ+FP73j25LbX3Lqgo3gCBhlgtoLiflYpmZq1GMoa57cv8CTYKJ6aWDN7MAAAAASUVORK5CYII=',{"lab":"CYAN ICE CRYSTAL"});
regImg('a5',14,14,'iVBORw0KGgoAAAANSUhEUgAAAA4AAAAOCAYAAAAfSC3RAAACDUlEQVR42pWSP0yTURRHz3vvey1QIFIJbcXwz5lExEggRUMiJKJlUUy6FHVBkZE4SthIHJhFSGS1ncBFTTQOJjqhlq2OBrETWjBtv6/vOiAJTOqd70l+OTnwH5dIJOTK+FXRxoj3L8Dg4JAMD1+ku6ebtbWnKBHU0Ye6ugYxRrO/v6cAotGTcn92lo7ObrTSLC09YiufJxSy6KPg5OQk8XgcQHp7+2Tm3gylH3sE1Qof3r9jK5/HeAbfDzCH0IWBAUmlUjzfWEdrQ1fXGQqFAkGtRtX/xerKKkprxNUQEaUBjDGSyUyxubnJ7u4uDZFGSqUSP0t7WKt5sryC7/torTDmQIsGGBpKUvxe5OWrF3g2xNm+fpqaG1Eacrks5XIZgGqliu/7BxMjkYjE4nEBBJCRkVF5lsvJ9Rs3pb29Ux7Oz8vtW3ck0hiRhYUFSafTYsNWOASamk/IpZHLks1mZXr6rthQWObmHsjG+rqMjo1JKBSSZDIpsVhMABQg0WgriVMdZDJpijs7PF5e5lpqgsAv8+njZwpfCiitECeHLpWqr2+Q0x09TKTGsZ5mcXGRWDyB1oZv218BjedpgiDAhi1+xVcAqqWlVc71n8daxZvXb6nVAgRwzhEOhxFx+NUKzrljsai2toSAoljcxvMsWhucc4g4nKuhlODc8cIAlNFWRARjNcEf1SKi/tbvb1B93X7EK+0OAAAAAElFTkSuQmCC',{"lab":"HULL SHARD"});
regImg('a6',18,14,'iVBORw0KGgoAAAANSUhEUgAAABIAAAAOCAYAAAAi2ky3AAAC3ElEQVR42p2TTWzbdBiHH8dOUjtxk7aJ4yxV2EK60TW0ZaxCIA2p40uIA18SiBMnxA1uPaKxIye4cOYECLEjN6QhtI60TYFuq7a0XZeuKkmTpbXd1rOTOn8ua4UEAonn9L7ST4/0Su8PHjF5Ji9y6YTgf6IcDaV8mk0tylbL/lsoNzwsgl6PkTNPMDl1nvK1ayxcvy79NXO8TJdOirN5g29m76AlkuRyOSanplDCYaZfeJHD4JCtP+qMlUrE4zE+mZnBsiyiqsYvP12V5CNRrWl92tUzl2YuX0ZVY3iuy4WL04xNTODuu2RzOWRZYX6uTFzXMU/k6CGjJxKsLN+6dCxKpVLisy8+51SxSEiJ4HkeluMwN7/AyuoaN5duUCg8xu6uRSSskBwcpFqtEolGUBQZGeDDjz4Wb7/zLn1xHduyCUkSejLJ2t17tJot7F0bv+PT358grsc57AYYqRTtB20sy8FzDwg9+fSUINzHyeLjrNy+w97+AVpMw/c7dDyPbtDD7LrEJWhsNzENg1azRTgSBkli3/OwbAfFyGb5tVLB81xeffklqmtrpFKDqJpK8XSRvugmrXYE0zQZf2qSHj3CkTCr6zUqc/Ps7LTp+h5Ks14nkcrw0H2IpMh0fB8hBLIcYnxiAtMwadTruJ7P7Pdfc2O9zsZGjVfefJ1Ox2N5cQFAkjRdF888P83I6Cj9egzLcsiYGbqdQ7Y27lKZnSWbVlG6VZzFAL8wTs8wyA6EWCzP095uSACKu7cn3V76XQSdDq1Wi9K5cyQGBmk2H+DYDtkhlban8Eb6BF/J26QLz2Kv/sjNn5f/+SGP0GIxcf65C3i+R3JgiMb9Krpis3LL4YNCH+XoaSqBgbN0RcqdvSjCUZXabz9I0n916L3X8mJAj/Llt6vSaCIm3h8Z4kpoDDVhcq9xn+6BQ2N94d9FI6eGxHBG42p58zj3Vj4tTDVgxw/4rmbTe3TVn/kEK6dvByI5AAAAAElFTkSuQmCC',{"lab":"DEAD SATELLITE POD"});
regImg('a7',16,13,'iVBORw0KGgoAAAANSUhEUgAAABAAAAANCAYAAACgu+4kAAACTklEQVR42pWTS08TYRSGn8GZsbTVolBoK5QALS1GUqwXvKCJYmI0ggsXujGQeIlx48af4T8wunLLQneGGC+JEhKVkAJFqW2BttzaUtoBepv5XBgxcQVneZLnzZuc58Aepq2jQ9y9/0AEgkHxdyfvBrx4xCIiWxJDjx6jyjLrmSy6LompyW9SzU6SLIv/waGjDeLldY/ostVyKdiDw93KqxfPMQyBx+/716D3zFnh9XpZXFwQy2tr/JielgAC9VaevYswU0K6d+WESEQjSMp+tGIJh+0g1ro6IQO43a04XS48nV5yG3kunOsTIyMjjMdWcV69iV3Li+aWFqKRObKJJEppiyanC19nF/va2z3idG8vhpBIJpewms0osoKsKqi+Hvw+P6nlFRxOB6WSzkBinGF3kdBKgYxqo6a7+xgmk4lypQIIBBLhcJhATxv9AzcwWcy0t7dhGAJdN6hVZSyKCVd1jdVsBjm1tAKKCcPQ0atVtGKRxPIyneVm5n7FUHSDfL7AIXsjBjrlEjSbFcjVYLZYkTPpNA0OB4ZuoKgqmwUNv7+L0dExDmuvWT3golxYpzt4nM1CgTfpMrGZDWLNJ5GzGjXR6JyUz+VQVJVicRu9WmFuNszsVEi61SQTyIZZiMfJbRTY3Nzmp93Nh/pTyA1O4rOhP2dMzM+TSiZILMSpVCrSXw++rlcY9Ldgt2UIvX/LteGHfB/7jMdRT3olhaZpkrQbE/uaasXEeoWhJ09ptDfw5dNHkosJpicnpL28AqpqEoO374jz/Zd3rP0NUNf7C8JRJecAAAAASUVORK5CYII=',{"lab":"THRUSTER NOZZLE"});
regImg('a8',15,14,'iVBORw0KGgoAAAANSUhEUgAAAA8AAAAOCAYAAADwikbvAAACSklEQVR42m2SS08TUQCFvzuvTgcoLZ0CQoktGIEQhICYGBN/hO75oboRH5EYF6QRHzylrYWW2mKZMtOZuXNdkBgRz/LknJzF+QT/UW4kpR4suhgGJIng836H5s8r8W/uhrF0L68erU1SLo2QsTXCQBLGCfUzj9OWT+Vbm+OaJ26Vl+/n1ebzebphn5N6D9PUKBYyaLFGq+tTcNMYCt7unPJyqyEAtOtFV20+m6cTXLL1+ohm7YKvuy0aFx6mpZPPpqg1QlA6awsuBddWf8qrCzmUnfBuu8pY3iGTsxkdtciOmui6YhBETLspKl/OMHWNcdcEQFtfKqjF+1mugoRYgucNuLgIGHNtZsYz9HsRw+kUMo5otDt0+j6lGQfNQBmTBQfT1IkGMetr00TEmIZOcTLL7s45x402T1bKpC2dpxtlQhkjhGRqIo3h+SF7R20u+wMerhZJlCKOJN7pAClhbmYcJ2sRawmZQGfnSwMhBMOOhlH52mGxnKM0beN5EVf9CMsURImkVMxh2oJu5LP95juzZZdYJVi6hZQ+WvfXQLyv1DFtnShUbH04YPfgjJEhG5lIIl3x6sUhw06K2TkXpRR9P8QPkuufbVtTjzeybCzcAVsnloohzULTEj5++kEsYHllgnbLo3nqsfutT7V+JQyAIEhEtearlNlkojCEk04hhaTV6ZEoyd3SGPt754SBZO/Qp1q/RvUGno5jqPG8QTZrUZxMY5o6hqnRuwzodGOa7YhqrS/+y/bfmp5y1MiQDkIQDuDopHcr+xtagQzeD6rWaQAAAABJRU5ErkJggg==',{"lab":"OLIVINE ROCK"});
regImg('a9',14,13,'iVBORw0KGgoAAAANSUhEUgAAAA4AAAANCAYAAACZ3F9/AAACDUlEQVR42m2SS08TUQCFvzvvpqUtMy0Nj0oiCkoTiGCMCYkLNm7c+zvcuPbvuNaYmPhYGIgmLhC1aGCo0gpF6HOm7XTa6XXBQgHP8uR8m3OO4IIsy5GT07fIZKaRUqFYfEfX3xMXc+eMtXuP5ERuAV2P43tNOn4bVZUUv76gevQRCMUlcH39sZyaWSXoB8hRxObGM+q1Kqt37nNldp793Q9823lOu+0KAAVgafmhNK08r988pd6oYhomhcJdCktrCEUQj5sk03mc7BqWZUsALZNdkjcLDziuVnCyk6THbcIwwMnMMJGbZTAIaDTbNJqn5HJX8dpzBEEdxbQcfM8nm53k+twKKjrDKGIwCPH9BopQ0YRCz2+gmzrj9iIomlSRPIlIEI0k3X6H5JiNKlR0Q6dUKhJGPazYGDP5ObzmKbXaCcgQxfN+iN9HW0QjwXDYJex3CIchpdIOtjOBphkk4glqJ1WOqxViVgpVNc/KGYR1To72CXp9FE2lVqtSPnQxYjFyTh537zOuu43vd/G8fVrN72dgu71Lp3uIwKRS3iU1lmLxxm0SsSRbWxt82n5PGAZcvbaI5/0kinri76CKKVPJBWxnFU2zMEwVQzfodlsE/YCpqXnKB5uUD16KS88BcDIrUtcd0uPzhKFPOu2QiDu47lt+VV6J/17uX9n2slRUg9FoRDTs02p9OZf9A8Nv4cvV6jMPAAAAAElFTkSuQmCC',{"lab":"COBALT ORE"});
regImg('big186',32,31,'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',{"lab":"BOULDER"});
regImg('j214',12,12,'iVBORw0KGgoAAAANSUhEUgAAAAwAAAAMCAYAAABWdVznAAABzklEQVR42k3SS08TURxA8XPv3OnLPqcCDbZAiBRpgsaNK0waghqDiZ/Dpd/KBYkLEoMbY4gLdy5IgDSFBdWW0FJamz6mM3P/rkg869/yKP5rZ3dDFktZ/EkAKLSCRDLBzJ+SyxsaZwP0Pd5/X5P6XpUHCZdcPkEmGyNbSHDVuuXlXpn6u6cMZwoF8OHjjjTOe7R/j3jztsrUD0AUs2lIqZwijKc4+tpi2Olj9vY3pf56nZW1PCLgz+Z4Oo4VIQxCbkYO379c0m/fkEonMdZqDg8vGN+OCCPBdTU2EkxM8/BRkePja/zJBG/B4+/tEJNMOPQ7Qy7bAV4xQ+uiS3nJYKOIXD5NOhvH96e4KRfpWUwYRBjX4dVumf5IWF8SFpfzHHw64ZlWzMdzsKCNQjkao7XGGIfmWQ9rLVtPCvz42aZUyaE0aK0I/YD51MGNG7R2FEEYMR5N2d5e5Ohbm+3nq5RLMSILbirOoHuHt5BDaYUWEcJ5RMGLcfD5nNJyhm67z2VzgEQW19VsvqjS/XPH1WlL6TCElY0lwkCo1TwKaYv1J1TWijixJDYQrpsdmr8aCkBVVj15vLWA62gEQaygHYUIjCfC6ckNg96duj/iH7xtycuh1ghsAAAAAElFTkSuQmCC',{"lab":"CARGO CRATE"});
regImg('j215',6,12,'iVBORw0KGgoAAAANSUhEUgAAAAYAAAAMCAYAAABBV8wuAAABFElEQVR42iXBTUvCYADA8f/scakYRXMF2TLyZlEheOsQQZ6CDgXZsQiib9G36dYt6BT0cooOoUFvSmPisOYy0Kmz6dOh3w/+ycLRsdw7OJGABFA2t3eliMWZ0jV8P+C336flOAhjPs3Wzj7ul4VZqaMlZri6OGckpKin0zGD6HiEctHk7f6FYukOZeMwK9tygh87oN3poS9E0cIeYnZVw8ymqFzaJOc04hmd5KOF6HV96EsK6ynazQ6N1gDf8wmp0TAEQwIvIDEZA6GgjgpCddPBVxWuyx5nt58MQgGN2jfCKyXwXRvsJvHBELcYwX0NEIaRJreYZywf5uHmCc/pYuklFIDl3JrMrCxhvdeofpSpWs/KH7FqdDCPmn9vAAAAAElFTkSuQmCC',{"lab":"POWER CELL"});
regImg('j25',22,14,'iVBORw0KGgoAAAANSUhEUgAAABYAAAAOCAYAAAArMezNAAACuklEQVR42p2TTWzLcRjHP7/2v3ZrV131r9OVWV82um5ep7K1Y2YHIisOSLwmLI7iwFEcxM2Fo3AQFycR4iUSEhIy8RLMSFhm2IbNbPtro6X9Pw4LIZGg38uTJ3meb/J8v98HikB7c1Q6WqLyo1/RXCPxmF9+nbH8D+HOzsUCoJkae1LLaE8GBaC9cR47Uq0UTRzx+zh1aKNcu9ujXrx8RzIaBqBtwTwcNoonPnziKhkjx5GuTjl+/jZLQ7WgWeTOsz6ePf345yWf2yUAutMqrbN0AVgT8QlAW3CqxnSXhF0eWbUwIqVosqgmIGVYxG93yUznNAGo0sun5FpaHRatYGV0YphF0bAMjgsNrVGGrj2QhlUJeozbElmyGMPRL3a3i/hsL48eD7Jrd4ob13vYtiVFxppl4Pl7Ih8M2dpQwaVXQ6INT07QFGvE2qFj0xWBYQXJCjwjPiaSdpy9bsrX+hDjLfFNc3nY/ZzwxmoGv31kxnIdfb+bgTMmvroQ9SPTOH3uJu/TBppmyfNVsljSFpxaGYZkGBn/TPZbnmwmTwGhoClyZoHRdIb0mIlzeo5sxoQy+JqDL4aJtaRAXkxKNMXA5BelVVW4GR96x9iTMUpm+ng9maV0BNSLPNbuPJY3BT5dnkC9Nfl8JY0jY0GGc8Tqq7h3v58Sh43QqNB7q48HBcWGUDlexy+Z9rhLBcDjtElidqUArA4FRENJwu8TK0oSlbpU2u3SVhuW1XXzpTkYlrg/KHM9XvE6y6bMc5XKX+MVC7gFIDRjynGXTZO961rk6L51AsjJA+sFkIPbO6RrTfzfP693aFIB9I8aCiDZUkfn8igHj12gK7UMU4FVKTqaIkQC3uIfZHNzlGNnb1AbrKIzUc+Ji/cpiKjuvkGm+8spGjZNE4CVTRFpmT/n5+nxxhqJR6t/k+I7D7wP7CDv1fAAAAAASUVORK5CYII=',{});
regImg('j57',12,11,'iVBORw0KGgoAAAANSUhEUgAAAAwAAAALCAYAAABLcGxfAAABnklEQVR42l2QzWoTYRiFn5k0mclPk5mJSRrSjPkrbU21ixbcFArqLquCuHbhBbhx703Ue7CiKFqktFBKKLYUBNOFJBZTNIKosTZpMpk608+FdiQ9q/PC+8A5R+KfQpGY6J8cS6Y5LSJxleP2gC+f3ktc0Mi5mSouULldEfnSFIoRZLqc4121IV49W+Hpk2UPlM+NnkjgD2QIhmP4LJnW4U8OPjfIlCaZn78lhoBUqiDmrtzA7R9R29vHds/Ye12j2nxJd7ZOSDaGI4W1EN22hWSrdJQfWLaNavjRWwW6byLoyf8dvGy57IwQZwEUdZRLSQPHOQXXT7fznd+yxUHjrTTUoXNyRCgcZSwzRm/QxpzIomdjuKrtPXtAOn1VpBJFFmfusLiwhORT6fcd7j24z6PHq1ybuyniRl4A+DLpWVFZuossK9T2N9jZXaNQuk5cS7O5+gLJHuVb6yv2wGV8fPLhiF9ROLUcPn5oUG9WJQiKilmkeVhne2uDzfXn5Mwyjgu93q+/uWKRktC0vLd1MlkW0ehl7w4ENGHoEwLgD0JJkRTTXcrlAAAAAElFTkSuQmCC',{"lab":"ROVER WRECK"});
regImg('j75',14,14,'iVBORw0KGgoAAAANSUhEUgAAAA4AAAAOCAYAAAAfSC3RAAACK0lEQVR42l2Su25UVxhG17/3PufM+MxkYMbCUojlAgkoqFIEIQQUKIoEr4B4gFR5jLwCUoTEK9BRpAoQEMhFIq6iQIANxoPtOZc5171/OgvzdUv6VreE7zYFvXlqynSodIWykyl384JPnZdvf4ewGsf6x5kJN85O2NhIofV0b2sWHzo+KtzeyvlrnlGEXg7FK8cHeuvqOqfHls19x/pEGbQV2W7AFrD9xTN0jhcN/PlpztNlLgbg93OrnEkN9960bG0uGJU1Zn/JKLR0pSeOBLTn0gr8MhgC4C6nqf42sWS7Jed/GuJmgVDXiDHEQyFJhVElPCuUnp6z8YALo7GaC6sJE+OxUSDMK7J5jQTPbibkjWEyC9Rt4P1CGQwdG9byYxRjjq1ZDnIhP/CsuMCJtQTjDH0nxMGzLAOpcfyaKk3R04gy73pMVyjWCx0RmR3w92fHorKszxQbCb6PWLSBKhGeNcJO55i5CPN6RyGKGaeOl0vLf//mbDeWgGGxF6GlENlAEgmlVyoRchT3pC54uDXmlGtxiXLt8pSTY2X+rqKt4P+iw8SGWCxNb6hoeLQ4wLwqG7mznfF8P2G67En2ahavG1wHuffUvVI3yuMDT6uW+0VBrl4cwL0iY80meFlhuddwemyZ58puEIbWUgXHShTxXFseNMXR5IxBr/8w42I8YiIWj2BQUidstp5XTcHDeknWLuWIeBi5tfrzYMyqAVGhNPBPmbHvj0b+FU9KEVNVecBDAAAAAElFTkSuQmCC',{"lab":"RED PLANETOID"});
regImg('j82',12,15,'iVBORw0KGgoAAAANSUhEUgAAAAwAAAAPCAYAAADQ4S5JAAAB8UlEQVR42mWSzU/ScRzHX7+fP8CHoJ+EBCGJzpmJsAJ7WqWu5ay8tU61vNiha8u1derepf+gWrcObXTo0ixdXmqMJYJmaxgFiiISojB50G+HilA+p8/D+7335wn+2sj9QdE/5BTsM5PbLG49vFbJy/+cUmMdBUv9fjztJ+1klHIllqqLbZe7hDhoZjuZQK8WaWpzsLWQYXEiVMEp1YRkVqbvnhd3R5FLrS1MxX7x7O7bPYqVlpQuhzj34Dz23SU2lpIEY2n0ugKnR0+hmGy1MygNGrTWA/gfZ8m/1DG3tsbMdJKoL4HaY6xV2N2RKZQVnKM6Jt8ESKVUImFQXc20WKRaQjH8VZocf0/cbOP6Kw/CP8cJbxN1pSTx6R8oBq0AkHpGjgmLq5vSVpbNxCorVg9H+g+hTWdYD+XYCIQwmhUcg93Ep78ho9EQ8f9k9nUQubUdpbmRzY0SZWM9xToFg9vOcuA7M08D7AgJed4XllLhBdx3zhKdTVNu0NHrSnPjQpm+KyZWFrc5Mz5MLrPOF19QkgF6rzopxVZJT4XRl3f4/E7hxfM88x9L5Cc+Ef0QoWOgs+rCnvY/e1at4uKT28Jw/KgASXgf3RRdA3YBYPPYa/4MgI6x4Uqhc2yoBiRXB0aHKvS55f+PlklzuFPdQ/oNtBS3715BBhcAAAAASUVORK5CYII=',{});
regImg('s1',20,17,'iVBORw0KGgoAAAANSUhEUgAAABQAAAARCAYAAADdRIy+AAADM0lEQVR42pWUX0ybBRTFf19Lkbb5uhbWBSZfCwYN4DSzJcbECMkSsSSa8DZ5MpugvGDGguuSGTcTs2VvPDjIfFyCjDhGCqiLbEYSFkyMOFrJsjIGK4Kl3SgF2vUP7fUNnYQMz+vNPfnl5pyrsIuam5tF0zRW12IcPFiGb8hHMBhUeIZ0uw2SySSxtTUO7N/Pvbv39mT2TJ357IyEQiExm8yy1x3FfsAuZWVlNL7TyMbmJgU6PZe+uqRUV9fI8MgIZrOJ/qvf0HWya5vwVbdbspkMdwOBHdS68vJyGurruXjhIkcajtDZeRKHQ5O5ufv09/dzc+wmsdUYwDaldd8+PjnRyWGXawe5YjKZxGg00tTUhGqxIPk8gUCAOrcbQdjYTPBybS2Tk5MMDg5uE31w7JikMhkG+vqeolT0er3kcjlaWlr4uL2dhYV5HJqTyV9uYzGaSaVSDI2M0NPTQzgcxu/3E4mscOVKH5WVDuaDs0RisW1T5dChV8RWrGI0G+k6cZrsVoY709OUWK3M/LUEOh1akYm8TqGttY31jQ2SyU3W4xuM/zpF3Zuvc+vadYaHh5mZmVH0BYbnztmKVRwVTl477OZJOgXAxNgtfkqs8iCxzvPpHBZ7MZFwhKlpP48jj1jaTNPz8zg6dHz5aReaphGNRs8VlBRbKdAXEF5aIZvNgkBkJcIb9W+hRsNs5fOUGFVy6S0aG98mkUiwFo8ze3+O95wa79a+RPtHbYxPTDA3O6soiqKIiHD06Pu0trbyMPSQiooKfvh+FKtqIZVK892NG3R3dxMKLTI9fYd4PI7P56OysoLffp8il839c0ObzSaFhYV4PB6KioyAsPjnIk6ng+XlZTKZLDXVNQT+CDD249j24vEPj8vjWAzf9aGns+h2u6Wjo0Py+bx8O3hNHszPi6ZpYjAY5PyF83L568viPe2Vf+fQXeeWrlOnpMRu35lDVVWltLQUj8fDk2QSQ2Ehvb29isvlktHRUTKZLAMDV/F6vf9thQD/r9+fnz0rCwsLYjKZ9txl/W6DhoYGSWfSBPx+bDYbwWDwi70Y7vq+VFXFZrURfRSlqqqKF16s2hPl35wcZGIH6EOWAAAAAElFTkSuQmCC',{});
regImg('s166',30,10,'iVBORw0KGgoAAAANSUhEUgAAAB4AAAAKCAYAAACjd+4vAAADPklEQVR42q2TvW8bdQCGn/Odff44x1/xVxo1dUmDkqIiNUVpKZQ2CxBUaBFInZAQ8A90YGZjYmBDYmBDzGEIEiogIFChuqUhIWqT1KkTx3b8deevO9t392MFpYiFZ3/fR3qlV+JfmHlqQlx67hRC2FjWgKEj8f0vD6k3DYn/gSMll184I2KaTL0bYGFhnpefhVjQw4HhoHcFn3y2zP3Nvf+UX16YFun0OCMhY/Y7rNxak46I61cD4p16jqpnghuXIjw/P8XNj1fIpJJ88OYc4YDMRsHg9kaDK2czrPy4yXrRITUex2gd8O61OT7/cpVsOoIiS5QPO7x+ZZbrr53n1NwMd/LrrK1tkctN8ukXP3Bv4wClcjEtEtfjJL4bx9jXMHsdSnsVkmMels4nKO6XUX0eJFfiRFZDN3q8/cpp3rBlkDz4lByTySDixkU0n4xHBn8wSLlh8s2tu2xt72ILD2efzjI7O8lyboJHlRFKZrUqsVoFNkFWRbUQIqKpZLIZRt44pf0q5XKBWDhIPBpitwTCHZKIapiDEfGIn5/vHGA5Hk5MJ2mbJqY5YL9UIxlVeVBoosiCvVKTcrNLW2/jui7K33f3MCSVnCKs+bFMk0q1wV6lRc+SCWo+KrpDu2NyLKUyHPVptAxGdpRmu4feGXEsIQBY32qwU9SRc2misSj2SKdvjvi900USLo4tkAFaN4+LR+Hpj5InL7B0Lsh7b53j218L4I6Ihb18+P4iiy+eIRzy89vdbbyyS7FUo1jScSXwegTbj+u0jD5buw0GIxvHEYRCQfqWTUCBTm9AdjzIxk6Thu6gFOeTIiS6WJ4Uhm7R7Q+pGxYg8epLz+A4Nl8t54lGQnz9U4HMmMzjSoe1PQWvN0ZJ11m6kKFaa7NR6OLKY0ylHDIRgT/gQ9OiqKJD3+rwoNBiYA0xrS7K8XxNIg/QZCwSF3klQcPoYg8tXGfIRCrG5k4Nq2+SjasYRpf8n0X84TSSa/PHboluq8b9rUMcx5EACttwdfG0ODxsUKs2MHoWD3frhAN+NgtVWkZfOvJHn6KIoW0zk0uSiqqoqortSoxpKrWWxe17O0/8/5PQgj4hAR3LBtf9R+YvLT2NIYt4Fw0AAAAASUVORK5CYII=',{});
regImg('s167',29,10,'iVBORw0KGgoAAAANSUhEUgAAAB0AAAAKCAYAAABIQFUsAAAC7UlEQVR42qWTTWhcZRSGn+/77s2deyczk/lp2jRtk2BbkbGUAReBUFCkCCIUF2qwK0EXCi7diqA7910rRV24My34h0UUhPhTI0UmSEjMNE1nMj+Z3zsz98533AhmEXDRZ3fgwAvvOY/iGLSjZTHp8lwxjS9twv6Yufk8vzQ8vlh/oHhEnKNDLqlltZjm2iWfdmgp7w75IfMsU6UFLla/5MbFDjLJy9qvjUcKVvL8Cfl0FPOhXeFMd4O1ayF3yhO2OpbFjOGFgzcprFzFzgS8urbKi4Um5cEUaV8TTyyCMLGgFQggAlortFi00fT7gu85dIi5dX9Eq2Ux717NvbdxznCvVWTcrOAedKibBOXdMZOR4cz210TffETq8hVa4yErvbs8edLlidmIWScmrxXFuQnz/oTFactjWUvWWoonLOezlpQIp4OYU65wvZSi1otwzI37/1Z1E0C6eY/q8JC3n8nRl4jLpQJ+z8f5/DVev5LmYZRBexFuAPEQQhHcQMCxOBpcF1o9QXyNSQj2UOF6inwmIm0O6XRH/930Uukpme5tc/PlBKsfN/hrJ2RaWxoubHaFp5c8sjnFrXJMMTZUGhprBRRsbGl8oxmMBVGacKzY2LVkXAUoXA3n5zXK19jIovpvLMgnYcRX3isMN7/lg+UD5nKG8sMQrQ0oxZQRvNBh7fc+P+3HDAV8z+A4DjlPMFj2epb5pKEztGil0MDslOWlJagNYn6uaQLX57sHA5xAgw1j1n/7kZn2PnftiHDRIxwL+50hCQP12KUVwp2DgMeTPdrBObIXimzt1ThJlaqT5ux0D5p7FDIenhLikXAYad75c5pkKk1YWGJ3Z4tK5R5HX1/OJg3vL6dJEzPxM9yuJ3F6DUp5w2ZniJ9KUe8MuL0zJpGZodbskmdEC5ecK5zyhbeWc/iu8NkfHdb/DnFOXyAVBNT3K1RrVQVwnG+yPJdgIWXYJk97DAvSZDixKIn5fm/8f47KUSWPW/gHnD9IxJCIbIwAAAAASUVORK5CYII=',{});
regImg('s168',27,9,'iVBORw0KGgoAAAANSUhEUgAAABsAAAAJCAYAAADDylfFAAACYklEQVR42qWTOUxUYRSFv3/ee/OWGWaRgQGBQZAgcYG4oUFjNNrbWZG4FVpY21lrbbS1trAhGDVKocTEJRpZjATHyOIoiCgz85g3y1t+C+NC1Ea/7uQW5+bccwV/RiaSSfp27aazPcPkxATPnj4V/Cfqr6Klc6P0HYdUUzOtrW0oikLVC5FubqO9fUk6Vj2prj6mhq+JfzbzzzbKNxmdc88PkM+9ZsfefSTX1TM1McZKPs+23u3UKmUcI8lK1wA9A/OyujiLUMO4bpWwohEEHrphMnjyFNVahTvDQwSBR63mMTk2JgB+bHjpQqu8eBU816ZkFxCKhvRd+vcfJGIaZLNv2bSpG9/1GTh0hECGWLWLqKqCQCAEuJ5HIplAUULouoamaSzkPjA+/oK5N9OsiUMoQlpmlOOnz5BKt1CpVSnZRSrlVWZnZtjSu5NUKs3KyjIhRcX3fFaLBaxIhJa2Nj4vL2PoYZxKhWLeJpGI4fkBsXicRw/ufjOTJzrkzXKJy+5h7MUc69u7MSNRTMvC0ARPHj1GET6T4xOk0s1UymUaGxtIpRspFItYhomu62hamKb1GZaWv6ApGtG6CNGYRSwWY3TkNurLvRlJpySdMFi6XiAoOTiOg+dL3s29o6NzA/179jB04zpHjw1imBZCwOjILR7ev0eqvoEg8AmCAMuK0t2zlXK5ipRVqhWb9/NZstOvsO38zxjPZ0LySiGBoapopklYDVMXi6NqGiFFZWF+Ft2M4tZKKIogNz+35ubf6d7cJz3X5W321W+zv1VY1sXjdHT1EAQ+HxdyfFpc/O8/+wpEkvXUmm087gAAAABJRU5ErkJggg==',{});
regImg('s169',32,9,'iVBORw0KGgoAAAANSUhEUgAAACAAAAAJCAYAAABT2S4KAAAC10lEQVR42q2UTW8bZQCEn3d3vfY6ju04iWPiJA61Exriqh8ULKAXQORQqRI/ghPiP/AD+APce0cVR0BClI8iVSVWIqo0diXHaeNvZx17ba933305IHqAHjOnOY1GM5oRvB5KExrFbIn82gaGoTMcObR7bQ4aj/F9X3BJMP4lnxbfVuXugHtXP+P6aglXjjntdCiktwkChT3qE97Woazx5/HvCtSlmDDapZxaviv5vLJCeFYidpZiJ32bSu8vmvY+rjsmrMdZT+RJmBHuFD4hO58jJhJK10OEjBBo4EkXFQQYmoEQBmbEwFCCwfCCzGIGLaShaRo6Gq7n80ftIb89/QFDWibiq4q4dl2o0xdHlKXFg4Nv2bt5j2gkTrPfYWV9xv0n37EUzeAFLoXMDutzW/RHLUKaSdg0caYTgkCQjKZo2g3kDJyZz8yzmBoa+WwBe2jjzBx2124gLZdn5wf8N0Z1a7PEcmKFjm2zbK1jCIEV17mRvYkpLH46/hXpTbizvcdR4wlhK8TAnrCZ2qE1qeFrE+RFlOFci8CoUD1+ydX0XT7a+pgmJ7T1OkX5Ht88+ho76P1j4PyLVfVlbZ0Gq6w001zbeBd73McZj5DKIx5JsrFwhZ+73zPzZtQrJ+ymP6AftJD+lJnn4oyHTJRD4Ae8v7nHYe8RUvTRVZhc/Dae5xKNRRhqPdTQoB4tU6s+x6h9uKaSt0zGZxJ3oDg6O+Z85BHT4pxoVXLkGShJezDgpVVnQS6idMVze5/MlR4p0yMIJFKPEBcBgRbh4eGPbCV36U5izBtJstEc+/1f6HZ9YmYSJ+hSfVbFGV2IVxWsppLK9iA//xaFhXdImstUOGQjyGOqCIkli6ezMktykQeP7zPxR6L4xqLSrQQq8HEDgaFrXEwnnDaar3STVlql57Kcjzt0xi/+t5zXTUm9mSgyF44jmSIIAQIzotGftjjr1PGVd2k/8DdXj0faGE8xugAAAABJRU5ErkJggg==',{});
regImg('s2',21,13,'iVBORw0KGgoAAAANSUhEUgAAABUAAAANCAYAAABGkiVgAAACi0lEQVR42qWTS29TdxDFf/9r+95rX9/4xhD8TLAdCNSNmkjBSSpAomqlblALVdXv1SVfATYsWCCyqlRVldrIahFUbRqg5GGHvOw4tu/T00XViDSVWDCr0czR0Zw5M/CeMVctyng6JQBL1+YEQP0XVMjaYiRivNrpqHcR3l2elb7nMfAjjt2Ao0jw/BDtbVAmZcrlcoFaqUDjSkX+rS9Ml6WWz8rb2K+vz8mxF/Ckuaa+f/ZC7Q98Lk5OsbXdVvHG1ZoEIyHyfW5encKNBD0eJ22ZOOmkrKz+xvV6leafm9QnC/Lop2fcWZ5FRFhp/n6iZqO1o3L5nNhjtmhxBUPPR4vHGIbC8dBnr9NlwjYZt5JMOGM0/2rTD0I8P2Bhukzgebzc2GTxUunU9D83f1W9o55SM6ULMoyEgpMmoSeQlM1uq0XtXIYJJ83WfgdBYeg6YRBw48MKD1a3yU7XKSUGeG/WefjD81P71/7YeqM22ruEgU/WGWOykCPrZIgEUnoM33XxXJfFepWuG7LZbrOf/ID0N/dYv3CTaNg/Y2C8MXNRul5ILKGztXuI1vM47PawnBR+OELF4vh+yOMff8FQivz5LJn1VV5/+yXFZEjHHZ0lNXWdjhfihyOWp/McuT69mE0159DtBzx91aKSyxJ6EQlT5/53TylmTA5bTbbReN46e3ra3lEPjRHucEB/MCAKPNJGHD+MWGvv0R24LF2exDYNrFSStfYBlmVRLpXPEC415sW2rVPmYRq6NK5U5NbCrOTPjZ8052tFqZx3ToG/+rguXyzWBSA3npHqVFFuf/7J/39UzrHEMAxe7xy886M++6gmZiKBKI1Wd8DBMASleO/49NrsiYLG/D/53xY7G/dUOP44AAAAAElFTkSuQmCC',{});
regImg('s219',25,18,'iVBORw0KGgoAAAANSUhEUgAAABkAAAASCAYAAACuLnWgAAAEKklEQVR42qWV22+TdRjHP2/fvu3brsetY4d2PW1soy4CMpCTAuIkUQLGG7kx8fRneIGHC731H/DCxEOMEYloNGocBIeRAmMnKVtb1rXr1rXr1rVd3/btzysTJ7AQ/N79njx5PsnveZ7vI/EIGjzyorB3+2g2DRgUnaW7MeajoxKPqG0Tew+eFH1PH8fi2Uk2FkNbS2C299EZCbAyGyUzcY2562PSY0Gsnj4RHh6he2gPuYUky7emkQ/rGPbZqX+7jJR20zfyHDabjL6e4IePP9wWJD8oOPT8W+cttJOZHqcQncR8RMH3Rj9vHjhCY6iVePQy5Zk1qg0j/uFDtPu955PRsXcfBjH8NxDYfUY0lyssJi/jHGgldHIEo2gjP1ciFUiRnFlFkgIcOncGq1pm7NIvdA7u4YmRs2Lb79o79r4Im8uMnr5KcNdBllKT9HSFUFIbuPbvYjNkZ3JqlBU9izvnwKaZaBncR5thlY75JJkDp+jyNPntm08wWmUWLt+kpcVN7zMHKBbSGJ+9+J5oyHXMokmtlqeaSRA0dpOvLyJeEuSnovjSnUR2HaZqyWIasqA2WrkXm0BPz2HV7RTvLWKsKRjLOivJDFCVyuUqA6EhMbGa29p4h71PBG2D1F51Ip/twmiWkc0mip+OM3C7l8axIPKOEqmvV2kNWUhEf0eTrdgVBVmxo5hVCskZsrErW+oa//0Ieo+i2ksUhu0c6mlwLtTDV6kKC68PUHwtRo/pKe5MJrCOzCPVXXiyYdz9IZobdXZEOthcraA3CmRjW3uyBSJLBkySiskiY1EsyAIshgqmooxklGhodRpaE5MkqGkaqxkNbWMBvV6lVCyi5cusp4rbj3BNK5831S1UtDp3m0YS3gqJlIHcZ3doibvROk3Y+q2sXVSwSC4yk+MUsnlEuURpPk9tvcLm+jKbleyWcZYOfveBUAMmevUKXx77HJujg6DFT6W9TmV/DTXuwJky03R60G0rmDqdKIqb9NQ44XIJn2zlimjHrpqYjV+lUsujrSQlAM/gXrHy103JcO30O1Ih36RYbWJW3Hj9w2S0AsaCguuLBuqSFd3hIq5PcUueYeKP68S+/xXzRjubnV7KdR27owPJacXUaaA9EgDsAhzC5W+n/8RRcZ8dhCMvi1anj/i9G9gCfrptYdaM8+ivqJx4u4s/L+ikP7pJoGU3ufVZmhsq/kiE2ckfWbzzYNO8b+Pj0xckSZaIRE4hLVVIXB+l0V2nLdxGf6KLLrsVgyvPQiKKsmkjtPNJUtNjDwU81CBVq0/sHDqJw+lnNb/I4tw0xhecuEY8bPxcoPFTGe9ABNXhZDlzm/jUJemxrb47cFzs8A6iqq1s5LLookizYabVF2R9bYHl9A1ymSnpf92Tf9Th2y8cbj8CA5JBkM/epbA0/shH62930b/1iy8DzAAAAABJRU5ErkJggg==',{});
regImg('s234',20,17,'iVBORw0KGgoAAAANSUhEUgAAABQAAAARCAYAAADdRIy+AAADPElEQVR42o2Uy29bZRDFf9992L73+hk7r5JC0qRIbZ5NWqiSoEhVUwjqqhskNrCAv4EtSAgQEkIqrJFAlA1dgCJVKShKWvFIsdo4aZsWE+Xdxm5iO7Ed29fG92NRhASNSWd7NGfmnBkdQY0SIiSbek/jDhvoLoP1mSilTFxwQGm1ACkzwl0Xkk2DY/gbgzS/OMjKtWmZ/uMm+Y2lmsT7AHUSyn9DOYxIJ67WE7QMDRNsMCllttlbibGzniFxYwrIiycIu8d3ZPK7zzibXyaRKVIYOYOq+BCGiwbLYeT6J1xcaUHvGER36QSOHUb1gJ1IoptJNn+bQHHXEb80LrSOz2elryFAqCtC4M4OC4sJnJIb8jboZfB5eKsnzA/RKD/ZXjyVNMtXEuimgWKY1PVUsZ4JU0ppuAJH5T/rHnnjA7n21Uc4ik647zR2Koti+anzeTn6MMba0HkOnzyBuyFE+s48Mx+/T1Xuiafw8PFNIp2jaJqHbDpF8OQITd3dPLx+lZC9iTcYoBg+gmYZpOZ+ZX3mqqhJ2DwwKtvPnMPf1Mrc5W84/ubbVMo28cvf8274F8qilXPP7zGVhntJk62XXiNl51manCA+OSGUJ1YTJu7Wfh49SGDn8lSqbjSfxZ8PFnmuqpDMaHiqkn5SnNUWmb/0NeHBUUK9L9eWLNRGeahnGKdcoej2YrV1oAkHdXaKxmKBoLCJmq04TfW0dXWydf82meUFcis3xT6EblnfP4TuCqL7AyipDXxOnlzPGIqjkC/sINxeTGljaQpbsZ9J3J7+t4cDMSntby/StTLPreg8be99SiUr0BtNmj0qH8bf4fUvN1k9doFIUGHpx3G2f7+xr0Lt+BfzspItYisOxUyeQ8+2c/fKJEZBIA2LVMhgNr/FXtnN7uwtdp0CdkkQHhjDE7Kk0VLAjBhsx3ZJTMf+O8EvQ/UNZMsSXXhwVBXDo2J6DFynXsWyvGjBIC6PQsXOUNopoWpr2KlZCo8cNq/NiAPTo/7UBRl64RXUqo2mVcktzZFbvU/qbhTwA2nxNI/9OCZ6z8v6vmGEbpNbXSIbXyC3Gv3fntoxpEakv70PVJVyeoNi8u6BagD+AjofRiTBGDktAAAAAElFTkSuQmCC',{});
regImg('s267',36,16,'iVBORw0KGgoAAAANSUhEUgAAACQAAAAQCAYAAAB+690jAAAEU0lEQVR42r2VS2xUVRjHf+fce+femc50Ou20004ftIXyFApVIr6IDxKJwQVoYhAXJsaNugLj0mBiYiSuTVQ2SsAIKkjiQqLGyAKlPlB5VNA2pRShz2He986993OhIVDQSIL8d+fky/n/zvm+832KK9QVj8nj7Rkcz+dUxWX/hQnFXGkthOG1+zdJ+srF2WJZeZM5ZLbAK11Zdgwskznx8v6iPl5b3Ce3BAhgqFgmWvHZ9cuvbJqc4rnursvm2jQ5PTLClmqRZxb0/G9QcyWbtSUvOQnZgZZ8qkF2Xv0i8i3IeG+3PNTSfNOhdHOmVVpas9Iyr1ueymbk9WULkVSSozWXC3aME7OXeOzcGLsW9orlONJsaj40DdKlIj3x2HUPfdIy5W7TEpauuWFg1b/mHgk9j5xb4fmzw9xnmlSCkE9dj2jg0KMN0l6BtVGTz9JN+MUSE6HgFgqsa06xYWKayTm2GwzFxpjDto3boXMxi37YyejhQ+SrQtn3/vVDqFhdXMJAaMi2sWhshE5tkFCKIQmZDAxSSmMriAcubTGHTBBhnlflkA7pU4qPOjspAdGITW5mFiub5dEfB3lZw+1BhHNvj3Fvj0OfvsiS+c3s3voCP328+x+hFCBN6TTJTIZNpSIDjUkmyhXqgTOFAkenZ7jD91jtgwe0K/CU5kvD4LhlQV2UI/WNJNOteG6NhCFs+36QJUpxwrF5M9lJIdDEEzE6Uib3F2exHYN9k9OcyRdY35bBLVcZqbh8k8v9BVRfn6SpPUv/6RF6TYuchOSBpYbigFumloiz2YngBgFF1wPTICKKHiUsj1gcUpqVbc0UbIu9I2O0GBECPyQtIVXfw9QKQbADyJsGCxqSdCTjxKs+bdWQDypFBmtVvp6cUWZdIo7ruYRKMWuGTFsBHiGtGr7whSKwvzFO0rbJlaosXrUSr+ay9echkoUqccqsFUWUCP3pOgYaW8gFAX4YcrTmsULVUTI0GcfmRFDjNhRuyeXi0Di+KF4t5zmIfzmFZt/yfnQYMlWu8nQYsDowuIBFHEWjuKwxFImR8+SBluYUx86dZ8/sJWZmcvRbDqdFcUxpvpiZRU9N0+77PKJsylpxMvS4oBUgBCguEvCbZVOrhayyoxwxwqtgAMzx4WE04Jsmp3CwQ5sZpXmnUmI7HiuAP4D5HVk+sR3e+32U+cpgfawOP3CRUPgq5jC+cAldTWk+P3kcu1hhXbHKw9Eke8MK3aIYwKT6t3Xc0ezzyxwsV9T1ivqyOpSWBq0ZE+EJrXjR91FAe0eGvUaEZ0fHaLUsFlgW5TAkqjVTgcfxAJoT9RjaoBwGOK7LA37AoF/DNjSjvs+WSJwHlcV5Ag6HHge8yn+bhzGQNzQyDjLdlJLventkZ0/XDTW4NMhdti1zri4rjIik0TfWLPe0toiXapB8Ni3v3rlKMvXxWzazrklZdyIum9taKFdrKNvireFRKkGgbiXQn3h83xniSsD2AAAAAElFTkSuQmCC',{});
regImg('s277',25,15,'iVBORw0KGgoAAAANSUhEUgAAABkAAAAPCAYAAAARZmTlAAADrUlEQVR42qWUW2tUZxSGn+/be89pT5LJnDLjJJlMYpySaJrYirUUi4WCgijYAy34N/oX+gt605v+gUJbbClCtSWtKYjaaOJommgOSpI5ZTKnjJN9+noXiJaCdF0u1suzYL3vErxUk5EZFc6HGJ+cwPSbKAWaJrl77zbVSplWo0G1Wha8RukvN/LxYwRHoyQHs2w+WwWliMXjDIzksJUgGU9TrZZfh8G/baQAckNvks9NYzsWO7Utyq0NKpUittV9RZOLHFNr9WVyiaMo6We9VDg0o0/2z6ia1cRIukR640hdQ2ga7XoLV7RBatjuHkezeWKRDIuFuUOA6cxpNdw/Rq8ZJWEmEXqQI6G0+nPtxgFIPz9+gfaegf5RF1OLYFtdTDPM9WvXkP0QT8TITKXxmSZ9fRF8P3uqbXVwHJuom+Lc+BX0d5PklqrY6RDOVpPIO6Pk755XD29c587TG0JcGb2q2g480uc4PXMOX49EuR4rSwXGT0xz8ePPWCk8YGn+L3RdsvOshawHsD2L45mTtPc7ONJG6hrKU4CBNDT6zD58QuOHuW8ObqIuXvyUqYkzWNJGeR6a57Hw+C7VRpOeYA/JkTE8BSEsfp/9lWgix363hac88DwEGtFojOLaKhvPCiAEhmGgPO3AXWIoO65uz93DFzIQQMBvMjw8SLjZJBUZ5f4HBoGpFLGv10kez5LKT6NLcJVEWQ6hUB+l9ces/Hb/4BaWax228JMnS0xMTyE08PsD2PsWC3/fYW1jm3SiTH8zi3PtOU3l0ljbxmvN86LbRNMNfFoAXIW71yHsC6u21T4AhXwhJU5G31M1Zwcjo5GIDYP0kAiKWxucOvs+H166zKP5edYeLqAbGjvbe3g1H939NieSpzD9EVy/A66L7XkYMoCSgp5gL1I4fHv7K8QXb32pOq7AvbJLvy/JfreD3wzy03ffk04OEIkl0f06/mCAsBll9pcfqTdqKOHQ62Y4c+QTrLMBrEKJwNEYqtQh8HaG8h9LrM7e4sH2TSEuj11VdatFe6TJcHKMTreFEQyyWyzhJ4iUQUqVVQyfoFJt8Xxz8VDQBqPjaiCcptQukgqnkcJkt7PNSmVe/GfizVCY4fQbZDOTOHg0G2V2W5tU6lvUa7VXNEO9I+p5c51sJIuQYdZrhxP/iuBS9nOVu5QnMZqmsHgP5XoMZoYoljdZXn6C4Qnmbt0U/+t3TfTNqBf9Nol0Ck3TkEIgpOTp02X2Wg0Cfo1yufhakH8ArT2HiCuqU1kAAAAASUVORK5CYII=',{});
regImg('s28',20,16,'iVBORw0KGgoAAAANSUhEUgAAABQAAAAQCAYAAAAWGF8bAAADVElEQVR42p2UW0wTZhTHf18vXEspLZWCgEVuLaAFL3MCBjUaLxnBxLhkWUaWuWVZfDIkyybb4uK2p20ZS9zFmMUZt0SzLWFBzdLpwCkDV2HILYyCzLaOlkuR0kIv8u2BzOgDmO08nZNzzv/8T85F8ARRgqzMT8BaruF+WMP5ljGxUvyyzgydVtaVpmHJNGDNMhJK8zCdFWI0qMXT6ufsj3+JJwIWXWiVmuoZurIOCICUuERpNZnZu6GeXHUtccZ6To84ufbT7LJEFP8q65scEp+KWEcGpd12CRCIzIsZ5V2+utHIrTE7CXMnqEKPVrVKrthy5Qe/yfZjWx9WXfvZz1JhmcO584Aoy82Ru4pMdN7xYbHu56kSLw/8XkbvebjiGqfn9sJjbMWm965Lx1vVAmCHvUsa9RoubCwSez+8KRWb4rm03Sby0wzy6Ku53Dbb8NzzYcjPYM3AMCVpi3R0j2O/Mc6AOyQAVGqlcmkIP1yUp3eWM+N0c/PLJlkjXHyy/wsOVdRIXZwG7x9rcZwbZHXuMNlzAfrvxHNG/zzP7VHyuq2Jb1rGpL19WggAQ+85+eLiNl5TR3jl0DaMC3PEFKt4dt8WfHeDWIszcLhvodN5KahKZ2zoPsf6DrM6c5HsxBhj3gXeKLjMd9emUAIsHqw9fjjFxo7SdKKLSbSMKClYX0Fnm5N4RSoRdQBHr4Y+dwBLSgKhyQCOoA5/UjGD/X1kht2sCU/QPhReAnxwtvnd5Jy641usZoaLLFzVmjlS+wIne/r5u/cy09NR9lWnMqoz8f2Ai4jGTNlsH4FJF9nqIOvmBzhzdQanZ2Kp5WcaW2XL+9vF7pc6pT8/gKNxl6h82S6N+Raa38yh3rYZvXESxToDugoT0akQsmMcEZjk+p9hfnHOAxHxcG32NLTJBOJo/mhpdcpOtUpNQE1HQ5XYmGeUcaoo6qCVuoIGoppPGfL56RqfosftA2Ji2UupOdkm9eYk8gxKPn56gwBQJilliaGQo7avIbOb30ff5vMrE8teiupRo+1IzWOBeXq9rC3ZzNbid3C5h9HGTjCoTuR/PYdkdaI8WGGiMEuLOTULhdnDSFoIQ3wyPb8GOfWtU/wnwEelUC9kVXk6vtkYlxz+FXP+ATC2RVy9DlzKAAAAAElFTkSuQmCC',{});
regImg('s3',23,10,'iVBORw0KGgoAAAANSUhEUgAAABcAAAAKCAYAAABfYsXlAAACSElEQVR42qWTTUhUYRSGn+/ON3NnHKexRlMzVCRSBO3HSpOIfkhoG0Q/izZBLdy0ClpFrdsE1kIosFpkElphEILUJsQWhmA/qAv70xFTU+/M3Dv3fqeFaZta+a7O4rwvB573KP7ozMmklFYYjGdY6ocHPx3FBmWtDVXVitv1NrcmonQVJ+jZViyt8ZgA8j/zzuqoXDhRKOVK/XNn/bqyOqSnLsFSnxDaGuFNBlQqyuFNRQzJAgP5RdKfA1wCIirEpHgq1VAgjy4nmLjp0W17pOM+45/c9VylNeL7cC5RQk1BBC8XoEuFs61J9LDLi4k0plXjbFa0DWVovlJIeiDCnUGHeWUwFmwRi6NNUVq229zrXabL5BjDV6qmvlGawvM83OdiaxsMmAph+fk8F79U0HftGQ1VFqlYhPGOu7QfecLV43MwHQYNxDW8tsiOGPywENMW09ri+jsHncvlCHQeJ5PFCTxEgUop7EgIjYLvC0xN55nyFPF8QMiyQRSQBwMEAUuuxVJOoTDosEYBgRGUZSHGwDEilKPwgVhKOL8/gT3s8WpxBco1+R1hmkczHGxP8m3QovOtw0pICIyQlBAte8McqrR53OfSS45RfLUOtLIxLH21CWZ6PKIlNv2LOdwY7Faa92GfkaRhajJgmTwxLGYIVPGuiHRfKuLjjSw9UcNcwmfsgwuyClSvhZ86EGVPbYhgJo78EL6KTaeToSPrrNKf+1uxXwQAJOeF+08dBmazzGLUf6t4uq1QyioMEkRZfunSNbfxJ/oNQwXzi0MLRFAAAAAASUVORK5CYII=',{});
regImg('s30',20,14,'iVBORw0KGgoAAAANSUhEUgAAABQAAAAOCAYAAAAvxDzwAAAC9ElEQVR42o2TXUxTdxjGf6cfp+W00GLpObZFahkMKR9mKlCXDqeGjYmyLVmymSXuaiYmXhoX450X3ni1xJiYeGXMvHGb4jYWnXF+bqioy7SZBVs+DlKsrK2gWHvofxcqk3gB7+X7vs+T9+N5YIGIVLWLra3bBIsM00IN41mdqH8ju7v2ziONrlgvtka2ic6GblGhqHM16fWmz1u/FOPZMS7Gf5+Xt6KIA50HuV+McXXgAq3+d6krb6TUUsZoPsljI0cqO8Z3149K0srK1UJ1qVR7anAVVRp9TVyZOM/hi9/OkXpL/GLX2r08t82gyAoek0Y8HeN25ga/3D0FIH1Q3yVGc0NIb6m1wuf2czl+QQKIejtEd9NnJIsxbo5do8UXobl8DbJdJjE8hF/VMCwFbiT7eD+8gdFcgp/7evkzfUl6Y2WAkCckNEclit3O15Gd/DbSS0ZKUV70EcqHKSmVcSh2CsYs0qzEOedJIpubOLOvj3OJs5LlFVFbICq+2bIHt7yEsZyO1bDxa6qHf9qv0rAuiPmPSYyzT5ktmpk2pnHZXEymMljzDu55+pDDBp2mj8XchDVlYVHjqUUullBR5mXKyLBcC5KV04xPpQi7m6koeBlouM6j0Sz5fjNS92Pq70XQLBpWh5XTN0+/uTJAm/aeaPO1czt1DVVZSntoPVPmSXpu/Uhwv53J40WWppeT6OjHHHeyVu8gOZ3gxOAxSarW3hYBV4BL8fMSwEeNm0VX7ScM6gOcud+LzaQQUdcxwxMe5HRMVQaOgoL3aSVD6WEM2wzp5xP89bD/xVPeqVotVKeGzxEgWBpimTPEydgJfhr8/n/ZWP3ii8avGJ/WUS1+ihjMygUmDJ2euz+8lM0moedGsNwa6Z8DrvJERObZvySfxOedwmIWrKpu4Pidv+kdPMXG4IfUO1dSp9VTXVaHYc4LPTtM7MEdaUFvBhxBcejTI2J7dMc864XcNaJ5yRrRokXFMndo0V6nWW0Rm1ZsWTTgP2vAHZkWPdKPAAAAAElFTkSuQmCC',{"noEng":true});
regImg('s38',22,12,'iVBORw0KGgoAAAANSUhEUgAAABYAAAAMCAYAAABm+U3GAAACfElEQVR42pXTO08UYRTG8f87t2WvDLvouosQLitIYoJampjYYOVHMLH2S9j7AawsLTWxsTBRG7UxCoL3oCSAussuLrjOzM7MO/POa4EWdnLqc37N8xwB6NpsldbFOUIvxrQMbMvi1b1ViuNlFi/PkwYpq3dfC44w1sK5S1QWMpauzhEPM2zHZrg/JNpVmFXB0rWzdFd+8Ob+uk7T7L9x69uXt5SiPMYDhRxKDMPA2wvwdwbYnsmLO6uIUHAUFODvsgZonGpi5gyGA5/JM5O03/eIZIS3OwAQ+VpeWzkTFWYMD4Yi7+a0aZmgIc0yjJyBjhXhQSwEQHP2jC45DkwMqM0cJ/IiysfLqEix9XKLk2dP8vnxBuOtOss3ltl8vsH67XdML0/RPD9F4kX0t3zqi2OMNko8vPkIC6BUrXPCPUZ4+jPluTqFIKJSK1KbHKO7uUvqK+yiQ/97n7UnH+h8+E5j/gS723uEuYwsSMmSDH9tgL3hUK5UDmEZxaQqI+j7KKePUhlxKtEWhL8kWv0iDiWjM2Va5Rqq4LHzqc3YgstUqw5akxsxSFJFJBOCzZ+HcOzvsxd7HHT3KfUUMkkI3BG+PtvG7/mIhkE4iHCVS2KbaNOi/fEbxekCX9c7CKUxcjYyiKnO1+hu9/4Nr7k0gZ238Ds+zcUJ+ts/UCi6H7sAQpim1nYGkRYAZk5oJfWfa3HImAKUFmJsvKkrswUuXD+PDBKkTPnZHtB+0sFyLVpXZkh6KU9vPSXyo//vcWNqATnqsfbwHZkHaZxSPOXiuA5GQSCBUCsymXLUHutCvUh1sooMUwzTwCzadFZ2cCoO44sN4kHE3pvOkR7kN3QBM58FJADPAAAAAElFTkSuQmCC',{});
regImg('s389',21,12,'iVBORw0KGgoAAAANSUhEUgAAABUAAAAMCAYAAACNzvbFAAACnElEQVR42oWSvU9TYRjFf297e++lLS1QSvkoakFRxAqCipiYiCE6yKouxsTBRRcX3TS6mujoH8Bm4qKJk9HEGEUFPylWBZSCSEsLtKXQ3tvevg4mxkTQZzwnv5OcPEfwn7t19o709IbI5At4FJXlkRiXh06JfzHKv8wHl17JpmP7WDFT6CsWzi4v3p5eOvyt8vjNvRsGb2gMXXksA4P9jLydpHH4I1bawNtTT+XhFnyyidEnjzh/dWBd3r6eeKbrotT6B4iMRKgfnWFY5LknsiTmsojn84ytphGbGqmP5659mHt3fcP6XcF2GXaFiOVKtB44gv/jLGo0yqJdp6CX0YoWmlIiv7qE61mauqRB2/7j9CcTMuiGN6nPjM9Pid/1Lx+4IDu2DaLW1DCeyzIxEaVzxzYUxUaFuUpzTTWWKGFDkEinEU4PqZUCY9+maQ1tZ4dNJZ9fY/LrA248uy3sJzpOy/Duc0wHFOY9DsIOje6MQWkhQaXqwVVTz/tcgqKuMZXJEtSbsC1m0H/EOahV4W4JMO5TWXLYaaATjyxcU4qs4RMFlhcEQaeJOfWVlc427NVuXMLizdhriqJMMm+SKxpErM9093RTDLeQzKyhf5igZXOIeBb8uolB9lf9g4275GD7SbR8FXuPHGW+UmLYFF5FI/iba4n2eUntqabtU5HA0xipbxl6t7fjdEhqc5KXD59QdsS5P3GX5zMR8eckZEPlZvq27KJaq2NPU5iZbI4qbwXqTj+OgI4rabAwGSeXKNDsdTMyF2Gx8J3R2S/MpWO/f7TezmSoto86VwOGsYTPGUSzwDJKWA47JVEkayVxqQFml6eZTr/4K2fD8TtVt9QVH51bD2FTHFhWGVGWmKVVxmPDmGaetdLyuvxPrNsSaaDUzCkAAAAASUVORK5CYII=',{"noRot":true,"noEng":true});
regImg('s39',20,20,'iVBORw0KGgoAAAANSUhEUgAAABQAAAAUCAYAAACNiR0NAAAEMElEQVR42oWUa2zTVRjGf+fff/9t19GtYyu7TxnCBo5tZFwzBoIQZVEQRFRM0ISLEk1AEgIGIn7QkBBM0ESMQNToBxCNiwbjhghLhCCQMcsmHWMwuivt2q1r18t6OX5YlA03fb+e8z7veZ73OY/gfyop1SCFAJ1QkAmBQDI4EBIT3R/3ICPLLIdDcUorM9myv5yAL0IMgYKOFEMSH71zidvNbjRNR68rICYEzCywyJzpyWzfNZ+W37xYEypd18LU2VvpCwTJy5pC+70e3n2jkifmTQWDjm3v1eDyBmnv6hdjAOdX5MsPvqymsb6LPdvPkGRSqVo4DYNe44ajgw/3rOW7s42kanpMUtDQ1IVUBMsXT2VWWQEHjtRypblTCIBUi1Ee3r2Gz49dIjNjEpZkIyuqCmn1uqmpaebgW09x8OQFinNtNN26S32LB/QqRGP/sPv47TXsOfrTGMpyNP2XKksw6RQCQ5JfbrayZFEh9/1GGvsVgi1XxtNeTriUqrJ86e4LkZRmQM2FypUFBO77OPaFgfJdxwi1nEGE2/H8cZVeu30MhvLwFLNekwtn5lKYl05uuZXSJdn0Boe47xesWqnh+nYfA+4g+c++TL/HT/GsLLluU7kc/UIJ8MruBQzcGWIoHuP89zdZs+lxZlozaLnopPdPF7ZHJmGuyiRijuFpbOPW5SHMthRSM6ysWF+E0xngm+NXEUaTXq5/vZRwBPrvDqEogrxMA4kZks7Peqg7sYHGw/WEsyfhcPr4yu1j9cFZhHsjpKYY+fVsJ6eO2Fn9YgV1P95AfaBnEKmE8Htj6NIyUQN6oj0DUOfAHY4S6xgk2B1iMC6ISBhWEoR0CaRuRMKoNgxK4gHljbvL8LWF6B8IcfGck+rnZ5NjVOg63YozLsnOMGJbmEYwx4DbcY9b1wKYJltIT7Uwd1UeXl+CmhMN/9qyNFlUtmydg6PJB9mCwkoLAW+YwfYYil/j2iWN2LQqZm+q5sLOzUxN8ZNflkbtSQeAGNc2i6pyZaA7jtGsohREmbf0Ubz3PJw6rVG68RBDHRcRum76bjTgsjdO+JclqoDYiL+fqZiOxagRDkpq7Q7WPl2CN5JE/Z0w/tsN4xpbqMqID9OtZvn14Q0sLsilunw6L1QWs25JESXF6fQM9nDh6Fai4QQzLFFKlI6R4Yr6t/ckID/bu55kozYqHObmyr3Hl9J0vo8DO2oxGzWeW1bMZGsStZfb2PHak9ibnVj0gmg4xu/XO9FUQVlRFsuXFbH/03Ncae4Yq6EtP1nmPZbK5p0V3LZ7sJn0tP3g5udmJwP+YWLxOMPDUQ69uZyqOTmgh23v1+IZjHC3+6H4Gl1T8iwyHBhm9gIbr+4sx+OKIDSJ0WQkzZrCJ/vqcVx3odd09Lr/I2DHK6NZlUiB3qCi0ykIBP19gQn7/gLaSLP+vfKKVwAAAABJRU5ErkJggg==',{"noEng":true});
regImg('s4',20,20,'iVBORw0KGgoAAAANSUhEUgAAABQAAAAUCAYAAACNiR0NAAADoklEQVR42pWVXWscZRiGr3dmdmZnZz+Tnd3tbpqPNk3iFxoRD1qLkiIK0nNFpKJ45oHH+hP8AYLgD/BIEBERqicWsWukbLBuFRKiab72e3d2ZnZ3Zl5PEmwqifY+fri4H+73uV/BGSqslaV6Xkd2Q5o39wicQPAfOnVg7u1FmXk5z2CrT/aKjVvv4d9z6d5q0au2xCMBZ95YkOdunGf749+J6iHipRjFNytEqIhAQ2769KotnN969H5pi1OBqecysvBqhcylLDufbWGNEwRRQKN2wMw7CyhPxwkGAbqlwxiMikmw73P4xQ6Nm/viBDA9l5WzH1zAWE2w+VEdfUslsZhksNvHvmjTP+gz8nyUsYIMJKqhERVAnTcoXC4yqHXZ/PSeOOHQyMclQ4m+bJIqpxnc7hK6AaWVAo7nkcglAIWd6jZW0cK57xDKEDWmYpRNvD+HD61cyMrii0Vca0R6bRocSW+9zaTu4teG5BdtYimdYWfI7sZ9cWooqdmsLL5SQVtV0ZcSjH/06X7fQF9KoC8niOU0goMx3e9ahJ0Jad+id9ijs9/5F1QIIWRyJo35uEUoAoQvUPsKhUIBEVfwJz7O0EFOCZJXc2izOr2vDvF+cggORjgtB/lAFv+EUshKPaEzvWDT/qtJMjKJp+JIRaJfjGPM6bQ32oyzIam3bIKJZLw7QtZ9gtqISSvg8NddoRwDJSCkQAgBCKQKQgGBQAhACsTRDFJCCKqiIBTlyJU8ciiQ6dks8SctJtEY4SuoXcG56SKKoeAOXfqhQ2SD9cI08fMG/a8PGK2P6P7RJPDCE5tqSIQAmS5nUJ7V0JZNgh9GNL5tos0ZaE9ZJDMJov0x/W8a9NoTslGSoC8JvFCceXqp6YwsrZUZWi7WtSnCoWRY7TC56zG665KfzxNL6bj9Ibu1M57NsWI5Q2qBILZiYuWTOLd7hN4Ee7mE67lYuQRIhZ31bUzbwj1wCMMQVVUxKiajPQ/1GGaWLVl+/wKlDxfwaw78HJJbmSIiIl3OEE4CvL7HpD/GMA3icYP4UpL01Txz711CMzU6682Tp2c9kZK5Kzap1Snan+9huSaBCGnc2aP07hyxZ0zCYUQsoSPcCK1kMj4c0fhyh/bD5fCg7OszsnBjnuYnm8h6SHhNxX69jCIFBDHk9ohBtctgo0Pvzhn1daKtX6vIqeslvLZH+nIet95hUvfo3GoxqHbEIzc2QO55WxqPWUTdkPb//AL+BtLBn33dN5gUAAAAAElFTkSuQmCC',{});
regImg('s400',23,10,'iVBORw0KGgoAAAANSUhEUgAAABcAAAAKCAYAAABfYsXlAAAB8klEQVR42o2SS08TYRiFn+/SudDphU5vlKKiQTfGxKW/XxaaGA0JqOA1BgErU2hn2s71+1wQFyZKfbfvOU9OTo7gP87dqNvh9g7tZgPH8/ny8YTLHxdinW+tYGfvsb3/YI+d3buoWo3Nbpd4dsWr/X0OXu7f6te3PR3XtxI4/faZk+MDQNIqchr1DZLKANjbAv4TPrr30PrBJrYq0I4mnkyoB03GOcjFNV/LnEdPnlJmmf10/FasrcUPWtbzPMLBGD9oUFnLaGuLsizQukY9CJhnKb7vUxOSfLHE2/BZzKZ8Oj7i/PspyyQWf8CV1jYc3qHT7aM9l6oocRyHZrNNrx8Sxwmu5yKkxBiDLUtG4xGTnxHLxRKJoNcNSdMF748O+fDukDxdCREOtmx/e5fBcEiUZ7ha0ut08ZRGSsFsNqfXH9AIAoy1KCVZrpbkWUqz1aYyljhLmU+nKMdl3O1wfnbGmxfPb5LX601rbMWzRoOlG1ApwaXjURY5WZ6hlEZpjZISYw1VUZJlOUFQRyjFKE2xwiCThNfpCt91mEaR0ACLxVwAxMLYqySlUIKiHVJVJbrm0my1sYAx1U2XvoD4mixN0bUaF9EEyoqOFqySRKySNTt3HN/m+Up0B9t2MwwBgTEGpSTGwtU0Iro4Fb91f2P8AkRx339Z4A1ZAAAAAElFTkSuQmCC',{"noRot":true,"noEng":true});
regImg('s419',20,12,'iVBORw0KGgoAAAANSUhEUgAAABQAAAAMCAYAAABiDJ37AAACpklEQVR42m2TTW8bVRSGn3tnPJ7x2Ekmduw4NEnz0VZQFSQ27aILpO5YsEWCFbv+BTb8AiR+Atuy6qYSWyQ2VNCmUhJIWwVa0sT58kccz9i+M3MPC1OplvpuzuI9Z/Oe51W8o8ZsTVY2NkhTg4iws/OMHBTvke/PSlgp0T5rTfnuZHiyfO9LwsYqxvOwuaEQhMzpKhd7v0pmzPRREMhMNEfjygZLy6vy594OeTJQAGrz6/viHL4hvf4VXmOddDhEIXjliIuth1Sai6RpARBQmjTuU712k+TslMvzY0r1Ja7cuM7Z1s/s/vidckteiBPM8e/BPsVUY8cDTGoolGpk7XN0uIQOIuyoj7UWlfsM4gBjPBx3BhND/dUT7J1viJ49FQUQVpriliNm5udxHU21XkM5RTrNz7ClOqZ7BDYDxwdAux5iMyQd4RTDSeatvxi8/mOSYXzZUp9+eE0aV2+y2Fjk1sebODbm+we/oW58Qe3259jMgppEKQgKQaxQCMpcvHxM4+AnXsT7b58CWWqIL/pc+mVOjtqMukdYqwjLIYNXzwGF0hrRGkRQgNgcUwwZnx3SPjnGDIcTJIKoKZXmGqWojqMVhaIPSsP6PfT8ClaNya1g8wysoD0fhaBE0IUimbVk53/T/uUB7q3734r75pBe9Q7h+ieMhwkmz/AqEf3HD1F9gzf/ATYdQW7Jh33C5Y8wvRbZOMGr1Li70Gdn+S5d7xFufNrB7ZyS5G2s/ENmDKKENBgw6vUInGNM0kVlY5QXgrWk568xnUOSXotCsczvyRqdvR/obT9Sb4GV8upt/GgJv6AQm6GLZdr72yQnu4CdAlu7gUTVBVbWNjFxl93tLfi/UVOLC9W6NK+uk5oULcKL3S1SkfdWL6zUZGZultbB/pT/H74ZOio1S/+oAAAAAElFTkSuQmCC',{});
regImg('s431',22,14,'iVBORw0KGgoAAAANSUhEUgAAABYAAAAOCAYAAAArMezNAAADPklEQVR42p2Uy2tcZQDFf999zZ13JjYJjTIxY6JNSmwSbVMLpgppsxHBheJGQdCdIO4ERfoX+D/oxp2PhbioUpAqIRiSySRxUpM+JvNw3pOZO3Pv3Jl7P3eWgpv27M/5bc45gv/RO/GIDAkoDAbc7PYFT6BHTJFoVF5dWycVCmGogrrtUAW2N/6geHLyWABtdW5MFiydeHKGxOTTJCMhzv79F01NY/zt91ibO8cLF5b46ovPAJg5vyADZhAtYHD38A6dekWcu7AswSeb3vkPrq1fmqSiztCZWOL+wTa94ATFuUXsoYdRq7Kf8RBGkOhIQj6bmmZmfoFOu4OiCEZH4uxv78ipqSTRkQTxeFxKT7L5+2+oV+bHbpQ6BuW+Sq9V4ziXo+a61H2fyskDDtM7WI6LaQZ594OPWLn6Oq1mDU03eP/jT6lVyixcusziyitcf/MtXNtmb3sL7ZfNfyj2GgzMPJ98+Tm//vg9q9fWGfWGuO6AYSzGt998TW8wpFqvooZCtDsWjm1TLOTpdbtYlkW5VqfZscikd5ieTqFsHNdErlQU0aiOGY7S69lUmi26Bwc0tzZJhMPMvriA6zoIRUWoGooQaJqGoRsIRUEVKgFNRwDuwOX09BTtYT8UvOEQKX10RWG4tIywbWxdx263YThEERDQFPSAgS99FEXB930QEkNTQFEIBcNEo7GHwYN+n4CuYwQCmOEwE5k0jXKV0sUVmq1ThBA02hai1abd7mB3uzyo1bEdh/ppB7/VRloW+fwJPauD+uFrU/L5+cUbtjmGikchl+Mws8uu45KVgoO9DH7fRQ+F8BstxvMn7JXLWK0W03aPO9Uqqt3n7P17FD2PdquB6vtos6koFT9GePIK2a3bjCdTWI6DIiWaN8RMPMXLr65y66cfaJQLlMwAlu3gDlxKxTxWo4bm9CjpOk0VPCnJZvfRsnd75IdF7MY2nqoSsZqcr5awdAOuvUHqmSRHx0dk09t0ahUqyxeJmSbBWJzvMrt0Czlml17iT00nd/Nn+pYlHpl0JBKTZiLB/PQslz2HXt+hkHwOb/QMR3tpDjZuiyf+CoDJM+NybSSM7kvutTvcqjeQUj72Ef0LZQ6OIj43YjMAAAAASUVORK5CYII=',{});
regImg('s432',22,14,'iVBORw0KGgoAAAANSUhEUgAAABYAAAAOCAYAAAArMezNAAACyUlEQVR42qWTzU9UZxjFf++973sHZJwPh4GBUcDODFIQTC3qNG0lEhM16qrRxL0bo/+AmqZJ/4Euuui6LhplIbZNm7Rp1IXRmrQFbEJUDBJB5nuG+eRymZnXhQYTahOjv+VZnJMn5zmwicjgkD549LhWlqV5D4zNwtBonC/OnuPYqTPv44vcLDz+dwZTKrxeH4AGhNfv16GeXoq5HMmlRfE2xiIaG9ShcBdjR46Qz+cxTEU+m+bJ7CyfjB/mxg/f82x+novffMdvE1f459490RcZ0Jl0imq58L8h0u8PsL0vyvCBz6jWqng8W0mns6yuN6jaq1y49CUz9+8zOTHBk7mntLa5dc8HEUzRJGdqHXMpck6DomWRSyUBBIAIhbbrLR4P4Vg/hmHS4lJUazZSmeRSKbb5/YydOElrcomuukOxf5iKY1MulfH6fLS7LOyGpq5Mfpm4xmqtil2rsHFKJDagw9EILZaFaZqsVmvYtk380DgPpqYY3buPtUwCuntpcSmCgXYKKyXsdQetNVIYtG5xsa29g5+vXnldXhODQCiM2+2h0WwQUhJPoIM7N/+gs6uTVpfCavOQKeTpGdmNFAbSMFBSodGYaOpOnVKpTLlSRezo6dO+YAedO3pJp5IopUAIms0mxUKeaP+HDMXjZG/folsazJotVJTE1BAMdRJwWdhNjSMMHk3/zZpdI51IIDuC3YSjUY6ePkVicQkpDdbrDW7/+hOhrkGiQyPcnJyk1mgw92Aaqet8vHcfzxbmSSSes8ftJuk4pLSA9bWN8mQyuUy2mCdbKmCYEkMqKisrZJYX2bmrjW+/uoiQivOXv8ZrNPnrz7sik81o27YBxEyl8uY//s/y9sf16KdjJObn+P3H6y/TLUv72oO43VtZePxQvNOkY4O7GdjzEYVCbkOrO47ILj8Xb2v6RmLDI/rzY8f1qzm/My8A+CYX3SCeZz8AAAAASUVORK5CYII=',{});
regImg('s437',26,14,'iVBORw0KGgoAAAANSUhEUgAAABoAAAAOCAYAAAAxDQxDAAAC6ElEQVR42q2Uy05bVxiFv31u9j7HBlqMjaEk2CoJGGhJEEXqRW2HVZUX6KwPU3WYp+igUl8go1RqU3XSS0TjkLQYgg0EgzHH2D637bM7iJpBhNRI5Bv+k7X0a60luITr79Z0sVgmURHSzdN6tkNz96ngChiXHfvnXYQWxEHMwbMdet0OV+UyIb12+1OWltapXK9BKvDPO+KqQgJgbjqvU6vI5sefU9/6Hcu0EUKQlR5ThWl+eXCPrCuJoojuafs/Ub2ytsl2/Q9UHP+vEfHZR+v6m69v8tW3dVaXb9H121z0O2SyHr7fIej1Kc/OU1lYpn3cQqcJsVKMVILjSHq9LhnHpjfSmKnitLVLmESoOMYALnxfAIjlGwt6ozrBd/e38WSWxZXbLK3eQo1S9v7ZptM5JI4iHFPy1mSJYTikMFUmCvvs7TymUCghpMfN1l8cmS5ieZ1xL09Wesis5MmjhzSePHzxuupCTdeWVqg/qmMYBpOTJQzD5OTkgELxHVbWNnjw8z1mCnOc9844OTnAtjIkKiSfm2AQDplH0VUjTkYgHYd2+wghNJ43Rq/XxQIQ2iQYDFEqASCMIlI9wnYklmHzd32LsB9glC2GwQBDGBimwEwtLMvGsmx0nGCYJo5lIaWHlJI01SBShCmw3qvd0F9+MMbd73/EdjKYhk2cpCBAoDk7OwZhUK0sMRz0icKQ+WqN/uCC/d3HZLMuw1gx3j0myOSwp+bIj01QnL6G1prO6SHoJlbOdfli8xo//BqwurqB73c4Pz8jk5E0mw16vs/MbJUwCmi1Grhujt3GNolKkN447c5zLMPmvjuJjSba26YRDklV/CKaWouX8S7kXI2cYHHxfZrNBqZpoXWK5bgUS7P8+dtP6FShVEIchy/jXanW2N9/ykgp8Vo9erWwH35yh1J5hvbRETs7Wzw/3H0zhX2VsfG3dXmmQhgEBOGQYb9Lv++/+a2TWRedKhzHojBVQrrelbfuX3WbSTd3eBuaAAAAAElFTkSuQmCC',{});
regImg('s447',22,14,'iVBORw0KGgoAAAANSUhEUgAAABYAAAAOCAYAAAArMezNAAAC+ElEQVR42pWU3WtbdRjHP7+TnJyetKnW2ibN8tbZdGvrsHuxIl3Rdc6JMkWdMMELLybuP/DSu6H7J0TxTpg3KqKgKJ2s+ELsBm3Sdl2WbGnTpeaYZMlJTs7jhSyabYj9Xj08X/g+D9/nBf4H5o8dljfPnBR2Ae9/kaPxkJx85Sj74lEWf1u+j5+e3i/xaBDHdVlKrZK7uanucp3g6WMHxd9r4DMM7HqD0paF+DTOv3eW8x98xE+Xr6h/i547+7qEIiGqlkWpXEWhKJctLn7+neoIn37rRSncLHHp+0UFEEvukRdOzXEju0lyJISpe1lYXKLP70fXdbweL7HRCOnVDb75eqFT8OjcYYmHh8lksqjxiYSge8gsratYLCJ+s5eVdFpFYsMyNvUYt28UmZocQyFUanXaLYfSbYtWs83G9RxW7Y6616JEbEQ6yUeGgvLUE/spVisk40GqlSqBgIltt6jVbASNdsvh4UETLWDgnxigfqvJnVSBra0iP6+sdRXQ7gbPvfwSO8pm0hF6RMfFxWf4qDeamKYPXIeJxyOo6WEKM36sfU2sIz6Sz47z9mvzDAYH5YFbsf3nNiLg8fsRFG0HnKaL03IxDQ2frqF7PbgtsPNCrtKkJ+DBKx4adhOR7m3stB8Oh2VmeorsdpFkIky1UiHQ10PbVdRqDUQE227S32/i6TFQIwNIo0Vz+RbrG3mW8/kuK9TkgaT0D/Ry+ceUiiUi4jcMVtLrKrE3LEPBR7GrNsmxKJqmqNcbuOKyU7JwWi6rmSxWrXbf8CKR4N/DO3XmhBTz2ywupBTA+ERC5p4/SO5aiWQsiK4pfvl1BbPXxNB1BI3k+CiZtet89cUPHeHZ2SMSCQ+xdi37jxWzxw/JQ/19KIGG7VDIFxBd58L75/jwwscsXPq9q7N333lVwtE9VMsV/ijX8Giws2Px2cVvVZfHD0J0NCTzz8xwYHIvqaurfPrJl+rek07EQzhN4crVNNncptrNP+HQk1PyxukTu3pCfwGk4jx1Us9gDQAAAABJRU5ErkJggg==',{"noEng":true});
regImg('s448',110,30,'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',{});
regImg('s466',23,14,'iVBORw0KGgoAAAANSUhEUgAAABcAAAAOCAYAAADE84fzAAAC6ElEQVR42q2US2hcBRSGv/v0zty5czMmTWfMpEnajlqJRhKpUIPSSq2tFF8IbroIiK40K92qGxcu3IgLtVBiRaHiVrpKK9iFKdXatD4aazJJmJnMDJPMZJ73dVwICu1CCP2WZ/EfOIfvV7iN7IAhM8/63NODtp7iq4tdVoodhR2g3j7wej6vHDvK4XefYMnpY6vuc5cw5ejUh3Lk+SXh/OfinHlb7EdSstM0ZfzgCUHXOFC6j3r/TezMvdS6mxQzVXpbLsawwa25eewAnP79NFohthlR3sgrtoqIqqES0gy443S6ZqWw3BS5+iQz022C4CyK0oeTiPNBOM6n5wvsOTRLb73IqalfOJBt8OPaKIuXcnI1tQ8tOYhhxBgJN6R28TTbYevfJQqA7Q5LKulieW1G79+LHtMJQpVfN/PsmhAqFZspq0UuYzHprPL1tcdp1nSmJhYIo4jQhwuF53hq1wv4Vy4wr69S6q78E5558ElJ7x4mMeihJaC+1GVwxGXd+YPZV/fwzWKF170FiiWTt6xz7J9M0/j2Y/YmSkgIIRE3lxXk9zXGOjFKSZ1aaxkdIPI96rUyzZaHkbTpFHxEU2ime6zMr7K60sTLqvRqKlRb1KVN1FIgpiEBqJqJbql4KqCGGKGCGYAGoBv6e9uFPEGwzNA+B3uwhaI3KV6vsVjoUa3GqPwVR9P6mH1gjvLlPH2bcGj3FdL+Olm/RKE3wYsPvclIQyjEQjpKgD59fEYS/f08dv1h3jj5A177NOK7JGMu77jHOffTOunhR7mxcYNJ5zd8Nc7YQJXFWz6feScQZ4iEoiD1q3zx3cu0wvZ/Dx0bf1pCNeJgJ0dl+08SbpatoEFxqEzXNiHTpji3gGNBfCBHfcvDsaFcyCs2iKJrGBKxGcr/WKybcmz6Izl8alm4fEasT94XbXRA7paipB3k2tmTcunnI/LSayPixtWdG3pncZky80yE2Y1omw5fft9lrdjbUXH9DejqPnTEj7V1AAAAAElFTkSuQmCC',{});
regImg('s6',47,20,'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',{});
regImg('s76',22,11,'iVBORw0KGgoAAAANSUhEUgAAABYAAAALCAYAAAB7/H1+AAACMklEQVR42rWTzU4TYRSGn+/rDEOnP5SU8BP+hComNQaLMYIGgWgiITEsdGW8CG/BGzAxrLwBvQA37iBxYYwaY0IUBUMpP2UqBMq0tJ3OzHFBwkLdqPFZn7xvzjnvCz+RTHQL/wmJgQD0kpT5vlt/ZaQAUtoWN92JdXsS02pBNEQiGsMLaGvtxg1cnKcL6k+E9WxmXh5OPWI43oenQxpaISHUc1c46O3CWj9kdGicnrEJAZjtmpOJiXsCMBm9LDNXH/x2I8NxS6wWVjiOCrFkmr6wnW05JHzzDjOWYK11jy/vX2BnzhERT7bb01TTJiqXk7ykSHQYqNGLIlYcRKGkCYU8CsMQfB9Gs2QHp7n7OcnjS0W8yRzWyyX86WvorSKRqkdjsAevuAGNJsZIFr/hgvi02Emmuo5QwOJxP+Gz56jrZ27KkD3AYptD2S2S9GPs3J/BLB2iDRMvFcf4uo5qi6OtKNqtoIIA7/xZwoaHQhFaJlFVQ9k2tXWH4MkCRkwSDMUv8LrqsD13g4pho4xWrP0qXsPFXCqgM0PwLU9dPAj8k5+XS1Dz4MiFVBu1eDsiAerTMoS+MgrVderbLvvxA0iMQaOJtbqJUS4z/aFOQydgxyGiTYZjIygVAa1QGyFKQBsme5s7fG8W+bj7FoeKOo0bgN2TkYHxOyBC4OyytfqKjloLm9X86cxY57iAEIqcnEACItrgOKiwsr+sfskxgB1PS1d/Fq1C3AOHUnFN/UvLfgCn8OkcetKTiAAAAABJRU5ErkJggg==',{});
regImg('s83',38,17,'iVBORw0KGgoAAAANSUhEUgAAACYAAAARCAYAAACxQt67AAAFhklEQVR42sWW22/bdxnGPz7/7J/PxziJktR1nDpzsjRtmrSlUhE0kzqpoME2gYaGEKhM2xX/wLIL7gZiN5MQQkho4rIVGwuldK0Go6XZuqR10iSNEzd1nMSn+Pzz2V8uEEgTUpCQVp7bV6/e530unudRcQhCkbCYmp4is53g+o1bKp4itIcNe/wBvvHSi1SKFYKRcfHuL95R/d+JSZIkSjmF3/7y13iCAdQaK57+cZFJpaCV+tIJqjwejxg4GuTVV75LudniIJdhfv4DXHYPssmMq9dP7qBCejtOy/UcM8/PcvVXb5OLXvk3ufPnvyosNgsTYxHe/tnP8Xl7CAwH0er1XJ+f/5+eUB+LjHMkOMLegaBQ6aIzOZk8MUO9XiGf3cdudRAIRfjRD15i887v2I2nuHjx64w/ExJqrVqo1WqxvLbKQHiUW7ducnbmDD/56ZsIycjM1Em+8/KLwmAwCJ/fL3w+j/B5fKK/3y8GAgFxqGInT50RJusA9v5RdBoFs91CciOKr8dPbGWJhlJk5vwsp0+M8t57VxFaGW23islipFTK09GrefjJXWYvXWJ/L0l8I0a5WePCt74NmSKP43FCsxeQXS6UUhWhdhP2akC0sbndvDP3JrLTx8LNa19QVqMWzD07NUNkeoKWksbR5yW58xi/08nQSJjd+Cqyw8zthXvIdjM2WUOn3qCEE2+fk3q7gdNg5oXLrxMOhYh+doc3Ll8mvZ/F6nLzOJ6gkCsjKjVSW09oFavYZJluS6FaraLT6RGTr3Hh5R/Ozb726pyhU5/bWl5+S2O19sx1Wx2EqkQuV6NZ19BstmnUqhyfnmBsbIToygrtrpZqpYxSqSAbJTx2I99/43Xu/PUWq/fXMJntJB49YnVlie/9+DKhsUn2MymOBkZwOpy4bWaeOTrAwf4urVaHwJFBUrs7rD54wP1HBzSlo+jcQySWb7D3cOWtf8knxscjDAWDiA6YHV52d7aYPD1FraGwl9jh+JmvYXZYUYsuqo4Ol8fH1tYa7//mXSJjx2kaJOgKPvrwA7556XkMeom+kRDh0eNkGnVMOjVyp4PkcPLHKx+yHdvmhVcuUi0VeLjwOX+69gmDg8eIb90jFo+rtADDoREGhkKUsmUmT07gH4lwynyRbrVLsbRLcOQ0i3c/Zmv9PlaTjMXiRmc0c33+Kj6XG//AMEgGWrU6ep2JTC7P4r1Fwqkmse0W9XodjUaNuiOotVvYTEaMJjN/uPI+dqcTp8PHxOQYelnG5DtLz+CA0ADYnENzbn+QSqWMwaBDY5BY38lSFF6UfJLU5h6VShl/r4e+4VEUFVglK9OnzrEZTyBZ/aQyB2RyJZ6sLxMOj5LP54kuRRkMTLC58neidz8imykiW7ykc2Usbh9KtUura6KkdGm1Qejd7CWf0Kjk/2mwahTymSSJRByTpEWyOMimCxTaq6gaDXwWO+sbMepdFc38n/H2eJC0RrQ6PflsEqesoS77aJVqSAYd6YMk/kEPPf1elm7/HrfbyYnpsyi1JqKdw6iDQqZMIvaIQj5FLp1GqeRU/+H8Pd5e9JIdi0lHS3R4sBol9ThOf28fHbRkq1kmZiKYvEeI37nG4GAf9xc+xeZxoShtMlWZbLWBXZbIHeRZXFjC5/MRPBak2djgbx9/hl6SoAvddp0uoNVqaTbbqkMjKRbfxNNbQzLpsdmN6Fot8jo1FosJRamj0mjJ7GwhkjE211cpFnIsLkVVADqjW2yu3aZYyVKwWtBLRrKZtKpRb4hKtcJ2fFsF0KzXv3C42WwfHkmHzETgyDiSbER2WRk+NsSnN2+wsRH/b3tfertQfeXcOTF17lmWlte4d/svbGzEn1q7OPTQicmTwmKTWVlbI7OXeqp97B9mxGKVpSLh8QAAAABJRU5ErkJggg==',{});
regImg('s97',28,12,'iVBORw0KGgoAAAANSUhEUgAAABwAAAAMCAYAAABx290PAAADkUlEQVR42qWUSW9bZRiFn+9+d/B07diO48ZphqZuU1KVthTaqgV2qGIVqRIb9iAhseAv8Af4GWxBXbAtQxdAqVQ6OqRDmqZO7di+19O17/ixQAQq2PWs3tU575HOOYJ/Ye29s2rhWI7JZIKQBkrFWLpBjMCQKe58t0Xn5TPBa0D/+xD5BXV6Y4HPlkz8cYiuG+QLM+x3OxRTFneCkKYzT3hLqf7OziuixZl15bgPRK10SnWcDoHaE+XiCdV1Gv95TgDYuSOqcGGO5fNZuGFTqVRACOYP1Wg8vEu5UKUxuMvilSJEkt17PR5db1Kc1TANDaEk7qjPfLWK0xtiyDSl8gxtt03z9jYwORDWP7lSV/dHp3hh7zBTLTCwArYe/wFawL67i9PvMA3HxEZCNmfx8tmI9z+uM7deYPXkDPa0QK8zput4vHPmKI+fd3ATBzWccn51hc2fqmTTafXkZpvGD7eEAKgtnFNeMaZ+NkM4CHhzY4koimntjpi6imOnyzxvOETDkLFhEE8jrIzEsC2sYYEoihj6HnNmAdcb4ZljLMMgndfxI4WZFrz1bo1rX/7CgdWUvaJObhxBP2Qyt5glnkSEoYaZlsR+jKbDw2ubnLn6Bt4wQJoGvX6ANMFK6Qwcj2o1Q7c5RkOjVLMJE4VMBLmqIhiEfPvF9/+EJiHgxYN9ct0UezdbBJ5E1wTWjERIDd/1qNTLJBq4zpSUnRBNApIIgv6YqZcwzpuESkCcMNgfEMeKymKRdFbHiHWKC3kkwPHjF1X2Up63PzrK5KmHKVOoJEREAVJBSjeoHC+y9sEiIzcgCKC8kOXE6TJHcysU0gUM3eTi2jpaIslUTJZX5rBrGjtbLtubHoX5PIOmj/z0w2OqL+uMci4ypbAaBTKxjRWaLM2t4js+i9ll9tstMssme49GlGp57v24izRNek8Sev0Jw2lIfzegve/Q6jo8utchQLJ9Y4+4O+b3b7Zo33nwV2gMo6ZmLx1m9XIe8XOO0mwZTUoOLy6xudmgZM/ysPUb5cs2e1sj4kHI4+stMCGTM7B0g/54SD6bYup5aGjYdo7uYErQ8oEX4pXih2FTtG+j6udO8vlVDeU30YROvuRzoepSyYX82i/x1dfb+M0Ok3bzgGDUg1SprqLejigunVftoWI43hKpYE0FvSf/X/yDxTixrrJlAzX1SVRCnMSYpoGKBdLK8PJ+j6D/9LWm7U9GqKCmg0fIMgAAAABJRU5ErkJggg==',{});
regImg('sat70',30,17,'iVBORw0KGgoAAAANSUhEUgAAAB4AAAARCAYAAADKZhx3AAAEL0lEQVR42r2Va0zTVxjGf/8iFCyUrgXUFopVCyIgXkBEkU5QHGJkOl3mMuOWecl0y4JGM03MjNn2ZVmcjmZbjAmbxmQTp1sw3jawDm8U8RIKUWzBFLAFlJZ7gXL2SWPmIuCHPZ/O+fC8v3Pe51zgX0rJXSb4HyQ9PzEVHhbK+EV47WcZar7J1ePHpLEWTJ81U2jlvZy68eClXtnTwdtf/iQC9Iu5XNaCL3gp8zcc4k2zRSzdun3EDkxMzxEAWdlviEhDwojQZzvO/rRI9GgWo9MFoVTKkDr6+KviEeO0MWjUgrhJrbRf/Y3z3x98oeCkvP3CtHUHqrqfOWP+FufDe6Pqkiz7w49FaOq7OFsGuXXBzmBHF3XuHubkxKDT+BjEh0ydQLfxE3L3l4h5m3aLxCUrBYBx9R6x89BeJig89OlyiZ8zf9SRyMqOFEmPSguZonWwaE0SN2392Ov9VJxuQBWjZE1uLL2dvTyofcRDVzSTkzOwXTpHRubrQh/SS6X5K4bafDR6hvFFmlCGjhdjPlx6U76YmLCAwOhlyCO16PVBVFqcKBUDxM2JputuOZYDOyhYs45tu3cS9uQuReYSrrrDCTCuwtNwjfsnd0ljBj9V/MKFIilvNf0RWdS7ggn0+NEOX6Tm9AEWZS5n7/5dhPttDNRasalXYD54BFefiqbqUzxxO14d/FQh4QoxI3894To9ZV/vkQIMOeKDwvfp9PayLtKFPzaDi1WNuO7dovTEcfw+r7S48HPR3e7GevQH6ZXBz0kATMnbR3JOHpr0adSVniN5uAlHvY3BgT5ammrRr9xBa3AufrkXQ4iD7usnsRwrll4JXLD+I5GVOQ91lIqSo8V0dcmQJa2jR2MksPIwzttnUEVFMWvjZ4QmZHP5vI3B/iFSjDqc3gCG2q101vyC/7ETu9UqjQqsn5YoZmXk4e1ox2jUU7h9I+bvirh9u5WeCUtxVx3CVXdDWvvNSXHLO5sexwPSV0wlbXIY1yrdXL/jQa6PYqpOyfKUIc4eK8b2ezHtjXYp4GXgiEnafclpC+jr7cd65TJXLNd5b8smSk+V0HrPgrveKhlmJAmZKoHQqVMYp1AS4GrjQqmdIEMUc+PDCJHLiIsc5s/yFh4+iSUxfz2Sp2af7GVgu+2O1Olx09/XgVodQmubh6qbtWgiFbQ2VEkADbU10t/mbZL7181ofWeRBw6gTkvG3iSn+v5jUmPAMzRAsFKOWjOeDsc1HJUV0ogZT09JFQkpswl7LZrmxiZCFAq6u5u59MeJ//Qm5phEuG4mqtRVTDRMp63hMeUX7KQtS2acu4yLX2ySRsw48621Isg/TLuziYX5qxj2yxHIqLKcprqifMRFmzZsEcQV0Ok3IDWfo/rHwtHdojjTEhGTmPLsCcxaXiAycvLG9F9HRISKue9sfsHzD+13r70PGdXXAAAAAElFTkSuQmCC',{});
regImg('sat78',25,17,'iVBORw0KGgoAAAANSUhEUgAAABkAAAARCAYAAAAougcOAAAD20lEQVR42p2V3U/bVRjHP6ftr++0MEppGRutjIGTt2E3Jqi4zeGML7hk8y1T4+LFXLJ45Y3xwkQvTFzijdmFf4AXZmpgc2/J3tyMEwgDOsvLLBXGyyiUrf1BS/srPd7hcGwkO5ff73me7/Oc8z3PETzGsvoD0l27g6WJPm53XxVr7dc9jCh0ueXDuPzmt2n46ls27m59gNu7b58MNDWviDU8LJGjwEVtQ6O8dP7k/yuV6mSK4OkoWmRhBXHw8BFZGXgBu8PC7YlP5fTooHhkJ5FbIXHp/EnR9NzOFVUpeoGydA+iU6Tm/hP58OjHcn1VgPafz7Kgptiz/z3srmIJsOZ5+ssrpSPfRTSmorc6GQ9dFYgiiVlAKgogmlpa5Iv73+fatSBCn8PrdePb5OfUiR/pvdAuDGuJRMJDQm+ekZUeSWvlIvOedVKyxF1V4nEVc/zMNFmZw+x0gVDQk2V6aobardV4/VX06tqlwWixyUxqYUVHeQ6nVBPxZczjVDj+zcv82qXDa8zy2SE/l28U0VgyxKnr38tA8276e/rJKzChLZrJLKbQyxzzswn81dvQtb7SRm2gURqNJul0OKTVYZeff32Mmpoaabfb5FN1Afla2256RzKcqzjMLwUHOP37DNmCfL77aRhP3R6e8G3AbLbgdjvwV65n/YZiEnOzeD1FGI0W9PWBbV9sDTxDal7l9TffZW42Rk1tA43Nz5NJa2iZRTIZjcPVIfQDfTSnO9nlGaJ4oYvhwXEG7nnZXOFDCit2q4V0Kom3tJA/r/xGVd1WQn09iLJNWyQIolMjGAwW7HkOtj+7l/BQL5G/h7Dm5VNR5uODkgFC8RwTsxqGqi1oS3MUTia4oG7mwME2uvvGSaZUbDYbucwSTruN6ZhKIja8urtM1jyZTqrLnNVkksf2b6TUb6UnnCDmbkJLxnlVGePIiQjb3zgEhnVkyZHLwtzULPUNTxLsDzE+0r36Y7xfAKC0opovz96ivNCKIpOYljqIZwTRYoWxqCoMVy7Kl976hODNILlUGp1eIISCUBTCfZ2saeGnm3bKjCYZjs0Tp5Bk7I5AKZDoDTCeAMBiVojPRCCtgS6HzW5DjSeITUYAhO5RAvXbW2QqtUCw67KAnEjGIgIgf8c7+I6ewb3rIwD+6usRFzt+oMhdxKIGhe4SwoNBgtc6Hj1WSsrK5aKWJXSj84F7y5ld4ChCw7SM3ZmIiMjgdSrKfShCY3QkuPaAnBwNMzkaXtUYRqedgiovyj953L0P7/3jnEgv3JXJeZWxkYHlWPE4/4lp4zZp89eg3bmJOtS5Zo5/Acf0j9ryUaXyAAAAAElFTkSuQmCC',{});
regImg('x1',29,11,'iVBORw0KGgoAAAANSUhEUgAAAB0AAAALCAYAAACDHIaJAAADoklEQVR42l2Uy24bdRjFf//LzHh8GYc4sS07NyeFNAVVFRXqBkrVFwDehEdgh1jAA/AISCAWqKxAKCAQqGpoKG2gJLQ0TeLL+Dq2xzOe+bNwxYKzP/p0jr7zEwBCCFPd2CbjuqxvNyjVN+j7PqmQCKlIkwQtBSAwGJLUIKTE0gKTGOIkASGQLOS6GZaKBX7+9huksui1m/jNMwABILZe2TO5whJ7117HsgTvf/gRBsmg7WNZFm6xQMfvYQGYhW2epEjbprTk4ShJy+8xnSfYloWrJEkcU9+o8uknH3Px9JRSpcb+nS/448HvhOOR0KVylZ3LVyiVV5lNBgz7fTrdEZNRgJd1mJ0+J5hO0Y6DVhZCGDCGVEge/Xof/9lTVutrlDe3uLu/T//sOaVajdvvvYswCZVKhWs33qD17Jgkgb8ePTTaGEjThAcHhyyvvsST0yZ+d0i/3cYreuTzeRCCp3+e0Ot0UFrBPCZNE44OD3l2fEK1vk5jb5eDH3/CJDGZbJYrb95iFsacPHxMfzDm4uyMSq2Gk3HQs3DKZDJBkDIdT9BK49gW58eP+e3igo2dbR7eu8d4GDDsd8m4LlIpEKC1xUp5lXAScPDD95Rrddrn59QbDZQWpEA0CwnGIzrNFrbtYlkSHUcRvXabMJyRmjlrazUs3ebya6/y5f1DhFQUl1e4fvMW9a11vv7sc0a9LkIKFp8hQMD1t29x7a2bzIZ9ViplNrcb/CIlnVYTJ5cnGIzIFxSdpo/K5rwPJqMAN5fDcbOkcYiWgnk4praxztbll9m81MAruOQcC9vWZAs5aut1ypUK3lKR1WqZUrVCNA1YLuZRcUjz5Ji7+/s4GZd43KPTaiG1zWwaoJXWWJbDbDqlvtngu6/uEM1CpFgkiGYRYIhmM5IkxiQJxeVl4ihi0O0BBm3b5LwlojDEb56jbRvLsqnUN5FSopXFfD5HolBKoadBgMhCkiQE/SFKWfgXf2PMYnPZQp5Lu1c5OrxLY28XJ5tDYJBSE0URWiuO7h/gZj2UtLj9zg1yBY9Bt0e/3aHTamFikEhG/S79bnMx1myuYEora2RzHqNhj55/jtYWUmqkhJXVNfqDFtu7V8l7HnYmg0AwHY8xKfxzcoTfPKO6vklta4f5fNGInclw+uSYYafHJBwQhSFhOBbiBURQyjHGJBiTIOULtkiJTCXzNAIp8fLLL5AiwRgQIJVFHE0Jgh4gMSbh/5JSk6bz/279CwlsndoKPwRFAAAAAElFTkSuQmCC',{});
regImg('x10',22,24,'iVBORw0KGgoAAAANSUhEUgAAABYAAAAYCAYAAAD+vg1LAAAE+ElEQVR42q2Va2yTdRTGf/+379tubWnLugLtLkyQMcbFzQ0EJRO5OoPEgCFcjJdEgxoTNaLGqBGjiUQTNcF44QOJGkAiF42CEgQRA8Kw4zYuImxuXTbWruu9Xd/uff9+MF6IKIb4fDs5J788H85zDvwHzZ83U/J/66nV06Se3yEfrqv8/+BPP36XlJnbpOx6TXY8NEXObrD+Z7jyT41nn5gjX18zBNH9FApQNaCyaVYVd9S55TWDH3xgkVz71g2QPYJhqUcZaSVTHWBkt2DDSh8zKu1/g08ud8irgkfnW+CTFFn9GRT/qyiOW7FOm0qPGscI2nC5L5/fu3WBXNtcenXHejwF27ehfrEZeWwTmAJtzlS6fy6l76KJTfttbmFDifz8g7FydnOBKqeLmQH1D9fqlcCqZoMRFZCMQvs28EeI7moiFsxSM9+B6DYAeOn522ickCDX56V2vMStDP27Y4tNgJbHHGtClYIYHifamcMwdKwWhcECjKt1y8ZCCONICeg1JIpLWDXL/6e5e5cvkTcp59jQepHg2UEBYI60gyeHZVQGhplgKCgaFBAUJIQzgsebbZinYuBIcv7ZVghKfiiN/QkePX4KjzYJls0b4KNzDvnk2gvCOtwDegqJCZYcGAaKNCkWKj5bEbmCpKRgoCSt5DSTnqDkFzXH14UR1DW4ZW8kjKKbZzCqm7B3aSw4ZMUmkELJgycPpgHOWdALjnw/yCFe+SmMy6/RiIdsSqO9LctnqgXvaw0cPLSEY9+s5p4p5SiWwbNYyga4EGygIpLm9iqNUMpAek3QNcALOYX+eIR99RUsfncsi5xWYuEiMIfYtSfDLS9PZ+myRcjcl7BlL1rrCZSPP7xAePcG/CtddF+ys9rnokQVCL0IGVMxLrRAxsous5ul79XQGKhmZCrD0Y4Mp4OScKOde6d/S7b7OPbShXD6IKrLixLqy4p1b1zCO/EAkRnDqB3wUNuZ4PABE1uuCDWtc/7TdpQRXm4oa0WmA7jGCxRTZWs+z4OrXJgnQ9hz5+l9P06sMk44kPtt3TYeN+hti1P/QoJOo4g7pZOvQikGu1Xo1Lh4No3PlsbimYtw9uBur2Jvn86k5SWMv9SLUnoT7Tvg5GM7ObRLYf2+pFAAOqKGeP8DA6dhIlcUI844mezW2N02iLhoZX/SSfOoVmSoixOPdeA3h5jYbLDCmgOLnXh/JUde7GLK1OG8cS5/eUC2nMoR2pihdmaWH3VBY5eLzh6FWEQwZpKJY/FchsJRokdjJEotFCIF2lsyUK8RfPE0N45RWS+ifNebFJeBf4oUxFs7ExSd6scx24oc0LDHNda1SSoDaezFUQa7BvGLGE5TwdHvQqkt8P3b4AzBAV+SNS394o/0/jXKnvJha2JtHqbbFE735sgub6S8aQTB7TnKRiUIdKU5ukflnewgNivo3RrxoyqTlhjct7+LZF6+fMVbMa08QFVTgMOmTsv1sPzuMEtWOKko09j0po9N6y2kl/l47hE3mmLjTKiYiTMsbOxxUe+puezeiL8WPqtHLhhXxtTqEsy8QPjTrLiun70Hi3GPLSWaN6keozE8kuXQ2WKO9AiC4QRFlhShVIKORFRcEfy7ivHIGp8DQ9eocHq5/2aVIauV1nadoWyecoedzT+HSeopzif6xDW+VEXWlfjlBGeZBGSgqFTWecuv+vd+BSrhHElVpLPkAAAAAElFTkSuQmCC',{});
regImg('x11',11,11,'iVBORw0KGgoAAAANSUhEUgAAAAsAAAALCAYAAACprHcmAAABiUlEQVR42l2QzW4SYQBFzzczTJkZCJQOtCm0WprSQDuNpDUGTY0RE1e+giuXvoNv4iP4EsYwO00xQmN/IlDAnwAVy4wzwOfCaKpndZN7NvcK/qNyWJFFZ5dGvU7ttSuud9qfcKe6K1VVoM0E8yBAU+H2vZIM5/Cu9kEAKADLSUvaS3EeF9e5W1njU/uYB092yNoxDFMnGY1IAAHw4llV1twT0oUV6q0hBFP0hEliQWV61mPzVp6Xr1yhbO7dlE1vxnbVYbtaJjOf8vzpfcLhGOdwj/zDMq0QbpTWpRYdX+G+aXCwv8Hpxx4pO04kZ7Ocs7l428SfwfFRC4s54lF5Q+qrNt5JC8vU6QwmOGspji4GpC2Db989soUcwdcRWqM7IisEOwdFFjJxQrdJYX+Lc6/JipPHuvTpnrbp9Ee/B6bihjTTi6CrLGmCZDHP2ftzQj9AUWDcHzK+8oUCIKImk9EPEpZBZOKz+rlP6qdHOpMkuJygRQ3+XncNWSnlmMVidHoDuu0v/zi/AC1Fi84aau+9AAAAAElFTkSuQmCC',{});
regImg('x12',12,11,'iVBORw0KGgoAAAANSUhEUgAAAAwAAAALCAYAAABLcGxfAAABrUlEQVR42k2Ru3ISUQBAz717d1cDrMSAEIyRBIQUSeFMGgtstHDGLp21jZWllb/gV/gDWmVGCzs7tXBk4ohGUd4Lya5keS33Wjg6OfWpzhGco1DcNGvqEvtqClrzdmIzXJzid7vin6MAChtFI2WSUmWPQiZHygJtNCU/xO10uHgha4SOaDa/CglQ3N7j0ZOnpLMpPh/XOZr6NBYBg7Mu1yp5Hjx8TP7qDgCimPJMrryPu5aidu8OzS9D1qsbKNuiXW+TLyR5/fKQOPqN//094qBSMWYS8TFdIrGiWAYBRXuGXi5pRBaxcDhbLKnZAbFto/qbVeJIcvNGHsfdov7qOQd5jxjJs6M22a0astfgV+469opBToITTv02rmvj2sB8gbs0JI3GFRbKdXFch37vmEG3hZBCmVu37zKdzbFsxbDZ4n45i5Tw4lMHnBRKCbxVjw/v3qC0iUW3FZjtnSqV3RLh7pxhMoEloex8w5ILTgYhQ7+J0VoogGDcp/XDodP5ySwak8lexmjDoO8jpERZDkHY+5v1/GkvnTGrV9ZJJjyWsSYMRoSjAePx6L/3B2pIsA0Jr22WAAAAAElFTkSuQmCC',{});
regImg('x13',11,10,'iVBORw0KGgoAAAANSUhEUgAAAAsAAAAKCAYAAABi8KSDAAABaklEQVR42l2RTS9jYQCFn1dLc6ut1gKtGiwawpAMQkRIpLbMahLxC+z8hUksLGfpH/gFtiYTKiMjJplIVNPSD1q0Ss390Nube99ZSMU4q5OcZ3FyDrxTrHdabs5vyVh4Rr7PRNNEP0zKTx+XsFSTYXWAtL/E7X2BVOEATS+KVzjcNSpXFjZYHF8greTZO9zFbwcZeuoko5b4ntrhr1EULQCBUBTD1EhaabKFc+qJEnq9StZbwbZdeDwhAFzx7jW5HP7MfvkHj40qg2mT7fVv/Dn6SR6N3F2S1egX2qzgV3fDMTHsBkpbAF17pqoJhC1wRAuqUUM4FoZVx2l27gqOy7FYnJGeKZKlBGpRpaPPw4Qywd7FL3L6MZXa2Utnx1Yp31yQzfwmqgxwFbrEbQhOrk8QPgev4vl/uk5fv+xoj+Bt9TMbmOOwlkA3n3Bcz1yXTwWAuwk/aHnxoOUZCk3LngjISo1C9Vi8PeUfMV6ZW/m/8rcAAAAASUVORK5CYII=',{});
regImg('x14',11,10,'iVBORw0KGgoAAAANSUhEUgAAAAsAAAAKCAYAAABi8KSDAAABWElEQVR42m2RPy8DcQCGnzuta3ttcyVHKFG9oCoWIpZKGMRsFIuEUWKQiEhMJDY+hcFi8QmMNkJCa/C3xV1RVdejej+DMJRnepI3eZYXaug3RsVialkkYylRu3l+pCM2LOLxARpVnWC0n2TEj1AjwsqmyRcyEoAM0KwnxOzUGnvzWyQ1nZ2TbRw3z9LEPCPd0wT8uvgt+8NRik+H7J675OpcjJsgFU3i+OwIx8njU3XssoU01jIpevtGMRWLl8Abg48+VhdWWN/cYN9vESooNJU1suYp8kPljmfbpOoInvNFEDJuVcatQrlkU1+nYLslzEoOCSCi9YihrnGioSCndg7tRvDa4aHlM8xV4Z7M/QEvxUtJBvj8KOL78GLIMQLvCmnNxGNDO200eJrwemWA7zJASG0TqqLTqSdpDbeTe73lwjrBeX+iULqW+A9djYmZxJyIasafU74AE/+Ce7w+p4gAAAAASUVORK5CYII=',{});
regImg('x17',26,25,'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',{});
regImg('x18',26,27,'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',{});
regImg('x19',18,28,'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',{});
regImg('x2',13,28,'iVBORw0KGgoAAAANSUhEUgAAAA0AAAAcCAYAAAC6YTVCAAADG0lEQVR42pWSS2skZRSGn7pXdVd1Um06SXeSTiZOJMngzCzCZBBBBMULA64Gd279Ce5c+5v8BQoKIkElJM4wMXe709e6dX3fcTHiqJ0WPLvvwMPznpcPZswH7zwS/s98/tnH8u3XX8nDrd1bQfPfi4f3tuXJ+3t0umMebG3RrG1MgfbfH3sPtuXLLz7Fd23EFN77cIfzkzHqRSFX/TPjVlO14hLPVciyjDTJiOZtVtdidPkfJqVMfvrxnOOjCxYaEZWqj1IlpdKzb3I9m3Ki8XwfTAsRjYimUMVsaGUhpnuToETTv0kZjDJ0qdhorMyOt7mxxGCUgCFkiaIUm2Z7juh5ZbYpLUpEQJcQ1RwGw5zWao2JKmdD5USzulpnNE4RDc1mjfFwgmNZt0PrrRVpLdY5Pe3SbMVYlonn2CiBWhCyvnhHpqDdnTZvbDXpXI8p85Iw9Bn0MiqhQ7wQIaUxXYTjQLeTEMcVVtZigsBGBLzAJKhYDJNkOl5gBSRpjlKKy4sBnc6IJC0YjwqGgzH1WjQNvbW/S5rm7D1ap5hMQIRK1UVpTRj61MN4Ol5tvsrRL2dMihJT4PysB5ZNez3+86eoaVNRZJRKcXR4hWUahGEFz3fRWqGVRm6rfDxKsW2D9p2YlY0Yw4Lj4wsQA6UE07ylvSRJKXPh+28OqUQu2/eW2d1Z4uy3AcutkKNfvX+a3n38pjQbddJcsf/2Jh89uU89jnj+rEPgWVRCj5ofsbn8uvwFZSUMejmBZxEEHqcnNxSFYmlxHts2KPMCxwZEXsU7ODjBVd8RhR7FgUaLRovguR6uY5IMFf1hxvWw+wraudvi6Sf79AcphmEABhrBwMDzbPr9hJ9/uOSyd80w6b2M93hvhxcnV2RpTiVw8H2TWtUhCi2eHZ7jOkIQuDSq9ZemWvCaLDXm+P2sR5HljAY55UThejaWLaRFQZYLrmtRj+aZDxfFvn+3jW8Ia6sx/V6CmGDbPuOkwDCguVzDdywWFlyG3YDVegOjvbwui/WIyUTjuRZag1IawwTDMLAsmzIv8TyLfn/CVa/LH1jlYsQ9qJdxAAAAAElFTkSuQmCC',{});
regImg('x20',25,28,'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',{});
regImg('x21',25,25,'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',{});
regImg('x22',20,25,'iVBORw0KGgoAAAANSUhEUgAAABQAAAAZCAYAAAAxFw7TAAAE2UlEQVR42pWVSYxc1RWGvzfXq/nV0PNgu+1Q7eCA3GqwwZIRNhIZSJRNyCBlw5oobNixYh0pi2QRkW0UIWWBwgKEkYisMFgIo8Y2jRvsLrvd3VVd1TW9evWm++qyIAQZcJPc7bn303/OPec/Ct9xFL0qM3kbVZW4XReZ9JSD7usHBc1sWT7xy5M0bieomoEzA/9+dU2O9nbuCVXvFUg70/J3fzhHsp/izpsm7qUM0fUCv/n9WYqzs/L/AhZmDsvn//gj4ljh9o2AY6dtZC7i43eHXHttxNPPPopdmpT/I9CUP/xVjdr9WXp7kpdeX+XFl4/y0+eKZJY95k8atNuCY6s1IC+/E+jMTfGL3y6x8UmL8+dn2d72Wb/i8eAjDosrNkd/YOE4kqXaBEq2eLBCFVWunD1M5XCR3dsRYRAxTqlMzKeYmrJ5cNXhTt2nMmnRavY4sTp18C+bZpmzGw4f/HkLrynJzllEax5RRqOuD9lpDuntRRhZFamDOVsAvSgRX7XSXQrnV2ZpPWbi/vUOtlSxGoLLrzRQS4KZqk6lZNJrx1gpnWLRpjKwsZ3St6ds2o6cX6rwvafL+KtF7vcU1v6yzaGfOSQTClpGoVEPKaRTnDs/zVTK4sjI4KGfHP02YErOLy8wf98Ek+k0xYxOYRgxOJ7BMVT8d0fs/qPDopWiF8W4fYFTS9EUPtm8jVVYkF8DZihaRcRAsj+I6O8EvN6JCdwA90+7dN/xKJdNHvh5lVze4uLFfZbWEuZjnXpdMFM7fLdCa8rmuJEnvuKysT7kEWxOhyaVN0Z8OK1z3wsL2Ofz1G8OIVG5cX3EjZFP9GObuapksNe/CygXT1SYfziNVQ9Y+pvL5U2XD+wE7/EcK88ewsGiuRlh2gZOWWNns0t8JkOfiDNPTDG5kEbRcvI/baNj2TmuNwJWBhpX9oc4L8xyajnP2BXYgWD7Qhd92mAw8vDCiJSuMnilQ+Fsjk/rPdKlPEY6T+S66KBiqBr7IsbQJZsTsDht0Pv7Ls03hwR+xORTVUqLDoEv6O4KVs9NYmyEdF5uceeEhRAqhqERfaHQpN8bk9ZVVEtHS8Z4LzXJfBIinsohsGhMJKy9sUWjFTK3mKPfDxlWYca2yH40oruQQ9HUL2uY4LU8FFvnljXmmUEacTXg7RUT44iGUVa59K8OcSg59VAJ1U5wR/DZRyNSiYpQUwg/QfjBl0Bf8fY6tANB1x5zLQh57SRUazr6hQHXLrik8xrLx3NIkbCzFTKMxlidmFs3+2zmFZJ+RDBsKP+d5dj3aa73KFHgauKz34fuW2NKN0Nmf13m0GyajesejbbAEBpWK2bLC+lFAhEIvJ39u80hGA6o9QNkPs9MqKB+6iLSJTYMnXY9pLcTYxU0DENhsJ2gDb4wh+VMlk40phdFXwENOy+/f2qJ5VGJt1od5qoOTw6LXEoiViKTy1ddxKKNvR7Sj0CECaKQRUtlaPqSY6qKsTJN2wpk83YP3UrnsTM5/rm+TuxJNjWfM7kKE9EYS4GpVkDXjziiZGjqPhM5i614hLvn8V4QwWc+tFUeOF2j03gf5ZuOa0ol41BdmKEcGaziUDJUWhmFi7kIOYxo37rFqLULRN94f8COTUkwmclVWNDzhKbk47hDPHAZi3vv5s8Bn14hdHqpo/8AAAAASUVORK5CYII=',{});
regImg('x23',14,16,'iVBORw0KGgoAAAANSUhEUgAAAA4AAAAQCAYAAAAmlE46AAACnklEQVR42mWT2W4bZQCFv/+fzR7vSxw7ad3YtGoW1ND0IrKQgDt4rD4Gz8AtD8AqVSBUtSRRVaNS6taJB8ee8TIZj2f5ubCoijhX5+LcnE3wAQqWpR4086xSwdyokIY3ZFhTtxLOHZ+JH4p/te/JcbulPt7dQgLkypSzBiQJ16GgGHtE6xU//zWlf+UIAB3gk1tltbfb4vjBAU/eOJSnA2pxihSSdSD42yjROz5klPYx0kBdOHOhlTOmaleyPL/0yJs6+60ie59+RYyGevglldtdrPmIF8Mpv/b/pJnPsAjXj/UkTRFKMLq+Ft/88ETtb9do9B2aBiSvL4nsCme/v8L1XJzFjegUm2odp0hdl2hy4/OzTg2LiGevBjydRzwbjvnux+/JpT5ftAsAZDUNTcpNOI92K2oVJ1w4c3HarqqdYp58VkcKgTubEaVwNlrwbhGKo0ZevZ7ebMIxdUGcphxs5VWcwB+TJQUd0C0WhQ7u2z6nd+q8Ox+yiBK61Rwyb+oqZ2g8v1qKu40SWxmwKtuo7gmZziG3tSWnLZuj7SwAJdPEzljopq4TpZs6v70Yit6dmto2XD7yPUazgEjPMllFfP3LAADDMpgFIXK/kWcWJgD0bhWVF8T01zl+kzsM6we4jQOcAI5qNoBq5gxejhdCT5XANsDKWspJLU4qCWXbx/V9oijGC9fcswXnSx3TNJn4ITXbVFoQ8bhZypBqFr3D+7iRYmyU6HS6FBst5srkagWP7nfxlisKMsHxV5s6CpahPj95SLdVYhhIpJ0jV28ghGQ5nWB5DnVb8uLS4+nFGWM/Eu9HXi+XVe/eDtVqlanK0g4HxOuEN/YeO8rFmQX89PIt8+VC/OcdH0Adde9SNwJiaTH2Y/qDwf+0/wB/riJZSNnKIAAAAABJRU5ErkJggg==',{});
regImg('x24',16,15,'iVBORw0KGgoAAAANSUhEUgAAABAAAAAPCAYAAADtc08vAAACsUlEQVR42n2TS28bZQBFz+cZj+0Zj+3Gdh52nAyJktDIECgggiqKALFAdIfE7+mvYcWiYoFA7FCFEBAEtJWqxHGa2Klje2zH48d4nh+LColH4a7vuatzBf+Tve2afL3+Mj17QNbU+fLrB+KfHfEicMfakIdv1SnmTeYzDxlDLH3Ky0t88dV3NJoX4oUD5eINefej2xzs75BUVYLA5aejxwSRglWrUioa2P0R9799wPFZSwCof+Hl3Q/f4fYbdTIZjRsFAykl9f1djn5vIKMYLxYMpj5v7m9z2bPlbOaKxJ/0Z598wHuHt3AXHu3LHienl4wdj98eNslmMuSSMb80bY46PqHQePfVmwAoAFZlVX768R0+/+ZnzgYeb9ctXDfkx6PHbFpV7v/Q4KJ1yc2tCl1fI3CnlI0EsZT3lLyRke8fHnDtBnSDFPqaRa8/JJfwiaOAruNzOgxZRPDKisZgFvC0N6aST9Ht2iQqy0V2rSqN9gA9a7IY9ajkkzjOFMuqIVyHvdoSsxB6owlbJuhmDjWVQk8pJKSE65nHVOikjCyVtRWeNU4wCyZbmxXiwKMYOwhF49f2nPW8yq2Cz6B9Tt8eoXqex/pqkYMAHl1dsVfaQC2XsDarDPpdhBBktQT7ZQ0RSB4+adLqDFjRYpZNDSWpinsZVSFvaLxUTFPKJpm6C7ZWTZzxBC1a0HF8MtGM5vExynwMeoFJAN8/aj4X6c5ru7JaLuNGMZ4fkstmUESMH0YYhsFFq0tKAduD7UKSzlRycn5Oxx4+9+DJxRXuYk5lSScOQ6ZBSGscYLvwtD+hVMxBMoWeTjNxJhyfndKxh+JvKud0TdZ3aqyXS0Rqmv5cEkUxOV0juRizcF2e9fu0eyOu3VD855lMPS13NtZQVQUhBAkkXXvMcDzher74V/8PR2A5AafDWHcAAAAASUVORK5CYII=',{});
regImg('x25',19,15,'iVBORw0KGgoAAAANSUhEUgAAABMAAAAPCAYAAAAGRPQsAAAC90lEQVR42o2US28bZRiFn++b6TgztieOb4mbW52mcmhaUClSAZUVZc2GBav+hP6nLtixrSgSFxGJtCDUFBBpojRpEsdxPPH4MjNOxh7PfCwqIYFa2nd7Xj06i3OO4C2uXMyp0XiEbWpUF8u03JCtnYb47594I0lLqU9uLTO3WCUa+px5HQq2TjBQbO46bD8/Fm8Fe6eaUdeXJ7m6EPP9Uw2zcJnqYhnHadFrt1ispDEMg+9+3mPv4FT8C3ZxOq8+vfMBppni24ePuH2zzI1alngc4SezPFw7xHE6rF67zMrKLK7bwW3WebdW5OsHf7x0ls1m1IfvLzA7k+VFY0ihVKJWq7C9vcfArSMn5ikVJomHPtl0Qstx2T4MufvlbZr1Q37daJCzL6BVF0pq9eoSUlygUk5TnU/Tbrs823KYmZmmMn+Jbt8nPu9y5+MiS3Np0tk0Oy8cNp5scvP6PL/92WK/MUCsLFeUZdmMxjGlgkVxKs1KNY1l6aw9bqAZab74/AY//vSEdttDCR1Naly6aLP2yw5TpTkO6g7jaIgMwxBppLj23kcYEzmcts8Pj1ts/NXh3t1V4sjn/v1v+OxWmWc7p9imTqmY4+mOjxdEpFISpRRKgRRSEseKQeCRyZaJyGOZGkiDrx7sU5nOEUZj1n8/xcpYnPQkm3sBQmjouiBO1MtYqAQphaTf7aFUQq/fo1zOY04ucdRs0/EiXE9wdDpitxHj9sZE4xhNQOvEZTCM0ZKY6CzgbBAgY6WAiPX1R/iBTxwneL5Hp+ORJIJOf0RKl2hSQ5Nw0nI5rh/hdV1sy8Dv9AnDELd/JvT9w7aoFCeVbRoc7u3S63bJZicBiUCga4JEKRCQJAmDnocCnO6ARCRsPT/5J6s6QLPdF4UpS+VMk3Hoc9DtU6vmkVKia+AHI4b+ADU8p9UdoJQSAK128P/dzNsZdeXKApZpMGHEHDc9xvE5yRjqzT7B+fC1FXylMF201UTKYDgacnLqY2cm8ILwjaPwN9/3Z6ik4EUbAAAAAElFTkSuQmCC',{});
regImg('x26',18,27,'iVBORw0KGgoAAAANSUhEUgAAABIAAAAbCAYAAABxwd+fAAAEy0lEQVR42m3UzW9cZxXH8e/zdu+8eMZjj+1JOrFTx0lxE4obUkAoq26QSiqQ2PYv4F9gA3v+kIpdViwqQUpAUauQtjQvDnFMEr/EHs848cx4Zu69z73PYRE1BTm7szmfc6Rz9FMA1z76hbzVXmBnp0uj1SIuEvLCI0ohISAISimUKAAkBHKtsMaw82SXm7e+UPbjX1+T3376R4pyhc7TbdKv1nn7Nx8y9h6lBaXA5QXVrT36K2dBgdt/wXy/z/76Bjv/ec7xaCxWIez/+a/4qSrD3T32kwL7+ReMdg+Jz5/GNOt4r4mzDP94CwKYO1+z8WSHqNlkMhkT8gwF8N7yqnzw44skohhNoCIpzkJkLYV1VMqWNGiSNCegmYuE7iDD25j1+/d4sHFXWYChQK3V5Iwacb6pkaiKcwpfGIwx+ExwRqjGGhUKdo811RdCHDLu3gsAWICnTx+q5901mVpr89WjLYphTpZkfNAWIpOQpYHCVbm9r1D1EnE1YhIcG9svePT4vnoNAdjY0lpaYXDU5yjp88t3hI+uZtBeg3IDHv+dz27kXN8xnG4KB3mFUZZ8147+rvjHX75k3OmR5oGD44JLKxFh8Sx+dpW01CSsXOa9iwVHRyk9XWX36RHf/PP+Saher7B29V3WlmssNQRXCuiZH4CfYF/cQ4UxUdVyedly7WxGszVNvR6fhLz39EcTXkYxWeLRuYdnn6GCR2YvoZzFGAVB6Loa2VQV+33791XIM8qxYSYKaAoKJWAc6uBbSHpQW4LqKVzZ0FqZJXKOACehZDzm5uf/AmNRgHIGShppX0WXTsPgEUrAaIXxYIJG/od6De329tTtm/c53DzAOI1kxzD7Q1QUoDQF8x++GiKCkCNK4E0brSyuyO9+/wlqeposOEwIEM2hfEbo3IL6KlLkIBoXGzb3h7wcDE9CzhhGE8+5doVaySJRDQbPCKJh4aewdx3xAwjC9tdb1NMxtUrtJITRoEHygNagogi6d1H9x6jSOZCAApIspXGmytRcDaPlDZCAFtAatORQXYDWFdT4JXRuwuE6aAtaEU85gjGEwJshhWLoIQ8GddyFo02k0kLNXARTQ5Ihohz9sUYrMMachCJlKYKgCk8RCophD7IhoX4WyVPwKYqYOHaUtEe8onjTQ47TACFQmDK+EKx1SFFA/wnoCKGM0yUKDMZonORMJtlJ6O2lBdZvPWTzm22ejx37XUG5Oq7WxPhdVHWBnf2cZ4dw405Kmo5ZWnzrNfQ6Ri6tznPlZy3+feM57Yrw6W1HYTwh/xuIEAF/+tLy/oWAbtTodA1nZhv/D52eb8vAW259u8cgTHHq/AwhLnP9YEKYbaC0QsST/sSznmTIXkpzWtNzEcuLK/Jke/NV1Fo9xfKZBcgDtcKTjHPKImgpwB9TiOCznIoCCsHoAlTgXHuWOw9eRYmtlVvy/pVV5i4ssPWgy9zlVUzZMN4+wCpNPFNh0BlQObeE1Zp8YxdfqhItNak/3OZH7yzTO+yJWXv30h8++dXPUXlBMsopEWhEllhpys4QCZRxhKNj5kqO6XqZMPKUCdgoptWo0Om8RLUXLsjiqXmSrEAI5JmnXNL4vEApjdYGrS1J6qlUXt3meJyiNRiliJxlp9Pjv7oJPdu1goFeAAAAAElFTkSuQmCC',{});
regImg('x27',23,27,'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',{});
regImg('x28',15,21,'iVBORw0KGgoAAAANSUhEUgAAAA8AAAAVCAYAAACZm7S3AAADhUlEQVR42mWUzW8bVRTFf+/NG4+/YsdJXNshbr5KqpS2BFoidZOqCMGuKhIrNuxZlT+CBUIIVUIqKxbAilXFskorFiWpoEJqQQEpKWoCsWM7duIZe8aer8eiIk3Uu7vSOTr3Xp1zBS+Xnn3zHcJraWQ6IvISqPs227/dQ4M4DlTHm6nZN/TIWJGF5ev47xbIXirhPmrgbj+GXpfQD/S/zx6Ll8i5QlkvLl3lVHWOjbX7+PeGDK0UsneI068xc36ZbC6HfVDXdrcp4NgYi0tv67H8FH2zharmECJCATGauO/Rq9koJ4/nd9jaWBcnlLutOtZUnvOffogps6yt/Y45jDCkJFN9hVLGpfnZj9hP9o/WlABSGTplZklfn2Uyt8ivP2zSfhpgDxT7bc3m+g67LYvyR5ewRApDKX2krJXAtCxiP8DpdAi6DtPvFSlUkwy6IVvfPyU5EiOaGiEFkTimvHL7c4KugzIUUVIi0oKoHyCUQWD7hEgGnsJMGAw6h6x888Xza9/UWnfW77Jp20hDIoWCXIbGZo++DWG7hxpNIg2JIQyCgcfswgJLWmslgMBz0RpMadCVARPnMlSm8xzseBSWSjQfNBjueYhxjRCCwHMRgPxSCKGVgSYmoRVRxSdZNfH9iMlrKXwzonC5SOVimjiOQQiUYXBLCCE/1oEOezbCNJFJgelo+h0fswKzI6dYLFWIwohc2SL0NEiJdnrc1L6WFxGMD4cIlSFpBZhlTeKM5Gx1EscPOFMqk5hMMj1TZCKviISF4bnMYaBiPOIoJDsxxt7XTezMT7x65QKphouv+9Slz9nxEQ5W19m9XaMyP88gCvHxUCFZ+rkR+u0Wp99fxvuqyeontxjNlYmJiIcBh/0GpdOvMXFlgeHGE1KjoxTIPPd2qjiu50qXKX0wz2itxF+PHjJ24y2ilRLphwf8890q1XOvY1zQNO9ss1l7QH+vLRSA12ozyDtEUhIQM2i1CMsm6uoM1EMie0gcR3hdH9vZp7/XfuEwlUgQxQN27/5Br3VAnFWoeoD17RbxjkecAb/r0Pr5T1y3g1TqRZ5D3xfKTOrccJzD/RqGadG+8wvu/j5YkuHQplH7m5Q1Rpj2icPwZCRbzWek80VSRoHp+TkGbpdUpkAcBggpGbg9EukETn3vKJInftLoWEkXxqdxe10MQyENg/h/oICD5g6ue3jE+Q8e7JbV9793fwAAAABJRU5ErkJggg==',{});
regImg('x29',13,31,'iVBORw0KGgoAAAANSUhEUgAAAA0AAAAfCAYAAAA89UfsAAAEH0lEQVR42pXVW2yTdRjH8e97aN92e9uu2zq6I+wEoxy2OQaNRCCyDDZGPAS5UDwR5UJNxMRowo0oRLgALtSYCBglkaAoxCxBk+F0Mi6AmG0wGcqG2zp27raup7drX/p6YcKp48Ln8p988nue/z95/vCI2rCkweD/1K4n3jUuf9xrlLvr5oXiwwc7ve8ZezbuRU/Gqchfz4qC1MQH0PPet4wP6/cRnQ4yN5dAi8/yWEkj3sXbjUeiwaDG1VAf1gwTFtGESy0APYlLKcae5roL5ftRAp0T3e0sTFepKsvjH18H8TsakiSBKM6flCGplKoF9Iem6A2PEIiNsSijgrriBqLx0PzIZIh4zaU4FZV4GFQxm5e8W/FFbyIY0vxoJDHO+alOSmM55DvcfL37EL7RSa70d2BRFOadySLYKXa4WOstJS3dzKEvv0CPWNi5+nUOXuhKTcp0lBnrSjazrWoN3/92ic9/bab52jmsziRezxISMSUVmUUL4wQ4/nsrYkgkGozzVGUjbzTt4JOfvgJZT21PlgQSuk5XqI/awhIaahuwhAXOnGllNjiHJAipKBwL4bY4efPFTdicJvr6Jnj/2GHMisiBZ/fzwtELhBl6EKVJeby89knGZyb59PQ5cops+BNTbCnaiMNt4o4hpyaVF1XTc3WUK8PdaIpGbM7M+oXreGfbTo6c+gaTaoHAQ0gSBX4YbcOt2SlX8miqriGn3sa3zS10TlxHMVlTkwxEsuOZNG2spqwkj0G/n137P8Byx8qu5a9xtOcIvQ9fuUvK5qNXt2NKE9l38hQn2lqJiFG6/V1Yl+o0eesefKfcjBpj9SovWlTj8NkfCRLGtUAlGA6zKreKNYUe+vonsJrzjbvIrEgMDE/R0nadxuLHyRYyCQU1nvE8zfE9Bznb2sqVW11YZPneTEZSJ5yIcHaqjUrrEuoX17B8WQEOp5XPTpzm5OXvcDgcGH7jHpJEAT2qs6molvoNlaiKzN+3p9h95AA3Az00VGwhEA1xa/LiPTQT0ajILeXtdXUc++UiPw9dwuXKYlgfpMLpwaOu5HygBVH4rz1BtWQZm5a+gixbWOaspn9uGkkWsGVb+eNSO7nmXHpmu9GNBJKU4Ja/HbkkezVba3YwHhngxkgnY9OziJJCob2cQHySkXAPWbZCCuwrkJJmorEZhKrCJqOiqB5FlliZ78FqERgN+vAFepkc07GaTMzF4kyGbiOhMKHdQNTiIRxWOyJmOnzX6BocQEy62Ox5jnAwyO2xIcYCQ1jNNuzpWWiJMFI0FtgbiIyQk+YmXcnCkAXEpIzP389fgx1kpueQaStAN2LcGG9lOjKMcP+OyLUtNxa5KilbUMuirELa/mxBFA0GZjoZnu1BT0aER34Aquw2qvMbDZe62ABSdvm/99eUk9dJfUAAAAAASUVORK5CYII=',{});
regImg('x3',17,28,'iVBORw0KGgoAAAANSUhEUgAAABEAAAAcCAYAAACH81QkAAADy0lEQVR42pXR224bVRTG8f/s2TPj8yG2Ywdjp3WTitAoVQH1wAUgBEjADZfwAkg8D0/BE3DBDYJKHBSoEAXRqqeE1k4cJ3EyHs9x780FElIlcui+XUs/fftbcMZ7e33DnLUjTxt+cuOa+fz9dcLUNT/f27RO2hMnDT64ftN8fLVLGAX0Sw1axRXzQsiNywPz2RttCp5HHCve2SiwurjMYqVvzo20ygU6NZckiTiap1jM6dQEWfICnVja8M2dkEf7GUt5cN0MlSky9PmRMEwZvGxzoefgjkIOLMEjS2GUOn8nnUabyI9YEQFJMeVJoEAp2pXF8yWp5Cqm3yxwZ8viOHEZHxi0nXJpyeHZ1CLnNk2UTKxTkzhCs3sYUq3Cpx+6XGtJDgObblMiMAhtn50kMQqMxYV+ju2Hil4X+pHD9iTDFpCRno306x1qpQL3HmbsjSQ1zyWNM7RJcIVHp9xi+/Dg9GLXeiWkI5kFiltXPHSjzN/jmHLepl6SZJk5+zqeNESpzes9BWLGK94uhWJGGCs8VzFPo7MRmzL+HFyZp+InjJ4kzGKLMM44DEJq+eLpSLfWNq9dXCTTivu7gjGSrbmDUqC1oeBC2atRcBfMiUinIalVPQ59QbMK06HgyLIw2lDwbIqexBgLRzonX+c4iJknGdoo1pseg75hYy/jy2OJtDU528a1HRKl/x9xKZkggtEkwmD4fU/zy1CTpDlcNyPLQpSxUFrTK1dxi665e/DMeu47713ps7zQ5ijQuLbN7QcHbAUhv27vkJcOu8eaSNsorakVy1wddJ/v5HJ90ax1CijhkHcc/JnPxorHm6s1wiijlHNwhIUw4EqLHT/iQrPMUrlp/kNurbYZTVOS0PD9HxPu7+3y6lKe3+4MmQQxP/31lH0/IlERh6FPqhRbY8X17r9ppCtL5lK7wQ9PpnzxUZ+Jn/LV7TmTqebxVLDcyrg5AM+VlDyH48ji7tBmHmd0qzmgYKQjLMIEnu47bN6zKHslBnWPQV3zrS3Z912CWY2pynicZAynCiEElqWJUgXYyCDxrWmImScB3/25Raok1waGvCMQxmE8i/j67ja2ZfDDhJiY1BgMC0jhAL4lq7mO+fHBnLfWGix3PHYmioejMcOdIzAZS1XBu2s1MqNAw+bjDEzK5nDCeqME5IzoVerkHQfXtDHRSxwceby5UqVdEay2UqLYY/+ozXi/TcHpsbzQwLPqFKSH68CtXherW+mZvFOg5OXJS83IP2bjYg4dpzw7NIxmmlapjNIG27YRpMQJzLKUek6BSfkHSX3CIpZs1koAAAAASUVORK5CYII=',{});
regImg('x30',20,26,'iVBORw0KGgoAAAANSUhEUgAAABQAAAAaCAYAAAC3g3x9AAAFA0lEQVR42q2UbWxTdRTGf/elve3abnShbGxuDFgGEwYONAooUVgkGo1CzHxBSVASoiJiQjTGKGBMNPpFYzCQGD+gJiaIUYwv0RAUx4Ib28J42dhwDPbSbn2/bW/b23vv309qiGJEfb6e5/xyTp6cA/9Qq9sfEvxf2vTaLnHALAn/skf+O7Ttxd3imBDiXV2Ixpf6hadq6d9C5b8rls0IiXt27qQATF5I46lpoXrNJv418MnPDuOv9JHEIpqQMfRp5m16hqbHd4hrBja2bRUP33ELhm0jHAXffC9OmcAnuVDlqmufcP7qeXRMO9QXbdyyhBZQWL3CQ+q7DmJD8asC1asVbMvkcEpGPmmwqsFkSaPGwPtfM37JizugXjvQEYLyGdAXLlIRKaBNTdL5U5jaJYuIZqU/+R9+brtIx82rr+woMk4OGElT4Zfp/2YYj9eNbVnAH5m0PbtdvD5wTqx89R26kyHkJes3i+bNe4X/upuuSM4pOSSPG5hxAzQVt+bCNm381dUoqut339y7nmLdwma++jiFbBqot97SSPOGp3i79j7qkh0idvproscOSDgyUt7CG3ATap3BpRMWwrFQfF4kTxkAc7Z+KBKeBez96BzTUT+aIpAH4zr1jTZVHgPXvPXMXPQggNCCQRRbob4lyFSmxFjUxBf0YcoC2yziVjThrlxE1shz8mcd59wgzlQe2fJ4SKPw/NbrkJRJ7NEMLm8T8ckknlqNWU0aQ50p0kUZX00VVizHVFcX0owV+EOziXSfp5TUsUWWfCaMqg9cZnLEYgoFb3aCtHATatyIcALMalGZ6DTIhQ08fh+xRITCibe4875FJDNeoic7qaibj11KU0iPEx89ihI5N7w7UruC/vE67mhSmB6dIpO1cJULZt68nIs/xbASBrnpi3hFP5vefJR1T29Bsmx+/vw4gYpK7ESCxPiPGOEeSYacRMd+qjSDY8Mas29fiuZR0buHOXXwDFY0jxoNM977LRt3tFN7+2oikTx98nLKKrLkB0cpUUSf6P7j9M58eVDKTpwhM5olHs1S3toEdgARLaJaNtn4GA3LGjhf0YJjlCiZMmqFwFMSKLaGlU9RTI5IV9zyidc24JKGiXaOgC+EMnMW2lQChTy56WHq2lZwatBm4KzBDx+c5dLhDlSzilJhksnTn1N//WLhCVQKqTzYIG7c2E7S8aHrMwnMXol+qpNCrBuR8aLNvQFRHKHmiW0Q8KOcHmK6J4XjOo1xtAuncSmtj93Nmg03sH/HLmRZklj74i6WbXmF0i82xtgp3MYQ+z59gdve2EAh1kVlXQNyWZAKDPT+CFqtQ66nC2d2DXMfuJ+5S5s49F4PualJ1FTionTo5b3CX7MORTiUeocQIZOx+nkEjIUEWx3y4QsEU1lyg+exdIPEyI+oRYlA8zLsWJoje/Yx0fMJ2WS3pACE+77fY0bt3QQ8+JpuRh/uo+94hFyqkoJuEun8guyFcQrhDPrlAVo3r0Kt0oj2jWGmckT6D5BN9EgAym+hCDO9u37lNoK3rqJ6cTO5sTSps+P4XGna33wcHRN94CyLt23EKgVI9WZwSSpmYQJLyVOID+8BuOKxhRbcJVxaNaGWe5HKy4gd+Y6F69fiXbic0Y5LSLEhPIvnMH7oMFa6h6KdJj3dewVD+qtf6Jb9wnQKqG4VxVtPWWABLlnBFibJsW7cfodCNv6Xvb8CqixfI2kk188AAAAASUVORK5CYII=',{});
regImg('x31',21,30,'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',{});
regImg('x32',17,29,'iVBORw0KGgoAAAANSUhEUgAAABEAAAAdCAYAAABMr4eBAAAErUlEQVR42pWVS2xUZRTHf9+98+q82mk7nZZpO6W2tbQ8SsujAUR5CYkSJDFEdKcSjW6NOxNZmRhNTFj7iGFnYgwB4gIQUKKIuCgi1mlL25l2ptPpvJ/3zr2fC6W1EkDO8jsnv5zvf17wCHt6/2H5qBjLw5wffPaFtAsvc3fn5cTkTfGgOOVBjvc+OSV7Nm1iYT7C1u172X/wJflYkI8//Vz2DW0mFZunweMhny8QbO9hz4Fj8n9CFFkxFXxeJ76mRnTTxOV1UOe2I7CgoMhHamJRbIyPT6LYXbx+/DCFmQTXS0Uy4QzppRQm5qOFddV70ApVIhfPc1EanD/7PYnoAuVCicYWH4piwTRrD4dIYUErl6imw7zz9mUagiEUVXJkg4NLsxoWiwNNKzxQEwmQS8WEUBQMIahzO6npFdLpNP29zeiVMppW+KfUQt4HGd6+a5lmVVUM4UStc7Cmq5e2YJDvbseoGSvfCAS6Vmeyccs2uX3P/uXHfLHKay/0s3nEy/VfrqFl0zTZHGg1bTmmOdC+GhJoD2IY1pVMbAqXLoRJTKRwOp20+OpIpHNIVpq2ppuAVS5DRnbsolrMAdDR0y/1skG6WqZYUfA1t+GxSbrbXdjsbgJtIQng9nhxOj3LmcjugX7MmsSuNsito4fQcyUy2TzxXA2HVWFibonFRAG3TeCxtwLIeo+PxubA35BtT+0mGcsQi8zz0Tdfse/lIxi5OHabxN1YT9uaICnDwejOdsxylNbRrbz64SkKWgF/SycWq0Mqof5+FGGhUMjR8/x+YobOYmSeULOfdCaFzVFHg89Hd6OTYGs9W04cY2DfCPGZGXp6h2hrW4s4dOy4HBk9SHRqgnQqhz/gR61p3Dh3llg+z7anDxKL3EXGp8kZKg1rn8Co1cgsLLJuwzC3x35AUVWVXCpDOasR6uwkNjmLIe10bd+GIUxUVcHpchPOVGhZux5ZrOGxeekI9VJIZ1EUgSKEwGq1cuvGT1z69jxNnX6qepk7v45htdoxTJ3m5lZ89S5y6QSVaonfbv1IZHqcJn8AEFjSiRRXp89w54+rApC1ks6+o0cwajVUoWKYAqSO1eLE6/GTTM6RzS+QTM3g9dajWixYrl04J+7NDSDic1Ny/u4MVoeNeGqBarlKrVLE7nDgcDpYSsbQ9ZIAGBu7Ig2pi3uzs9yKO44+hykMWjo7wRREoxOkkou4nG60SpFAcGVmDKmL+zbb+oEtcmiwi9lIjOHhId584wTh32/i9wdweepJprOs617DQN/wqu0mAFragvLA0RcpLuUwYmHqGpxYPO0sLBaYDE+wrm+QyfA4vR2CcqFKNAVTs3epGVlxDyJfeetdRp99hkopz/zkDNGxMaqZOZbSSaYiZYS0Ua3msFklsWSO/r4hhIRIdJxCOSFU4GSlVHq/mK+glST+QCvdIxtJ6wo/X76GYQoqlQoVTUMKF4PrhlAUlVIpRyobxzS1k6sOUveTG2XP4BCtoW727d3JlTNfc/rL0zQ1dOBx+7A7HOhamanp22h6VvDfqvzbQr0DcnTnbmKzk0z9GadzTR/xxDRzsQmq1azgMU0qql26Xf6H3uO/AD8NDambkrp6AAAAAElFTkSuQmCC',{});
regImg('x33',12,13,'iVBORw0KGgoAAAANSUhEUgAAAAwAAAANCAYAAACdKY9CAAAB0klEQVR42m2SS1OScRSHH5DgBZRrJL4vgXIRdGIgnKxxNJrJaWNSa3fNuG3VF2mmtn0FV9W6XQscL41OeSlCiIsgiIjcpH8LRhtnOKsz5zy/M+cGAywe8IsFv18MyqmvnJeRe9eAT7IyabDeAB+4PQJAcxWYs41xHhoSo+EoisFGIVsh8Xhc7Pz4zpuoh43jY5JH6f+Cj+U2y09fYAs62SvkyWXLBH0TeGUne4cHfNhKqm60NP9kHvUdE79/Zgh8+cxKM0P97JTa3xbJlmDKHRbXAtkxKRrVBjsHGTRb66y+m2HluQpvuUrxvElAcdHu9gurEpGosOi9lESHrlFHYkTN68VTmmkVq596GO+H2dz+hqfXQegaqAAehuIiphOYh5ts6nwst7LsXhg5lEzYzD0S7RzvUx2+pjdUaoD9/C/SutuU9G509R5rQwr7Bj2WES2powJrNYk/7Vb/DrPj02JUGqZcK9Nt3yJXydOqlpDtZtZ3t7lrGqPY1uAwmJly+oXaanewEI/xzKLl5KSAJBlxaAzkUkX0WiNn9QqLDolus8TFZac/+YziFnOe/tpk+4RYCs2Kt0uvhOIICjQm8cgVFBHZNfBVAJDNThFTpgcC/wBura7doU9NMQAAAABJRU5ErkJggg==',{});
regImg('x37',8,15,'iVBORw0KGgoAAAANSUhEUgAAAAgAAAAPCAYAAADZCo4zAAABmElEQVR42i3BzWoTURiA4fc7c2ISlPx0TKYo6W+kkS6qggsR3QQjsaiguPAWvAPvqpsKFtxUilA3KgUpKQmZDobWdNKY1mSSmXPc+DzCf/cfNW1lpYKIxW/5fN37KADana/aau0eN5ZXefpyEy3Ch61trMraTusHOpef42G9wWx8yu7OFrMo4rpbYrn6nGHYRxeKeYaDgE97B2TdEmY8YTpo86TxgGu5DNJ8/dYeHfo8fvee426PaDLlrOMTdXZZrS6hMAnFm0t4C2Uu/YDxrx65xUXmvApKQCdRhFPII55HfmMdoxS6XCbs7BPHI7QVwcQx2hGylTIxkBhFEkfYFGgsKKWQOCHu9ZkBFF2U0ghTlFhLklgSgYvfp0zCMzKZNIkRjEmQxos3VlyP8/waOhHEcbgcDSmNu3DRRwXBCc36M+b//ET7X3Dan1kwPs3mJsfBCXo0COm0uty+dYe1VzUcR2gdHNFtB4zCAVJwPbt+t046c5WUToGAtYZp9Jdv+zsIwJV02q7UNii4ZTBwHp7SPvzONIrkH9jBryBpz+QuAAAAAElFTkSuQmCC',{});
regImg('x38',12,15,'iVBORw0KGgoAAAANSUhEUgAAAAwAAAAPCAYAAADQ4S5JAAACIElEQVR42mWSS08TYRSGn28ubem02lJBRNpqQyEiqAsNRgMxarzFsPHumo3/whj/gRvd6MKVRuNCE43GtWKMxJKCTQGpWMQCYpl22k5n+rkwNKBndZLzPsnJeY7gnxocPi2j3XGk1Jj9kmH83Vuxca6tN34jJI+PXKcrnmTg4D6qVZtwZDexRL98+eQ+ds0Um4CRS6MMnT3Fw7t3WJ7PMjM1Qbg1xJkL16hYVV4/uweAsg7E4lHKKwWSh0+QESHqySMsbtvL40dP0TT1/5WEW+fTxBRbjp0nmqiwmMriCYfQnVWEdDcDHiUkp2dzZH/kcY0EfqeKOZPD1xlFriyDruNVDFlrlIWyI9YtL98axduyBcssItZszIUi3rbt1Osu5lKBrmiSi7dv0LmrR2qRgTBBX4SiUkNXNFiZREfgFKH8dYqWho3m8eFTPQT3GGjKWgBrHipWhaGrV7DbNYI1m7EHLzhqpOg/N8qrbJ2un348pa2oujBuzmbStHd20NvXj2O6hL2txNrb2OnrIBTfT2puhrE3z7FLNbTvubQA5ODwSTTHQTUtSlJwwKORiyXJew0CqsL83GcAsX5WgapJ22lQLZcQZYv3ArwqRGo2qq7/zWz0kE1PYFkNEvE2kn29SClJp9KMf5zE/L3Q9NA0vVrIs/RtGs0fBN2P8AQwghHWlhf4Vcg3gU2fqKo+2TNwCH/AjxACq2SRSX3AdavN3B/wl898eRTsVgAAAABJRU5ErkJggg==',{});
regImg('x39',11,14,'iVBORw0KGgoAAAANSUhEUgAAAAsAAAAOCAYAAAD5YeaVAAAB3klEQVR42m2SwU4TYQCEv79sbReWdmtpKLSCShpjG1KNGCPGWDUcuFgTw8WTMfHkO/g+evMF1ESTxhiiQUKNUCi0C2nLsnS7UHbb7u/BaBSd00y+28zAKaUncvLB0jOZiM3I00z5M9xeKMrCncdoo2eJhM7z/u1LuW2siF/8t3n6/IVMp3O02yZu20b2FQ4aDnutNT6UXgmAAED+2n1ZWHhE9dt3KtUNymqErZAgrAuCIsaNuaIEEFfm7sn5Ww85ow7TbB/iT+dYLTUYKC4qFa6PJFgrl9l3Vgl09tu43T6ucoKpJZnKZ8g+uYnrBPGjeVqORUTTEX2VQKW6LMxdAzc4ih2a4CChkconiOcvILwhrFiM5t4WmzufCej6uJRKj7rVobmyyYnjM5XSUQY97O06XakiGeLEtYW4enlR5jMFVrwNQjNFjmyPeC6N8a6MOh5EbS2TsAaUjY8ottNkx9jkYmaaSr+GU3M5rjXQs2nC2gDvUxNTATU48rPneDQj53NFhpIaVvoSx12B53Xoll4zGU5hHKyzXn8jFABdi2JYdWJHcfyWSbNnETMlutSxvUN8v/33goCcHJsle+4uE2NJNox1aq0v7Jpf8f2+4H8aDk/K2ZlFORwa++dIPwCaqsy2z+JzIgAAAABJRU5ErkJggg==',{});
regImg('x4',24,25,'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',{});
regImg('x40',16,14,'iVBORw0KGgoAAAANSUhEUgAAABAAAAAOCAYAAAAmL5yKAAACYElEQVR42oWT3UuTcRiGr3fu023OffhufsyZMz/mJpaoURiKCFZ0KFEknXXSnxH9DdFZ0EEIRYVBHaQWSFEJamnBZrq2ITq36dTptlffXweRUio9hzc3Fw/Pc98SJ4x8yiNqmjxsp3OEP0cApON8x4rtA2fE8J1+mgN1rKaS3L83yrepBLnVtSN+zb+CwyuLKzcv0N3ZxP5ennyhwOD1HjrbOzAYzeK/gGZnLXqphIVojJVkhsTPDYqqQqCoQn73wOcutYpjAb5Kmdj0KvOLayQSm6gqlCgqksdIsCoIwO1Qn3CabEc3cJrtwlLhQKto2Rnb5uv7OEpCYH6toomY8PkaMWMWsr4aodEfAtrkOgGQzq1LC5NxdGMKuZk87rielfEMiel11KyG3WgKg17HbDKN3+o+BDQ5vQC02fzCpRhJ5xSMOQNbM1C5bCapFEgVcmzuFagxBBmNv5L8Fjvn5XahuRUYEKWqhcGGoBhuuYzTUkZPg5fwxhptVS4+pKO4jKUoqsTFyhYk7RahCr+oNvlJ7e6geRsLE7TXM1RzlQdz43gcOqaWo8xmvuMtN+Gy6JnLLOE0a3j8Y4J6q5NL3l5GIpOEt8KSBKDDIFRK2GdHGqjuEiGHnyeRabo89Qw1dPNo4SWL2TXOuUPMZZb4lJw7CKEGQKEgBWQnADmRJV/UklUzvIi949qbu/itfnq9rTxf/Eig7DQ3Gvtpc/0+vPbPCz3GCnrPtgt1z8DT8ATZYlIC6PO1ilpbOSOReTLFpPRw4RkdroD4kooerUGV1f5XVH3lshhu6RM2rUOcVLpfF8/rLA1KQocAAAAASUVORK5CYII=',{});
regImg('x42',13,13,'iVBORw0KGgoAAAANSUhEUgAAAA0AAAANCAYAAABy6+R8AAABjklEQVR42o2SzWsTYRCHn/fNuxvz4S4xjSZpmkOhaYtSkCi1pNiDF4Va8eBJEMSzB/8r6V2sFymlgiANKdooKomoDTb9MO02anbbjBcRo8H2d56HmWdm4Fey+SkZGpkRjhEDcCZ/QSYuXSd60sUyIam9fqr+B2kAkQO0sbBMjHMX5xgpzsmRUPNTRVWX5ukGP3DcFHYoxOnksNy9c1+K5y9L3/EAPnx+oezFqCjXIb25TOnKLW7cu8nQQh7xtJTfL6p/IIBmYw3HH2a2VCKd8nn0eBX5ssthV/fvFLOSErYSpBMpmokxCk4Ls7bCy2qdt/VXvU62SkpUZ6QdbKsN753qBC12vls8rLuovX1iZoOJ6VkKZ6/+djO+bCv/D9XVN89UWNtCV7PU3mJ0fIpcdpKB7D5tb1PWP64o02+lW62v1BoVlTk1JiqS4/CgQ9g+weTMbcrP46L/BmImJbVGRQHs7jWplp8QBB3icRfP28EPvvUCET3QcxPHHhSAzGBRStceHP1iiUhOsm6hb+FPcoKQKyfRUZ4AAAAASUVORK5CYII=',{});
regImg('x43',10,11,'iVBORw0KGgoAAAANSUhEUgAAAAoAAAALCAYAAABGbhwYAAABb0lEQVR42l2QUUvbUABGz01vjallJm5VM62IDizSDYaKxQdhm2P/QXwVX9zv8NfsdYooMgRBRbapIIhgW5uqrW2kMTWxxuuDKGPn6cAH5+ET/MOb7pzKfM7RvPCoHB5SutoW/E92ZlH93HfUr2qovi/n1cLSpprIzavnXXuWiblZjoIHfqz9pVkpERmS9NiXl5AEyEzNKu+4yOr2Lv5JgZZ/y6vBPuLSxO7/pM5LG0IDGH+Xwt//TXnnANcp4nt1nD8HhGdHfJ3OPhU7jC7V15Pi4weTq8skcX0UoWmEzRvakjqVYg09nlTyfWacZhCR6ErSqxu0J3RiAly3HS8IuK7eMDyQRbujhZkysW2LUIG+t0LaaOC3IszOBFbaos0wkLJwyn0hz47U6H1rIaZmcFyPWKPOyXUC3a0iy3kEwDd7SA1aFrGhNI1Iou4iOqQi5pQ4rtVYPz8VL89PvrbVSGeciAcQgjCA8q1gq14UAI/pXpDonELLlwAAAABJRU5ErkJggg==',{});
regImg('x44',10,11,'iVBORw0KGgoAAAANSUhEUgAAAAoAAAALCAYAAABGbhwYAAABf0lEQVR42kWQS08TARRGz+1D+rAMOIVYSkeqWERsfQUFiUYjC+PC/+DKnT/GnQuDgQUxNXGjOxuXKqkNIZAY46IKmJEyQ+hrpobhusDHWX/J+XKEPxQL17Tb9LEKOfyOz/tPFZnIF1WCKJ+/1yQCcGtuXs9lStjbDoNmnE6yyXDSUjOWxm15AEQA4kaCn/s/WP+6QfF4idv3bvDr0GOt+oW6syEAIQDHblG8coGCNcmpCYvycpnhjMXl6UvcnJ3Xf8Nq7Z0YgynGJrOUn72k5wUcS8Swt3exxs78VwMsL5V59PghrWabA0/Yqm8ylEszOm4hEtcQwNXpOQ0CWHjygvxUnvPXz9Jr9Bgxs7hbLqpypOYgxP07D9BemOdPF6l+WCGIHeLu7bK6WgO6R3kaTge7vUmucBL5FlCpvEXUp4Mnf69FZvoNnS1ZeP1RjIER3J09+vqU05LEUUPrvi0Acjd/UbNmm4+NBClzgHAQJbnfJTGV4fWbV5IOn9DRVIzfrr+NpsRDWhoAAAAASUVORK5CYII=',{});
regImg('x45',12,13,'iVBORw0KGgoAAAANSUhEUgAAAAwAAAANCAYAAACdKY9CAAABj0lEQVR42o2SX0uTcRiGr9+0+b5jk6V752ZNJ/PPJrx1kENNcq1lBiKe9AnMs75EJ0Zn4hcQPPJgIBiEhFBk0DZBLENxSGPxlrH+GGZOM/TpJF8YQnof3xfXzcMD/xLr6BPOkWqArusjEokNIapGNjaeq/8BjlCoV1rCAxztbNHWkaDl8i3b5PdGxKsFK8wOy8qoN9kF/uz+xlvrpNVMAmC298j4xENuJ+9SpzfZkANgszCnCsU8+4ceDr5mqMYlDYZBlVIEjABN/mbbULHX770i4VYfo6P3eDq7iPVhi0gwyvJ6luK3VWUbTnLR2CWViBM2o9y808329y/ML6Xtsn0lgP7+pDx49JjSvuLFk3nyL5+xs/cR54U6yvzgFDCYGqbe8YvC2ibruffkS2XMqwlqavvY/rwiK2tpVQHMTM1iFT/xs1Qmt5xBtAMaQ8N43Dq+NpPjKuTtalrZ29x6UAx3ALem8856rQAuNfRIdzyFU/Pg0jVeLUyf/QrhxgEZuz8p16I3zvU6AAT9nXb5L6UNgbMnyoUmAAAAAElFTkSuQmCC',{});
regImg('x46',29,12,'iVBORw0KGgoAAAANSUhEUgAAAB0AAAAMCAYAAACeGbYxAAAD+klEQVR42k2U3W4UZQCGn29mvpnZ2e1u2m5pt9DS8tMS/mNJkErsgcEbMCZegvECuAmj1+KJmGAkEQViDJCCUCxtqbTd7XZmd2f/Z3Zmvs8DBX0P37w/Z4/gH+njC6fxPJfrn3zK0VFAv99HCXBdl7yXpx2GaK3/TSsQAoEgVQqhNaZholEkWcZkeYryWI77P97Fccfwa/s0/UMAASDmT5/T5ekZBsMBX92+zedffMb65ls0AtO2yeU84n6PdDQi0xoN2I6N67oIKdFJSjqMSNME25bEWjPKMlbPzvHt19+w8WqHY9Mz/HznO/Z3d+i2Q2EVJ8a5emOV2t42wpZsHAQcHB4x6veYrczgV6sgwDRNBJAozcHuLipNkKaFADSayskFgloVv37E9MI8BddmmGQsnF5k5caHhM1DDGnz+vm6tpRSuJ5NUG/gH7V4vbvPYBjx2737dBsB19bWePzgAWHDJ18oIAyL6s4bsizDMC3QijiOuXpzlb3XmwSHPieXlrC+nKR65JN2+0Rxgl+rsbh0FiHA0EpTfVtj7802jjTxpEkSD1m+eJ4kHvHLne/xcg7nLlwABftbW1iWRJoG0jTIFwqUj5XZfvYcQ5g4jk3U6+CaJsVCgVYQUD+o0Wq2OPjrLdKysLIsI01ikijCyuewLIkF7G9vkamMiakpWkGTo70qGs1YqcT07CzlSoVWEPDm5QajZITj5li9dQu/VmWyUqFcHqfouTQbDQzboVmvYzs5kizG6ocdtv7YwBsr8urxUz5Y/RgBKCVIM400BNfW1ihXZomHER2/TlCvc3iwT7fVZHyqzNzyEjNzc5y5dIXLUmIJxZjnsfniBV4+T87ziAZDRCbISBEnTi5pw5TMLs4z6HQ4d/k8Z66sEDRblEolsjgmSkaAQAhBr91lNOyRHyuSaUiSGMt2MCyJlBKVjFhYmOPRT/fwD+ocXzyFRvHw7g/kc0WSLMIyDBPQSMvi6vWPeLn+lM2NLSqVCnruBDs7u/S7HbqdFsN+DyltDMsizVJyjksUDciSjPJ0hSRNCGpVlFLkCkXGiiU2nv7OseNzWNImSUaEYQuRL5T0RHmWQrHExHSFJInZ23mFaUp0luG4HqeWL7L95zMuXVvBcV2GwyGmMNCAVyjw5OGveN44hjRYuXmdTqtNK2iggTDw6YUDOi2ffq9N2PKF1e+1hdZKqywjHaUM+m167RBhGKCh12ljaJN4OMCvNZDSwck5AETDiEa1RRKl7Ow9Z3p2nvVHT4jjCK0UUjoE9RoqUfS7IWEYiPdYeodCwzDAANR/pmFZKKWQ0qRQmCQdpVjSQqPRSqE1pGnMoN8BQKn/ld9tGAZKqfdffwNdkAa1QOjvNQAAAABJRU5ErkJggg==',{});
regImg('x5',22,23,'iVBORw0KGgoAAAANSUhEUgAAABYAAAAXCAYAAAAP6L+eAAAFeklEQVR42oWVS2hc5xXHf9/97p1778wdzYxGI41eliXZsvWILbsyaezYSZzGEOwGUjA0lC5Cuu2iC2Poyu2iNFAImHZpaPrAJC10Uwfa0NSua6VUSW2ixNHLip4jaUavGc3zvr4uSh2cJvVZHc7h/Phz4Jw/PCa6Lz+nLuW+o564dV4BXFU/UX2nz6jHzYmvahy1M+r8vnb0jjhFo0ryjW5WbnmMrA4RvnuPsubx8eo6v1ue+VKG/LLi1+ysun5mjFbPYH6pwnJS8b7m8sF4g9rbd3nlYBIrHuW5bDsdevzK7ULuR19kaF8sDDk96urREW4v5vm9qDEw2obXUSeTMnAcg/zF/Vza2EazAtrtgLNajIvNA+r/gruiLeq19i4WPY+nXjjI5d427uTXuW2kGNxnMHxUY93UsTMGcwHcWKkyWy7zzeYsh2Kd6ivBJ6IZRjMWJ4bbmLi9wDPLs/zxQgIj7hANJAeHQav5BOkIarFMV7dDrdvh1l6JbyUy9JkJ9T/gtEyqUaeJxTDkjZtzzPc28+rYQU5OV8kmGpSlIOoo+kZ8/iVcakGFFhRZ2+J7z/dS1GxSMvVQpP7f5Hi0mVZbZ7izif6hFsZzW1SIcYoEIi24nw9gTdDbr+jKtnBtdYELssHbn27w3Wabcy06nxTtR1eR0tLqpJ3k230J5MoOb/1lipoRkql6TMwWMB1Bu6NxuMNkIQeHewVDr7Tym5UtfnbuAL+aLqJqDZ5NJtjvdKmH4AutWcYcm5vrRa6XPF594QjluOJ6ZJXci5K1LY+LZ0zuPSgz2GLzp/E6A4fT/M0KkTGbA0dbmS5BiyZ52mn9j2LDbFLnUikmawqjq40fjHVy6a2/86ZRoedCB22D7Rw6HnJ3eZd9GYNnT3q0JHTG7+8SHouTL/m8dqiZmKaQysdWOsiY0scSafxtj56OJoZEwLt3VhjpamW7rjg7nOQX13I0pXXSmxB1LX56bZeIqXH+VDP2/QLmfJ5izSOUkhkXCENMYaLVPcmUjNAU1Hlvco3h00NcffkUlzdbeefHc/QfcVj8WMMMo8zv+EzdczjQ2kK00WDh19tM5at4viJQih7LZ4cInfFOpCeiV3QZww98zg62kSzs8v7ELC89eZjUbIMbOwV6TiZZWwwxYooTT2ok01UaV3bIFk0qlkEQhtyreZTqPqXsKBU9QDM1yRNSoKRFpVDmwwfbrDZg/IMpXhzaz/M3Gsz8o8joMZvBHh3/ToXR10OeydtM1ANS6Fzf2GPAkbgBRO0Ehm6gR6XJbBjDKVWZ9KrM+gIzBDO3R1iZ5OmmFP+UHt7ODryp+H7ZIYrH6xubHGuKkKvVqQQ6DhKBht9w8ap1tLpfwaIBIkTKgH1JmxVfp2DE+HDbJdHdzNJv1/nGz3V+aLVRqVb55dwWTsTAQXK3XKczorFYdpnR2wk02Nj5DH2luiqmZUwNmBlu1uCECBixNSaKIRJJ/2drDMgoazsN/rC0QFWT7ApJQkaZcQOEUKS1kL8WBZ19/ZTy06yX54UOMFvZwpIZLGFTdqs4GgQKPKVYqfmctk3eydfYH42xUS8zZEVYC0yKvkdCFyy5Ia7U2dh6wN31ic9Pei/cEg9qa9TrLn8uhRRUiC8gF2rkhIkrIxyJR5ivbuMrncm6xbrboKp0PBXBJYYGrG9N02W1P2pNT8WPqX4Z5ZNgl4YI6TXiZCMmS4Eko3lUfUWTCCAMCbUIS55NQIiPhiYCCt4G3TgIUee9vY/EQ/CA1aUSusSTOstug5JX5+uxJCpw2BISTRhIoWELQc2vsumVOG5l6QgsPg02+UjPUdwriMea6efvz1KapqFrOiAJEbj+tgDokBnVrkUp4jLnrT3C+jf6fVP55SpkigAAAABJRU5ErkJggg==',{});
regImg('x6',18,40,'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',{});
regImg('x7',19,40,'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',{});
regImg('x8',22,35,'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',{});
regImg('x9',19,24,'iVBORw0KGgoAAAANSUhEUgAAABMAAAAYCAYAAAAYl8YPAAAEK0lEQVR42q2VSWxVdRTGf/87vHl+dHhlqGUukBYIJGgxpnFicIGgRjTGmKiYqCtMXMhCo2zQxERcGDdqFEENbowkilpigGKphVLmvlKkLZZOad9433v33uMCg4GIqdGzPckv3znn+3JgGrV+Vlz4P6oxlZTed16Xt/zB/wZsnFMnPXveFzn8s3QbSdkR/meF2u0aCxvqZe8r21jW1Ewxm+eELaxSAZYauvwr2F1LFsu+bU/RPHsOds7G61Qo43Da0XktUM1azSPThm1tWcWK1rspBSMY5RJSGmeFx0YEEsrLxlh4+sp0Xad0pIMKQCTMWLqf4YrBGhM0r0Jsd/owZWpIezdXXtqOG9Bg9jxMXdGn6+SmHFxHpg9zlcI3K475+wCZF7ehdxziN81DV8nB0hXubQ5g/K0yjweUC14fpb4hzKo6BlyTsi0kghC8hfbh849L14U+tIfW3SufzbtDdhimLAlfN6buOrhZm0xZEa714kZihLw6iYCObSscUTdA761fLc893MpUTz/anSuW8+S7O3k2HmNvOM5MkELRQtk2YilyGQMnkyfp1agO6EyWBffPlX2QSsjLzzxK5w8XGJgYQ7POdcPKJi5tfYS2q2Ns0EwsR1AxHyICGBgXz+PkKoyPlsjaFfKOw+JYSB5rnEt53kqixSLzAe3YT530PvgEq+pTjMRm0KKiTPx4hN7OM1i2ge3RGalJcl4cxkIuVxpiXCu7tMSriBoGvfu/J3T8GIlQCO1gZlK9eaaH0a+/Y9PG1Uwqh+TAOB8dOkFVKED36UlGxcuvusPbu99gU1WAbytF1igTbWaUtj1fsuv4IIeVc/2a7UE/J3vSRJWGnkpQ29RE1HT4/JuDrPPHGei6yObNrSTrakifH6YZwYj4ONef4Zdrgj/k0JEtKg0gnS+qrqiOc+4yXYPDPL3rVbZsWMdRxyKnFEOZAs31M3H1IL5QhJaG2UwOjfPpmXE0x6Lbyd7ss50DV5XrrxZPMsJwWxv5vjGW6kEO5i2ii2pZ6+TJnG1nxCeIa3JqIsNldBYYDh3FiropAdsbGuQoYTo1D5ndn8DgKdbPncVZt0Cv7WIqm/S+L/i4b5S+/hFcM8ICDdKaQdAfkRuwZf6EzI+kaKiJUKP7OTk0RSGXp6SZNGoG9yMwNMKpMZhyLI5qBqbtMOFWuCeW4IWq1F9jlkWxv+wQD/hYGY3wlWuwfLRCMWvRYQSozpbJn+znyMUJRrwx6g2hQzQMfxR/MMhU2UJX+vVc+D1hme/VsIxatqTqUHaBgXyRRXUBToxbPBAzab+a5VLBpTUeZNA0uOQIuXyWweIEBadCpjCp1K0hN82ohA0fNd4Q982IkFcmwyWbdLZCS8KD7locGB0h5pZJW3nKrq2m8U40SQVmyMJ4naCHJWWGpCmUFOC2P+APNwXWPuBgDmMAAAAASUVORK5CYII=',{});
const SHIPIMG=["s267", "s466", "s1", "s3", "s234", "s28", "s30", "s447", "s2", "s4", "s39", "s219", "s389", "s400", "s38", "s76", "s166", "s167", "s168", "s169", "s432", "s277", "s97", "s419", "s6", "s83", "s431", "s437", "s448"],SHIPR=["fighter", "fighter", "fighter", "fighter", "fighter", "fighter", "alien", "alien", "alien", "alien", "alien", "alien", "saucer", "saucer", "medium", "medium", "medium", "medium", "medium", "medium", "medium", "medium", "medium", "medium", "heavy", "heavy", "heavy", "heavy", "capital"],SHIPN=["RED STRIKE FIGHTER", "PRISM RACER", "WHITE SPARROW", "ROCKET SKIFF", "BLUE INTERCEPTOR", "AURIC SKIRMISHER", "SQUID BROODSHIP", "CARAPACE HUNTER", "BEETLE GUNBOAT", "EMERALD STINGER", "XENO DRONE", "HIVE MOTHERSHIP", "VIOLET SAUCER", "SHADOW SAUCER", "GREEN TRANSPORT", "TEAL PATROL BOAT", "DESERT HAULER", "AMBER FREIGHTER", "GREY CORVETTE", "VIOLET GUNSHIP", "LANCE GUNBOAT", "PURPLE FRIGATE", "LEAF CARGO BARGE", "BLUE SHUTTLE", "YELLOW MEGA-HAULER", "STEEL CRUISER", "BOX FREIGHTER", "BLACK DESTROYER", "DREADNOUGHT"],SHIPSPD2=[null, [40, 30], null, null, null, null, null, null, null, null, null, [6, 8], null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, [5, 5]];
const ROCKKEYS=["a1", "a2", "a3", "a4", "a8", "a9", "a10", "a11", "a154", "a159", "a166", "a167", "a168", "a169", "a188", "a189", "a194", "a437", "a441", "a442", "a443", "a445", "j75"],JUNKKEYS=["a5", "a6", "a7", "a12", "j57", "j214", "j215"],SATPICK=["sat70", "sat78", "sat70", "sat78", "sat70", "sat78", "x42", "x45", "x44"],XCAT={"cargo": {"w": 4, "glow": 0, "keys": ["x17", "x18", "x19", "x23", "x24", "x25", "x38", "x43"]}, "cryo": {"w": 1.2, "glow": 1, "keys": ["x1", "x46", "x20", "x28", "x37"]}, "monolith": {"w": 2, "glow": 1, "keys": ["x2", "x3", "x6", "x7", "x8", "x26", "x29", "x32"]}, "crystal": {"w": 1.6, "glow": 1, "keys": ["x9", "x10", "x30"]}, "bio": {"w": 2.2, "glow": 1, "keys": ["x4", "x5", "x21", "x22", "x27", "x31", "x40", "j25", "j82"]}, "mine": {"w": 1.4, "glow": 0, "keys": ["x11", "x12", "x13", "x14", "x33", "x39"]}};
const SPDROLE={"fighter": [26, 32], "alien": [14, 26], "saucer": [6, 10], "medium": [13, 20], "heavy": [9, 10], "capital": [5, 5]};
/* native colours (tc) or palette-snapped, optional hue shift; cached per sprite */
function imgCanvas(s,tc,hue){
  if(!s.ready)return null;
  if(!tc&&s._cp!==P.i){for(const k of [...s.v.keys()])if(k[0]==='p')s.v.delete(k);s._cp=P.i;}
  const key=(tc?'t':'p')+hue;
  let c=s.v.get(key);if(c)return c;
  if(tc&&!hue)c=s.cv;
  else{
    c=document.createElement('canvas');c.width=s.w;c.height=s.h;
    const g=c.getContext('2d');g.drawImage(s.cv,0,0);
    const id=g.getImageData(0,0,s.w,s.h),d=id.data;
    for(let i=0;i<d.length;i+=4){
      if(d[i+3]<128)continue;
      let r=d[i],gg=d[i+1],b=d[i+2];
      if(hue){const q=rgb2hsl(r,gg,b),n=hsl2rgb(q[0]+hue,q[1],q[2]);r=n[0];gg=n[1];b=n[2];}
      if(!tc){
        const kk=((r>>3)<<10)|((gg>>3)<<5)|(b>>3);let ci=P.cache[kk];
        if(ci===0){ci=nearest(r,gg,b)+1;P.cache[kk]=ci;}
        ci--;r=P.R[ci];gg=P.G[ci];b=P.B[ci];
      }
      d[i]=r;d[i+1]=gg;d[i+2]=b;
    }
    g.putImageData(id,0,0);
  }
  s.v.set(key,c);return c;
}
function blitImg(s,sc,tc,hue,px,py,ang,flipV){
  const c=imgCanvas(s,tc,hue||0);if(!c)return;
  lctx.save();lctx.translate(Math.round(px),Math.round(py));
  if(ang)lctx.rotate(ang);
  if(flipV)lctx.scale(1,-1);
  lctx.imageSmoothingEnabled=false;
  const w=s.w*sc,h=s.h*sc;
  lctx.drawImage(c,0,0,s.w,s.h,-(w>>1),-(h>>1),w,h);
  lctx.restore();
}
/* special rotation rules: rocks always spin, junk tumbles, monoliths stay mostly upright */
function rotSpecial(o){
  const r=mulberry(((o.rs|0)^0x7A11)>>>0),u=r(),sg=r()<.5?-1:1,a0=(r()-.5)*.7,A=.1+r()*.25,f=.25+r()*.5,ph=r()*TAU;
  if(o.rk)return{m:'spin',a0:r()*TAU,A:0,f:0,w:sg*(o.rk==='rock'?.03+r()*.22:.12+r()*.6),ph:0};
  switch(o.cat){
    case'monolith':return{m:u<.3?'still':u<.6?'tilt':'sway',a0:a0*.6,A:A*.6,f:f*.6,w:0,ph};
    case'bio':return{m:u<.15?'tilt':'sway',a0,A:A*1.4,f,w:0,ph};
    case'crystal':return u<.5?{m:'spin',a0,A:0,f:0,w:sg*(.05+r()*.1),ph:0}:{m:'sway',a0,A,f,w:0,ph};
    case'mine':return{m:'spin',a0,A:0,f:0,w:sg*(.2+r()*.6),ph:0};
    default:return{m:u<.2?'tilt':u<.7?'spin':'tumble',a0,A,f,w:sg*(u<.7?.06+r()*.2:.4+r()*.5),ph};
  }
}
function pickW(R,tbl){let tot=0;for(const k in tbl)tot+=tbl[k].w;let v=R()*tot;for(const k in tbl){v-=tbl[k].w;if(v<=0)return k;}return Object.keys(tbl)[0];}
function drawImgItem(o,sx,sy,s){
  const sp=IMGS[o.img];if(!sp||!sp.ready)return;
  const sc=Math.max(1,Math.round((o.scale||1)*S.sprSize*s)),fo=floatOff(o),px=sx+fo[0]*s,py=sy+fo[1]*s;
  if(o.glow){
    if(o._gp!==P.i){o._gc=CS(nearT(sp.glowC[0],sp.glowC[1],sp.glowC[2]));o._gp=P.i;}
    lctx.fillStyle=o._gc;lctx.globalAlpha=(.05+.04*Math.sin(wt*1.3+o.ph))*(S.glow*.75+.25)*1.6;
    circ(px,py,Math.max(sp.w,sp.h)*sc*.8);lctx.globalAlpha=1;
  }
  const ang=rotAng(o,false),hue=o.hue||0;
  if(S.crisp)spriteQ.push(()=>blitImg(sp,sc,true,hue,px,py,ang,false));
  else blitImg(sp,sc,false,hue,px,py,ang,false);
}
function drawField(o,sx,sy,s){
  const a=wt*o.w,ca=Math.cos(a),sa=Math.sin(a);
  for(const p of o.parts)drawImgItem(p,sx+(p.dx*ca-p.dy*sa)*s,sy+(p.dx*sa+p.dy*ca)*s,s);
}
/* satellites: the two supplied sprites (plus three small ones); falls back to the old pixel sat while loading */
function drawSat(o,sx,sy,s){
  if(!o.img){o.img=SATPICK[(o.rs>>>4)%SATPICK.length];o.scale=1;o.hitR=14;}
  const sp=IMGS[o.img];
  if(!sp||!sp.ready){drawSatLegacy(o,sx,sy,s);return;}
  drawImgItem(o,sx,sy,s);
}
/* ---- image ships ---- */
const IMGSHIP0=SPRITES.length;
SHIPKIND.alien=[];SHIPKIND.saucer=[];
SHIPIMG.forEach((k,i)=>{
  const r=SHIPR[i],si=IMGSHIP0+i;
  SHIPCLASS.push(SHIPN[i]);
  (SHIPKIND[r==='capital'?'heavy':r]||(SHIPKIND[r]=[])).push(si);
  SHIPSPD[si]=SHIPSPD2[i]||SPDROLE[r];
  SIPOOL.push(si,si);
});
const ESCORTS=[0,6,8,10,15].concat(SHIPKIND.fighter.filter(q=>q>=IMGSHIP0));
const ALIENSM=[SHIPIMG.indexOf('s30'),SHIPIMG.indexOf('s39')].map(i=>IMGSHIP0+i);
function shipLen(si){if(si>=IMGSHIP0){const s=IMGS[SHIPIMG[si-IMGSHIP0]];return Math.max(s.w,s.h);}return SPRITES[si][0].length;}
function drawImgShip(sh,sx,sy,sc){
  const sp=IMGS[SHIPIMG[sh.si-IMGSHIP0]];if(!sp||!sp.ready)return;
  let ang,flipV=false;
  if(sp.noRot)ang=Math.max(-.35,Math.min(.35,sh.vy/(Math.abs(sh.vx)+8)*.9));   /* saucers stay level and bank into turns */
  else{ang=sh.ang;flipV=Math.cos(ang)<0;}
  const tc=S.crisp,hue=sh.hue||0;
  const run=()=>{
    blitImg(sp,sc,tc,hue,sx,sy,ang,flipV);
    if(sp.noEng||!sp.eng.length)return;
    lctx.save();lctx.translate(Math.round(sx),Math.round(sy));
    if(ang)lctx.rotate(ang);
    if(flipV)lctx.scale(1,-1);
    lctx.fillStyle=tc?sp.fc:CS(sh.cols.E);
    for(const e of sp.eng){
      const len=sc*(1+Math.random()*2.5)*Math.max(1,sp.h/24),th=Math.max(sc,Math.min(e[2]*sc*.5,4*sc));
      lctx.globalAlpha=.4+Math.random()*.5;
      lctx.fillRect(Math.floor(e[0]*sc-len),Math.floor(e[1]*sc-th/2),len,th);
    }
    lctx.restore();lctx.globalAlpha=1;
  };
  if(tc)spriteQ.push(run);else run();
}
function drawUfoImg(u,sx,sy){
  const sp=IMGS[u.img];if(!sp||!sp.ready)return;
  const sc=Math.max(1,Math.round(zoom*S.sprSize)),y=sy+Math.sin(wt*1.4+u.ph)*1.5,hh=sp.h*sc/2;
  if(Math.sin(rt*.7+u.ph)>.2){
    const fl=.5+.5*Math.sin(rt*17);
    lctx.fillStyle=CS(nearT(255,225,110));lctx.globalAlpha=.14+.2*fl;
    lctx.beginPath();lctx.moveTo(sx-1.5*sc,y+hh);lctx.lineTo(sx+1.5*sc,y+hh);
    lctx.lineTo(sx+6*sc,y+hh+16*sc);lctx.lineTo(sx-6*sc,y+hh+16*sc);lctx.closePath();lctx.fill();
    lctx.globalAlpha=.7*fl;lctx.fillRect((sx-1)|0,(y+hh+14*sc-((rt*8)%(12*sc)))|0,2,2);lctx.globalAlpha=1;
  }
  const ang=Math.sin(wt*.9+u.ph)*.08;
  if(S.crisp)spriteQ.push(()=>blitImg(sp,sc,true,0,sx,y,ang,false));
  else blitImg(sp,sc,false,0,sx,y,ang,false);
}

const ENG=SPRITES.map(sp=>{const a=[];sp.forEach((row,r)=>{for(let c=0;c<row.length;c++)if(row[c]==='E')a.push([c,r]);});return a;});
/* per-object motion: some sit still, some tilt, sway, slowly circle the clock, a few tumble */
function rotInfo(o,isSign){
  if(o._ri)return o._ri;
  if(o.rk||o.cat)return o._ri=rotSpecial(o);
  const r=mulberry(((o.rs|0)^0x7A11)>>>0),u=r(),sg=r()<.5?-1:1;
  let m,a0=(r()-.5)*.8,w;
  const A=.12+r()*.28,f=.25+r()*.6;
  if(isSign){m=u<.15?'still':u<.55?'tilt':u<.8?'sway':'spin';a0=(r()-.5)*.6;w=sg*(.03+r()*.09);}
  else{m=u<.22?'still':u<.47?'tilt':u<.72?'sway':u<.92?'spin':'tumble';w=sg*(m==='tumble'?.5+r()*.6:.05+r()*.22);}
  return o._ri={m,a0,A,f,w,ph:r()*TAU};
}
function rotAng(o,isSign){
  const q=rotInfo(o,isSign);
  if(q.m==='still')return 0;
  if(q.m==='tilt')return q.a0;
  if(q.m==='sway')return q.a0*.4+Math.sin(wt*q.f+q.ph)*q.A;
  return q.a0+wt*q.w;
}
/* gentle bounded drift so nothing is nailed to the background */
function floatOff(o){
  if(!o._fo){const r=mulberry(((o.rs|0)^0x3C3C)>>>0);
    o._fo={ax:3+r()*9,ay:2+r()*7,fx:.12+r()*.25,fy:.1+r()*.22,px:r()*TAU,py:r()*TAU};}
  const q=o._fo;return[Math.sin(wt*q.fx+q.px)*q.ax,Math.cos(wt*q.fy+q.py)*q.ay];
}
/* pre-rendered sprite at pixel scale sc (outline included) so it can be rotated with hard pixel edges */
function sprCanvas(o,fi,sc,tc){
  if(o._cp!==P.i){o._sc=null;o._cp=P.i;}
  const key=fi+'|'+sc+'|'+(tc?1:0);
  const m=o._sc||(o._sc=new Map());
  let c=m.get(key);if(c)return c;
  if(m.size>12)m.clear();
  const spr=o.frames[fi],w=spr[0].length,h=spr.length;
  c=document.createElement('canvas');c.width=w*sc+2;c.height=h*sc+2;
  const g=c.getContext('2d');
  if(tc&&o.ol){
    g.fillStyle='#0b0908';
    for(let r=0;r<h;r++)for(let q=0;q<spr[r].length;q++)if(spr[r][q]!=='.')g.fillRect(q*sc,r*sc,sc+2,sc+2);
  }
  for(let r=0;r<h;r++){
    const row=spr[r];
    for(let q=0;q<row.length;q++){
      const ch=row[q];
      if(ch==='.'||(o.eng&&ch==='E'))continue;
      let col;
      if(tc)col=o.tcols[ch];else{const ci=o.cols[ch];col=ci===undefined?null:CS(ci);}
      if(!col)continue;
      g.fillStyle=col;g.fillRect(1+q*sc,1+r*sc,sc,sc);
    }
  }
  m.set(key,c);return c;
}
function blitSpr(o,fi,sc,tc,px,py,ang){
  const c=sprCanvas(o,fi,sc,tc);
  lctx.save();lctx.translate(Math.round(px),Math.round(py));
  if(ang)lctx.rotate(ang);
  lctx.imageSmoothingEnabled=false;
  lctx.drawImage(c,-(c.width>>1),-(c.height>>1));
  lctx.restore();
}
function drawSpr(o,sx,sy,s,glowR,glowA,bobSp){
  const sc=Math.max(1,Math.round(o.scale*S.sprSize*s));
  const fo=floatOff(o);
  const px=sx+fo[0]*s,py=sy+fo[1]*s+Math.sin(wt*bobSp+o.ph)*1.2;
  lctx.fillStyle=CS(P.top[0]);lctx.globalAlpha=glowA;circ(px,py,glowR*sc);lctx.globalAlpha=1;
  let fi=0;
  if(o.frames.length>1&&o.fps>0){
    const k=Math.floor(wt*o.fps+o.ph*3);
    fi=o.seq?o.seq[k%o.seq.length]:k%o.frames.length;
  }
  const ang=rotAng(o,false);
  if(S.crisp)spriteQ.push(()=>blitSpr(o,fi,sc,true,px,py,ang));
  else blitSpr(o,fi,sc,false,px,py,ang);
}
/* ---- neon sign renderer: draws into a scratch canvas, then blits rotated with hard edges ---- */
let sgC=null,sgG=null;
function renderSign(o,sx,sy,s,tc){
  const sc=Math.max(1,Math.round(2.4*s)),LH=7,ADV=6.5,GAP=3,nl=o.lineL.length;
  const wMax=Math.max.apply(null,o.lineL.map(l=>l.length));
  const W=Math.ceil((wMax*ADV-1.5)*sc),H=(nl*LH+(nl-1)*GAP)*sc,pad=Math.ceil(sc*5);
  const CW=W+pad*2,CH=H+pad*2,ox=CW>>1,oy=CH>>1;
  if(!sgC){sgC=document.createElement('canvas');sgG=sgC.getContext('2d');}
  if(sgC.width<CW||sgC.height<CH){sgC.width=Math.max(CW,sgC.width);sgC.height=Math.max(CH,sgC.height);}
  const g=sgG;g.setTransform(1,0,0,1,0,0);g.globalAlpha=1;g.clearRect(0,0,sgC.width,sgC.height);
  g.translate(ox,oy);
  const C=k=>tc?o.css[k]:CS(o.idx[k]);
  g.fillStyle=C('on');g.globalAlpha=.06*(S.glow*.75+.25);g.fillRect(-W/2-sc*4,-H/2-sc*4,W+sc*8,H+sc*8);
  g.fillStyle=tc?'#07050b':CS(P.deep[0]);g.globalAlpha=.8;g.fillRect(-W/2-sc*1.5,-H/2-sc*2,W+sc*3,H+sc*4);
  g.globalAlpha=.9;
  const bx=-W/2-sc,by=-H/2-sc*1.5,bw=W+sc*2,bh=H+sc*3,th=Math.max(1,Math.round(sc*.55));
  g.strokeStyle=C('fr');g.fillStyle=C('fr');g.lineWidth=th;
  if(o.fr===1||o.fr===2)g.strokeRect(bx,by,bw,bh);
  if(o.fr===2)g.strokeRect(bx-sc*1.2,by-sc*1.2,bw+sc*2.4,bh+sc*2.4);
  if(o.fr===3){g.fillRect(bx,by,bw,th);g.fillRect(bx,by+bh-th,bw,th);}
  if(o.fr){g.fillStyle=C('on');
    for(let q=0;q<4;q++){
      if(Math.sin(rt*4+q*1.57+o.rs%10)>.3){g.globalAlpha=1;
        g.fillRect(Math.floor((q%2?1:-1)*(W/2+sc)-sc/2),Math.floor((q<2?-1:1)*(H/2+sc*1.5)-sc/2),Math.max(1,sc),Math.max(1,sc));}}
  }
  const nAll=o.letters.length;let ia=0;
  o.lineL.forEach((line,li)=>{
    const lw=(line.length*ADV-1.5)*sc,x0=-lw/2,y0=-H/2+li*(LH+GAP)*sc;
    const kOn=o.lineCol[li]?'on2':'on',kDim=o.lineCol[li]?'dim2':'dim';
    for(let i=0;i<line.length;i++){
      const L=line[i];if(L.ch===' ')continue;
      const gl=FONT5[L.ch];if(!gl){ia++;continue;}
      let lit;
      if(L.mode==='on')lit=.92+.08*Math.sin(wt*3+L.ph);
      else if(L.mode==='blink')lit=Math.sin(wt*L.sp+L.ph)>0?1:.06;
      else if(L.mode==='flicker')lit=(Math.sin(wt*7+L.ph)+Math.sin(wt*13.7+L.ph*2))>.2?1:.08;
      else lit=.07;
      if(o.anim==='blink')lit*=Math.sin(wt*2.4+o.ph)>0?1:.1;
      else if(o.anim==='chase'){const pos=(wt*5+o.ph)%(nAll+3);lit*=Math.abs(ia-pos)<1.6?1:.18;}
      for(let r=0;r<LH;r++){
        const row=gl[r];
        for(let c=0;c<5;c++){
          if(row[c]!=='1')continue;
          const px=Math.floor(x0+i*ADV*sc+c*sc),py=Math.floor(y0+r*sc);
          if(lit>.45){
            g.fillStyle=C(kOn);g.globalAlpha=.22*lit;g.fillRect(px-1,py-1,sc+2,sc+2);
            g.globalAlpha=lit;g.fillRect(px,py,sc,sc);
          }else{g.fillStyle=C(kDim);g.globalAlpha=.45;g.fillRect(px,py,sc,sc);}
        }
      }
      ia++;
    }
  });
  g.globalAlpha=1;
  lctx.save();lctx.translate(Math.round(sx),Math.round(sy));
  const ang=rotAng(o,true);if(ang)lctx.rotate(ang);
  lctx.imageSmoothingEnabled=false;
  lctx.drawImage(sgC,0,0,CW,CH,-ox,-oy,CW,CH);
  lctx.restore();lctx.globalAlpha=1;
}
/* ---- ship formations ---- */
function spawnShip(){
  const vr=viewRect();
  const si=pick(Math.random,SIPOOL),len=shipLen(si),M=60+len;
  const side=(Math.random()*4)|0;
  let x,y;
  if(side===0){x=vr.x0-M;y=vr.y0+Math.random()*(vr.y1-vr.y0);}
  else if(side===1){x=vr.x1+M;y=vr.y0+Math.random()*(vr.y1-vr.y0);}
  else if(side===2){y=vr.y0-M;x=vr.x0+Math.random()*(vr.x1-vr.x0);}
  else{y=vr.y1+M;x=vr.x0+Math.random()*(vr.x1-vr.x0);}
  const tx=cam.x+(Math.random()-.5)*(vr.x1-vr.x0)*.7,ty=cam.y+(Math.random()-.5)*(vr.y1-vr.y0)*.7;
  const d=Math.hypot(tx-x,ty-y)||1;
  const warp=len<40&&Math.random()<.12;
  const ss=SHIPSPD[si],sp=warp?220+Math.random()*120:(ss?ss[0]+Math.random()*ss[1]:16+Math.random()*34);
  const vx=(tx-x)/d*sp,vy=(ty-y)/d*sp,dx=vx/sp,dy=vy/sp,nx=-dy,ny=dx,ang=Math.atan2(vy,vx);
  const cols={H:pick(Math.random,P.hi),D:pick(Math.random,P.lo),
    W:P.top[0],A:pick(Math.random,P.top),E:P.hot[P.hot.length-1]};
  const K=SHIPKIND,cls=K.fighter.includes(si)?'fighter':K.heavy.includes(si)?'heavy':K.alien.includes(si)?'alien':K.saucer.includes(si)?'saucer':'light';
  const r=Math.random();
  let form='solo';
  if(warp)form=r<.7?'solo':'pair';
  else if(cls==='fighter')form=r<.28?'solo':r<.42?'pair':r<.62?'wedge':r<.78?'swarm':r<.9?'patrol':'column';
  else if(cls==='heavy')form=r<.32?'solo':r<.50?'column':r<.75?'escort':'pair';
  else if(cls==='alien')form=r<.30?'solo':r<.55?'pair':r<.8?'swarm':'patrol';
  else if(cls==='saucer')form=r<.55?'solo':r<.8?'pair':'patrol';
  else form=r<.6?'solo':r<.85?'pair':'patrol';
  if(len>90&&form==='column')form='escort';
  const f=Math.max(1,len/18),J=n=>(Math.random()-.5)*n;
  const offs=[[0,0,si]];                       /* [ahead, side, sprite] */
  if(form==='pair')offs.push([-(10+Math.random()*8),(Math.random()<.5?1:-1)*(8+Math.random()*6),si]);
  else if(form==='patrol')offs.push([-14,10,si],[-14,-10,si]);
  else if(form==='wedge'){const n=1+((Math.random()*2)|0);for(let k=1;k<=n;k++)offs.push([-k*13,k*10,si],[-k*13,-k*10,si]);}
  else if(form==='column'){const n=2+((Math.random()*3)|0);
    for(let k=1;k<=n;k++)offs.push([-k*(16+Math.random()*5),J(5),(si<IMGSHIP0&&Math.random()<.25)?pick(Math.random,[2,3,11,16]):si]);}
  else if(form==='escort'){const pos=[[16,-15],[16,15],[-18,-17],[-18,17]],n=2+((Math.random()*3)|0);
    for(let k=0;k<n;k++)offs.push([pos[k][0],pos[k][1],pick(Math.random,ESCORTS)]);}
  else if(form==='swarm'){const n=4+((Math.random()*5)|0),small=cls==='alien'?ALIENSM:[4,15];
    for(let k=0;k<n;k++)offs.push([-Math.random()*44,J(46),Math.random()<.5?pick(Math.random,small):si]);}
  const turn=(form==='solo'&&!warp&&len<60&&Math.random()<.35)?(Math.random()<.5?-1:1)*(.02+Math.random()*.03):0;
  const hue=(si>=IMGSHIP0&&Math.random()<.45)?(1+((Math.random()*5)|0))*60:0;
  for(const o of offs){
    const j=form==='swarm'?.94+Math.random()*.12:1,a=o[0]*f,b=o[1]*f;
    ships.push({cls:'ship',form,x:x+dx*a+nx*b,y:y+dy*a+ny*b,vx:vx*j,vy:vy*j,ang,warp,si:o[2],cols,turn,hue,
      hr:shipLen(o[2])/2,rs:(Math.random()*1e9)|0});
  }
}
/* ---- supernova event ---- */
function spawnNova(){
  const vr=viewRect();
  novas.push({cls:'nova',x:vr.x0+(.1+Math.random()*.8)*(vr.x1-vr.x0),y:vr.y0+(.1+Math.random()*.8)*(vr.y1-vr.y0),
    t:0,life:7,r:26+Math.random()*34,ci:pick(Math.random,P.hot),rs:(Math.random()*1e9)|0,
    rays:8+((Math.random()*6)|0),ra:Math.random()*TAU});
}
function drawNovas(z,cx0,cy0){
  for(const n of novas){
    const sx=(n.x-cam.x)*z+cx0,sy=(n.y-cam.y)*z+cy0,k=n.t/n.life;
    if(sx<-200||sy<-200||sx>lw+200||sy>lh+200)continue;
    const R=n.r*z*(1-Math.pow(1-k,3)),hot=CS(P.hot[P.hot.length-1]),col=CS(n.ci);
    if(k<.12){lctx.fillStyle=hot;lctx.globalAlpha=(1-k/.12)*.9;circ(sx,sy,6+n.r*z*.25*(k/.12+.3));}
    lctx.strokeStyle=col;lctx.lineWidth=Math.max(1,2*(1-k)*z);lctx.globalAlpha=(1-k)*.8;
    lctx.beginPath();lctx.arc(sx,sy,R,0,TAU);lctx.stroke();
    lctx.globalAlpha=(1-k)*.4;lctx.beginPath();lctx.arc(sx,sy,R*.7,0,TAU);lctx.stroke();
    lctx.fillStyle=col;lctx.globalAlpha=(1-k)*.12;circ(sx,sy,R*.8);
    lctx.strokeStyle=hot;lctx.lineWidth=1;
    for(let q=0;q<n.rays;q++){
      const a=n.ra+q*TAU/n.rays,l0=R*.2,l1=R*(.55+.4*((q*37)%7)/7);
      lctx.globalAlpha=(1-k)*.55;lctx.beginPath();
      lctx.moveTo(sx+Math.cos(a)*l0,sy+Math.sin(a)*l0);lctx.lineTo(sx+Math.cos(a)*l1,sy+Math.sin(a)*l1);lctx.stroke();
    }
    lctx.fillStyle=hot;lctx.globalAlpha=Math.min(1,(1-k)*1.4);lctx.fillRect((sx|0)-1,(sy|0)-1,2,2);
  }
  lctx.globalAlpha=1;
}
/* ---- wormhole / void jelly / lighthouse rock ---- */
function drawWorm(o,sx,sy,s){
  const r=Math.max(6,o.r*s);
  lctx.fillStyle=CS(o.cC);lctx.globalAlpha=.07*(S.glow*.75+.25);circ(sx,sy,r*1.9);
  lctx.globalAlpha=.1*(S.glow*.75+.25);circ(sx,sy,r*1.4);
  for(let a=0;a<o.arms;a++)for(let k=0;k<34;k++){
    const q=k/34,an=a*TAU/o.arms+q*5.5*o.dir+wt*o.sp*o.dir,rr=r*(1.15-q*.95);
    lctx.fillStyle=CS(q<.3?o.cA:q<.7?o.cB:o.cC);lctx.globalAlpha=Math.min(1,.55+.5*q);
    const sz=(q>.5&&s>.8)||s>1.8?2:1;
    lctx.fillRect((sx+Math.cos(an)*rr)|0,(sy+Math.sin(an)*rr*.8)|0,sz,sz);
  }
  lctx.globalAlpha=1;lctx.fillStyle=P.str[0];circ(sx,sy,r*.3);
  lctx.strokeStyle=CS(o.cA);lctx.lineWidth=1;lctx.globalAlpha=.85;
  lctx.beginPath();lctx.arc(sx,sy,r*.33,0,TAU);lctx.stroke();
  const pu=.55+.12*Math.sin(wt*1.7);
  lctx.globalAlpha=.3;lctx.strokeStyle=CS(o.cB);lctx.beginPath();lctx.ellipse(sx,sy,r*pu,r*pu*.8,0,0,TAU);lctx.stroke();
  lctx.globalAlpha=1;
}
function drawJelly(o,sx,sy,s){
  const R=Math.max(3,o.r*s),pu=Math.sin(wt*o.sp+o.ph),bw=R*(1+.14*pu),bh=R*(.75-.14*pu);
  lctx.save();lctx.translate(sx,sy+Math.sin(wt*.4+o.ph)*2);lctx.rotate(Math.sin(wt*.3+o.ph)*.22);
  lctx.fillStyle=CS(o.cC);lctx.globalAlpha=.1*(S.glow*.75+.25);
  lctx.beginPath();lctx.ellipse(0,0,bw*1.6,bh*1.6,0,0,TAU);lctx.fill();
  const sz=R>8?2:1;
  for(let i=0;i<o.n;i++){
    const bx=(i-(o.n-1)/2)*bw*.45;
    for(let k=1;k<=9;k++){
      const xx=bx+Math.sin(k*.8-wt*o.sp*1.4+i*1.3)*R*.12*k/3;
      lctx.fillStyle=CS(k<4?o.cB:o.cC);lctx.globalAlpha=1-k/11;
      lctx.fillRect(Math.floor(xx),Math.floor(k*R*.2),sz,sz);
    }
  }
  lctx.globalAlpha=.9;lctx.fillStyle=CS(o.cA);lctx.beginPath();lctx.ellipse(0,0,bw,bh,0,Math.PI,TAU);lctx.closePath();lctx.fill();
  lctx.globalAlpha=.7;lctx.fillStyle=CS(o.cB);lctx.beginPath();lctx.ellipse(0,0,bw*.6,bh*.55,0,Math.PI,TAU);lctx.fill();
  lctx.globalAlpha=.9;lctx.fillStyle=CS(P.top[0]);lctx.fillRect(-1,-Math.max(1,(bh*.5)|0),2,1);
  lctx.restore();lctx.globalAlpha=1;
}
function drawBeacon(o,sx,sy,s){
  const r=Math.max(4,o.r*s),a=wt*o.w+o.ph,L=o.L*s,fl=.75+.25*Math.sin(wt*5+o.ph),ly=sy-r*1.15;
  lctx.fillStyle=CS(o.cBeam);
  for(let q=0;q<2;q++){
    const b=a+q*Math.PI;
    lctx.globalAlpha=.2*fl;lctx.beginPath();lctx.moveTo(sx,ly);
    lctx.lineTo(sx+Math.cos(b-.12)*L,ly+Math.sin(b-.12)*L*.5);lctx.lineTo(sx+Math.cos(b+.12)*L,ly+Math.sin(b+.12)*L*.5);
    lctx.closePath();lctx.fill();
  }
  lctx.globalAlpha=1;lctx.fillStyle=CS(o.cRock);lctx.beginPath();
  lctx.moveTo(sx+o.pts[0][0]*r,sy+o.pts[0][1]*r*.7);
  for(let k=1;k<o.pts.length;k++)lctx.lineTo(sx+o.pts[k][0]*r,sy+o.pts[k][1]*r*.7);
  lctx.closePath();lctx.fill();
  lctx.fillStyle=CS(P.top[0]);lctx.fillRect((sx-r*.09)|0,(ly)|0,Math.max(1,(r*.18)|0),Math.max(2,(r*1.0)|0));
  lctx.fillStyle=CS(o.cBeam);lctx.globalAlpha=.6+.4*fl;lctx.fillRect((sx-r*.16)|0,(ly-1)|0,Math.max(2,(r*.32)|0),2);
  lctx.globalAlpha=1;
}
/* night-side city lights + aurora on some worlds (called inside the planet clip) */
function planetExtras(o,sx,sy,r,ca,sa){
  if(r<4)return;
  const k=o.kind;
  if(k==='earth'||k==='terra'){
    if(o._cl===undefined){const q=mulberry((o.rs^0xC17E)>>>0);o._cl=null;
      if(q()<.55){o._cl=[];for(let i=0;i<30;i++){const a=q()*TAU,d=Math.sqrt(q())*.95;o._cl.push([Math.cos(a)*d,Math.sin(a)*d,q()]);}}}
    if(o._cl){
      if(o._lcp!==P.i){o._lc=CS(nearT(255,214,120));o._lcp=P.i;}
      lctx.fillStyle=o._lc;
      for(const q of o._cl)if(q[0]*ca+q[1]*sa>.3){
        lctx.globalAlpha=.55+.45*Math.abs(Math.sin(wt*.6+q[2]*9));
        lctx.fillRect((sx+q[0]*r)|0,(sy+q[1]*r)|0,1,1);}
      lctx.globalAlpha=1;
    }
  }
  if(k==='earth'||k==='ice'){
    if(o._au===undefined)o._au=mulberry((o.rs^0xA0A0)>>>0)()<.35;
    if(o._au){
      if(o._acp!==P.i){o._ac=CS(nearT(90,255,170));o._acp=P.i;}
      lctx.fillStyle=o._ac;
      for(let i=-5;i<=5;i++){
        const h=1+Math.abs(Math.sin(wt*1.3+i*.7))*Math.max(1,r*.18);
        lctx.globalAlpha=.35+.4*Math.abs(Math.sin(wt*.9+i));
        lctx.fillRect((sx+i*r*.12)|0,(sy-r*.78+Math.abs(i)*r*.03-h)|0,1,h|0);}
      lctx.globalAlpha=1;
    }
  }
}

/* ---------------- URL sync / copy link ---------------- */
function buildQuery(){
  const q=new URLSearchParams();
  q.set('seed',seedStr);
  if(P.i!==0)q.set('pal',PALETTES[P.i].n.toLowerCase().replace(/\s+/g,'-'));
  for(const k in NUMR)if(+S[k]!==+S_DEF[k])q.set(k,(+S[k]).toFixed(3).replace(/\.?0+$/,''));
  for(const k of BOOLK)if(S[k]!==S_DEF[k])q.set(k,S[k]?1:0);
  const on=TYPELIST.filter(t=>ON[t[0]]).map(t=>t[0]),off=TYPELIST.filter(t=>!ON[t[0]]).map(t=>t[0]);
  if(off.length&&on.length<=off.length)q.set('only',on.join(','));   /* shorter of the two lists */
  else if(off.length)q.set('off',off.join(','));
  if(panel.classList.contains('hidden'))q.set('panel',0);
  if(uiLocked)q.set('ui',0);
  if(window._noFault)q.set('fault',0);
  return q.toString();
}
let syncT=0;
function syncURL(){
  clearTimeout(syncT);
  syncT=setTimeout(()=>{try{history.replaceState(null,'','?'+buildQuery());}catch(e){}},300);
}
['input','change','click'].forEach(ev=>panel.addEventListener(ev,syncURL));
panelToggle.addEventListener('click',syncURL);
addEventListener('keydown',e=>{if(e.key.toLowerCase()==='s')syncURL();});

/* ---------------- init ---------------- */
setPalette(0);
rebuildLUT();
resize();
zoom=S.ztzoom;
applySeed(URLC.seed||Math.random().toString(36).slice(2,8).toUpperCase());
{
  const si=panel.querySelector('.seedrow input');
  if(si)si.value=seedStr;
}
setPanel(URLC.panel!==null?URLC.panel:innerWidth>760);
if(URLC.pal!==null&&URLC.pal!==0){const b=panel.querySelectorAll('.pal')[URLC.pal];if(b)b.click();}
if(S.autoReroll)nextReroll=Date.now()+S.rerollMin*60000;
if(URLC.ui===false){uiLocked=true;hideUI();}
if(ON.ship)spawnShip();
requestAnimationFrame(frame);
</script>
</body>
</html>