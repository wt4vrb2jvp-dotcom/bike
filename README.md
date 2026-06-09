# bike
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8"/>
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0"/>
  <meta name="apple-mobile-web-app-capable" content="yes"/>
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent"/>
  <title>🚲 Bike Flip</title>
  <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
  <style>
    html,body{margin:0;padding:0;background:#0d0f14;-webkit-tap-highlight-color:transparent;}
    *{-webkit-text-size-adjust:100%;}
  </style>
</head>
<body>
<div id="root"></div>
<script type="text/babel">
const { useState, useEffect, useMemo } = React;


// ─── КОНСТАНТЫ ───────────────────────────────────────────────
const BIKE_CATS    = ["Велосипед","Запчасти","Дорога (топливо/платные)","Реклама","Инструменты","Транспорт","Прочее"];
const EXPENSE_CATS = ["Реклама","Запчасти","Дорога (топливо/платные)","Транспорт","Инструменты","Прочее"];
const today = () => new Date().toISOString().slice(0,10);
const fmt = (n, sign=false) => {
  const s = Math.abs(n).toLocaleString("ru-RU",{minimumFractionDigits:0,maximumFractionDigits:0});
  if (sign) return (n>=0?"+":"−")+s+" €";
  return s+" €";
};
const totalCost = b => b.costs.reduce((s,c)=>s+c.amount,0);
const bikeProfit = b => b.salePrice!=null ? b.salePrice-totalCost(b) : null;

const SPLIT = [
  {label:"Реинвест в товар",pct:60,color:"#60a5fa"},
  {label:"Резерв / запчасти",pct:20,color:"#fbbf24"},
  {label:"Себе",pct:20,color:"#4ade80"},
];
const TXN_META = {
  sale:       {icon:"▲",color:"#4ade80",bg:"#0a1a0f"},
  purchase:   {icon:"🚲",color:"#fbbf24",bg:"#1a1600"},
  expense:    {icon:"▼",color:"#f87171",bg:"#1a0a0a"},
  withdrawal: {icon:"💸",color:"#c084fc",bg:"#130a1a"},
  debt:       {icon:"🔴",color:"#fb923c",bg:"#1a0f00"},
};

// ─── НАЧАЛЬНЫЕ ДАННЫЕ ─────────────────────────────────────────
const INIT = {
  cash: { paypal: 0, cash: 100 },
  debts: [{ id:1, label:"Займ на KTM + Cube", amount:500, date:"2026-06-06" }],
  parts: [{ id:1, name:"Батарея Bosch", cost:150, date:"2026-06-05", note:"" }],
  bikes: [
    { id:1, name:"Cube", status:"stock",
      costs:[{label:"Велосипед",amount:800,date:"2026-06-01"},{label:"Дорога (топливо/платные)",amount:50,date:"2026-06-01"},{label:"Дорога (топливо/платные)",amount:30,date:"2026-06-01"},{label:"Запчасти",amount:20,date:"2026-06-09"}],
      salePrice:null, soldDate:null },
    { id:2, name:"Карбон (черный)", status:"stock",
      costs:[{label:"Велосипед",amount:800,date:"2026-06-02"},{label:"Дорога (топливо/платные)",amount:50,date:"2026-06-02"}],
      salePrice:null, soldDate:null },
    { id:3, name:"Focus e-bike", status:"stock",
      costs:[{label:"Велосипед",amount:1300,date:"2026-06-03"},{label:"Дорога (топливо/платные)",amount:75,date:"2026-06-03"}],
      salePrice:null, soldDate:null },
    { id:4, name:"Cannondale", status:"stock",
      costs:[{label:"Велосипед",amount:1350,date:"2026-06-03"},{label:"Дорога (топливо/платные)",amount:75,date:"2026-06-03"}],
      salePrice:null, soldDate:null },
    { id:5, name:"KTM", status:"stock",
      costs:[{label:"Велосипед",amount:1200,date:"2026-06-05"},{label:"Дорога (топливо/платные)",amount:50,date:"2026-06-05"}],
      salePrice:null, soldDate:null },
    { id:6, name:"Flyer", status:"stock",
      costs:[{label:"Велосипед",amount:450,date:"2026-05-20"},{label:"Мотор",amount:150,date:"2026-05-20"},{label:"Дисплей",amount:150,date:"2026-05-20"}],
      salePrice:null, soldDate:null },
    { id:7, name:"Scott", status:"sold",
      costs:[{label:"Велосипед",amount:700,date:"2026-05-15"},{label:"Зарядка",amount:50,date:"2026-05-15"},{label:"Ключ",amount:20,date:"2026-05-15"}],
      salePrice:1850, soldDate:"2026-05-21" },
    { id:8, name:"Bianchi XL", status:"sold",
      costs:[{label:"Велосипед",amount:1300,date:"2026-05-10"}],
      salePrice:1700, soldDate:"2026-06-06" },
  ],
  txns: [
    {id:1,date:"2026-05-15",type:"purchase",category:"Закупка",desc:"Scott — закупка",amount:-770,account:"cash"},
    {id:2,date:"2026-05-20",type:"purchase",category:"Закупка",desc:"Flyer — закупка",amount:-750,account:"cash"},
    {id:3,date:"2026-05-21",type:"sale",category:"Продажа",desc:"Продажа Scott",amount:1850,account:"cash"},
    {id:4,date:"2026-05-21",type:"expense",category:"Реклама",desc:"Реклама",amount:-8,account:"paypal"},
    {id:5,date:"2026-06-01",type:"purchase",category:"Закупка",desc:"Cube — закупка",amount:-880,account:"cash"},
    {id:6,date:"2026-06-02",type:"purchase",category:"Закупка",desc:"Карбон — закупка",amount:-850,account:"cash"},
    {id:7,date:"2026-06-03",type:"purchase",category:"Закупка",desc:"Focus e-bike — закупка",amount:-1375,account:"cash"},
    {id:8,date:"2026-06-03",type:"purchase",category:"Закупка",desc:"Cannondale — закупка",amount:-1425,account:"cash"},
    {id:9,date:"2026-06-05",type:"purchase",category:"Закупка",desc:"KTM — закупка",amount:-1250,account:"cash"},
    {id:10,date:"2026-06-05",type:"purchase",category:"Запчасти",desc:"Батарея Bosch",amount:-150,account:"cash"},
    {id:11,date:"2026-06-05",type:"debt",category:"Займ",desc:"Займ на KTM + Cube",amount:2000,account:"cash"},
    {id:12,date:"2026-06-05",type:"withdrawal",category:"Личные расходы",desc:"Себе — мама массаж",amount:-150,account:"paypal"},
    {id:13,date:"2026-06-05",type:"withdrawal",category:"Личные расходы",desc:"Себе — авторемкомплект",amount:-80,account:"cash"},
    {id:14,date:"2026-06-06",type:"sale",category:"Продажа",desc:"Продажа Bianchi XL",amount:1700,account:"cash"},
    {id:16,date:"2026-06-08",type:"withdrawal",category:"Личные расходы",desc:"Себе — дорога, бензин",amount:-100,account:"cash"},
    {id:17,date:"2026-06-08",type:"withdrawal",category:"Личные расходы",desc:"Себе — автострада, магазин, дорога",amount:-50,account:"cash"},
    {id:18,date:"2026-06-09",type:"withdrawal",category:"Личные расходы",desc:"Себе — личный вывод",amount:-50,account:"cash"},
    {id:19,date:"2026-06-09",type:"expense",category:"Погашение долга",desc:"Погашение долга (частично)",amount:-1500,account:"cash"},
    {id:20,date:"2026-06-09",type:"expense",category:"Запчасти",desc:"Cube — замок",amount:-20,account:"cash"},
    {id:21,date:"2026-06-09",type:"withdrawal",category:"Личные расходы",desc:"Себе — Оле, личный резерв",amount:-50,account:"cash"},
    {id:15,date:"2026-05-10",type:"purchase",category:"Закупка",desc:"Bianchi XL — закупка",amount:-1300,account:"cash"},
  ],
  nextBikeId: 20,
  nextTxnId: 30,
  nextPartId: 10,
};

const STORAGE_KEY = "bikeflip_data";
const VERSION_KEY  = "bikeflip_version";
const CURRENT_VERSION = "v1.5"; // bump this every time INIT data is updated

function loadState() {
  try {
    const savedVersion = localStorage.getItem(VERSION_KEY);
    const savedData    = localStorage.getItem(STORAGE_KEY);
    // If version matches and data exists — use saved (user edits preserved)
    if (savedVersion === CURRENT_VERSION && savedData) {
      return JSON.parse(savedData);
    }
    // New version or first load — use fresh INIT data and save version
    localStorage.setItem(VERSION_KEY, CURRENT_VERSION);
    localStorage.setItem(STORAGE_KEY, JSON.stringify(INIT));
    return INIT;
  } catch { return INIT; }
}
function saveState(s) {
  try {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(s));
    localStorage.setItem(VERSION_KEY, CURRENT_VERSION);
  } catch {}
}

// ─── КОМПОНЕНТ ────────────────────────────────────────────────
function App() {
  const [st, setSt] = useState(loadState);

  const update = fn => setSt(prev => {
    const next = fn(prev);
    saveState(next);
    return next;
  });

  const { cash, bikes, txns, debts, parts } = st;

  const stockBikes  = bikes.filter(b=>b.status==="stock");
  const soldBikes   = bikes.filter(b=>b.status==="sold");
  const stockValue  = stockBikes.reduce((s,b)=>s+totalCost(b),0);
  const partsValue  = parts.reduce((s,p)=>s+p.cost,0);
  const totalMoney  = cash.paypal+cash.cash;
  const totalDebt   = debts.reduce((s,d)=>s+d.amount,0);
  const totalAssets = totalMoney+stockValue+partsValue;
  const netAssets   = totalAssets-totalDebt;
  const totalProfit = soldBikes.reduce((s,b)=>s+(bikeProfit(b)||0),0);
  const totalWithdrawn = txns.filter(t=>t.type==="withdrawal").reduce((s,t)=>s+Math.abs(t.amount),0);

  // Monthly analytics
  const MONTH_NAMES = ["Янв","Фев","Мар","Апр","Май","Июн","Июл","Авг","Сен","Окт","Ноя","Дек"];
  const monthlyStats = useMemo(() => {
    const map = {};
    txns.forEach(t => {
      const key = t.date.slice(0,7); // "2026-05"
      if (!map[key]) map[key] = { key, revenue:0, invested:0, expenses:0, withdrawn:0, sold:0, bought:0 };
      const m = map[key];
      if (t.type === "sale")       { m.revenue   += t.amount; m.sold++; }
      if (t.type === "purchase")   { m.invested  += Math.abs(t.amount); m.bought++; }
      if (t.type === "expense")    { m.expenses  += Math.abs(t.amount); }
      if (t.type === "withdrawal") { m.withdrawn += Math.abs(t.amount); }
    });
    return Object.values(map).sort((a,b)=>b.key.localeCompare(a.key)).map(m => ({
      ...m,
      profit: m.revenue - m.invested - m.expenses,
      label: (() => { const [y,mo] = m.key.split("-"); return MONTH_NAMES[parseInt(mo)-1]+" "+y; })()
    }));
  }, [txns]);

  const [view, setView] = useState("overview");
  const [logFilter, setLogFilter] = useState("all"); // all | cash | paypal | withdrawal
  const [selling, setSelling] = useState(null);
  const [saleInput, setSaleInput] = useState({price:"",method:"cash"});
  const [bikeCostModal, setBikeCostModal] = useState(null);
  const [bikeCostForm, setBikeCostForm] = useState({category:BIKE_CATS[0],amount:"",account:"cash",date:today()});
  const [addForm, setAddForm] = useState({name:"",costs:[{category:BIKE_CATS[0],amount:""}]});
  const [expForm, setExpForm] = useState({amount:"",category:EXPENSE_CATS[0],desc:"",account:"cash",date:today()});
  const [showExpForm, setShowExpForm] = useState(false);
  const [showWithdrawForm, setShowWithdrawForm] = useState(false);
  const [withdrawForm, setWithdrawForm] = useState({amount:"",account:"cash",desc:"",date:today()});
  const [showDebtForm, setShowDebtForm] = useState(false);
  const [debtForm, setDebtForm] = useState({label:"",amount:"",date:today()});
  const [showPartForm, setShowPartForm] = useState(false);
  const [partForm, setPartForm] = useState({name:"",cost:"",date:today(),note:""});

  function addTxn(state, txn) {
    return { ...state, txns:[{...txn,id:state.nextTxnId},...state.txns], nextTxnId:state.nextTxnId+1 };
  }
  function deductCash(state, amt, account) {
    const c = {...state.cash};
    if (account==="cash") c.cash -= amt; else c.paypal -= amt;
    return {...state, cash:c};
  }
  function addCash(state, amt, account) {
    const c = {...state.cash};
    if (account==="cash") c.cash += amt; else c.paypal += amt;
    return {...state, cash:c};
  }

  function handleAddExpense() {
    const amt = Number(expForm.amount);
    if (!amt||amt<=0) return;
    update(s => addTxn(deductCash(s,amt,expForm.account), {date:expForm.date,type:"expense",category:expForm.category,desc:expForm.desc||expForm.category,amount:-amt,account:expForm.account}));
    setExpForm({amount:"",category:EXPENSE_CATS[0],desc:"",account:"cash",date:today()});
    setShowExpForm(false);
  }

  function handleWithdraw() {
    const amt = Number(withdrawForm.amount);
    if (!amt||amt<=0) return;
    update(s => addTxn(deductCash(s,amt,withdrawForm.account), {date:withdrawForm.date,type:"withdrawal",category:"Личные расходы",desc:withdrawForm.desc?`Себе — ${withdrawForm.desc}`:"Себе (вывод)",amount:-amt,account:withdrawForm.account}));
    setWithdrawForm({amount:"",account:"cash",desc:"",date:today()});
    setShowWithdrawForm(false);
  }

  function handleAddDebt() {
    const amt = Number(debtForm.amount);
    if (!amt||amt<=0||!debtForm.label.trim()) return;
    update(s => {
      let s2 = {...s, debts:[...s.debts,{id:s.nextTxnId,label:debtForm.label,amount:amt,date:debtForm.date}], nextTxnId:s.nextTxnId+1};
      s2 = addCash(s2,amt,"cash");
      s2 = addTxn(s2,{date:debtForm.date,type:"debt",category:"Займ",desc:debtForm.label,amount:amt,account:"cash"});
      return s2;
    });
    setDebtForm({label:"",amount:"",date:today()});
    setShowDebtForm(false);
  }

  function handleRepayDebt(id) {
    update(s => {
      const debt = s.debts.find(d=>d.id===id);
      if (!debt) return s;
      let s2 = deductCash(s,debt.amount,"cash");
      s2 = addTxn(s2,{date:today(),type:"expense",category:"Погашение долга",desc:`Погашен: ${debt.label}`,amount:-debt.amount,account:"cash"});
      s2 = {...s2, debts:s2.debts.filter(d=>d.id!==id)};
      return s2;
    });
  }

  function handleAddPart() {
    const cost = Number(partForm.cost);
    if (!cost||cost<=0||!partForm.name.trim()) return;
    update(s => {
      let s2 = {...s, parts:[...s.parts,{id:s.nextPartId,name:partForm.name,cost,date:partForm.date,note:partForm.note}], nextPartId:s.nextPartId+1};
      s2 = deductCash(s2,cost,"cash");
      s2 = addTxn(s2,{date:partForm.date,type:"expense",category:"Запчасти",desc:partForm.name,amount:-cost,account:"cash"});
      return s2;
    });
    setPartForm({name:"",cost:"",date:today(),note:""});
    setShowPartForm(false);
  }

  function handleDeletePart(id) {
    update(s => ({...s, parts:s.parts.filter(p=>p.id!==id)}));
  }

  function handleAddBikeCost() {
    const amt = Number(bikeCostForm.amount);
    if (!amt||amt<=0) return;
    const bike = bikes.find(b=>b.id===bikeCostModal);
    update(s => {
      let s2 = deductCash(s,amt,bikeCostForm.account);
      s2 = {...s2, bikes:s2.bikes.map(b=>b.id===bikeCostModal?{...b,costs:[...b.costs,{label:bikeCostForm.category,amount:amt,date:bikeCostForm.date}]}:b)};
      s2 = addTxn(s2,{date:bikeCostForm.date,type:"purchase",category:bikeCostForm.category,desc:`${bike?.name} — ${bikeCostForm.category}`,amount:-amt,account:bikeCostForm.account});
      return s2;
    });
    setBikeCostForm({category:BIKE_CATS[0],amount:"",account:"cash",date:today()});
    setBikeCostModal(null);
  }

  function handleAddBike() {
    if (!addForm.name.trim()) return;
    const costs = addForm.costs.filter(c=>c.amount!==""&&Number(c.amount)>0).map(c=>({label:c.category,amount:Number(c.amount),date:today()}));
    if (costs.length===0) return;
    const total = costs.reduce((s,c)=>s+c.amount,0);
    update(s => {
      let s2 = deductCash(s,total,"cash");
      s2 = {...s2, bikes:[...s2.bikes,{id:s2.nextBikeId,name:addForm.name,status:"stock",costs,salePrice:null,soldDate:null}], nextBikeId:s2.nextBikeId+1};
      s2 = addTxn(s2,{date:today(),type:"purchase",category:"Закупка",desc:`${addForm.name} — ${costs.map(c=>c.label).join(", ")}`,amount:-total,account:"cash"});
      return s2;
    });
    setAddForm({name:"",costs:[{category:BIKE_CATS[0],amount:""}]});
    setView("bikes");
  }

  function handleSell() {
    const price = Number(saleInput.price);
    if (!price||price<=0) return;
    const bike = bikes.find(b=>b.id===selling);
    update(s => {
      let s2 = addCash(s,price,saleInput.method);
      s2 = {...s2, bikes:s2.bikes.map(b=>b.id===selling?{...b,status:"sold",salePrice:price,soldDate:today()}:b)};
      s2 = addTxn(s2,{date:today(),type:"sale",category:"Продажа",desc:`Продажа ${bike?.name||""}`,amount:price,account:saleInput.method});
      return s2;
    });
    setSelling(null); setSaleInput({price:"",method:"cash"});
  }

  function handleDeleteBike(id) {
    const bike = bikes.find(b=>b.id===id);
    update(s => {
      let s2 = s;
      if (bike?.status==="stock") s2 = addCash(s2,totalCost(bike),"cash");
      return {...s2, bikes:s2.bikes.filter(b=>b.id!==id)};
    });
  }

  const AccountToggle = ({value,onChange}) => (
    <div style={{display:"flex",gap:8}}>
      {[["cash","Наличные"],["paypal","PayPal"]].map(([v,l])=>(
        <button key={v} onClick={()=>onChange(v)} style={{flex:1,padding:"9px",borderRadius:7,border:"1px solid",borderColor:value===v?"#4a6fa5":"#2a2d38",background:value===v?"#1a2a40":"transparent",color:value===v?"#7eb3ff":"#555",cursor:"pointer",fontFamily:"inherit",fontSize:12}}>{l}</button>
      ))}
    </div>
  );

  function goLog(filter) { setLogFilter(filter); setView("log"); }

  const NAV = [["overview","Баланс"],["bikes","Склад"],["add","+ Вел"],["parts","Запчасти"],["log","Журнал"],["stats","📊"],["split","Распред."]];

  return (
    <div style={{minHeight:"100vh",background:"#0d0f14",fontFamily:"'Space Mono','Courier New',monospace",color:"#ddd8cc"}}>
      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&family=Syne:wght@700;800&display=swap');
        *{box-sizing:border-box;margin:0;padding:0}
        ::-webkit-scrollbar{width:4px}::-webkit-scrollbar-thumb{background:#2a2d38;border-radius:2px}
        .card{background:#13161e;border:1px solid #222530;border-radius:10px;padding:18px}
        .nav-btn{background:none;border:none;cursor:pointer;padding:7px 9px;font-family:inherit;font-size:9px;letter-spacing:.1em;text-transform:uppercase;border-radius:6px;color:#555;transition:all .15s}
        .nav-btn:hover{color:#999;background:rgba(255,255,255,.04)}
        .nav-btn.active{background:#1e2130;color:#ddd8cc}
        .inp{background:#1a1d26;border:1px solid #2a2d38;border-radius:7px;color:#ddd8cc;font-family:inherit;font-size:13px;padding:10px 12px;width:100%;outline:none;transition:border-color .15s}
        .inp:focus{border-color:#4a6fa5}
        .btn{border:none;border-radius:7px;cursor:pointer;font-family:inherit;font-size:12px;font-weight:700;letter-spacing:.06em;padding:10px 16px;transition:opacity .15s}
        .btn:hover{opacity:.82}
        .btn-primary{background:#2a3f6f;color:#7eb3ff}
        .btn-green{background:#1a3328;color:#4ade80}
        .btn-yellow{background:#2a2010;color:#fbbf24;border:none}
        .btn-red{background:none;border:1px solid #3a2020;color:#f87171}
        .btn-ghost{background:#1a1d26;border:none;color:#666}
        .btn-purple{background:#1a1028;border:none;color:#c084fc}
        .btn-orange{background:#1a0f00;border:none;color:#fb923c}
        label{font-size:10px;letter-spacing:.1em;color:#555;text-transform:uppercase;display:block;margin-bottom:6px}
        .modal-bg{position:fixed;inset:0;background:rgba(0,0,0,.75);display:flex;align-items:center;justify-content:center;z-index:100;padding:16px}
        .modal{background:#13161e;border:1px solid #2a2d38;border-radius:12px;padding:22px;width:100%;max-width:380px;max-height:90vh;overflow-y:auto}
        .row{display:flex;align-items:center;gap:12px;padding:11px 0;border-bottom:1px solid #1a1d26}
        .row:last-child{border-bottom:none}
        .cost-row{display:flex;justify-content:space-between;font-size:11px;padding:3px 0;color:#888}
      `}</style>

      {/* Header */}
      <div style={{borderBottom:"1px solid #1a1d26",padding:"0 10px",position:"sticky",top:0,background:"#0d0f14",zIndex:10}}>
        <div style={{maxWidth:600,margin:"0 auto",display:"flex",alignItems:"center",justifyContent:"space-between",height:50}}>
          <span style={{fontFamily:"'Syne',sans-serif",fontWeight:800,fontSize:13,color:"#7eb3ff"}}>🚲 BIKE FLIP</span>
          <div style={{display:"flex",gap:1,flexWrap:"wrap"}}>
            {NAV.map(([v,l])=><button key={v} className={`nav-btn${view===v?" active":""}`} onClick={()=>setView(v)}>{l}</button>)}
          </div>
        </div>
      </div>

      <div style={{maxWidth:600,margin:"0 auto",padding:"18px 12px",display:"flex",flexDirection:"column",gap:12}}>

        {/* ── БАЛАНС ── */}
        {view==="overview" && <>
          <div className="card">
            <div style={{fontSize:10,color:"#444",letterSpacing:".12em",textTransform:"uppercase",marginBottom:12}}>Чистые активы (без долга)</div>
            <div style={{fontFamily:"'Syne',sans-serif",fontWeight:800,fontSize:34,color:netAssets>=0?"#ddd8cc":"#f87171",marginBottom:4}}>{fmt(netAssets)}</div>
            <div style={{fontSize:11,color:"#555"}}>активы {fmt(totalAssets)} − долг {fmt(totalDebt)}</div>
          </div>

          <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10}}>
            <div className="card" onClick={()=>goLog("paypal")} style={{cursor:"pointer",transition:"border-color .15s",borderColor:"#2a3a5a"}} onMouseEnter={e=>e.currentTarget.style.borderColor="#4a6fa5"} onMouseLeave={e=>e.currentTarget.style.borderColor="#2a3a5a"}><label>PayPal ↗</label><div style={{fontFamily:"'Syne',sans-serif",fontWeight:700,fontSize:20,color:"#7eb3ff"}}>{fmt(cash.paypal)}</div><div style={{fontSize:10,color:"#444",marginTop:4}}>нажми — журнал</div></div>
            <div className="card" onClick={()=>goLog("cash")} style={{cursor:"pointer",transition:"border-color .15s",borderColor:"#2a3a5a"}} onMouseEnter={e=>e.currentTarget.style.borderColor="#4a6fa5"} onMouseLeave={e=>e.currentTarget.style.borderColor="#2a3a5a"}><label>Наличные ↗</label><div style={{fontFamily:"'Syne',sans-serif",fontWeight:700,fontSize:20,color:"#7eb3ff"}}>{fmt(cash.cash)}</div><div style={{fontSize:10,color:"#444",marginTop:4}}>нажми — журнал</div></div>
          </div>

          <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10}}>
            <div className="card" onClick={()=>setView("bikes")} style={{cursor:"pointer",transition:"border-color .15s",borderColor:"#2a2a10"}} onMouseEnter={e=>e.currentTarget.style.borderColor="#fbbf24"} onMouseLeave={e=>e.currentTarget.style.borderColor="#2a2a10"}>
              <label>Склад велов ({stockBikes.length}) ↗</label>
              <div style={{fontFamily:"'Syne',sans-serif",fontWeight:700,fontSize:20,color:"#fbbf24"}}>{fmt(stockValue)}</div>
              <div style={{fontSize:10,color:"#444",marginTop:4}}>нажми — склад</div>
            </div>
            <div className="card" onClick={()=>setView("parts")} style={{cursor:"pointer",transition:"border-color .15s",borderColor:"#2a2a10"}} onMouseEnter={e=>e.currentTarget.style.borderColor="#fbbf24"} onMouseLeave={e=>e.currentTarget.style.borderColor="#2a2a10"}>
              <label>Запчасти ({parts.length}) ↗</label>
              <div style={{fontFamily:"'Syne',sans-serif",fontWeight:700,fontSize:20,color:"#fbbf24"}}>{fmt(partsValue)}</div>
              <div style={{fontSize:10,color:"#444",marginTop:4}}>нажми — запчасти</div>
            </div>
          </div>

          <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10}}>
            <div className="card" onClick={()=>setView("stats")} style={{cursor:"pointer",transition:"border-color .15s",borderColor:"#0a2a1a"}} onMouseEnter={e=>e.currentTarget.style.borderColor="#4ade80"} onMouseLeave={e=>e.currentTarget.style.borderColor="#0a2a1a"}>
              <label>Прибыль с продаж ↗</label>
              <div style={{fontFamily:"'Syne',sans-serif",fontWeight:700,fontSize:20,color:"#4ade80"}}>{fmt(totalProfit)}</div>
              <div style={{fontSize:10,color:"#444",marginTop:4}}>нажми — аналитика</div>
            </div>
            <div className="card" onClick={()=>{ const el=document.getElementById("debt-section"); if(el) el.scrollIntoView({behavior:"smooth"}); }} style={{cursor:"pointer",border:"1px solid #2a1a00",transition:"border-color .15s"}} onMouseEnter={e=>e.currentTarget.style.borderColor="#fb923c"} onMouseLeave={e=>e.currentTarget.style.borderColor="#2a1a00"}>
              <label>Долг 🔴 ↗</label>
              <div style={{fontFamily:"'Syne',sans-serif",fontWeight:700,fontSize:20,color:"#fb923c"}}>{fmt(totalDebt)}</div>
              <div style={{fontSize:10,color:"#444",marginTop:4}}>нажми — к долгам</div>
            </div>
          </div>

          {totalWithdrawn>0 && (
            <div className="card" onClick={()=>goLog("withdrawal")} style={{cursor:"pointer",border:"1px solid #2a1040",transition:"border-color .15s"}} onMouseEnter={e=>e.currentTarget.style.borderColor="#c084fc"} onMouseLeave={e=>e.currentTarget.style.borderColor="#2a1040"}>
              <label>💸 Выведено себе ↗</label>
              <div style={{fontFamily:"'Syne',sans-serif",fontWeight:700,fontSize:20,color:"#c084fc"}}>{fmt(totalWithdrawn)}</div>
              <div style={{fontSize:10,color:"#444",marginTop:4}}>нажми — журнал</div>
            </div>
          )}

          {/* Долги список */}
          {debts.length>0 && (
            <div id="debt-section" className="card">
              <div style={{fontSize:10,color:"#444",letterSpacing:".1em",textTransform:"uppercase",marginBottom:12}}>Долги</div>
              {debts.map(d=>(
                <div key={d.id} style={{display:"flex",justifyContent:"space-between",alignItems:"center",padding:"10px 0",borderBottom:"1px solid #1a1d26"}}>
                  <div>
                    <div style={{fontSize:13}}>{d.label}</div>
                    <div style={{fontSize:10,color:"#555",marginTop:2}}>{d.date}</div>
                  </div>
                  <div style={{display:"flex",alignItems:"center",gap:10}}>
                    <div style={{color:"#fb923c",fontWeight:700}}>{fmt(d.amount)}</div>
                    <button className="btn btn-ghost" style={{fontSize:10,padding:"5px 9px"}} onClick={()=>handleRepayDebt(d.id)}>Погасить</button>
                  </div>
                </div>
              ))}
              <button className="btn btn-orange" style={{width:"100%",marginTop:12}} onClick={()=>setShowDebtForm(v=>!v)}>+ Новый долг</button>
              {showDebtForm && (
                <div style={{marginTop:14,paddingTop:14,borderTop:"1px solid #1a1d26"}}>
                  <div style={{marginBottom:10}}><label>Описание</label><input className="inp" placeholder="От кого / на что" value={debtForm.label} onChange={e=>setDebtForm(f=>({...f,label:e.target.value}))}/></div>
                  <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10,marginBottom:14}}>
                    <div><label>Сумма €</label><input className="inp" type="number" value={debtForm.amount} onChange={e=>setDebtForm(f=>({...f,amount:e.target.value}))}/></div>
                    <div><label>Дата</label><input className="inp" type="date" value={debtForm.date} onChange={e=>setDebtForm(f=>({...f,date:e.target.value}))}/></div>
                  </div>
                  <button className="btn btn-orange" style={{width:"100%"}} onClick={handleAddDebt}>Записать долг</button>
                </div>
              )}
            </div>
          )}

          <div className="card">
            <div style={{fontSize:10,color:"#444",letterSpacing:".1em",textTransform:"uppercase",marginBottom:12}}>Обновить остатки</div>
            <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10}}>
              <div><label>PayPal €</label><input className="inp" type="number" value={cash.paypal} onChange={e=>update(s=>({...s,cash:{...s.cash,paypal:Number(e.target.value)||0}}))}/></div>
              <div><label>Наличные €</label><input className="inp" type="number" value={cash.cash} onChange={e=>update(s=>({...s,cash:{...s.cash,cash:Number(e.target.value)||0}}))}/></div>
            </div>
          </div>

          {soldBikes.length>0 && (
            <div className="card">
              <div style={{fontSize:10,color:"#444",letterSpacing:".1em",textTransform:"uppercase",marginBottom:12}}>Проданные</div>
              {soldBikes.map(b=>(
                <div key={b.id} style={{display:"flex",justifyContent:"space-between",alignItems:"center",padding:"10px 0",borderBottom:"1px solid #1a1d26"}}>
                  <div>
                    <div style={{fontSize:13}}>{b.name}</div>
                    <div style={{fontSize:10,color:"#555",marginTop:2}}>Себест: {fmt(totalCost(b))} → {fmt(b.salePrice)}</div>
                  </div>
                  <div style={{textAlign:"right"}}>
                    <div style={{color:(bikeProfit(b)||0)>=0?"#4ade80":"#f87171",fontWeight:700}}>{fmt(bikeProfit(b)||0,true)}</div>
                    <div style={{fontSize:10,color:"#444"}}>{b.soldDate}</div>
                  </div>
                </div>
              ))}
            </div>
          )}
        </>}

        {/* ── СКЛАД ВЕЛОВ ── */}
        {view==="bikes" && <>
          {stockBikes.length===0 && <div style={{textAlign:"center",color:"#444",padding:"40px 0"}}>Склад пустой</div>}
          {stockBikes.map(b=>(
            <div key={b.id} className="card">
              <div style={{display:"flex",justifyContent:"space-between",alignItems:"flex-start",marginBottom:12}}>
                <div>
                  <div style={{fontFamily:"'Syne',sans-serif",fontWeight:700,fontSize:15}}>{b.name}</div>
                  <span style={{display:"inline-block",marginTop:4,padding:"2px 9px",borderRadius:20,fontSize:10,background:"#1e2e1a",color:"#86efac"}}>На складе</span>
                </div>
                <div style={{textAlign:"right"}}>
                  <div style={{fontFamily:"'Syne',sans-serif",fontWeight:700,fontSize:19,color:"#fbbf24"}}>{fmt(totalCost(b))}</div>
                  <div style={{fontSize:10,color:"#444"}}>вложено</div>
                </div>
              </div>
              <div style={{background:"#0d0f14",borderRadius:6,padding:"10px 12px",marginBottom:12}}>
                {b.costs.map((c,i)=>(
                  <div key={i} className="cost-row">
                    <span>{c.label}</span>
                    <div style={{display:"flex",gap:12}}>
                      <span style={{color:"#555"}}>{c.date?.slice(5).replace("-",".")}</span>
                      <span style={{color:"#f87171"}}>{fmt(c.amount)}</span>
                    </div>
                  </div>
                ))}
              </div>
              <div style={{display:"flex",gap:8}}>
                <button className="btn btn-yellow" style={{flex:1}} onClick={()=>{setBikeCostModal(b.id);setBikeCostForm({category:BIKE_CATS[0],amount:"",account:"cash",date:today()})}}>+ Расход</button>
                <button className="btn btn-green" style={{flex:1}} onClick={()=>{setSelling(b.id);setSaleInput({price:"",method:"cash"})}}>💰 Продать</button>
                <button className="btn btn-red" onClick={()=>handleDeleteBike(b.id)}>✕</button>
              </div>
            </div>
          ))}
        </>}

        {/* ── ДОБАВИТЬ ВЕЛ ── */}
        {view==="add" && (
          <div className="card">
            <div style={{fontFamily:"'Syne',sans-serif",fontWeight:800,fontSize:16,marginBottom:18}}>Новый велосипед</div>
            <div style={{marginBottom:14}}>
              <label>Название / модель</label>
              <input className="inp" placeholder="Напр. Trek FX3" value={addForm.name} onChange={e=>setAddForm(f=>({...f,name:e.target.value}))}/>
            </div>
            <div style={{marginBottom:10}}>
              <label>Затраты</label>
              {addForm.costs.map((c,i)=>(
                <div key={i} style={{display:"grid",gridTemplateColumns:"1fr auto",gap:8,marginBottom:8}}>
                  <select className="inp" value={c.category} onChange={e=>setAddForm(f=>{const costs=[...f.costs];costs[i]={...costs[i],category:e.target.value};return{...f,costs};})}>
                    {BIKE_CATS.map(cat=><option key={cat}>{cat}</option>)}
                  </select>
                  <input className="inp" style={{width:86}} type="number" placeholder="€" value={c.amount} onChange={e=>setAddForm(f=>{const costs=[...f.costs];costs[i]={...costs[i],amount:e.target.value};return{...f,costs};})}/>
                </div>
              ))}
              <button onClick={()=>setAddForm(f=>({...f,costs:[...f.costs,{category:BIKE_CATS[0],amount:""}]}))} style={{background:"none",border:"1px dashed #2a2d38",color:"#555",borderRadius:6,padding:"7px 14px",cursor:"pointer",fontFamily:"inherit",fontSize:11,width:"100%",marginTop:4}}>+ добавить статью</button>
            </div>
            {addForm.costs.some(c=>c.amount) && (
              <div style={{background:"#0d0f14",borderRadius:6,padding:"10px 14px",marginBottom:14,fontSize:12,display:"flex",justifyContent:"space-between"}}>
                <span style={{color:"#555"}}>Итого:</span>
                <span style={{color:"#fbbf24",fontWeight:700}}>{fmt(addForm.costs.reduce((s,c)=>s+(Number(c.amount)||0),0))}</span>
              </div>
            )}
            <button className="btn btn-primary" style={{width:"100%"}} onClick={handleAddBike}>Добавить на склад</button>
          </div>
        )}

        {/* ── ЗАПЧАСТИ ── */}
        {view==="parts" && <>
          <div style={{display:"flex",justifyContent:"space-between",alignItems:"center"}}>
            <div style={{fontFamily:"'Syne',sans-serif",fontWeight:800,fontSize:15}}>Склад запчастей</div>
            <button className="btn btn-yellow" style={{padding:"8px 12px",fontSize:11}} onClick={()=>setShowPartForm(v=>!v)}>+ Добавить</button>
          </div>
          {showPartForm && (
            <div className="card">
              <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10,marginBottom:10}}>
                <div><label>Название</label><input className="inp" placeholder="Батарея, мотор..." value={partForm.name} onChange={e=>setPartForm(f=>({...f,name:e.target.value}))}/></div>
                <div><label>Цена €</label><input className="inp" type="number" value={partForm.cost} onChange={e=>setPartForm(f=>({...f,cost:e.target.value}))}/></div>
              </div>
              <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10,marginBottom:14}}>
                <div><label>Дата</label><input className="inp" type="date" value={partForm.date} onChange={e=>setPartForm(f=>({...f,date:e.target.value}))}/></div>
                <div><label>Примечание</label><input className="inp" placeholder="Необязательно" value={partForm.note} onChange={e=>setPartForm(f=>({...f,note:e.target.value}))}/></div>
              </div>
              <button className="btn btn-yellow" style={{width:"100%"}} onClick={handleAddPart}>Добавить запчасть</button>
            </div>
          )}
          {parts.length===0 && <div style={{textAlign:"center",color:"#444",padding:"40px 0"}}>Запчастей нет</div>}
          <div className="card">
            {parts.map(p=>(
              <div key={p.id} className="row">
                <div style={{width:32,height:32,borderRadius:8,background:"#1a1600",display:"flex",alignItems:"center",justifyContent:"center",fontSize:16,flexShrink:0}}>🔧</div>
                <div style={{flex:1,minWidth:0}}>
                  <div style={{fontSize:13}}>{p.name}</div>
                  {p.note && <div style={{fontSize:10,color:"#555",marginTop:2}}>{p.note}</div>}
                </div>
                <div style={{textAlign:"right",flexShrink:0}}>
                  <div style={{color:"#fbbf24",fontWeight:700}}>{fmt(p.cost)}</div>
                  <div style={{fontSize:10,color:"#444"}}>{p.date?.slice(5).replace("-",".")}</div>
                </div>
                <button className="btn btn-red" style={{padding:"4px 8px",fontSize:12}} onClick={()=>handleDeletePart(p.id)}>✕</button>
              </div>
            ))}
            {parts.length>0 && (
              <div style={{borderTop:"1px solid #1a1d26",paddingTop:10,marginTop:6,display:"flex",justifyContent:"space-between",fontSize:12}}>
                <span style={{color:"#555"}}>Итого запчасти:</span>
                <span style={{color:"#fbbf24",fontWeight:700}}>{fmt(partsValue)}</span>
              </div>
            )}
          </div>
        </>}

        {/* ── ЖУРНАЛ ── */}
        {view==="log" && <>
          <div style={{display:"flex",justifyContent:"space-between",alignItems:"center"}}>
            <div style={{fontFamily:"'Syne',sans-serif",fontWeight:800,fontSize:15}}>Журнал</div>
            <div style={{display:"flex",gap:6}}>
              <button className="btn btn-yellow" style={{padding:"7px 11px",fontSize:10}} onClick={()=>{setShowExpForm(v=>!v);setShowWithdrawForm(false)}}>+ Расход</button>
              <button className="btn btn-purple" style={{padding:"7px 11px",fontSize:10}} onClick={()=>{setShowWithdrawForm(v=>!v);setShowExpForm(false)}}>💸 Себе</button>
            </div>
          </div>

          {showExpForm && (
            <div className="card">
              <div style={{fontSize:10,color:"#fbbf24",letterSpacing:".1em",textTransform:"uppercase",marginBottom:12}}>Новый расход</div>
              <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10,marginBottom:10}}>
                <div><label>Сумма €</label><input className="inp" type="number" placeholder="0" value={expForm.amount} onChange={e=>setExpForm(f=>({...f,amount:e.target.value}))}/></div>
                <div><label>Дата</label><input className="inp" type="date" value={expForm.date} onChange={e=>setExpForm(f=>({...f,date:e.target.value}))}/></div>
              </div>
              <div style={{marginBottom:10}}><label>Категория</label><select className="inp" value={expForm.category} onChange={e=>setExpForm(f=>({...f,category:e.target.value}))}>{EXPENSE_CATS.map(c=><option key={c}>{c}</option>)}</select></div>
              <div style={{marginBottom:10}}><label>Комментарий</label><input className="inp" placeholder="Необязательно" value={expForm.desc} onChange={e=>setExpForm(f=>({...f,desc:e.target.value}))}/></div>
              <div style={{marginBottom:14}}><label>Счёт</label><AccountToggle value={expForm.account} onChange={v=>setExpForm(f=>({...f,account:v}))}/></div>
              <div style={{display:"flex",gap:8}}>
                <button className="btn btn-red" style={{flex:1}} onClick={handleAddExpense}>Записать</button>
                <button className="btn btn-ghost" onClick={()=>setShowExpForm(false)}>Отмена</button>
              </div>
            </div>
          )}

          {showWithdrawForm && (
            <div className="card" style={{border:"1px solid #3a1a5a"}}>
              <div style={{fontSize:10,color:"#c084fc",letterSpacing:".1em",textTransform:"uppercase",marginBottom:4}}>💸 Вывод личных денег</div>
              <div style={{fontSize:11,color:"#555",marginBottom:14}}>Деньги которые вы забрали из бизнеса</div>
              <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10,marginBottom:10}}>
                <div><label>Сумма €</label><input className="inp" type="number" placeholder="0" value={withdrawForm.amount} onChange={e=>setWithdrawForm(f=>({...f,amount:e.target.value}))}/></div>
                <div><label>Дата</label><input className="inp" type="date" value={withdrawForm.date} onChange={e=>setWithdrawForm(f=>({...f,date:e.target.value}))}/></div>
              </div>
              <div style={{marginBottom:10}}><label>На что</label><input className="inp" placeholder="Еда, аренда, личное..." value={withdrawForm.desc} onChange={e=>setWithdrawForm(f=>({...f,desc:e.target.value}))}/></div>
              <div style={{marginBottom:14}}><label>Откуда</label><AccountToggle value={withdrawForm.account} onChange={v=>setWithdrawForm(f=>({...f,account:v}))}/></div>
              <div style={{display:"flex",gap:8}}>
                <button onClick={handleWithdraw} style={{flex:1,padding:"11px",borderRadius:7,border:"none",background:"#2a1040",color:"#c084fc",cursor:"pointer",fontFamily:"inherit",fontSize:12,fontWeight:700}}>Записать вывод</button>
                <button className="btn btn-ghost" onClick={()=>setShowWithdrawForm(false)}>Отмена</button>
              </div>
            </div>
          )}

          {/* Filter tabs */}
          <div style={{display:"flex",gap:6,flexWrap:"wrap"}}>
            {[["all","Все"],["sale","Продажи"],["purchase","Закупки"],["expense","Расходы"],["withdrawal","💸 Себе"],["cash","Наличные"],["paypal","PayPal"]].map(([v,l])=>(
              <button key={v} onClick={()=>setLogFilter(v)} style={{padding:"5px 10px",borderRadius:6,border:"1px solid",borderColor:logFilter===v?"#4a6fa5":"#2a2d38",background:logFilter===v?"#1a2a40":"transparent",color:logFilter===v?"#7eb3ff":"#555",cursor:"pointer",fontFamily:"inherit",fontSize:10,letterSpacing:".06em"}}>{l}</button>
            ))}
          </div>

          <div className="card">
            {txns.length===0 && <div style={{textAlign:"center",color:"#444",padding:"20px 0"}}>Нет операций</div>}
            {[...txns]
              .filter(t => {
                if (logFilter==="all") return true;
                if (["sale","purchase","expense","withdrawal","debt"].includes(logFilter)) return t.type===logFilter;
                if (logFilter==="cash")   return t.account==="cash";
                if (logFilter==="paypal") return t.account==="paypal";
                return true;
              })
              .sort((a,b)=>b.date.localeCompare(a.date)).map(t=>{
              const meta=TXN_META[t.type]||TXN_META.expense;
              return (
                <div key={t.id} className="row">
                  <div style={{width:32,height:32,borderRadius:8,display:"flex",alignItems:"center",justifyContent:"center",background:meta.bg,color:meta.color,fontSize:13,flexShrink:0}}>{meta.icon}</div>
                  <div style={{flex:1,minWidth:0}}>
                    <div style={{fontSize:12,color:"#ccc",whiteSpace:"nowrap",overflow:"hidden",textOverflow:"ellipsis"}}>{t.desc}</div>
                    <div style={{fontSize:10,color:"#444",marginTop:2}}>{t.category} · {t.account==="paypal"?"PayPal":"Нал."} · {t.date.slice(5).replace("-",".")}</div>
                  </div>
                  <div style={{fontFamily:"'Syne',sans-serif",fontWeight:700,fontSize:13,color:meta.color,flexShrink:0}}>{t.amount>0?"+":"−"}{fmt(Math.abs(t.amount))}</div>
                </div>
              );
            })}
          </div>
        </>}


        {/* ── АНАЛИТИКА ── */}
        {view==="stats" && <>
          <div style={{fontFamily:"'Syne',sans-serif",fontWeight:800,fontSize:15,marginBottom:4}}>Аналитика по месяцам</div>
          <div style={{fontSize:11,color:"#555",marginBottom:14}}>Все суммы в €. Прибыль = выручка − вложения − расходы.</div>

          {monthlyStats.length===0 && <div style={{textAlign:"center",color:"#444",padding:"40px 0"}}>Нет данных</div>}

          {monthlyStats.map(m => {
            const profitColor = m.profit > 0 ? "#4ade80" : m.profit < 0 ? "#f87171" : "#888";
            return (
              <div key={m.key} className="card" style={{padding:"16px 18px"}}>
                <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:14}}>
                  <div style={{fontFamily:"'Syne',sans-serif",fontWeight:800,fontSize:16}}>{m.label}</div>
                  <div style={{fontFamily:"'Syne',sans-serif",fontWeight:800,fontSize:18,color:profitColor}}>
                    {m.profit>=0?"+":""}{m.profit.toLocaleString("ru-RU")} €
                  </div>
                </div>
                <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:"8px 16px"}}>
                  {[
                    {label:"💰 Выручка",    value:m.revenue,   color:"#4ade80", sub:`${m.sold} прод.`},
                    {label:"🚲 Вложил",     value:m.invested,  color:"#fbbf24", sub:`${m.bought} вел.`},
                    {label:"📋 Расходы",    value:m.expenses,  color:"#f87171", sub:"операц."},
                    {label:"💸 Себе",       value:m.withdrawn, color:"#c084fc", sub:"вывод"},
                  ].map(({label,value,color,sub}) => (
                    <div key={label} style={{background:"#0d0f14",borderRadius:8,padding:"10px 12px"}}>
                      <div style={{fontSize:10,color:"#555",marginBottom:4}}>{label}</div>
                      <div style={{fontFamily:"'Syne',sans-serif",fontWeight:700,fontSize:16,color}}>{value.toLocaleString("ru-RU")} €</div>
                      <div style={{fontSize:10,color:"#444",marginTop:2}}>{sub}</div>
                    </div>
                  ))}
                </div>
                {/* Profit bar */}
                {m.revenue > 0 && (
                  <div style={{marginTop:12}}>
                    <div style={{display:"flex",justifyContent:"space-between",fontSize:10,color:"#444",marginBottom:5}}>
                      <span>Рентабельность</span>
                      <span style={{color:profitColor}}>{((m.profit/m.revenue)*100).toFixed(1)}%</span>
                    </div>
                    <div style={{background:"#1a1d26",borderRadius:4,height:5,overflow:"hidden"}}>
                      <div style={{width:`${Math.max(0,Math.min(100,(m.profit/m.revenue)*100))}%`,height:"100%",background:profitColor,borderRadius:4}}/>
                    </div>
                  </div>
                )}
              </div>
            );
          })}

          {/* Итого */}
          {monthlyStats.length > 1 && (() => {
            const total = monthlyStats.reduce((s,m)=>({
              revenue:   s.revenue+m.revenue,
              invested:  s.invested+m.invested,
              expenses:  s.expenses+m.expenses,
              withdrawn: s.withdrawn+m.withdrawn,
              profit:    s.profit+m.profit,
            }), {revenue:0,invested:0,expenses:0,withdrawn:0,profit:0});
            const pc = total.revenue>0 ? ((total.profit/total.revenue)*100).toFixed(1) : "—";
            return (
              <div className="card" style={{border:"1px solid #2a3a2a",background:"#0f160f"}}>
                <div style={{fontFamily:"'Syne',sans-serif",fontWeight:800,fontSize:14,color:"#4ade80",marginBottom:14}}>Всего за период</div>
                <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:"8px 16px"}}>
                  {[
                    {label:"Выручка",  value:total.revenue,   color:"#4ade80"},
                    {label:"Вложено",  value:total.invested,  color:"#fbbf24"},
                    {label:"Расходы",  value:total.expenses,  color:"#f87171"},
                    {label:"Себе",     value:total.withdrawn, color:"#c084fc"},
                  ].map(({label,value,color}) => (
                    <div key={label} style={{background:"#0d0f14",borderRadius:8,padding:"10px 12px"}}>
                      <div style={{fontSize:10,color:"#555",marginBottom:4}}>{label}</div>
                      <div style={{fontFamily:"'Syne',sans-serif",fontWeight:700,fontSize:15,color}}>{value.toLocaleString("ru-RU")} €</div>
                    </div>
                  ))}
                </div>
                <div style={{marginTop:12,display:"flex",justifyContent:"space-between",alignItems:"center",padding:"12px",background:"#0d0f14",borderRadius:8}}>
                  <span style={{fontSize:11,color:"#555"}}>Итого прибыль ({pc}%)</span>
                  <span style={{fontFamily:"'Syne',sans-serif",fontWeight:800,fontSize:18,color:total.profit>=0?"#4ade80":"#f87171"}}>{total.profit>=0?"+":""}{total.profit.toLocaleString("ru-RU")} €</span>
                </div>
              </div>
            );
          })()}
        </>}
        {/* ── РАСПРЕДЕЛЕНИЕ ── */}
        {view==="split" && <>
          <div className="card">
            <div style={{fontFamily:"'Syne',sans-serif",fontWeight:800,fontSize:15,marginBottom:14}}>Как делить прибыль</div>
            {SPLIT.map(({label,pct,color})=>(
              <div key={label} style={{marginBottom:14}}>
                <div style={{display:"flex",justifyContent:"space-between",marginBottom:6}}>
                  <span style={{fontSize:13,color}}>{label}</span>
                  <span style={{fontFamily:"'Syne',sans-serif",fontWeight:700,color}}>{pct}%</span>
                </div>
                <div style={{background:"#1a1d26",borderRadius:4,height:6,overflow:"hidden"}}>
                  <div style={{width:`${pct}%`,height:"100%",background:color,borderRadius:4}}/>
                </div>
              </div>
            ))}
          </div>
          <div className="card">
            <div style={{fontSize:10,color:"#444",letterSpacing:".1em",textTransform:"uppercase",marginBottom:14}}>Калькулятор продажи</div>
            <SplitCalc/>
          </div>
        </>}
      </div>

      {/* ── MODAL: расход к велу ── */}
      {bikeCostModal && (
        <div className="modal-bg" onClick={()=>setBikeCostModal(null)}>
          <div className="modal" onClick={e=>e.stopPropagation()}>
            <div style={{fontFamily:"'Syne',sans-serif",fontWeight:800,fontSize:15,marginBottom:4}}>+ Расход к: {bikes.find(b=>b.id===bikeCostModal)?.name}</div>
            <div style={{fontSize:11,color:"#555",marginBottom:18}}>Уже вложено: {fmt(totalCost(bikes.find(b=>b.id===bikeCostModal)||{costs:[]}))}</div>
            <div style={{marginBottom:10}}><label>Категория</label><select className="inp" value={bikeCostForm.category} onChange={e=>setBikeCostForm(f=>({...f,category:e.target.value}))}>{BIKE_CATS.map(c=><option key={c}>{c}</option>)}</select></div>
            <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10,marginBottom:10}}>
              <div><label>Сумма €</label><input className="inp" type="number" placeholder="0" autoFocus value={bikeCostForm.amount} onChange={e=>setBikeCostForm(f=>({...f,amount:e.target.value}))}/></div>
              <div><label>Дата</label><input className="inp" type="date" value={bikeCostForm.date} onChange={e=>setBikeCostForm(f=>({...f,date:e.target.value}))}/></div>
            </div>
            <div style={{marginBottom:16}}><label>Счёт</label><AccountToggle value={bikeCostForm.account} onChange={v=>setBikeCostForm(f=>({...f,account:v}))}/></div>
            <div style={{display:"flex",gap:8}}>
              <button className="btn btn-yellow" style={{flex:1}} onClick={handleAddBikeCost}>Добавить</button>
              <button className="btn btn-ghost" onClick={()=>setBikeCostModal(null)}>Отмена</button>
            </div>
          </div>
        </div>
      )}

      {/* ── MODAL: продать ── */}
      {selling && (
        <div className="modal-bg" onClick={()=>setSelling(null)}>
          <div className="modal" onClick={e=>e.stopPropagation()}>
            <div style={{fontFamily:"'Syne',sans-serif",fontWeight:800,fontSize:16,marginBottom:6}}>Продать: {bikes.find(b=>b.id===selling)?.name}</div>
            <div style={{fontSize:11,color:"#555",marginBottom:18}}>Себестоимость: {fmt(totalCost(bikes.find(b=>b.id===selling)||{costs:[]}))}</div>
            <div style={{marginBottom:12}}><label>Цена продажи €</label><input className="inp" type="number" placeholder="0" autoFocus value={saleInput.price} onChange={e=>setSaleInput(s=>({...s,price:e.target.value}))}/></div>
            {saleInput.price&&(()=>{
              const bike=bikes.find(b=>b.id===selling);
              const cost=totalCost(bike);
              const p=Number(saleInput.price)-cost;
              return (
                <div style={{background:p>=0?"#0a1a0f":"#1a0a0a",border:`1px solid ${p>=0?"#1a3020":"#3a1a1a"}`,borderRadius:8,padding:"12px 14px",marginBottom:14}}>
                  <div style={{display:"flex",justifyContent:"space-between",fontSize:12}}>
                    <span style={{color:"#555"}}>Прибыль:</span>
                    <span style={{color:p>=0?"#4ade80":"#f87171",fontWeight:700}}>{fmt(p,true)}</span>
                  </div>
                  {p>0 && (
                    <div style={{marginTop:10,borderTop:"1px solid #1a2a1a",paddingTop:10}}>
                      {SPLIT.map(({label,pct,color})=>(
                        <div key={label} style={{display:"flex",justifyContent:"space-between",fontSize:11,padding:"2px 0"}}>
                          <span style={{color:"#555"}}>{label} ({pct}%)</span>
                          <span style={{color}}>{fmt(Math.round(p*pct/100))}</span>
                        </div>
                      ))}
                    </div>
                  )}
                </div>
              );
            })()}
            <div style={{marginBottom:14}}><label>Получено на</label><AccountToggle value={saleInput.method} onChange={v=>setSaleInput(s=>({...s,method:v}))}/></div>
            <div style={{display:"flex",gap:8}}>
              <button className="btn btn-green" style={{flex:1}} onClick={handleSell}>Подтвердить</button>
              <button className="btn btn-red" onClick={()=>setSelling(null)}>Отмена</button>
            </div>
          </div>
        </div>
      )}
    </div>
  );
}

function SplitCalc() {
  const [sale,setSale]=useState("");
  const [cost,setCost]=useState("");
  const p=Number(sale)-Number(cost);
  const valid=sale&&cost&&p>0;
  return (
    <div>
      <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10,marginBottom:12}}>
        <div><label>Цена продажи €</label><input className="inp" type="number" value={sale} onChange={e=>setSale(e.target.value)} placeholder="0"/></div>
        <div><label>Себестоимость €</label><input className="inp" type="number" value={cost} onChange={e=>setCost(e.target.value)} placeholder="0"/></div>
      </div>
      {valid && (
        <div style={{background:"#0d0f14",borderRadius:8,padding:"12px 14px"}}>
          <div style={{display:"flex",justifyContent:"space-between",fontSize:12,marginBottom:10,paddingBottom:10,borderBottom:"1px solid #1a1d26"}}>
            <span style={{color:"#555"}}>Прибыль</span>
            <span style={{color:"#4ade80",fontWeight:700}}>+{p} €</span>
          </div>
          {SPLIT.map(({label,pct,color})=>(
            <div key={label} style={{display:"flex",justifyContent:"space-between",fontSize:12,padding:"4px 0"}}>
              <span style={{color:"#666"}}>{label}</span>
              <span style={{color,fontWeight:700}}>{Math.round(p*pct/100)} €</span>
            </div>
          ))}
        </div>
      )}
      {sale&&cost&&p<=0&&<div style={{color:"#f87171",fontSize:12,textAlign:"center",padding:"10px 0"}}>⚠️ Убыток {p} €</div>}
    </div>
  );
}

// Mount
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(React.createElement(App));
</script>
</body>
</html>
