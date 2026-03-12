import { useState, useMemo, useCallback } from "react";

const MONTHS = ["Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"];
const DEFAULT_GOALS = { rothLimit:7500, hysaTarget:36000, travelTarget:2000, brokerageMonthly:1500, brokerageAge50:800000, rothAge50:300000 };
const DEFAULT_BALANCES = { hysaStart:28300, travelStart:234, rothContributed:0 };
const DEFAULT_RENT = { total:1700, myShare:425 };
const DEFAULT_BROK_ALLOC = { FXAIX:60, FSPSX:30, FSELX:10 };

const DEFAULT_EXPENSES = [
  { id:1, name:"Groceries", amount:200, cc:"AMEX" },
  { id:2, name:"ConEd (Electricity)", amount:40, cc:"Regular" },
  { id:3, name:"Utilities", amount:70, cc:"Regular" },
  { id:4, name:"Personal Care", amount:150, cc:"Regular" },
  { id:5, name:"Cat Expenses", amount:50, cc:"Regular" },
  { id:6, name:"Transportation", amount:150, cc:"Regular" },
  { id:7, name:"Shopping", amount:50, cc:"Regular" },
  { id:8, name:"Home", amount:75, cc:"Regular" },
  { id:9, name:"Dining", amount:100, cc:"AMEX" },
  { id:10, name:"Entertainment", amount:80, cc:"AMEX" },
  { id:11, name:"Misc", amount:150, cc:"Regular" },
];

const DEFAULT_SUBS = [
  { id:101, name:"Spotify", amount:11, cc:"Regular" },
  { id:102, name:"Amazon Prime", amount:7, cc:"Regular" },
  { id:103, name:"New Yorker", amount:12, cc:"Regular" },
  { id:104, name:"Netflix", amount:11, cc:"Regular" },
  { id:105, name:"NYT All Access", amount:25, cc:"Regular" },
  { id:106, name:"NYT Games", amount:5, cc:"Regular" },
  { id:107, name:"Flo", amount:5, cc:"Regular" },
  { id:108, name:"AppleCare", amount:9.99, cc:"Regular" },
  { id:109, name:"FlightRadar24", amount:4.99, cc:"Regular" },
  { id:110, name:"Claude", amount:21.0, cc:"Regular" },
];

const fmt = (n) => {
  if (n===undefined||n===null||isNaN(n)) return "$0.00";
  const a = Math.abs(n).toFixed(2).replace(/\B(?=(\d{3})+(?!\d))/g,",");
  return n<0?`-$${a}`:`$${a}`;
};
const fmtInt = (n) => `$${Math.round(n||0).toLocaleString()}`;

/* ── palette ── */
const C = {
  bg: "#FBF8F3", bgAlt: "#F5F0E8", bgCard: "#FFFFFF", bgWarm: "#FAF6EF",
  border: "#E8E0D4", borderLight: "#F0EBE3",
  text: "#3D3229", textMed: "#6B5D50", textLight: "#9B8E80", textFaint: "#C4B8AA",
  sage: "#7A9E7E", sageBg: "#EFF5F0", sageBorder: "#D4E5D6",
  terracotta: "#C67B5C", terraBg: "#FDF2ED", terraBorder: "#F0D5C7",
  gold: "#C9A84C", goldBg: "#FDF8ED", goldBorder: "#EDE0B8",
  blue: "#6B8DB5", blueBg: "#EDF2F8", blueBorder: "#CDDAEB",
  plum: "#9B7BA8", plumBg: "#F5F0F8", plumBorder: "#DDD0E5",
  rose: "#C47A7A", roseBg: "#FDF0F0", roseBorder: "#F0CCCC",
  red: "#C45D4D", green: "#5D9B62",
};

function computeMonth(mi, pay, hBal, tBal, rYTD, cfg) {
  const { goals, rent, jhu, taxRate, expenses, subs } = cfg;
  const brokPer = Math.round((goals.brokerageMonthly/3)*100)/100;

  let rI=0,rP1=0,rP2=0;
  if(mi<=3){rI=500;rP1=500;rP2=500;}
  else if(mi===4){const rem=Math.max(0,goals.rothLimit-rYTD);rI=Math.min(750,rem);rP1=Math.min(750,Math.max(0,rem-rI));}
  const rothTot=rI+rP1+rP2;

  let hI=0,hP1=0,hP2=0;
  const hNeed=Math.max(0,goals.hysaTarget-hBal);
  if(mi<=4){hI=268;hP1=268;hP2=268;}
  else if(mi>=5&&mi<=7){hI=500;hP1=500;hP2=500;}
  else if(mi===8){hI=Math.min(500,hNeed);hP1=Math.min(500,Math.max(0,hNeed-hI));hP2=Math.min(120,Math.max(0,hNeed-hI-hP1));}
  let hRaw=hI+hP1+hP2;
  if(hRaw>hNeed&&hNeed>=0&&hRaw>0){const s=hNeed/hRaw;hI=Math.round(hI*s*100)/100;hP1=Math.round(hP1*s*100)/100;hP2=Math.round(Math.max(0,hNeed-hI-hP1)*100)/100;}
  const hTot=hI+hP1+hP2;

  const tAct=mi<=7;
  const tI=tAct?83.33:0,tP1=tAct?83.33:0,tP2=tAct?83.34:0;
  const tTot=tI+tP1+tP2;

  const all=[...expenses,...subs];
  const amex=all.filter(e=>e.cc==="AMEX").reduce((s,e)=>s+(parseFloat(e.amount)||0),0);
  const reg=all.filter(e=>e.cc==="Regular").reduce((s,e)=>s+(parseFloat(e.amount)||0),0);
  const ccTot=amex+reg;

  const iG=pay.imslp,iTx=Math.round(iG*taxRate*100)/100,iAT=iG-iTx;
  const iAll=rent.myShare+jhu+rI+brokPer+hI+tI;
  const iSur=Math.round((iAT-iAll)*100)/100;

  const p1G=pay.paycheck1,p1R=rent.total-rent.myShare;
  const p1All=p1R+jhu+rP1+brokPer+hP1+tP1;
  const p1Sur=Math.round((p1G-p1All)*100)/100;

  const p2G=pay.paycheck2;
  const p2All=jhu+ccTot+rP2+brokPer+hP2+tP2;
  const p2Sur=Math.round((p2G-p2All)*100)/100;

  return {
    month:MONTHS[mi],mi,pay,tax:iTx,
    rent:{imslp:rent.myShare,p1:p1R,total:rent.total},jhu,
    roth:{imslp:rI,p1:rP1,p2:rP2,total:rothTot,ytd:rYTD+rothTot},
    brok:{per:brokPer,total:brokPer*3},
    hysa:{imslp:hI,p1:hP1,p2:hP2,total:hTot,end:hBal+hTot},
    travel:{imslp:tI,p1:tP1,p2:tP2,total:tTot,end:tBal+tTot},
    cc:{amex,regular:reg,total:ccTot},
    checks:{
      imslp:{gross:iG,tax:iTx,afterTax:iAT,rent:rent.myShare,jhu,roth:rI,brok:brokPer,hysa:hI,travel:tI,surplus:iSur},
      p1:{gross:p1G,rent:p1R,jhu,roth:rP1,brok:brokPer,hysa:hP1,travel:tP1,surplus:p1Sur},
      p2:{gross:p2G,jhu,roth:rP2,brok:brokPer,hysa:hP2,travel:tP2,cc:ccTot,surplus:p2Sur},
    },
    surplus:iSur+p1Sur+p2Sur,
  };
}

function Field({label,value,onChange,prefix="$",suffix,small,style:sx}){
  return(
    <div style={{flex:1,minWidth:small?70:120,...sx}}>
      {label&&<label style={{fontSize:10,color:C.textLight,textTransform:"uppercase",letterSpacing:1.5,display:"block",marginBottom:5,fontWeight:500}}>{label}</label>}
      <div style={{position:"relative"}}>
        {prefix&&<span style={{position:"absolute",left:10,top:"50%",transform:"translateY(-50%)",fontSize:small?11:12,color:C.textFaint,fontWeight:500}}>{prefix}</span>}
        <input type="number" value={value} onChange={e=>onChange(parseFloat(e.target.value)||0)}
          style={{background:C.bgWarm,border:`1.5px solid ${C.border}`,borderRadius:10,color:C.text,padding:small?"6px 10px":"9px 12px",
          fontFamily:"'Nunito',sans-serif",fontSize:small?12:13,fontWeight:600,width:"100%",outline:"none",
          paddingLeft:prefix?24:12,paddingRight:suffix?30:12,transition:"border-color 0.2s, box-shadow 0.2s"}} />
        {suffix&&<span style={{position:"absolute",right:11,top:"50%",transform:"translateY(-50%)",fontSize:12,color:C.textFaint,fontWeight:500}}>{suffix}</span>}
      </div>
    </div>
  );
}

function GoalCard({title,current,target,color,colorBg,colorBorder,icon,deadline}){
  const pct=target>0?Math.min(100,(current/target)*100):0;
  return(
    <div style={{background:colorBg,border:`1.5px solid ${colorBorder}`,borderRadius:16,padding:"18px 20px",position:"relative",overflow:"hidden",transition:"transform 0.2s",cursor:"default"}}>
      <div style={{position:"absolute",top:-30,right:-30,width:90,height:90,background:`${color}10`,borderRadius:"50%"}} />
      <div style={{display:"flex",alignItems:"center",gap:8,marginBottom:10}}>
        <span style={{fontSize:20}}>{icon}</span>
        <span style={{fontFamily:"'Playfair Display',serif",fontSize:15,color:C.text,fontWeight:600}}>{title}</span>
      </div>
      <div style={{fontFamily:"'Nunito',sans-serif",fontSize:28,fontWeight:800,color,letterSpacing:-1}}>{fmtInt(current)}</div>
      <div style={{fontSize:11,color:C.textLight,margin:"4px 0 12px",fontWeight:500}}>{fmtInt(target)} goal · {deadline}</div>
      <div style={{height:8,background:`${color}18`,borderRadius:4,overflow:"hidden"}}>
        <div style={{height:"100%",width:`${pct}%`,background:`linear-gradient(90deg,${color},${color}cc)`,borderRadius:4,transition:"width 0.7s cubic-bezier(0.22,1,0.36,1)"}} />
      </div>
      <div style={{fontSize:10,color:C.textLight,marginTop:5,fontWeight:600,textAlign:"right"}}>{pct.toFixed(1)}%</div>
    </div>
  );
}

function CheckCard({label,data,showTax,showRent,showCC,accent}){
  const rows=[];
  if(showTax) rows.push({l:"1099 Tax (30%)",v:data.tax,c:C.rose,neg:true});
  if(showRent) rows.push({l:`Rent (${label==="IMSLP"?"¼":"¾"})`,v:data.rent});
  rows.push({l:"JHU Loan",v:data.jhu});
  if(data.roth>0) rows.push({l:"Roth → FXAIX",v:data.roth,c:C.plum});
  rows.push({l:"Brokerage",v:data.brok,c:C.blue});
  if(data.hysa>0) rows.push({l:"HYSA",v:data.hysa,c:C.sage});
  if(data.travel>0) rows.push({l:"Travel",v:data.travel,c:C.gold});
  if(showCC) rows.push({l:"Credit Cards",v:data.cc,c:C.terracotta});
  return(
    <div style={{background:C.bgCard,border:`1.5px solid ${C.border}`,borderRadius:16,padding:18,flex:1,minWidth:220,boxShadow:"0 2px 12px rgba(61,50,41,0.04)"}}>
      <div style={{display:"flex",alignItems:"center",gap:8,marginBottom:6}}>
        <div style={{width:8,height:8,borderRadius:"50%",background:accent}} />
        <span style={{fontFamily:"'Playfair Display',serif",fontSize:15,color:C.text,fontWeight:600}}>{label}</span>
      </div>
      <div style={{fontFamily:"'Nunito',sans-serif",fontSize:24,fontWeight:800,color:C.text,letterSpacing:-0.5}}>{fmt(data.gross)}</div>
      {showTax&&<div style={{fontSize:11,color:C.rose,fontWeight:600,marginBottom:4}}>After tax: {fmt(data.afterTax)}</div>}
      <div style={{borderTop:`1.5px solid ${C.borderLight}`,marginTop:8,paddingTop:8}}>
        {rows.map((r,i)=>(
          <div key={i} style={{display:"flex",justifyContent:"space-between",padding:"5px 0",fontSize:12,fontFamily:"'Nunito',sans-serif"}}>
            <span style={{color:C.textMed,fontWeight:500}}>{r.l}</span>
            <span style={{color:r.c||C.text,fontWeight:700}}>{r.neg?`-${fmt(r.v)}`:fmt(r.v)}</span>
          </div>
        ))}
        <div style={{display:"flex",justifyContent:"space-between",padding:"8px 0 0",fontSize:13,borderTop:`1.5px solid ${C.borderLight}`,marginTop:6,fontFamily:"'Nunito',sans-serif"}}>
          <span style={{color:data.surplus>=0?C.green:C.red,fontWeight:700}}>{data.surplus>=0?"Surplus":"Deficit"}</span>
          <span style={{color:data.surplus>=0?C.green:C.red,fontWeight:800,fontSize:14}}>{fmt(data.surplus)}</span>
        </div>
      </div>
    </div>
  );
}

export default function BudgetApp(){
  const [mo,setMo]=useState(0);
  const [tab,setTab]=useState("overview");
  const [pays,setPays]=useState(Array.from({length:12},()=>({imslp:2200,paycheck1:3200,paycheck2:3200})));
  const [bal,setBal]=useState({...DEFAULT_BALANCES});
  const [goals,setGoals]=useState({...DEFAULT_GOALS});
  const [rent,setRent]=useState({...DEFAULT_RENT});
  const [jhu,setJhu]=useState(50);
  const [taxRate,setTaxRate]=useState(0.30);
  const [exps,setExps]=useState(DEFAULT_EXPENSES.map(e=>({...e})));
  const [subs,setSubs]=useState(DEFAULT_SUBS.map(s=>({...s})));
  const [brokA,setBrokA]=useState({...DEFAULT_BROK_ALLOC});

  const updatePay=useCallback((f,v)=>setPays(p=>{const n=p.map(x=>({...x}));n[mo]={...n[mo],[f]:v};return n;}),[mo]);
  const updateExp=useCallback((id,f,v)=>setExps(p=>p.map(e=>e.id===id?{...e,[f]:v}:e)),[]);
  const updateSub=useCallback((id,f,v)=>setSubs(p=>p.map(s=>s.id===id?{...s,[f]:v}:s)),[]);

  const cfg=useMemo(()=>({goals,rent,jhu,taxRate,expenses:exps,subs}),[goals,rent,jhu,taxRate,exps,subs]);
  const year=useMemo(()=>{
    const r=[];let hB=bal.hysaStart,tB=bal.travelStart,rY=bal.rothContributed;
    for(let i=0;i<12;i++){const m=computeMonth(i,pays[i],hB,tB,rY,cfg);r.push(m);hB=m.hysa.end;tB=m.travel.end;rY=m.roth.ytd;}return r;
  },[pays,bal,cfg]);
  const cur=year[mo];

  const tabDef=[{id:"overview",label:"Overview"},{id:"paychecks",label:"Paychecks"},{id:"investments",label:"Investments"},{id:"expenses",label:"Expenses"},{id:"settings",label:"Settings"},{id:"yearly",label:"12-Month"}];

  const inSm={background:C.bgWarm,border:`1.5px solid ${C.border}`,borderRadius:8,color:C.text,padding:"6px 10px",fontFamily:"'Nunito',sans-serif",fontSize:12,fontWeight:600,width:"100%",outline:"none"};
  const addBtn={padding:"7px 16px",fontSize:11,fontFamily:"'Nunito',sans-serif",fontWeight:700,color:C.sage,background:C.sageBg,border:`1.5px solid ${C.sageBorder}`,borderRadius:8,cursor:"pointer",transition:"all 0.15s"};
  const rmBtn={padding:"4px 10px",fontSize:10,fontFamily:"'Nunito',sans-serif",fontWeight:700,color:C.rose,background:C.roseBg,border:`1.5px solid ${C.roseBorder}`,borderRadius:6,cursor:"pointer"};
  const card={background:C.bgCard,border:`1.5px solid ${C.border}`,borderRadius:16,padding:18,boxShadow:"0 2px 12px rgba(61,50,41,0.04)"};
  const secT={fontFamily:"'Playfair Display',serif",fontSize:22,color:C.text,marginBottom:16,fontWeight:700};
  const subT={fontFamily:"'Playfair Display',serif",fontSize:15,color:C.textMed,marginBottom:12,fontWeight:600};

  return(
    <div style={{minHeight:"100vh",background:C.bg,color:C.text,fontFamily:"'Nunito',sans-serif"}}>
      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Nunito:ital,wght@0,300;0,400;0,500;0,600;0,700;0,800;0,900;1,400&family=Playfair+Display:ital,wght@0,400;0,500;0,600;0,700;0,800;1,400;1,500&display=swap');
        *{box-sizing:border-box;margin:0;padding:0}
        input[type="number"]::-webkit-inner-spin-button{opacity:0.4}
        input:focus{border-color:${C.terracotta} !important;box-shadow:0 0 0 3px ${C.terraBg} !important}
        select{background:${C.bgWarm};border:1.5px solid ${C.border};border-radius:8px;color:${C.text};padding:6px 10px;font-family:'Nunito',sans-serif;font-size:12px;font-weight:600;outline:none}
        select:focus{border-color:${C.terracotta};box-shadow:0 0 0 3px ${C.terraBg}}
        ::-webkit-scrollbar{width:6px;height:6px}
        ::-webkit-scrollbar-track{background:${C.bgAlt}}
        ::-webkit-scrollbar-thumb{background:${C.border};border-radius:3px}
        @keyframes fadeUp{from{opacity:0;transform:translateY(12px)}to{opacity:1;transform:translateY(0)}}
      `}</style>

      {/* ═══ HEADER ═══ */}
      <div style={{background:`linear-gradient(180deg, ${C.bgAlt} 0%, ${C.bg} 100%)`,borderBottom:`1.5px solid ${C.border}`,padding:"28px 32px 0"}}>
        <div style={{display:"flex",alignItems:"baseline",gap:10,marginBottom:2}}>
          <h1 style={{fontFamily:"'Playfair Display',serif",fontSize:30,fontWeight:700,color:C.text,letterSpacing:-0.5}}>Budget Command Center</h1>
          <span style={{fontSize:14,color:C.textLight,fontStyle:"italic",fontFamily:"'Playfair Display',serif",fontWeight:400}}>2026</span>
        </div>
        <p style={{fontSize:11,color:C.textLight,letterSpacing:2,textTransform:"uppercase",marginBottom:20,fontWeight:600}}>
          Millionaire by 45 · Roth Max by May · HYSA $36K by Sept
        </p>
        <div style={{display:"flex",gap:2}}>
          {MONTHS.map((m,i)=>(
            <button key={m} onClick={()=>setMo(i)} style={{
              flex:1,padding:"10px 0",fontSize:12,fontWeight:mo===i?800:500,fontFamily:"'Nunito',sans-serif",
              color:mo===i?C.terracotta:C.textLight,background:mo===i?C.bgCard:"transparent",
              border:"none",borderBottom:mo===i?`3px solid ${C.terracotta}`:"3px solid transparent",
              cursor:"pointer",borderRadius:"10px 10px 0 0",letterSpacing:0.5,transition:"all 0.2s"
            }}>{m}</button>
          ))}
        </div>
      </div>

      {/* ═══ TABS ═══ */}
      <div style={{display:"flex",gap:0,padding:"0 32px",background:C.bgCard,borderBottom:`1.5px solid ${C.border}`,overflowX:"auto"}}>
        {tabDef.map(t=>(
          <button key={t.id} onClick={()=>setTab(t.id)} style={{
            padding:"12px 20px",fontSize:11,fontWeight:tab===t.id?800:500,fontFamily:"'Nunito',sans-serif",
            color:tab===t.id?C.terracotta:C.textLight,background:"transparent",border:"none",
            borderBottom:tab===t.id?`3px solid ${C.terracotta}`:"3px solid transparent",
            cursor:"pointer",letterSpacing:1,textTransform:"uppercase",transition:"all 0.2s"
          }}>{t.label}</button>
        ))}
      </div>

      <div style={{padding:"24px 32px",maxWidth:1120}}>

        {/* ═══ OVERVIEW ═══ */}
        {tab==="overview"&&(<div style={{animation:"fadeUp 0.4s ease"}}>
          <div style={{display:"flex",gap:12,marginBottom:22,flexWrap:"wrap"}}>
            <Field label="IMSLP (1099)" value={pays[mo].imslp} onChange={v=>updatePay("imslp",v)} />
            <Field label="Paycheck 1 (W-2)" value={pays[mo].paycheck1} onChange={v=>updatePay("paycheck1",v)} />
            <Field label="Paycheck 2 (W-2)" value={pays[mo].paycheck2} onChange={v=>updatePay("paycheck2",v)} />
            <div style={{flex:1,minWidth:130,display:"flex",alignItems:"flex-end"}}>
              <div style={{background:C.terraBg,border:`1.5px solid ${C.terraBorder}`,borderRadius:10,padding:"9px 14px",width:"100%"}}>
                <div style={{fontSize:10,color:C.textLight,letterSpacing:1,textTransform:"uppercase",fontWeight:600}}>Total Gross</div>
                <div style={{fontSize:18,fontWeight:800,color:C.terracotta}}>{fmt(cur.pay.imslp+cur.pay.paycheck1+cur.pay.paycheck2)}</div>
              </div>
            </div>
          </div>

          <div style={{display:"grid",gridTemplateColumns:"repeat(auto-fit,minmax(210px,1fr))",gap:14,marginBottom:22}}>
            <GoalCard title="Roth IRA 2026" current={cur.roth.ytd} target={goals.rothLimit} color={C.plum} colorBg={C.plumBg} colorBorder={C.plumBorder} icon="✦" deadline="Max by May" />
            <GoalCard title="HYSA" current={cur.hysa.end} target={goals.hysaTarget} color={C.sage} colorBg={C.sageBg} colorBorder={C.sageBorder} icon="🌿" deadline="$36K by Sep" />
            <GoalCard title="Travel Fund" current={cur.travel.end} target={goals.travelTarget} color={C.gold} colorBg={C.goldBg} colorBorder={C.goldBorder} icon="☀" deadline="$2K by Aug" />
            <GoalCard title="Brokerage" current={cur.brok.total} target={goals.brokerageMonthly} color={C.blue} colorBg={C.blueBg} colorBorder={C.blueBorder} icon="◈" deadline="$1,500/mo" />
          </div>

          <div style={{...card,background:`linear-gradient(135deg, ${C.bgCard}, ${C.bgWarm})`,marginBottom:18}}>
            <div style={secT}>{MONTHS[mo]} 2026</div>
            <div style={{display:"grid",gridTemplateColumns:"repeat(auto-fit,minmax(115px,1fr))",gap:10}}>
              {[
                {l:"1099 Taxes",v:cur.tax,c:C.rose,bg:C.roseBg},
                {l:"Rent",v:cur.rent.total,c:C.textMed,bg:C.bgAlt},
                {l:"JHU Loan",v:jhu*3,c:C.textMed,bg:C.bgAlt},
                {l:"Roth IRA",v:cur.roth.total,c:C.plum,bg:C.plumBg},
                {l:"Brokerage",v:cur.brok.total,c:C.blue,bg:C.blueBg},
                {l:"HYSA",v:cur.hysa.total,c:C.sage,bg:C.sageBg},
                {l:"Travel",v:cur.travel.total,c:C.gold,bg:C.goldBg},
                {l:"Credit Cards",v:cur.cc.total,c:C.terracotta,bg:C.terraBg},
                {l:"Surplus",v:cur.surplus,c:cur.surplus>=0?C.green:C.red,bg:cur.surplus>=0?C.sageBg:C.roseBg},
              ].map((x,i)=>(
                <div key={i} style={{background:x.bg,borderRadius:12,padding:"10px 12px",border:`1px solid ${C.borderLight}`}}>
                  <div style={{fontSize:9,color:C.textLight,textTransform:"uppercase",letterSpacing:1.2,marginBottom:3,fontWeight:600}}>{x.l}</div>
                  <div style={{fontSize:16,fontWeight:800,color:x.c}}>{fmt(x.v)}</div>
                </div>
              ))}
            </div>
          </div>

          {cur.surplus>0&&<div style={{background:C.sageBg,border:`1.5px solid ${C.sageBorder}`,borderRadius:14,padding:"14px 18px"}}>
            <div style={{fontFamily:"'Playfair Display',serif",fontSize:14,color:C.sage,marginBottom:5,fontWeight:700}}>Surplus → {fmt(cur.surplus)}</div>
            <div style={{fontSize:12,color:C.textMed,fontWeight:500}}>{cur.hysa.end<goals.hysaTarget?"→ Route to HYSA to accelerate $36K target":cur.travel.end<goals.travelTarget&&mo<=7?"→ Route to Travel Fund":"→ Route to Extra Brokerage (FXAIX 60% / FSPSX 30% / FSELX 10%)"}</div>
          </div>}
          {cur.surplus<0&&<div style={{background:C.roseBg,border:`1.5px solid ${C.roseBorder}`,borderRadius:14,padding:"14px 18px"}}>
            <div style={{fontFamily:"'Playfair Display',serif",fontSize:14,color:C.red,marginBottom:5,fontWeight:700}}>⚠ Deficit — {fmt(cur.surplus)}</div>
            <div style={{fontSize:12,color:C.textMed,fontWeight:500}}>Expenses exceed income. Adjust paycheck amounts or reduce spending.</div>
          </div>}
        </div>)}

        {/* ═══ PAYCHECKS ═══ */}
        {tab==="paychecks"&&(<div style={{animation:"fadeUp 0.4s ease"}}>
          <div style={secT}>{MONTHS[mo]} — Per-Paycheck Allocation</div>
          <div style={{display:"flex",gap:14,flexWrap:"wrap",marginBottom:18}}>
            <CheckCard label="IMSLP" data={cur.checks.imslp} showTax showRent showCC={false} accent={C.terracotta} />
            <CheckCard label="Paycheck 1" data={cur.checks.p1} showTax={false} showRent showCC={false} accent={C.sage} />
            <CheckCard label="Paycheck 2" data={cur.checks.p2} showTax={false} showRent={false} showCC accent={C.blue} />
          </div>
          <div style={card}>
            <div style={subT}>Allocation Waterfall</div>
            <div style={{fontSize:12,color:C.textMed,lineHeight:2,fontWeight:500}}>
              <strong style={{color:C.terracotta}}>IMSLP:</strong> Tax ({(taxRate*100).toFixed(0)}%) → ¼ Rent → ${jhu} JHU → Roth → Brokerage → HYSA → Travel → Surplus<br/>
              <strong style={{color:C.sage}}>Paycheck 1:</strong> ¾ Rent → ${jhu} JHU → Roth → Brokerage → HYSA → Travel → Surplus<br/>
              <strong style={{color:C.blue}}>Paycheck 2:</strong> ${jhu} JHU → All CC → Roth → Brokerage → HYSA → Travel → Surplus
            </div>
          </div>
        </div>)}

        {/* ═══ INVESTMENTS ═══ */}
        {tab==="investments"&&(<div style={{animation:"fadeUp 0.4s ease"}}>
          <div style={secT}>{MONTHS[mo]} — Investments</div>
          <div style={{display:"flex",gap:14,flexWrap:"wrap",marginBottom:18}}>
            <div style={{...card,flex:1,minWidth:230,borderColor:C.plumBorder}}>
              <div style={{fontFamily:"'Playfair Display',serif",fontSize:15,color:C.plum,marginBottom:10,fontWeight:700}}>Roth IRA — {fmt(cur.roth.total)}/mo</div>
              <div style={{display:"flex",justifyContent:"space-between",fontSize:13,fontWeight:600}}><span style={{color:C.textMed}}>FXAIX (100%)</span><span style={{color:C.plum}}>{fmt(cur.roth.total)}</span></div>
              <div style={{fontSize:11,color:C.textLight,marginTop:8,fontWeight:600}}>YTD: {fmtInt(cur.roth.ytd)} / {fmtInt(goals.rothLimit)}</div>
            </div>
            <div style={{...card,flex:1,minWidth:230,borderColor:C.blueBorder}}>
              <div style={{fontFamily:"'Playfair Display',serif",fontSize:15,color:C.blue,marginBottom:10,fontWeight:700}}>Brokerage — {fmt(cur.brok.total)}/mo</div>
              {Object.entries(brokA).map(([f,p])=>(
                <div key={f} style={{display:"flex",justifyContent:"space-between",fontSize:13,padding:"3px 0",fontWeight:600}}>
                  <span style={{color:C.textMed}}>{f} ({p}%)</span><span style={{color:C.blue}}>{fmt(cur.brok.total*p/100)}</span>
                </div>
              ))}
            </div>
          </div>
          <div style={{marginBottom:14}}>
            <div style={{display:"flex",justifyContent:"space-between",fontSize:11,color:C.textLight,marginBottom:4,fontWeight:600}}>
              <span>2026 Roth Contributions</span>
              <span>{fmtInt(cur.roth.ytd)} / {fmtInt(goals.rothLimit)} ({(Math.min(100,cur.roth.ytd/goals.rothLimit*100)).toFixed(1)}%)</span>
            </div>
            <div style={{height:10,background:C.plumBg,borderRadius:5,overflow:"hidden",border:`1px solid ${C.plumBorder}`}}>
              <div style={{height:"100%",width:`${Math.min(100,cur.roth.ytd/goals.rothLimit*100)}%`,background:`linear-gradient(90deg,${C.plum},${C.plum}bb)`,borderRadius:5,transition:"width 0.7s ease"}} />
            </div>
          </div>
          <div style={card}>
            <div style={subT}>Fund Lineup — Locked</div>
            <div style={{fontSize:12,color:C.textMed,lineHeight:2,fontWeight:500}}>
              <strong style={{color:C.plum}}>Roth IRA:</strong> 100% FXAIX (S&P 500)<br/>
              <strong style={{color:C.blue}}>Brokerage:</strong> 60% FXAIX · 30% FSPSX (Small-Cap) · 10% FSELX (Semiconductors)<br/>
              <strong style={{color:C.textLight}}>401k (Voya):</strong> 6% salary, 3% match — managed separately
            </div>
          </div>
        </div>)}

        {/* ═══ EXPENSES ═══ */}
        {tab==="expenses"&&(<div style={{animation:"fadeUp 0.4s ease"}}>
          <div style={secT}>Monthly Expenses</div>
          <div style={{...card,marginBottom:16}}>
            <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:12}}>
              <span style={subT}>Fixed / Variable</span>
              <button onClick={()=>setExps(p=>[...p,{id:Date.now(),name:"New Expense",amount:0,cc:"Regular"}])} style={addBtn}>+ Add</button>
            </div>
            {exps.map(e=>(
              <div key={e.id} style={{display:"flex",gap:8,alignItems:"center",marginBottom:8,flexWrap:"wrap"}}>
                <input value={e.name} onChange={ev=>updateExp(e.id,"name",ev.target.value)} style={{...inSm,flex:2,minWidth:120}} />
                <div style={{flex:1,minWidth:85,position:"relative"}}>
                  <span style={{position:"absolute",left:10,top:"50%",transform:"translateY(-50%)",fontSize:11,color:C.textFaint,fontWeight:600}}>$</span>
                  <input type="number" value={e.amount} onChange={ev=>updateExp(e.id,"amount",parseFloat(ev.target.value)||0)} style={{...inSm,paddingLeft:22}} />
                </div>
                <select value={e.cc} onChange={ev=>updateExp(e.id,"cc",ev.target.value)} style={{width:95}}>
                  <option value="AMEX">AMEX</option><option value="Regular">Regular</option>
                </select>
                <button onClick={()=>setExps(p=>p.filter(x=>x.id!==e.id))} style={rmBtn}>✕</button>
              </div>
            ))}
          </div>
          <div style={{...card,marginBottom:16}}>
            <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:12}}>
              <span style={subT}>Subscriptions</span>
              <button onClick={()=>setSubs(p=>[...p,{id:Date.now(),name:"New Sub",amount:0,cc:"Regular"}])} style={addBtn}>+ Add</button>
            </div>
            {subs.map(s=>(
              <div key={s.id} style={{display:"flex",gap:8,alignItems:"center",marginBottom:8,flexWrap:"wrap"}}>
                <input value={s.name} onChange={ev=>updateSub(s.id,"name",ev.target.value)} style={{...inSm,flex:2,minWidth:120}} />
                <div style={{flex:1,minWidth:85,position:"relative"}}>
                  <span style={{position:"absolute",left:10,top:"50%",transform:"translateY(-50%)",fontSize:11,color:C.textFaint,fontWeight:600}}>$</span>
                  <input type="number" value={s.amount} onChange={ev=>updateSub(s.id,"amount",parseFloat(ev.target.value)||0)} style={{...inSm,paddingLeft:22}} />
                </div>
                <select value={s.cc} onChange={ev=>updateSub(s.id,"cc",ev.target.value)} style={{width:95}}>
                  <option value="AMEX">AMEX</option><option value="Regular">Regular</option>
                </select>
                <button onClick={()=>setSubs(p=>p.filter(x=>x.id!==s.id))} style={rmBtn}>✕</button>
              </div>
            ))}
          </div>
          <div style={{...card,display:"flex",justifyContent:"space-around",flexWrap:"wrap",gap:16,background:`linear-gradient(135deg,${C.bgCard},${C.bgWarm})`}}>
            {[{l:"AMEX",v:cur.cc.amex,c:C.gold},{l:"Regular CC",v:cur.cc.regular,c:C.blue},{l:"All Cards",v:cur.cc.total,c:C.terracotta}].map((x,i)=>(
              <div key={i} style={{textAlign:"center"}}>
                <div style={{fontSize:10,color:C.textLight,textTransform:"uppercase",letterSpacing:1.2,fontWeight:600}}>{x.l}</div>
                <div style={{fontSize:22,fontWeight:800,color:x.c,marginTop:2}}>{fmt(x.v)}</div>
              </div>
            ))}
          </div>
        </div>)}

        {/* ═══ SETTINGS ═══ */}
        {tab==="settings"&&(<div style={{animation:"fadeUp 0.4s ease"}}>
          <div style={secT}>Settings</div>
          <div style={{...card,marginBottom:16}}>
            <div style={subT}>Starting Balances (Jan 2026)</div>
            <div style={{display:"flex",gap:12,flexWrap:"wrap"}}>
              <Field label="HYSA Balance" value={bal.hysaStart} onChange={v=>setBal(p=>({...p,hysaStart:v}))} />
              <Field label="Travel Fund" value={bal.travelStart} onChange={v=>setBal(p=>({...p,travelStart:v}))} />
              <Field label="Roth YTD Contributed" value={bal.rothContributed} onChange={v=>setBal(p=>({...p,rothContributed:v}))} />
            </div>
          </div>
          <div style={{...card,marginBottom:16}}>
            <div style={subT}>Goal Targets</div>
            <div style={{display:"flex",gap:12,flexWrap:"wrap"}}>
              <Field label="Roth 2026 Limit" value={goals.rothLimit} onChange={v=>setGoals(p=>({...p,rothLimit:v}))} />
              <Field label="HYSA Target" value={goals.hysaTarget} onChange={v=>setGoals(p=>({...p,hysaTarget:v}))} />
              <Field label="Travel Target" value={goals.travelTarget} onChange={v=>setGoals(p=>({...p,travelTarget:v}))} />
              <Field label="Brokerage /month" value={goals.brokerageMonthly} onChange={v=>setGoals(p=>({...p,brokerageMonthly:v}))} />
            </div>
          </div>
          <div style={{...card,marginBottom:16}}>
            <div style={subT}>Long-Term Targets (Age 50)</div>
            <div style={{display:"flex",gap:12,flexWrap:"wrap"}}>
              <Field label="Brokerage Target" value={goals.brokerageAge50} onChange={v=>setGoals(p=>({...p,brokerageAge50:v}))} />
              <Field label="Roth IRA Target" value={goals.rothAge50} onChange={v=>setGoals(p=>({...p,rothAge50:v}))} />
            </div>
          </div>
          <div style={{...card,marginBottom:16}}>
            <div style={subT}>Rent & Fixed Obligations</div>
            <div style={{display:"flex",gap:12,flexWrap:"wrap"}}>
              <Field label="Total Rent" value={rent.total} onChange={v=>setRent(p=>({...p,total:v}))} />
              <Field label="My Share" value={rent.myShare} onChange={v=>setRent(p=>({...p,myShare:v}))} />
              <Field label="JHU per Paycheck" value={jhu} onChange={setJhu} />
              <Field label="1099 Tax Rate" value={Math.round(taxRate*100)} onChange={v=>setTaxRate(v/100)} prefix="" suffix="%" />
            </div>
          </div>
          <div style={{...card,marginBottom:16}}>
            <div style={subT}>Brokerage Fund Allocation (%)</div>
            <div style={{display:"flex",gap:12,flexWrap:"wrap"}}>
              {Object.entries(brokA).map(([f,p])=>(
                <Field key={f} label={f} value={p} onChange={v=>setBrokA(pr=>({...pr,[f]:v}))} prefix="" suffix="%" />
              ))}
            </div>
            {(()=>{const t=Object.values(brokA).reduce((s,v)=>s+v,0);return t!==100
              ?<div style={{fontSize:11,color:C.red,marginTop:8,fontWeight:700}}>⚠ Sum is {t}% — should be 100%</div>
              :<div style={{fontSize:11,color:C.sage,marginTop:8,fontWeight:700}}>✓ Allocations = 100%</div>;})()}
          </div>
          <div style={card}>
            <div style={subT}>Bulk Paycheck Edit</div>
            <div style={{fontSize:12,color:C.textLight,marginBottom:12,fontWeight:500}}>Edit January values, then apply to all 12 months.</div>
            <div style={{display:"flex",gap:12,flexWrap:"wrap",alignItems:"flex-end"}}>
              <Field label="IMSLP" value={pays[0].imslp} onChange={v=>setPays(p=>{const n=[...p];n[0]={...n[0],imslp:v};return n;})} />
              <Field label="Paycheck 1" value={pays[0].paycheck1} onChange={v=>setPays(p=>{const n=[...p];n[0]={...n[0],paycheck1:v};return n;})} />
              <Field label="Paycheck 2" value={pays[0].paycheck2} onChange={v=>setPays(p=>{const n=[...p];n[0]={...n[0],paycheck2:v};return n;})} />
              <button onClick={()=>setPays(Array.from({length:12},()=>({...pays[0]})))} style={{...addBtn,height:38,fontSize:12}}>Apply to All</button>
            </div>
          </div>
        </div>)}

        {/* ═══ 12-MONTH ═══ */}
        {tab==="yearly"&&(<div style={{animation:"fadeUp 0.4s ease"}}>
          <div style={secT}>2026 — Full Year</div>
          <div style={{overflowX:"auto",marginBottom:22,...card,padding:0}}>
            <table style={{width:"100%",borderCollapse:"collapse",fontSize:12,fontFamily:"'Nunito',sans-serif"}}>
              <thead><tr style={{borderBottom:`2px solid ${C.border}`}}>
                {["Mo","Gross","Tax","Rent","JHU","Roth","Brok","HYSA","Travel","CC","Surplus"].map(h=>
                  <th key={h} style={{padding:"12px 8px",textAlign:"right",color:C.textLight,fontWeight:700,fontSize:10,textTransform:"uppercase",letterSpacing:1.2}}>{h}</th>
                )}
              </tr></thead>
              <tbody>{year.map((m,i)=>{
                const g=m.pay.imslp+m.pay.paycheck1+m.pay.paycheck2;
                return <tr key={i} onClick={()=>{setMo(i);setTab("overview");}} style={{borderBottom:`1px solid ${C.borderLight}`,cursor:"pointer",background:i===mo?C.bgAlt:"transparent",transition:"background 0.15s"}}>
                  <td style={{padding:"10px 8px",textAlign:"right",color:C.text,fontWeight:700}}>{m.month}</td>
                  <td style={{padding:"10px 8px",textAlign:"right",color:C.text,fontWeight:600}}>{fmtInt(g)}</td>
                  <td style={{padding:"10px 8px",textAlign:"right",color:C.rose,fontWeight:600}}>{fmtInt(m.tax)}</td>
                  <td style={{padding:"10px 8px",textAlign:"right",color:C.textMed,fontWeight:600}}>{fmtInt(m.rent.total)}</td>
                  <td style={{padding:"10px 8px",textAlign:"right",color:C.textMed,fontWeight:600}}>{fmtInt(jhu*3)}</td>
                  <td style={{padding:"10px 8px",textAlign:"right",color:C.plum,fontWeight:600}}>{fmtInt(m.roth.total)}</td>
                  <td style={{padding:"10px 8px",textAlign:"right",color:C.blue,fontWeight:600}}>{fmtInt(m.brok.total)}</td>
                  <td style={{padding:"10px 8px",textAlign:"right",color:C.sage,fontWeight:600}}>{fmtInt(m.hysa.total)}</td>
                  <td style={{padding:"10px 8px",textAlign:"right",color:C.gold,fontWeight:600}}>{fmtInt(m.travel.total)}</td>
                  <td style={{padding:"10px 8px",textAlign:"right",color:C.terracotta,fontWeight:600}}>{fmtInt(m.cc.total)}</td>
                  <td style={{padding:"10px 8px",textAlign:"right",color:m.surplus>=0?C.green:C.red,fontWeight:800}}>{fmtInt(m.surplus)}</td>
                </tr>;
              })}</tbody>
            </table>
          </div>
          <div style={{display:"grid",gridTemplateColumns:"repeat(auto-fit,minmax(260px,1fr))",gap:14}}>
            <div style={{...card,borderColor:C.sageBorder}}>
              <div style={{fontSize:13,color:C.sage,fontWeight:800,marginBottom:8}}>HYSA Balance</div>
              {year.map((m,i)=><div key={i} style={{display:"flex",justifyContent:"space-between",padding:"4px 0",fontSize:12,fontWeight:600,color:m.hysa.end>=goals.hysaTarget?C.sage:C.textLight}}>
                <span>{m.month}</span><span>{fmtInt(m.hysa.end)}{m.hysa.end>=goals.hysaTarget?" ✓":""}</span>
              </div>)}
            </div>
            <div style={{...card,borderColor:C.goldBorder}}>
              <div style={{fontSize:13,color:C.gold,fontWeight:800,marginBottom:8}}>Travel Fund</div>
              {year.filter((_,i)=>i<=7).map((m,i)=><div key={i} style={{display:"flex",justifyContent:"space-between",padding:"4px 0",fontSize:12,fontWeight:600,color:m.travel.end>=goals.travelTarget?C.gold:C.textLight}}>
                <span>{m.month}</span><span>{fmtInt(m.travel.end)}{m.travel.end>=goals.travelTarget?" ✓":""}</span>
              </div>)}
            </div>
            <div style={{...card,borderColor:C.plumBorder}}>
              <div style={{fontSize:13,color:C.plum,fontWeight:800,marginBottom:8}}>Roth IRA YTD</div>
              {year.filter((_,i)=>i<=4).map((m,i)=><div key={i} style={{display:"flex",justifyContent:"space-between",padding:"4px 0",fontSize:12,fontWeight:600,color:m.roth.ytd>=goals.rothLimit?C.plum:C.textLight}}>
                <span>{m.month}</span><span>{fmtInt(m.roth.ytd)} / {fmtInt(goals.rothLimit)}{m.roth.ytd>=goals.rothLimit?" ✓":""}</span>
              </div>)}
            </div>
          </div>
        </div>)}

      </div>
    </div>
  );
}
