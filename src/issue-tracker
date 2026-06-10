import { useState, useRef } from "react";

// ─── CONSTANTS ────────────────────────────────────────────────
const MONTHS_EN = ["Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"];
const MONTHS_TH = ["ม.ค.","ก.พ.","มี.ค.","เม.ย.","พ.ค.","มิ.ย.","ก.ค.","ส.ค.","ก.ย.","ต.ค.","พ.ย.","ธ.ค."];
const CUR_YEAR  = new Date().getFullYear();
const CUR_MONTH = new Date().getMonth(); // 0-based
const ALL_STATUSES = ["รอดำเนินการ","กำลังดำเนินการ","รอตรวจสอบ","เสร็จแล้ว"];
const ALL_PRI      = ["High","Medium","Low"];
const ALL_TYPES    = ["Improvement","E-memo","Safety","Quality","Maintenance","Process"];
const planKey = (mi, w) => `${CUR_YEAR}-${String(mi+1).padStart(2,"0")}-W${w}`;

// ─── HELPERS ──────────────────────────────────────────────────
const dOff = n => { const x=new Date(); x.setDate(x.getDate()+n); return x.toISOString().split("T")[0]; };
const fmt  = s => s ? new Date(s).toLocaleDateString("th-TH",{day:"numeric",month:"short",year:"2-digit"}) : "—";
const dLeft= s => s ? Math.ceil((new Date(s)-new Date())/864e5) : null;

// ─── INITIAL DATA ─────────────────────────────────────────────
const INIT_MEMBERS = [
  {id:1,name:"สมชาย ใจดี",  role:"Engineer",   abbr:"สช",skills:["Mechanical","Electrical"]},
  {id:2,name:"วิภา รักงาน", role:"Technician", abbr:"วภ",skills:["Quality","Process"]},
  {id:3,name:"ธนา สุขใจ",   role:"Supervisor", abbr:"ธน",skills:["Management","Process"]},
  {id:4,name:"นภา ชัยดี",   role:"Engineer",   abbr:"นภ",skills:["Electrical","PLC"]},
  {id:5,name:"อรุณ มานะ",   role:"Technician", abbr:"อร",skills:["Mechanical","Welding"]},
];

// plan rows: [{id, label, cells:{key:true}}]
const mkPlan = (label, keys) => [{id:1, label, cells: Object.fromEntries(keys.map(k=>[k,true]))}];

const INIT_ISSUES = [
  {id:1, title:"ปรับปรุงสายพานลำเลียงสายการผลิต A", desc:"สายพานมีการสึกหรอ ต้องเปลี่ยนและปรับแนว",       type:"Improvement",pri:"High",  status:"กำลังดำเนินการ",asgn:1,s:dOff(-10),due:dOff(3), pct:60, planRows:mkPlan("วางแผนปรับปรุงสายพาน",[planKey(0,1),planKey(0,2),planKey(1,1)]),acts:[{id:1,kind:"status",txt:"เปลี่ยนสถานะ → กำลังดำเนินการ",date:dOff(-5),by:"สมชาย"}],img:null},
  {id:2, title:"จัดทำ E-memo ขั้นตอนการตรวจสอบ QC", desc:"สร้างเอกสาร E-memo สำหรับกระบวนการ QC",         type:"E-memo",     pri:"Medium",status:"รอดำเนินการ",   asgn:2,s:dOff(-3), due:dOff(7), pct:0,  planRows:mkPlan("ร่างเอกสาร QC",[planKey(1,2),planKey(1,3)]),acts:[],img:null},
  {id:3, title:"ติดตั้งระบบป้องกันความปลอดภัย Press #3", desc:"ติดตั้ง Light Curtain & Safety Relay",    type:"Safety",     pri:"High",  status:"รอตรวจสอบ",     asgn:4,s:dOff(-15),due:dOff(1), pct:90, planRows:mkPlan("ติดตั้งและทดสอบ Safety",[planKey(0,1),planKey(0,2),planKey(0,3)]),acts:[],img:null},
  {id:4, title:"ปรับปรุงกระบวนการ Welding ลดของเสีย", desc:"อัตราของเสียจาก Welding สูงกว่า Target 2%", type:"Quality",    pri:"High",  status:"กำลังดำเนินการ",asgn:5,s:dOff(-7), due:dOff(5), pct:45, planRows:mkPlan("วิเคราะห์และแก้ไข Welding",[planKey(2,1),planKey(2,2),planKey(2,3),planKey(2,4)]),acts:[],img:null},
  {id:5, title:"PM ระบบไฮดรอลิก Injection #5",        desc:"ถึงกำหนด PM เปลี่ยนน้ำมันและฟิลเตอร์",       type:"Maintenance",pri:"Medium",status:"เสร็จแล้ว",      asgn:1,s:dOff(-20),due:dOff(-5),pct:100,planRows:mkPlan("PM Hydraulic",[planKey(0,3),planKey(0,4)]),acts:[],img:null},
  {id:6, title:"ออกแบบ Jig ใหม่สำหรับชิ้นงาน Part-X", desc:"Jig เดิมไม่รองรับขนาดชิ้นงานใหม่",          type:"Improvement",pri:"Low",   status:"รอดำเนินการ",   asgn:3,s:dOff(2),  due:dOff(20),pct:0,  planRows:mkPlan("ออกแบบ Jig",[planKey(3,1),planKey(3,2)]),acts:[],img:null},
  {id:7, title:"จัดทำ E-memo SOP การบำรุงรักษารายวัน", desc:"เอกสาร SOP ยังไม่ครอบคลุมการ PM ประจำวัน",   type:"E-memo",     pri:"Medium",status:"กำลังดำเนินการ",asgn:2,s:dOff(-5), due:dOff(10),pct:30, planRows:mkPlan("ร่างและรีวิว SOP",[planKey(1,1),planKey(1,2),planKey(1,3)]),acts:[],img:null},
  {id:8, title:"ติดตั้ง Sensor วัดอุณหภูมิ Oven",      desc:"Oven บ่มชิ้นงานไม่มีการแสดงอุณหภูมิ Realtime",type:"Improvement",pri:"Medium",status:"รอตรวจสอบ",   asgn:4,s:dOff(-8), due:dOff(2), pct:85, planRows:mkPlan("จัดซื้อและติดตั้ง Sensor",[planKey(0,2),planKey(0,3)]),acts:[],img:null},
  {id:9, title:"ลด Setup Time CNC 45→30 นาที",         desc:"วิเคราะห์และปรับปรุงขั้นตอน Setup",          type:"Process",    pri:"High",  status:"รอดำเนินการ",   asgn:3,s:dOff(1),  due:dOff(25),pct:0,  planRows:mkPlan("ปรับปรุง Setup Process",[planKey(4,1),planKey(4,2),planKey(4,3)]),acts:[],img:null},
  {id:10,title:"ซ่อมระบบไฟฟ้าสายการผลิต C",           desc:"พบปัญหาไฟฟ้าตกบ่อยครั้งในสายการผลิต C",      type:"Maintenance",pri:"High",  status:"เสร็จแล้ว",      asgn:4,s:dOff(-12),due:dOff(-8),pct:100,planRows:mkPlan("ตรวจสอบและซ่อมระบบไฟฟ้า",[planKey(0,1),planKey(0,2)]),acts:[],img:null},
];

// ─── DESIGN TOKENS ────────────────────────────────────────────
const C = {
  bg:"#eef2fb", panel:"#ffffff", card:"#f7f9ff",
  line:"#d0daf0", line2:"#e8eef8",
  accent:"#1a56db", accentBright:"#2563eb",
  gold:"#d97706", red:"#dc2626", green:"#16a34a",
  textPrimary:"#0f172a", textSec:"#334d7a", textMuted:"#64748b",
};
const STATUS_CFG = {
  "รอดำเนินการ":   {bg:"#f1f5f9",color:"#475569",border:"#cbd5e1"},
  "กำลังดำเนินการ":{bg:"#dbeafe",color:"#1d4ed8",border:"#93c5fd"},
  "รอตรวจสอบ":    {bg:"#fef3c7",color:"#b45309",border:"#fcd34d"},
  "เสร็จแล้ว":    {bg:"#dcfce7",color:"#15803d",border:"#86efac"},
};
const PRI_CFG = {
  High:  {bg:"#fee2e2",color:"#b91c1c",border:"#fca5a5"},
  Medium:{bg:"#fef3c7",color:"#b45309",border:"#fcd34d"},
  Low:   {bg:"#dcfce7",color:"#15803d",border:"#86efac"},
};
const TYPE_CFG = {
  Improvement:{color:"#1d4ed8",bg:"#dbeafe"},
  "E-memo":   {color:"#6d28d9",bg:"#ede9fe"},
  Safety:     {color:"#b91c1c",bg:"#fee2e2"},
  Quality:    {color:"#065f46",bg:"#d1fae5"},
  Maintenance:{color:"#9a3412",bg:"#ffedd5"},
  Process:    {color:"#0e7490",bg:"#cffafe"},
};
const AVT_COLORS = ["#1a56db","#0f766e","#7c3aed","#c2410c","#0369a1","#be185d","#065f46","#92400e"];

// chart bar colors by status tier
const STATUS_BAR_COLOR = {
  "รอดำเนินการ":   "#94a3b8",
  "กำลังดำเนินการ":"#2563eb",
  "รอตรวจสอบ":    "#f59e0b",
  "เสร็จแล้ว":    "#16a34a",
};

// ─── SHARED UI ATOMS ─────────────────────────────────────────
const Avt = ({m,size=28})=>{
  const bg=AVT_COLORS[((m?.id||1)-1)%AVT_COLORS.length];
  return <div style={{width:size,height:size,borderRadius:6,background:bg,display:"flex",alignItems:"center",justifyContent:"center",fontSize:size*0.35,fontWeight:800,color:"#fff",flexShrink:0,fontFamily:"monospace",letterSpacing:"0.03em"}}>{m?.abbr||"?"}</div>;
};
const StatusChip=({status,sm})=>{
  const c=STATUS_CFG[status]||STATUS_CFG["รอดำเนินการ"];
  return <span style={{display:"inline-flex",alignItems:"center",gap:4,padding:sm?"2px 8px":"3px 10px",fontSize:sm?10:11,fontWeight:700,borderRadius:20,color:c.color,background:c.bg,border:`1px solid ${c.border}`,whiteSpace:"nowrap"}}>{status}</span>;
};
const PriChip=({pri})=>{
  const c=PRI_CFG[pri]||PRI_CFG.Low;
  return <span style={{display:"inline-flex",padding:"2px 8px",fontSize:10,fontWeight:700,borderRadius:4,color:c.color,background:c.bg,border:`1px solid ${c.border}`}}>{pri}</span>;
};
const TypeChip=({type})=>{
  const c=TYPE_CFG[type]||{color:C.textSec,bg:C.line2};
  return <span style={{display:"inline-flex",padding:"2px 8px",fontSize:10,fontWeight:700,borderRadius:4,color:c.color,background:c.bg,border:`1px solid ${c.color}44`}}>{type}</span>;
};
const Bar=({pct,h=4,color})=>(
  <div style={{height:h,background:C.line,borderRadius:2,overflow:"hidden"}}>
    <div style={{width:`${pct}%`,height:"100%",background:color||(pct===100?C.green:C.accent),borderRadius:2,transition:"width .4s"}}/>
  </div>
);
const Panel=({children,style={},...p})=>(
  <div style={{background:C.panel,border:`1px solid ${C.line}`,borderRadius:10,boxShadow:"0 1px 4px rgba(26,86,219,0.06)",...style}} {...p}>{children}</div>
);
const SHdr=({label,right,sub})=>(
  <div style={{display:"flex",alignItems:"center",padding:"10px 16px",background:C.line2,borderBottom:`1px solid ${C.line}`,borderRadius:"10px 10px 0 0",gap:8}}>
    <div style={{flex:1}}>
      <div style={{fontSize:12,fontWeight:700,color:C.textSec,letterSpacing:"0.05em"}}>{label}</div>
      {sub&&<div style={{fontSize:10,color:C.textMuted,marginTop:1}}>{sub}</div>}
    </div>
    {right&&<div>{right}</div>}
  </div>
);
const Inp=({style={},...p})=>(
  <input style={{background:"#fff",border:`1px solid ${C.line}`,color:C.textPrimary,padding:"8px 10px",fontSize:13,borderRadius:7,width:"100%",fontFamily:"inherit",boxSizing:"border-box",...style}} {...p}/>
);
const Sel=({children,style={},...p})=>(
  <select style={{background:"#fff",border:`1px solid ${C.line}`,color:C.textPrimary,padding:"8px 10px",fontSize:13,borderRadius:7,fontFamily:"inherit",...style}} {...p}>{children}</select>
);
const Btn=({children,variant="primary",style={},...p})=>{
  const vs={
    primary:{background:C.accent,color:"#fff",border:"none",boxShadow:"0 1px 4px rgba(26,86,219,.25)"},
    ghost:  {background:"#fff",color:C.textSec,border:`1px solid ${C.line}`},
    danger: {background:"#fee2e2",color:C.red,border:"1px solid #fca5a5"},
    ai:     {background:"#ede9fe",color:"#6d28d9",border:"1px solid #c4b5fd"},
    amber:  {background:"#fef3c7",color:"#b45309",border:"1px solid #fcd34d"},
  };
  return <button style={{...vs[variant],padding:"7px 14px",fontSize:12,fontWeight:700,borderRadius:7,cursor:"pointer",fontFamily:"monospace",display:"inline-flex",alignItems:"center",gap:6,transition:"all .15s",...style}} {...p}>{children}</button>;
};

// ─── PLAN GRID COMPONENT ─────────────────────────────────────
// planRows = [{id, label, cells:{key:true}}]
// Multi-row Gantt: each row has a label + clickable week cells
function PlanGrid({planRows=[], onChange, readonly=false}){
  const weeks=[1,2,3,4];

  const toggleCell=(rowId,key)=>{
    if(readonly||!onChange) return;
    onChange(planRows.map(r=>{
      if(r.id!==rowId) return r;
      const cells={...r.cells};
      if(cells[key]) delete cells[key]; else cells[key]=true;
      return {...r,cells};
    }));
  };

  const addRow=()=>{
    if(readonly||!onChange) return;
    const newRow={id:Date.now(),label:"รายละเอียดใหม่",cells:{}};
    onChange([...planRows,newRow]);
  };

  const deleteRow=(rowId)=>{
    if(readonly||!onChange) return;
    onChange(planRows.filter(r=>r.id!==rowId));
  };

  const updateLabel=(rowId,val)=>{
    if(readonly||!onChange) return;
    onChange(planRows.map(r=>r.id===rowId?{...r,label:val}:r));
  };

  return (
    <div style={{overflowX:"auto"}}>
      <table style={{borderCollapse:"collapse",fontSize:11,minWidth:900,width:"100%"}}>
        <thead>
          {/* Year */}
          <tr>
            <th style={{minWidth:180,padding:"5px 10px",background:C.line2,border:`1px solid ${C.line}`,textAlign:"left",fontSize:11,fontWeight:700,color:C.textSec}} rowSpan={2}>รายละเอียด</th>
            <th colSpan={48} style={{padding:"4px",background:C.accent,color:"#fff",border:`1px solid ${C.line}`,textAlign:"center",fontSize:11,fontWeight:800}}>{CUR_YEAR}</th>
            {!readonly&&<th rowSpan={2} style={{width:32,background:C.line2,border:`1px solid ${C.line}`}}></th>}
          </tr>
          {/* Months */}
          <tr>
            {MONTHS_EN.map((mo,mi)=>(
              <th key={mi} colSpan={4} style={{padding:"3px 2px",background:C.line2,border:`1px solid ${C.line}`,textAlign:"center",fontSize:10,fontWeight:700,color:C.textSec,minWidth:56}}>{mo}</th>
            ))}
          </tr>
          {/* W1-W4 */}
          <tr>
            <th style={{padding:"3px 8px",background:"#f1f5f9",border:`1px solid ${C.line}`,fontSize:9,color:C.textMuted}}></th>
            {MONTHS_EN.map((_,mi)=>[1,2,3,4].map(w=>(
              <th key={`${mi}-${w}`} style={{padding:"2px 0",background:"#f1f5f9",border:`1px solid ${C.line}`,textAlign:"center",fontSize:8,color:C.textMuted,fontWeight:600,width:14}}>W{w}</th>
            )))}
            {!readonly&&<th style={{background:"#f1f5f9",border:`1px solid ${C.line}`}}></th>}
          </tr>
        </thead>
        <tbody>
          {planRows.map((row,ri)=>(
            <tr key={row.id} style={{background:ri%2===0?"#fff":"#f8faff"}}>
              {/* Label cell */}
              <td style={{padding:"4px 6px",border:`1px solid ${C.line}`,minWidth:180}}>
                {readonly?(
                  <span style={{fontSize:11,color:C.textPrimary,fontWeight:ri===0?600:400}}>{row.label}</span>
                ):(
                  <input value={row.label} onChange={e=>updateLabel(row.id,e.target.value)}
                    style={{width:"100%",border:"none",background:"transparent",fontSize:11,color:C.textPrimary,fontFamily:"inherit",outline:"none",fontWeight:ri===0?600:400}}
                    placeholder="ระบุรายละเอียด..."/>
                )}
              </td>
              {/* Week cells */}
              {MONTHS_EN.map((_,mi)=>[1,2,3,4].map(w=>{
                const key=planKey(mi,w);
                const active=!!(row.cells&&row.cells[key]);
                return (
                  <td key={key}
                    onClick={()=>toggleCell(row.id,key)}
                    style={{border:`1px solid ${C.line}`,background:active?"#1a56db":"#fff",width:14,height:22,padding:0,cursor:readonly?"default":"pointer",transition:"background .1s"}}
                    title={`${MONTHS_EN[mi]} W${w}`}
                  />
                );
              }))}
              {/* Delete row btn */}
              {!readonly&&(
                <td style={{border:`1px solid ${C.line}`,textAlign:"center",padding:"2px 4px",background:"#fff"}}>
                  <button onClick={()=>deleteRow(row.id)}
                    style={{background:"none",border:"none",color:"#fca5a5",fontSize:13,cursor:"pointer",lineHeight:1}}
                    title="ลบแถวนี้">✕</button>
                </td>
              )}
            </tr>
          ))}
          {planRows.length===0&&(
            <tr>
              <td colSpan={50} style={{padding:16,textAlign:"center",color:C.textMuted,fontSize:11,border:`1px solid ${C.line}`,fontStyle:"italic"}}>
                ยังไม่มีรายละเอียด — กดปุ่ม "+ เพิ่มรายละเอียด" ด้านล่าง
              </td>
            </tr>
          )}
        </tbody>
      </table>
      {!readonly&&(
        <button onClick={addRow}
          style={{marginTop:8,background:"none",border:`1px dashed ${C.accent}`,color:C.accent,padding:"5px 14px",borderRadius:6,fontSize:11,cursor:"pointer",fontFamily:"monospace",fontWeight:700}}>
          + เพิ่มรายละเอียด
        </button>
      )}
      {!readonly&&<div style={{fontSize:10,color:C.textMuted,marginTop:4}}>💡 คลิกช่องเพื่อวางแผน (สีน้ำเงิน = มีแผน) · คลิกอีกครั้งเพื่อยกเลิก · แก้ไขชื่อได้โดยตรง</div>}
    </div>
  );
}

// ─── DASHBOARD CHART ─────────────────────────────────────────
// แสดงเดือนที่ผ่านมา (summary) + สัปดาห์ปัจจุบัน
function DashboardChart({issues}){
  const now = new Date();
  const cm  = now.getMonth();   // current month 0-based
  const cw  = Math.ceil(now.getDate()/7); // approx current week 1-4

  // Build monthly summary for past months (Jan → last month)
  // Each bar stacked by status count
  const monthBars = [];
  for(let mi=0; mi<cm; mi++){
    const mo = String(mi+1).padStart(2,"0");
    // issues that have any plan cell in this month
    const inMonth = issues.filter(i=>{
      if(!i.planRows) return false;
      return i.planRows.some(r=>r.cells && Object.keys(r.cells).some(k=>k.startsWith(`${CUR_YEAR}-${mo}-`)));
    });
    const byStatus = {};
    ALL_STATUSES.forEach(s=>{ byStatus[s]=inMonth.filter(i=>i.status===s).length; });
    monthBars.push({label:MONTHS_TH[mi], type:"month", byStatus, total:inMonth.length});
  }

  // Build weekly bars for current month
  const weekBars = [];
  for(let w=1; w<=5; w++){
    const mo = String(cm+1).padStart(2,"0");
    const key = `${CUR_YEAR}-${mo}-W${w}`;
    const inWeek = issues.filter(i=>{
      if(!i.planRows) return false;
      return i.planRows.some(r=>r.cells&&r.cells[key]);
    });
    const byStatus = {};
    ALL_STATUSES.forEach(s=>{ byStatus[s]=inWeek.filter(i=>i.status===s).length; });
    weekBars.push({label:`W${w}`, type:"week", byStatus, total:inWeek.length, isCurrent:w===cw});
  }

  const allBars = [...monthBars, ...weekBars];
  const maxVal  = Math.max(...allBars.map(b=>b.total), 1);
  const BAR_H   = 120;

  return (
    <div style={{padding:"14px 16px"}}>
      <div style={{display:"flex",alignItems:"flex-end",gap:3,height:BAR_H+24}}>
        {allBars.map((bar,bi)=>{
          const totalH = (bar.total/maxVal)*BAR_H;
          let cumH = 0;
          return (
            <div key={bi} style={{flex:1,display:"flex",flexDirection:"column",alignItems:"center",gap:2,position:"relative"}}>
              {/* Stacked bar */}
              <div style={{width:"100%",height:BAR_H,display:"flex",flexDirection:"column",justifyContent:"flex-end",position:"relative"}}>
                {/* separator line between months and weeks */}
                {bi===monthBars.length&&(
                  <div style={{position:"absolute",left:-4,top:0,bottom:0,width:2,background:C.accent,opacity:0.3,borderRadius:1}}/>
                )}
                <div style={{width:"100%",height:totalH,display:"flex",flexDirection:"column-reverse",borderRadius:"3px 3px 0 0",overflow:"hidden",boxShadow:bar.isCurrent?"0 0 0 2px #f59e0b":"none"}}>
                  {ALL_STATUSES.map(s=>{
                    const cnt=bar.byStatus[s]||0;
                    if(!cnt) return null;
                    const h=(cnt/bar.total)*totalH;
                    return (
                      <div key={s}
                        style={{width:"100%",height:h,background:STATUS_BAR_COLOR[s],flexShrink:0}}
                        title={`${s}: ${cnt}`}/>
                    );
                  })}
                </div>
                {bar.total>0&&(
                  <div style={{position:"absolute",top:-16,width:"100%",textAlign:"center",fontSize:9,fontWeight:700,color:C.textSec,fontFamily:"monospace"}}>{bar.total}</div>
                )}
              </div>
              {/* Label */}
              <div style={{fontSize:9,fontFamily:"monospace",color:bar.isCurrent?C.gold:bar.type==="month"?C.textSec:C.textMuted,fontWeight:bar.isCurrent?800:bar.type==="month"?600:400,textAlign:"center",lineHeight:1.2}}>
                {bar.label}
                {bar.type==="month"&&<div style={{fontSize:7,color:C.textMuted,letterSpacing:"0.05em"}}>SUM</div>}
              </div>
            </div>
          );
        })}
      </div>

      {/* Legend */}
      <div style={{display:"flex",flexWrap:"wrap",gap:10,marginTop:10,alignItems:"center"}}>
        {ALL_STATUSES.map(s=>(
          <div key={s} style={{display:"flex",alignItems:"center",gap:4}}>
            <div style={{width:10,height:10,borderRadius:2,background:STATUS_BAR_COLOR[s]}}/>
            <span style={{fontSize:10,color:C.textMuted}}>{s}</span>
          </div>
        ))}
        <div style={{marginLeft:"auto",fontSize:10,color:C.textMuted}}>
          <span style={{color:C.gold,fontWeight:700}}>■</span> = สัปดาห์ปัจจุบัน &nbsp;|&nbsp;
          <span style={{color:C.textSec,fontWeight:600}}>SUM</span> = สรุปรายเดือน
        </div>
      </div>
    </div>
  );
}

// ─── MAIN APP ─────────────────────────────────────────────────
export default function App(){
  const [issues,  setIssues]  = useState(INIT_ISSUES);
  const [members, setMembers] = useState(INIT_MEMBERS);
  const [types,   setTypes]   = useState([...ALL_TYPES]);
  const [view,    setView]    = useState("dashboard");
  const [sel,     setSel]     = useState(null);
  const [showNotif,setShowNotif]=useState(false);
  const [showNew,  setShowNew] = useState(false);
  const [aiState,  setAiState] = useState({open:false,loading:false,title:"",text:""});
  const [notifs,   setNotifs]  = useState([
    {id:1,text:"Issue #3 — Safety Press #3 ครบกำหนดพรุ่งนี้",level:"danger",read:false,ts:"05:23"},
    {id:2,text:"Issue #8 — เปลี่ยนสถานะ → รอตรวจสอบ",        level:"info",  read:false,ts:"08:41"},
  ]);
  const unread=notifs.filter(n=>!n.read).length;

  const upsert=upd=>{setIssues(p=>p.map(i=>i.id===upd.id?upd:i));if(sel?.id===upd.id)setSel(upd);};
  const removeIssue=id=>{setIssues(p=>p.filter(i=>i.id!==id));if(sel?.id===id){setSel(null);setView("list");}};
  const addIssue=iss=>{setIssues(p=>[{...iss,id:Date.now(),acts:[],pct:0},...p]);setShowNew(false);};
  const openDetail=iss=>{setSel(iss);setView("detail");};

  // AI
  const callAI=async(title,prompt)=>{
    setAiState({open:true,loading:true,title,text:""});
    try{
      const res=await fetch("https://api.anthropic.com/v1/messages",{method:"POST",headers:{"Content-Type":"application/json"},body:JSON.stringify({model:"claude-sonnet-4-20250514",max_tokens:1000,messages:[{role:"user",content:prompt}]})});
      const d=await res.json();
      setAiState(s=>({...s,loading:false,text:d.content?.map(c=>c.text||"").join("\n")||"ไม่สามารถวิเคราะห์ได้"}));
    }catch(e){setAiState(s=>({...s,loading:false,text:"❌ "+e.message}));}
  };
  const aiSummary=()=>callAI("สรุปสถานการณ์วันนี้",`คุณเป็น Production Manager AI วิเคราะห์ issue ต่อไปนี้ (วันนี้: ${new Date().toLocaleDateString("th-TH")}):\n${JSON.stringify(issues.filter(i=>i.status!=="เสร็จแล้ว").map(i=>({id:i.id,title:i.title,status:i.status,priority:i.pri,dueDate:i.due,progress:i.pct+"%"})),null,2)}\nสรุปเป็นภาษาไทย: 1) สิ่งน่าเป็นห่วง 2) ใกล้ deadline 3) ลำดับความสำคัญ`);
  const aiReport=()=>callAI("รายงานประจำสัปดาห์",`เขียนรายงานสรุปประจำสัปดาห์ทีม PRODUCTION:\n${JSON.stringify(issues.map(i=>({id:i.id,title:i.title,type:i.type,status:i.status,pri:i.pri,assignee:members.find(m=>m.id===i.asgn)?.name,pct:i.pct+"%"})),null,2)}\nรายงานภาษาไทย: บทสรุป, ประเภทปัญหาบ่อย, insight, คำแนะนำ`);
  const exportCSV=()=>{
    const rows=[["#","ชื่อปัญหา","ประเภท","Priority","ผู้รับผิดชอบ","วันกำหนด","สถานะ","ความคืบหน้า"]];
    issues.forEach(i=>rows.push([i.id,i.title,i.type,i.pri,members.find(m=>m.id===i.asgn)?.name||"-",i.due,i.status,i.pct+"%"]));
    const a=document.createElement("a");a.href=URL.createObjectURL(new Blob(["\uFEFF"+rows.map(r=>r.join(",")).join("\n")],{type:"text/csv;charset=utf-8;"}));a.download="issues.csv";a.click();
  };

  const NAV=[{id:"dashboard",label:"DASHBOARD",ico:"▦"},{id:"list",label:"ISSUE LIST",ico:"≡"},{id:"kanban",label:"KANBAN",ico:"⊞"},{id:"gantt",label:"GANTT",ico:"▤"}];

  return (
    <div style={{minHeight:"100vh",background:C.bg,color:C.textPrimary,fontFamily:"'IBM Plex Sans Thai','Sarabun','Noto Sans Thai',sans-serif"}}>
      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=IBM+Plex+Sans+Thai:wght@400;500;600;700&family=Sarabun:wght@400;500;600;700&display=swap');
        *{box-sizing:border-box;margin:0;padding:0;}
        ::-webkit-scrollbar{width:5px;height:5px;}
        ::-webkit-scrollbar-track{background:${C.line2};}
        ::-webkit-scrollbar-thumb{background:${C.line};border-radius:2px;}
        .row-h:hover{background:${C.line2}!important;}
        .nav-a{background:rgba(255,255,255,0.2)!important;border-bottom:3px solid #fff!important;color:#fff!important;}
        .nav-btn:hover{background:rgba(255,255,255,0.12)!important;color:#fff!important;}
        .k-card:hover{border-color:${C.accentBright}!important;transform:translateY(-1px);box-shadow:0 4px 12px rgba(26,86,219,.15)!important;}
        .k-card{transition:all .15s;}
        .fade{animation:fd .2s ease;}
        @keyframes fd{from{opacity:0;transform:translateY(6px)}to{opacity:1;transform:none}}
        @keyframes spin{to{transform:rotate(360deg)}}
        .grid-bg{background-image:linear-gradient(rgba(26,86,219,.03) 1px,transparent 1px),linear-gradient(90deg,rgba(26,86,219,.03) 1px,transparent 1px);background-size:32px 32px;}
        input:focus,select:focus,textarea:focus{outline:none;border-color:${C.accentBright}!important;box-shadow:0 0 0 3px rgba(37,99,235,.1);}
      `}</style>

      {/* ── TOP BAR ── */}
      <div style={{height:52,background:C.accent,display:"flex",alignItems:"center",padding:"0 20px",gap:2,boxShadow:"0 2px 8px rgba(26,86,219,.3)",position:"sticky",top:0,zIndex:50}}>
        <div style={{display:"flex",alignItems:"center",gap:10,marginRight:16}}>
          <div style={{width:34,height:34,background:"#fff",borderRadius:8,display:"flex",alignItems:"center",justifyContent:"center",fontSize:13,fontWeight:900,color:C.accent,fontFamily:"monospace"}}>PT</div>
          <div>
            <div style={{fontSize:13,fontWeight:800,color:"#fff",letterSpacing:"0.1em"}}>PRODUCTION</div>
            <div style={{fontSize:9,color:"rgba(255,255,255,.6)",letterSpacing:"0.18em"}}>ISSUE TRACKER</div>
          </div>
        </div>

        {NAV.map(n=>(
          <button key={n.id} className={`nav-btn${view===n.id?" nav-a":""}`} onClick={()=>setView(n.id)}
            style={{background:"transparent",border:"none",borderBottom:"3px solid transparent",color:"rgba(255,255,255,.75)",padding:"0 14px",height:52,fontSize:11,fontWeight:700,letterSpacing:"0.08em",fontFamily:"monospace",display:"flex",alignItems:"center",gap:5,cursor:"pointer"}}>
            <span style={{fontSize:13}}>{n.ico}</span>{n.label}
          </button>
        ))}

        <div style={{marginLeft:"auto",display:"flex",gap:8,alignItems:"center"}}>
          <Btn variant="ai" onClick={aiSummary} style={{fontSize:11,padding:"5px 12px"}}>⬡ AI สรุป</Btn>
          <Btn variant="ai" onClick={aiReport}  style={{fontSize:11,padding:"5px 12px"}}>⬡ รายงาน</Btn>
          <Btn variant="ghost" onClick={exportCSV} style={{fontSize:11,padding:"5px 12px",background:"rgba(255,255,255,.15)",color:"#fff",border:"1px solid rgba(255,255,255,.3)"}}>↓ Export</Btn>
          <button onClick={()=>setShowNew(true)} style={{background:"#fff",color:C.accent,border:"none",padding:"7px 16px",borderRadius:7,fontSize:12,fontWeight:800,cursor:"pointer",fontFamily:"monospace"}}>+ NEW ISSUE</button>
          <div style={{position:"relative"}}>
            <button onClick={()=>setShowNotif(!showNotif)} style={{width:36,height:36,background:"rgba(255,255,255,.15)",border:"1px solid rgba(255,255,255,.3)",borderRadius:7,color:"#fff",fontSize:16,display:"flex",alignItems:"center",justifyContent:"center",cursor:"pointer"}}>
              🔔
              {unread>0&&<span style={{position:"absolute",top:-4,right:-4,width:17,height:17,background:C.red,borderRadius:"50%",fontSize:9,fontWeight:800,color:"#fff",display:"flex",alignItems:"center",justifyContent:"center",border:"2px solid #fff"}}>{unread}</span>}
            </button>
            {showNotif&&<NotifPanel notifs={notifs} onRead={id=>setNotifs(p=>p.map(n=>n.id===id?{...n,read:true}:n))} onClose={()=>setShowNotif(false)}/>}
          </div>
        </div>
      </div>

      {/* ── CONTENT ── */}
      <div className="grid-bg" style={{padding:20,minHeight:"calc(100vh - 52px)"}}>
        <div className="fade" key={view}>
          {view==="dashboard"&&<Dashboard issues={issues} members={members} onView={openDetail} onViewAll={()=>setView("list")}/>}
          {view==="list"     &&<IssueList issues={issues} members={members} types={types} onView={openDetail} onUpdate={upsert} onDelete={removeIssue}/>}
          {view==="kanban"   &&<Kanban    issues={issues} members={members} onUpdate={upsert} onView={openDetail}/>}
          {view==="gantt"    &&<GanttView issues={issues} members={members}/>}
          {view==="detail"   &&sel&&<Detail issue={sel} members={members} onUpdate={upsert} onDelete={removeIssue} onBack={()=>setView("list")} callAI={callAI}/>}
        </div>
      </div>

      {showNew&&<NewIssueModal onClose={()=>setShowNew(false)} onSave={addIssue} types={types} setTypes={setTypes} members={members} setMembers={setMembers}/>}

      {aiState.open&&(
        <div style={{position:"fixed",inset:0,background:"rgba(15,23,42,.6)",display:"flex",alignItems:"center",justifyContent:"center",zIndex:200,padding:24}}>
          <Panel style={{width:"100%",maxWidth:640,maxHeight:"80vh",overflow:"auto"}} className="fade">
            <SHdr label={`⬡ AI — ${aiState.title}`} right={!aiState.loading&&<button onClick={()=>setAiState(s=>({...s,open:false}))} style={{background:"none",border:"none",color:C.textMuted,fontSize:18,cursor:"pointer"}}>✕</button>}/>
            <div style={{padding:20}}>
              {aiState.loading
                ?<div style={{textAlign:"center",padding:40}}><div style={{width:40,height:40,border:`3px solid ${C.line}`,borderTop:`3px solid ${C.accent}`,borderRadius:"50%",animation:"spin .8s linear infinite",margin:"0 auto 16px"}}/><div style={{color:C.textSec,fontSize:13}}>กำลังวิเคราะห์...</div></div>
                :<div style={{fontSize:13,lineHeight:1.85,color:C.textPrimary,whiteSpace:"pre-wrap"}}>{aiState.text}</div>}
            </div>
          </Panel>
        </div>
      )}
    </div>
  );
}

// ─── NOTIF PANEL ──────────────────────────────────────────────
function NotifPanel({notifs,onRead,onClose}){
  const lv={danger:C.red,info:C.accent};
  return (
    <div style={{position:"absolute",right:0,top:46,width:320,background:C.panel,border:`1px solid ${C.line}`,borderTop:`3px solid ${C.accent}`,zIndex:300,boxShadow:"0 8px 24px rgba(26,86,219,.15)",borderRadius:"0 0 8px 8px"}}>
      <SHdr label="NOTIFICATIONS" right={<button onClick={onClose} style={{background:"none",border:"none",color:C.textMuted,cursor:"pointer",fontSize:16}}>✕</button>}/>
      {notifs.map(n=>(
        <div key={n.id} onClick={()=>onRead(n.id)} style={{padding:"10px 14px",borderBottom:`1px solid ${C.line2}`,cursor:"pointer",background:n.read?"transparent":C.line2,display:"flex",gap:10,alignItems:"flex-start"}}>
          <div style={{width:3,minHeight:32,background:lv[n.level]||C.accent,borderRadius:2,flexShrink:0}}/>
          <div style={{flex:1}}>
            <div style={{fontSize:12,color:n.read?C.textMuted:C.textPrimary,lineHeight:1.5}}>{n.text}</div>
            <div style={{fontSize:10,color:C.textMuted,marginTop:3}}>{n.ts}</div>
          </div>
          {!n.read&&<div style={{width:7,height:7,borderRadius:"50%",background:C.accent,marginTop:5,flexShrink:0}}/>}
        </div>
      ))}
    </div>
  );
}

// ─── DASHBOARD ────────────────────────────────────────────────
function Dashboard({issues,members,onView,onViewAll}){
  const byS=s=>issues.filter(i=>i.status===s).length;
  const total=issues.length,open=byS("รอดำเนินการ"),prog=byS("กำลังดำเนินการ"),rev=byS("รอตรวจสอบ"),done=byS("เสร็จแล้ว"),high=issues.filter(i=>i.pri==="High"&&i.status!=="เสร็จแล้ว").length;
  const CARDS=[
    {label:"Issue ทั้งหมด",  val:total,color:"#1a56db",bg:"#dbeafe"},
    {label:"รอดำเนินการ",    val:open, color:"#475569",bg:"#f1f5f9"},
    {label:"กำลังดำเนินการ", val:prog, color:"#1d4ed8",bg:"#dbeafe"},
    {label:"รอตรวจสอบ",      val:rev,  color:"#b45309",bg:"#fef3c7"},
    {label:"เสร็จแล้ว",      val:done, color:"#15803d",bg:"#dcfce7"},
    {label:"HIGH PRIORITY",  val:high, color:"#b91c1c",bg:"#fee2e2"},
  ];
  const recent=[...issues].sort((a,b)=>b.id-a.id).slice(0,5);
  const nearDL=issues.filter(i=>{const d=dLeft(i.due);return d!==null&&d<=3&&d>=0&&i.status!=="เสร็จแล้ว";});

  return (
    <div style={{display:"flex",flexDirection:"column",gap:16}}>
      {/* Stat Cards */}
      <div style={{display:"grid",gridTemplateColumns:"repeat(6,1fr)",gap:10}}>
        {CARDS.map(c=>(
          <div key={c.label} style={{background:c.bg,border:`1px solid ${c.color}33`,borderLeft:`4px solid ${c.color}`,borderRadius:8,padding:"14px 16px",boxShadow:"0 1px 4px rgba(0,0,0,.06)"}}>
            <div style={{fontSize:32,fontWeight:800,color:c.color,fontFamily:"monospace",lineHeight:1}}>{c.val}</div>
            <div style={{fontSize:11,color:c.color,marginTop:6,fontWeight:600,opacity:.85}}>{c.label}</div>
          </div>
        ))}
      </div>

      {/* Chart + Workload */}
      <div style={{display:"grid",gridTemplateColumns:"2fr 1fr",gap:16}}>
        <Panel>
          <SHdr label="TREND — รายเดือน (SUM) + รายสัปดาห์ปัจจุบัน" sub={`${MONTHS_TH[CUR_MONTH]} ${CUR_YEAR} — แยกสีตามสถานะ`}/>
          <DashboardChart issues={issues}/>
        </Panel>
        <Panel>
          <SHdr label="TEAM WORKLOAD"/>
          <div style={{padding:12}}>
            {members.map(m=>{
              const load=issues.filter(i=>i.asgn===m.id&&i.status!=="เสร็จแล้ว").length;
              const col=load>=3?C.red:load>=2?C.gold:C.green;
              return (
                <div key={m.id} style={{display:"flex",alignItems:"center",gap:8,marginBottom:10}}>
                  <Avt m={m} size={26}/>
                  <div style={{flex:1}}>
                    <div style={{display:"flex",justifyContent:"space-between",fontSize:11,marginBottom:4}}>
                      <span style={{color:C.textPrimary,fontWeight:500}}>{m.name}</span>
                      <span style={{fontFamily:"monospace",color:col,fontWeight:800}}>{load}</span>
                    </div>
                    <Bar pct={Math.min(load*25,100)} h={4} color={col}/>
                  </div>
                </div>
              );
            })}
          </div>
        </Panel>
      </div>

      {/* Recent + Near Deadline */}
      <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:16}}>
        <Panel>
          <SHdr label="RECENT ISSUES" right={<button onClick={onViewAll} style={{background:"none",border:"none",color:C.accent,fontSize:11,cursor:"pointer",fontFamily:"monospace",fontWeight:600}}>ดูทั้งหมด →</button>}/>
          {recent.map(i=>{
            const m=members.find(x=>x.id===i.asgn);
            return (
              <div key={i.id} className="row-h" onClick={()=>onView(i)} style={{display:"flex",alignItems:"center",gap:10,padding:"10px 14px",borderBottom:`1px solid ${C.line2}`,cursor:"pointer"}}>
                <span style={{fontFamily:"monospace",fontSize:10,color:C.textMuted,width:28,flexShrink:0}}>#{i.id}</span>
                <div style={{flex:1,minWidth:0}}>
                  <div style={{fontSize:12,fontWeight:600,color:C.textPrimary,whiteSpace:"nowrap",overflow:"hidden",textOverflow:"ellipsis"}}>{i.title}</div>
                  <div style={{display:"flex",gap:6,marginTop:3,alignItems:"center"}}>
                    <TypeChip type={i.type}/>
                    {m&&<span style={{fontSize:10,color:C.textMuted}}>{m.name}</span>}
                  </div>
                </div>
                <StatusChip status={i.status} sm/>
              </div>
            );
          })}
        </Panel>
        <Panel>
          <SHdr label="⚠ NEAR DEADLINE"/>
          {nearDL.length===0
            ?<div style={{padding:24,textAlign:"center",color:C.textMuted,fontSize:12,fontFamily:"monospace"}}>ไม่มี issue ใกล้ deadline</div>
            :nearDL.map(i=>{
              const d=dLeft(i.due);
              return (
                <div key={i.id} className="row-h" onClick={()=>onView(i)} style={{display:"flex",alignItems:"center",gap:10,padding:"10px 14px",borderBottom:`1px solid ${C.line2}`,cursor:"pointer",borderLeft:`3px solid ${d===0?C.red:C.gold}`}}>
                  <div style={{flex:1}}>
                    <div style={{fontSize:12,fontWeight:600,color:C.textPrimary}}>{i.title}</div>
                    <div style={{fontSize:10,color:d===0?C.red:C.gold,fontWeight:700,marginTop:2}}>{d===0?"TODAY!":"D-"+d+" ("+fmt(i.due)+")"}</div>
                  </div>
                  <PriChip pri={i.pri}/>
                </div>
              );
            })}
        </Panel>
      </div>
    </div>
  );
}

// ─── ISSUE LIST ───────────────────────────────────────────────
function IssueList({issues,members,types,onView,onUpdate,onDelete}){
  const [q,setQ]=useState("");
  const [fTyp,setFTyp]=useState("ALL");
  const [fSt,setFSt]=useState("ALL");
  const [fPri,setFPri]=useState("ALL");
  const [fAsgn,setFAsgn]=useState("ALL");
  const [editing,setEditing]=useState({});
  const [confirmDel,setConfirmDel]=useState(null);

  const list=issues.filter(i=>
    (i.title.toLowerCase().includes(q.toLowerCase())||i.desc?.toLowerCase().includes(q.toLowerCase()))
    &&(fTyp==="ALL"||i.type===fTyp)
    &&(fSt==="ALL"||i.status===fSt)
    &&(fPri==="ALL"||i.pri===fPri)
    &&(fAsgn==="ALL"||String(i.asgn)===String(fAsgn))
  );

  const startEdit=i=>setEditing(p=>({...p,[i.id]:i.status}));
  const save=i=>{
    if(editing[i.id]!==undefined&&editing[i.id]!==i.status) onUpdate({...i,status:editing[i.id]});
    setEditing(p=>{const n={...p};delete n[i.id];return n;});
  };

  const SEL_STYLE={minWidth:130,fontSize:12};

  return (
    <div style={{display:"flex",flexDirection:"column",gap:12}}>
      <Panel>
        <div style={{padding:"10px 14px",display:"flex",gap:8,alignItems:"center",flexWrap:"wrap"}}>
          <div style={{position:"relative",flex:"1 1 160px"}}>
            <span style={{position:"absolute",left:10,top:"50%",transform:"translateY(-50%)",color:C.textMuted,fontSize:14}}>⌕</span>
            <Inp value={q} onChange={e=>setQ(e.target.value)} placeholder="ค้นหา issue..." style={{paddingLeft:30}}/>
          </div>
          <Sel value={fTyp} onChange={e=>setFTyp(e.target.value)} style={SEL_STYLE}>
            <option value="ALL">ประเภท: ทั้งหมด</option>
            {types.map(t=><option key={t} value={t}>{t}</option>)}
          </Sel>
          <Sel value={fSt} onChange={e=>setFSt(e.target.value)} style={SEL_STYLE}>
            <option value="ALL">สถานะ: ทั้งหมด</option>
            {ALL_STATUSES.map(s=><option key={s} value={s}>{s}</option>)}
          </Sel>
          <Sel value={fPri} onChange={e=>setFPri(e.target.value)} style={{...SEL_STYLE,minWidth:110}}>
            <option value="ALL">Priority: ทั้งหมด</option>
            {ALL_PRI.map(p=><option key={p} value={p}>{p}</option>)}
          </Sel>
          <Sel value={fAsgn} onChange={e=>setFAsgn(e.target.value)} style={{...SEL_STYLE,minWidth:150}}>
            <option value="ALL">ผู้รับผิดชอบ: ทั้งหมด</option>
            {members.map(m=><option key={m.id} value={m.id}>{m.name}</option>)}
          </Sel>
          <span style={{fontSize:11,color:C.textMuted,marginLeft:"auto",whiteSpace:"nowrap"}}>{list.length}/{issues.length}</span>
        </div>
      </Panel>

      <Panel>
        <div style={{overflowX:"auto"}}>
          <table style={{width:"100%",borderCollapse:"collapse",fontSize:12}}>
            <thead>
              <tr style={{background:C.line2,borderBottom:`1px solid ${C.line}`}}>
                {["#","ชื่อปัญหา","ประเภท","Priority","ผู้รับผิดชอบ","วันกำหนด","ความคืบหน้า","สถานะ",""].map(h=>(
                  <th key={h} style={{padding:"9px 12px",textAlign:"left",fontSize:10,fontWeight:700,color:C.textSec,whiteSpace:"nowrap"}}>{h}</th>
                ))}
              </tr>
            </thead>
            <tbody>
              {list.map((i,idx)=>{
                const m=members.find(x=>x.id===i.asgn);
                const dl=dLeft(i.due);
                const urgent=dl!==null&&dl<=2&&i.status!=="เสร็จแล้ว";
                const isEd=i.id in editing;
                return (
                  <tr key={i.id} className="row-h" style={{borderBottom:`1px solid ${C.line2}`,background:idx%2===0?"transparent":"rgba(238,242,251,.4)"}}>
                    <td style={{padding:"10px 12px",fontFamily:"monospace",fontSize:10,color:C.textMuted}}>#{i.id}</td>
                    <td style={{padding:"10px 12px",maxWidth:220}}>
                      <button onClick={()=>onView(i)} style={{background:"none",border:"none",color:C.textPrimary,fontSize:12,fontWeight:600,textAlign:"left",cursor:"pointer",overflow:"hidden",textOverflow:"ellipsis",whiteSpace:"nowrap",maxWidth:220,display:"block"}}>{i.title}</button>
                    </td>
                    <td style={{padding:"10px 12px"}}><TypeChip type={i.type}/></td>
                    <td style={{padding:"10px 12px"}}><PriChip pri={i.pri}/></td>
                    <td style={{padding:"10px 12px"}}>
                      <div style={{display:"flex",alignItems:"center",gap:6}}>
                        <Avt m={m} size={22}/>
                        <span style={{fontSize:11,color:C.textSec,whiteSpace:"nowrap"}}>{m?.name||"—"}</span>
                      </div>
                    </td>
                    <td style={{padding:"10px 12px",fontFamily:"monospace",fontSize:11,color:urgent?C.red:C.textSec,fontWeight:urgent?700:400,whiteSpace:"nowrap"}}>
                      {fmt(i.due)}
                      {urgent&&<div style={{fontSize:9,color:C.red}}>⚠ อีก {dl} วัน</div>}
                    </td>
                    <td style={{padding:"10px 12px",width:110}}>
                      <div style={{fontSize:10,color:C.textMuted,marginBottom:3}}>{i.pct}%</div>
                      <Bar pct={i.pct}/>
                    </td>
                    <td style={{padding:"10px 12px"}}>
                      {isEd?(
                        <div style={{display:"flex",flexDirection:"column",gap:4}}>
                          <Sel value={editing[i.id]} onChange={e=>setEditing(p=>({...p,[i.id]:e.target.value}))} style={{fontSize:11,padding:"4px 6px",width:"100%"}}>
                            {ALL_STATUSES.map(s=><option key={s} value={s}>{s}</option>)}
                          </Sel>
                          <div style={{display:"flex",gap:4}}>
                            <Btn variant="primary" onClick={()=>save(i)} style={{flex:1,fontSize:10,padding:"4px 8px",justifyContent:"center"}}>บันทึก</Btn>
                            <Btn variant="ghost"   onClick={()=>setEditing(p=>{const n={...p};delete n[i.id];return n;})} style={{flex:1,fontSize:10,padding:"4px 8px",justifyContent:"center"}}>ยกเลิก</Btn>
                          </div>
                        </div>
                      ):(
                        <button onClick={()=>startEdit(i)} style={{background:"none",border:"none",cursor:"pointer"}} title="คลิกเพื่อเปลี่ยนสถานะ">
                          <StatusChip status={i.status} sm/>
                        </button>
                      )}
                    </td>
                    <td style={{padding:"10px 12px"}}>
                      <button onClick={()=>setConfirmDel(i.id)} style={{background:"none",border:`1px solid ${C.line}`,color:C.red,padding:"4px 8px",borderRadius:6,fontSize:12,cursor:"pointer"}}>🗑</button>
                    </td>
                  </tr>
                );
              })}
              {list.length===0&&<tr><td colSpan={9} style={{padding:40,textAlign:"center",color:C.textMuted,fontFamily:"monospace",fontSize:12}}>— ไม่พบข้อมูล —</td></tr>}
            </tbody>
          </table>
        </div>
      </Panel>

      {confirmDel&&(
        <div style={{position:"fixed",inset:0,background:"rgba(15,23,42,.5)",display:"flex",alignItems:"center",justifyContent:"center",zIndex:200}}>
          <Panel style={{padding:28,maxWidth:360,width:"100%",textAlign:"center"}} className="fade">
            <div style={{fontSize:36,marginBottom:12}}>🗑️</div>
            <div style={{fontSize:15,fontWeight:700,color:C.textPrimary,marginBottom:6}}>ลบ Issue นี้?</div>
            <div style={{fontSize:12,color:C.textMuted,marginBottom:20}}>การลบไม่สามารถย้อนกลับได้</div>
            <div style={{display:"flex",gap:10}}>
              <Btn variant="ghost"  onClick={()=>setConfirmDel(null)} style={{flex:1,justifyContent:"center"}}>ยกเลิก</Btn>
              <Btn variant="danger" onClick={()=>{onDelete(confirmDel);setConfirmDel(null);}} style={{flex:1,justifyContent:"center"}}>ลบเลย</Btn>
            </div>
          </Panel>
        </div>
      )}
    </div>
  );
}

// ─── KANBAN ───────────────────────────────────────────────────
function Kanban({issues,members,onUpdate,onView}){
  const [dragging,setDragging]=useState(null);
  const [over,setOver]=useState(null);
  return (
    <div style={{display:"grid",gridTemplateColumns:"repeat(4,1fr)",gap:12}}>
      {ALL_STATUSES.map(st=>{
        const col=issues.filter(i=>i.status===st);
        const cfg=STATUS_CFG[st];
        return (
          <div key={st}
            onDragOver={e=>{e.preventDefault();setOver(st);}}
            onDrop={e=>{e.preventDefault();if(dragging&&dragging.status!==st)onUpdate({...dragging,status:st});setDragging(null);setOver(null);}}
            onDragLeave={()=>setOver(null)}>
            <div style={{background:C.panel,border:`1px solid ${C.line}`,borderTop:`3px solid ${cfg.color}`,borderRadius:"8px 8px 0 0",padding:"10px 14px",display:"flex",alignItems:"center",justifyContent:"space-between"}}>
              <span style={{fontSize:12,fontWeight:700,color:C.textPrimary}}>{st}</span>
              <span style={{fontSize:12,fontWeight:800,color:cfg.color,background:cfg.bg,border:`1px solid ${cfg.border}`,padding:"1px 8px",borderRadius:10}}>{col.length}</span>
            </div>
            <div style={{minHeight:400,padding:6,background:over===st?C.line2:"transparent",border:over===st?`2px dashed ${C.accent}`:"2px dashed transparent",borderRadius:"0 0 8px 8px",display:"flex",flexDirection:"column",gap:8,transition:"all .15s"}}>
              {col.map(i=>{
                const m=members.find(x=>x.id===i.asgn);
                const dl=dLeft(i.due);
                const urgent=dl!==null&&dl<=2&&i.status!=="เสร็จแล้ว";
                return (
                  <div key={i.id} className="k-card"
                    draggable onDragStart={()=>setDragging(i)} onDragEnd={()=>{setDragging(null);setOver(null);}}
                    onClick={()=>onView(i)}
                    style={{background:"#fff",border:`1px solid ${urgent?C.red:C.line}`,borderRadius:8,padding:"10px 12px",cursor:"pointer",borderLeft:`4px solid ${urgent?C.red:PRI_CFG[i.pri]?.color||C.line}`,opacity:dragging?.id===i.id?.5:1,boxShadow:"0 1px 3px rgba(0,0,0,.06)"}}>
                    <div style={{display:"flex",justifyContent:"space-between",marginBottom:6}}>
                      <span style={{fontFamily:"monospace",fontSize:9,color:C.textMuted}}>#{i.id}</span>
                      <PriChip pri={i.pri}/>
                    </div>
                    <div style={{fontSize:12,fontWeight:600,color:C.textPrimary,lineHeight:1.4,marginBottom:6}}>{i.title}</div>
                    {i.img&&<div style={{marginBottom:8,borderRadius:6,overflow:"hidden",border:`1px solid ${C.line}`,height:80}}><img src={i.img} alt="" style={{width:"100%",height:"100%",objectFit:"cover"}}/></div>}
                    <TypeChip type={i.type}/>
                    <div style={{marginTop:8}}>
                      <div style={{display:"flex",justifyContent:"space-between",fontSize:10,marginBottom:3}}>
                        <span style={{color:C.textMuted}}>PROGRESS</span>
                        <span style={{fontFamily:"monospace",color:C.textSec}}>{i.pct}%</span>
                      </div>
                      <Bar pct={i.pct}/>
                    </div>
                    <div style={{display:"flex",alignItems:"center",justifyContent:"space-between",marginTop:8}}>
                      <div style={{display:"flex",alignItems:"center",gap:5}}>
                        <Avt m={m} size={20}/>
                        <span style={{fontSize:10,color:C.textSec}}>{m?.name?.split(" ")[0]||"—"}</span>
                      </div>
                      <span style={{fontSize:10,fontFamily:"monospace",color:urgent?C.red:C.textMuted}}>{fmt(i.due)}</span>
                    </div>
                  </div>
                );
              })}
              {col.length===0&&<div style={{padding:20,textAlign:"center",color:C.textMuted,fontSize:11,fontFamily:"monospace",border:`1px dashed ${C.line}`,borderRadius:6}}>DROP HERE</div>}
            </div>
          </div>
        );
      })}
    </div>
  );
}

// ─── GANTT VIEW ───────────────────────────────────────────────
function GanttView({issues,members}){
  return (
    <Panel>
      <SHdr label={`GANTT CHART — ${CUR_YEAR}`} right={<span style={{fontSize:10,color:C.textMuted}}>วันนี้: {new Date().toLocaleDateString("th-TH")}</span>}/>
      <div style={{overflowX:"auto"}}>
        <table style={{borderCollapse:"collapse",fontSize:11,minWidth:1000}}>
          <thead>
            <tr>
              <th style={{width:200,padding:"6px 12px",background:C.line2,border:`1px solid ${C.line}`,textAlign:"left",fontSize:11,fontWeight:700,color:C.textSec}} rowSpan={2}>Issue</th>
              <th colSpan={48} style={{padding:"5px",background:C.accent,color:"#fff",border:`1px solid ${C.line}`,textAlign:"center",fontSize:11,fontWeight:700}}>{CUR_YEAR}</th>
            </tr>
            <tr>
              {MONTHS_EN.map((mo,mi)=>(
                <th key={mi} colSpan={4} style={{padding:"4px 2px",background:C.line2,border:`1px solid ${C.line}`,textAlign:"center",fontSize:10,fontWeight:700,color:C.textSec}}>{mo}</th>
              ))}
            </tr>
            <tr>
              <th style={{padding:"3px 8px",background:"#f1f5f9",border:`1px solid ${C.line}`,fontSize:9,color:C.textMuted}}></th>
              {MONTHS_EN.map((_,mi)=>[1,2,3,4].map(w=>(
                <th key={`${mi}-${w}`} style={{padding:"3px 1px",background:"#f1f5f9",border:`1px solid ${C.line}`,textAlign:"center",fontSize:8,color:C.textMuted,width:14}}>W{w}</th>
              )))}
            </tr>
          </thead>
          <tbody>
            {issues.map((i,idx)=>{
              const m=members.find(x=>x.id===i.asgn);
              // merge all plan rows cells
              const allCells={};
              (i.planRows||[]).forEach(r=>Object.assign(allCells,r.cells||{}));
              return (
                <tr key={i.id} style={{background:idx%2===0?"#fff":"#f8faff",borderBottom:`1px solid ${C.line2}`}}>
                  <td style={{padding:"7px 12px",border:`1px solid ${C.line}`}}>
                    <div style={{display:"flex",alignItems:"center",gap:6}}>
                      <Avt m={m} size={20}/>
                      <div style={{minWidth:0}}>
                        <div style={{fontSize:11,fontWeight:600,color:C.textPrimary,whiteSpace:"nowrap",overflow:"hidden",textOverflow:"ellipsis",maxWidth:150}}>{i.title}</div>
                        <StatusChip status={i.status} sm/>
                      </div>
                    </div>
                  </td>
                  {MONTHS_EN.map((_,mi)=>[1,2,3,4].map(w=>{
                    const key=planKey(mi,w);
                    const active=!!allCells[key];
                    return <td key={key} style={{border:`1px solid ${C.line}`,background:active?"#1a56db":"#fff",width:14,height:22,padding:0}}/>;
                  }))}
                </tr>
              );
            })}
          </tbody>
        </table>
      </div>
    </Panel>
  );
}

// ─── ISSUE DETAIL ─────────────────────────────────────────────
function Detail({issue,members,onUpdate,onDelete,onBack,callAI}){
  const [cmt,setCmt]=useState("");
  const [confirmDel,setConfirmDel]=useState(false);
  const [editActId,setEditActId]=useState(null); // id ของ activity ที่กำลังแก้
  const [editActVal,setEditActVal]=useState("");
  const m=members.find(x=>x.id===issue.asgn);

  const addCmt=()=>{
    if(!cmt.trim()) return;
    onUpdate({...issue,acts:[...(issue.acts||[]),{id:Date.now(),kind:"comment",txt:`💬 "${cmt}"`,date:new Date().toISOString().split("T")[0],by:"ผู้ใช้"}]});
    setCmt("");
  };

  // แก้ไข activity
  const startEditAct=(act)=>{setEditActId(act.id);setEditActVal(act.txt);};
  const saveEditAct=(actId)=>{
    onUpdate({...issue,acts:(issue.acts||[]).map(a=>a.id===actId?{...a,txt:editActVal}:a)});
    setEditActId(null);setEditActVal("");
  };
  const deleteAct=(actId)=>{
    onUpdate({...issue,acts:(issue.acts||[]).filter(a=>a.id!==actId)});
  };

  return (
    <div style={{maxWidth:1100}}>
      <div style={{display:"flex",alignItems:"center",justifyContent:"space-between",marginBottom:16}}>
        <button onClick={onBack} style={{background:"none",border:`1px solid ${C.line}`,color:C.textSec,padding:"6px 14px",fontSize:11,fontFamily:"monospace",cursor:"pointer",borderRadius:6}}>← กลับ</button>
        <Btn variant="danger" onClick={()=>setConfirmDel(true)} style={{fontSize:11}}>🗑 ลบ Issue</Btn>
      </div>

      <div style={{display:"grid",gridTemplateColumns:"1fr 290px",gap:16}}>
        {/* Left */}
        <div style={{display:"flex",flexDirection:"column",gap:12}}>

          {/* Header */}
          <Panel>
            <div style={{padding:20}}>
              <div style={{display:"flex",alignItems:"flex-start",justifyContent:"space-between",gap:12,marginBottom:12}}>
                <div>
                  <div style={{display:"flex",alignItems:"center",gap:8,marginBottom:8,flexWrap:"wrap"}}>
                    <span style={{fontFamily:"monospace",fontSize:11,color:C.textMuted}}>#{issue.id}</span>
                    <TypeChip type={issue.type}/><PriChip pri={issue.pri}/>
                  </div>
                  <h2 style={{fontSize:17,fontWeight:700,color:C.textPrimary,lineHeight:1.3}}>{issue.title}</h2>
                </div>
                <StatusChip status={issue.status}/>
              </div>
              <p style={{fontSize:13,color:C.textSec,lineHeight:1.75}}>{issue.desc}</p>
              {issue.img&&<img src={issue.img} alt="" style={{marginTop:12,maxHeight:200,borderRadius:8,border:`1px solid ${C.line}`}}/>}
            </div>
          </Panel>

          {/* Progress */}
          <Panel>
            <SHdr label="PROGRESS"/>
            <div style={{padding:16,display:"flex",alignItems:"center",gap:16}}>
              <div style={{flex:1}}>
                <div style={{display:"flex",justifyContent:"space-between",fontSize:11,marginBottom:6}}>
                  <span style={{color:C.textMuted}}>COMPLETION</span>
                  <span style={{color:issue.pct===100?C.green:C.accent,fontWeight:800,fontFamily:"monospace"}}>{issue.pct}%</span>
                </div>
                <Bar pct={issue.pct} h={10}/>
              </div>
              <Inp type="number" min={0} max={100} value={issue.pct}
                onChange={e=>onUpdate({...issue,pct:Math.min(100,Math.max(0,Number(e.target.value)))})}
                style={{width:64,textAlign:"center",fontFamily:"monospace",fontWeight:800,fontSize:14}}/>
              <span style={{fontSize:11,color:C.textMuted}}>%</span>
            </div>
          </Panel>

          {/* Plan Grid */}
          <Panel>
            <SHdr label={`PLAN ${CUR_YEAR}`} sub="คลิกช่องเพื่อวางแผน · แก้ไขรายละเอียดได้โดยตรง · + เพิ่มรายละเอียดแถวใหม่"/>
            <div style={{padding:14,overflowX:"auto"}}>
              <PlanGrid
                planRows={issue.planRows||[]}
                onChange={rows=>onUpdate({...issue,planRows:rows})}
              />
            </div>
          </Panel>

          {/* Activity Log */}
          <Panel>
            <SHdr label="ACTIVITY LOG" sub="แก้ไขหรือลบรายการได้"/>
            <div style={{padding:16}}>
              {(issue.acts||[]).length===0
                ?<div style={{textAlign:"center",color:C.textMuted,fontSize:11,padding:16,fontStyle:"italic"}}>— ยังไม่มีกิจกรรม —</div>
                :(issue.acts||[]).map(a=>(
                  <div key={a.id} style={{display:"flex",gap:10,marginBottom:12,alignItems:"flex-start"}}>
                    <div style={{width:28,height:28,background:C.line2,border:`1px solid ${C.line}`,display:"flex",alignItems:"center",justifyContent:"center",fontSize:12,color:C.textMuted,flexShrink:0,borderRadius:6}}>
                      {a.kind==="status"?"🔄":a.kind==="comment"?"💬":"📎"}
                    </div>
                    <div style={{flex:1,borderBottom:`1px solid ${C.line2}`,paddingBottom:10}}>
                      {editActId===a.id?(
                        <div style={{display:"flex",gap:6}}>
                          <Inp value={editActVal} onChange={e=>setEditActVal(e.target.value)} style={{flex:1,fontSize:12}}/>
                          <Btn variant="primary" onClick={()=>saveEditAct(a.id)} style={{fontSize:10,padding:"4px 10px"}}>บันทึก</Btn>
                          <Btn variant="ghost"   onClick={()=>setEditActId(null)} style={{fontSize:10,padding:"4px 10px"}}>ยกเลิก</Btn>
                        </div>
                      ):(
                        <div style={{fontSize:12,color:C.textPrimary}}>{a.txt}</div>
                      )}
                      <div style={{display:"flex",alignItems:"center",gap:8,marginTop:4}}>
                        <span style={{fontSize:10,color:C.textMuted}}>{a.by} · {fmt(a.date)}</span>
                        <button onClick={()=>startEditAct(a)} style={{background:"none",border:"none",color:C.accent,fontSize:10,cursor:"pointer",fontFamily:"monospace"}}>✏ แก้ไข</button>
                        <button onClick={()=>deleteAct(a.id)}  style={{background:"none",border:"none",color:C.red,fontSize:10,cursor:"pointer",fontFamily:"monospace"}}>✕ ลบ</button>
                      </div>
                    </div>
                  </div>
                ))}
              {/* Comment input */}
              <div style={{display:"flex",gap:8,marginTop:8}}>
                <Inp value={cmt} onChange={e=>setCmt(e.target.value)} placeholder="เพิ่มความคิดเห็น..." onKeyDown={e=>e.key==="Enter"&&addCmt()} style={{flex:1,fontSize:12}}/>
                <Btn variant="primary" onClick={addCmt} style={{whiteSpace:"nowrap"}}>ส่ง</Btn>
              </div>
            </div>
          </Panel>
        </div>

        {/* Right sidebar */}
        <div style={{display:"flex",flexDirection:"column",gap:12}}>
          <Panel>
            <SHdr label="ISSUE INFO"/>
            <div style={{padding:12}}>
              {[["ID",`#${issue.id}`],["START",fmt(issue.s)],["DUE",fmt(issue.due)],["TYPE",issue.type],["PRIORITY",issue.pri]].map(([k,v])=>(
                <div key={k} style={{display:"flex",justifyContent:"space-between",padding:"7px 0",borderBottom:`1px solid ${C.line2}`,fontSize:11}}>
                  <span style={{fontFamily:"monospace",fontSize:9,letterSpacing:"0.1em",color:C.textMuted}}>{k}</span>
                  <span style={{color:C.textPrimary,fontWeight:600}}>{v}</span>
                </div>
              ))}
            </div>
          </Panel>

          <Panel>
            <SHdr label="ASSIGNED TO"/>
            <div style={{padding:14}}>
              {m?(
                <div style={{display:"flex",alignItems:"center",gap:10}}>
                  <Avt m={m} size={36}/>
                  <div>
                    <div style={{fontSize:14,fontWeight:700,color:C.textPrimary}}>{m.name}</div>
                    <div style={{fontSize:10,color:C.textMuted,marginTop:2}}>{m.role}</div>
                    <div style={{display:"flex",flexWrap:"wrap",gap:4,marginTop:6}}>
                      {m.skills.map(s=>(
                        <span key={s} style={{fontSize:9,background:C.line2,color:C.accent,padding:"2px 6px",borderRadius:3,fontFamily:"monospace"}}>{s}</span>
                      ))}
                    </div>
                  </div>
                </div>
              ):<div style={{color:C.textMuted,fontSize:11,padding:4}}>ยังไม่ได้มอบหมาย</div>}
            </div>
          </Panel>

          {/* Change Status — แก้ไข/ลบประวัติได้ */}
          <Panel>
            <SHdr label="CHANGE STATUS" sub="คลิกเพื่อเปลี่ยน · ประวัติแก้ไข/ลบได้"/>
            <div style={{padding:12,display:"flex",flexDirection:"column",gap:6}}>
              {ALL_STATUSES.map(s=>{
                const cfg=STATUS_CFG[s];
                const active=issue.status===s;
                return (
                  <button key={s}
                    onClick={()=>{
                      if(active) return;
                      const newAct={id:Date.now(),kind:"status",txt:`เปลี่ยนสถานะ → ${s}`,date:new Date().toISOString().split("T")[0],by:"ผู้ใช้"};
                      onUpdate({...issue,status:s,acts:[...(issue.acts||[]),newAct]});
                    }}
                    style={{display:"flex",alignItems:"center",gap:8,padding:"9px 12px",borderRadius:7,cursor:active?"default":"pointer",background:active?cfg.bg:"transparent",border:`1px solid ${active?cfg.border:C.line2}`,color:active?cfg.color:C.textSec,fontWeight:active?700:400,fontSize:12,fontFamily:"inherit",textAlign:"left",transition:"all .15s"}}>
                    <span style={{width:8,height:8,borderRadius:"50%",background:active?cfg.color:C.line,display:"inline-block",flexShrink:0}}/>
                    {s}
                    {active&&<span style={{marginLeft:"auto",fontSize:9,fontFamily:"monospace",color:cfg.color,letterSpacing:"0.06em"}}>✓ CURRENT</span>}
                  </button>
                );
              })}
            </div>
          </Panel>
        </div>
      </div>

      {confirmDel&&(
        <div style={{position:"fixed",inset:0,background:"rgba(15,23,42,.5)",display:"flex",alignItems:"center",justifyContent:"center",zIndex:200}}>
          <Panel style={{padding:28,maxWidth:380,width:"100%",textAlign:"center"}} className="fade">
            <div style={{fontSize:36,marginBottom:12}}>🗑️</div>
            <div style={{fontSize:15,fontWeight:700,color:C.textPrimary,marginBottom:6}}>ลบ Issue #{issue.id}?</div>
            <div style={{fontSize:12,color:C.textSec,marginBottom:4,padding:"0 12px"}}>{issue.title}</div>
            <div style={{fontSize:11,color:C.textMuted,marginBottom:20}}>การลบไม่สามารถย้อนกลับได้</div>
            <div style={{display:"flex",gap:10}}>
              <Btn variant="ghost"  onClick={()=>setConfirmDel(false)} style={{flex:1,justifyContent:"center"}}>ยกเลิก</Btn>
              <Btn variant="danger" onClick={()=>{onDelete(issue.id);setConfirmDel(false);}} style={{flex:1,justifyContent:"center"}}>ลบเลย</Btn>
            </div>
          </Panel>
        </div>
      )}
    </div>
  );
}

// ─── NEW ISSUE MODAL ─────────────────────────────────────────
function NewIssueModal({onClose,onSave,types,setTypes,members,setMembers}){
  const [f,setF]=useState({title:"",desc:"",type:types[0],pri:"Medium",asgn:members[0]?.id||1,s:new Date().toISOString().split("T")[0],due:"",status:"รอดำเนินการ",planRows:[{id:1,label:"รายละเอียดแผนงาน",cells:{}}],img:null});
  const [showAddMember,setShowAddMember]=useState(false);
  const [newM,setNewM]=useState({name:"",role:"",abbr:""});
  const set=(k,v)=>setF(p=>({...p,[k]:v}));

  const handleImg=e=>{
    const file=e.target.files[0];if(!file) return;
    if(file.size>1048576){alert("รูปใหญ่เกิน 1MB");e.target.value="";return;}
    const r=new FileReader();r.onload=ev=>set("img",ev.target.result);r.readAsDataURL(file);
  };
  const addMember=()=>{
    if(!newM.name.trim()) return;
    const abbr=newM.abbr.trim()||newM.name.trim().slice(0,2);
    const nm={id:Date.now(),name:newM.name.trim(),role:newM.role.trim()||"Staff",abbr,skills:[]};
    setMembers(p=>[...p,nm]);set("asgn",nm.id);setNewM({name:"",role:"",abbr:""});setShowAddMember(false);
  };
  const addType=()=>{
    const n=prompt("ชื่อประเภทใหม่:");
    if(n&&n.trim()&&!types.includes(n.trim())){setTypes(p=>[...p,n.trim()]);set("type",n.trim());}
  };

  return (
    <div style={{position:"fixed",inset:0,background:"rgba(15,23,42,.55)",display:"flex",alignItems:"center",justifyContent:"center",zIndex:200,padding:20}}>
      <Panel style={{width:"100%",maxWidth:960,maxHeight:"92vh",overflow:"hidden",display:"flex",flexDirection:"column"}} className="fade">

        {/* Header */}
        <div style={{padding:"14px 20px",background:C.accent,display:"flex",justifyContent:"space-between",alignItems:"center",borderRadius:"10px 10px 0 0",flexShrink:0}}>
          <span style={{fontSize:13,fontWeight:800,color:"#fff",letterSpacing:"0.08em",fontFamily:"monospace"}}>+ CREATE NEW ISSUE</span>
          <button onClick={onClose} style={{background:"rgba(255,255,255,.2)",border:"none",color:"#fff",fontSize:16,cursor:"pointer",width:28,height:28,borderRadius:6,display:"flex",alignItems:"center",justifyContent:"center"}}>✕</button>
        </div>

        <div style={{overflow:"auto",flex:1}}>
          <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:0}}>
            {/* Left column */}
            <div style={{padding:20,borderRight:`1px solid ${C.line}`,display:"flex",flexDirection:"column",gap:14}}>
              <div>
                <label style={{display:"block",fontSize:12,fontWeight:600,color:C.textSec,marginBottom:6}}>ชื่อปัญหา *</label>
                <Inp value={f.title} onChange={e=>set("title",e.target.value)} placeholder="ระบุชื่อปัญหา..."/>
              </div>
              <div>
                <label style={{display:"block",fontSize:12,fontWeight:600,color:C.textSec,marginBottom:6}}>รายละเอียด</label>
                <textarea value={f.desc} onChange={e=>set("desc",e.target.value)} rows={3} placeholder="อธิบายรายละเอียด..."
                  style={{background:"#fff",border:`1px solid ${C.line}`,color:C.textPrimary,padding:"8px 10px",fontSize:13,width:"100%",borderRadius:7,resize:"vertical",fontFamily:"inherit",boxSizing:"border-box"}}/>
              </div>
              <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:12}}>
                <div>
                  <label style={{display:"block",fontSize:12,fontWeight:600,color:C.textSec,marginBottom:6}}>ประเภท</label>
                  <div style={{display:"flex",gap:6}}>
                    <Sel value={f.type} onChange={e=>set("type",e.target.value)} style={{flex:1,fontSize:12}}>
                      {types.map(t=><option key={t} value={t}>{t}</option>)}
                    </Sel>
                    <Btn variant="ghost" onClick={addType} style={{padding:"7px 10px",fontSize:13}}>+</Btn>
                  </div>
                </div>
                <div>
                  <label style={{display:"block",fontSize:12,fontWeight:600,color:C.textSec,marginBottom:6}}>ความสำคัญ</label>
                  <div style={{display:"flex",gap:5}}>
                    {ALL_PRI.map(p=>{const c=PRI_CFG[p];return(
                      <button key={p} onClick={()=>set("pri",p)} style={{flex:1,padding:"7px 4px",borderRadius:6,cursor:"pointer",fontSize:11,fontWeight:700,background:f.pri===p?c.bg:"#fff",color:f.pri===p?c.color:C.textMuted,border:`1px solid ${f.pri===p?c.border:C.line}`}}>{p}</button>
                    );})}
                  </div>
                </div>
              </div>
              <div>
                <label style={{display:"block",fontSize:12,fontWeight:600,color:C.textSec,marginBottom:6}}>ผู้รับผิดชอบ</label>
                <div style={{display:"flex",gap:6}}>
                  <Sel value={f.asgn} onChange={e=>set("asgn",Number(e.target.value))} style={{flex:1,fontSize:12}}>
                    {members.map(m=><option key={m.id} value={m.id}>{m.name}</option>)}
                  </Sel>
                  <Btn variant="ghost" onClick={()=>setShowAddMember(!showAddMember)} style={{padding:"7px 10px",fontSize:13}} title="เพิ่มผู้รับผิดชอบใหม่">+</Btn>
                </div>
                {showAddMember&&(
                  <div style={{marginTop:8,padding:12,background:C.line2,borderRadius:8,border:`1px solid ${C.line}`,display:"flex",flexDirection:"column",gap:7}}>
                    <div style={{fontSize:11,fontWeight:700,color:C.textSec}}>➕ เพิ่มผู้รับผิดชอบใหม่</div>
                    <Inp placeholder="ชื่อ-นามสกุล *" value={newM.name} onChange={e=>setNewM(p=>({...p,name:e.target.value}))} style={{fontSize:12}}/>
                    <Inp placeholder="ตำแหน่ง เช่น Engineer" value={newM.role} onChange={e=>setNewM(p=>({...p,role:e.target.value}))} style={{fontSize:12}}/>
                    <Inp placeholder="ตัวย่อ 2 ตัว เช่น สช" value={newM.abbr} onChange={e=>setNewM(p=>({...p,abbr:e.target.value}))} style={{fontSize:12}} maxLength={2}/>
                    <div style={{display:"flex",gap:6}}>
                      <Btn variant="primary" onClick={addMember} style={{flex:1,justifyContent:"center",fontSize:11}}>เพิ่ม</Btn>
                      <Btn variant="ghost"   onClick={()=>setShowAddMember(false)} style={{flex:1,justifyContent:"center",fontSize:11}}>ยกเลิก</Btn>
                    </div>
                  </div>
                )}
              </div>
              <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:12}}>
                <div>
                  <label style={{display:"block",fontSize:12,fontWeight:600,color:C.textSec,marginBottom:6}}>วันเริ่มต้น</label>
                  <Inp type="date" value={f.s} onChange={e=>set("s",e.target.value)}/>
                </div>
                <div>
                  <label style={{display:"block",fontSize:12,fontWeight:600,color:C.textSec,marginBottom:6}}>วันกำหนดเสร็จ *</label>
                  <Inp type="date" value={f.due} onChange={e=>set("due",e.target.value)}/>
                </div>
              </div>
              <div>
                <label style={{display:"block",fontSize:12,fontWeight:600,color:C.textSec,marginBottom:6}}>📎 แนบรูปภาพ</label>
                <label style={{display:"flex",alignItems:"center",gap:12,background:C.line2,border:`2px dashed ${C.line}`,borderRadius:8,padding:"12px 14px",cursor:"pointer"}}>
                  <span style={{fontSize:22}}>🖼️</span>
                  <div>
                    <div style={{fontSize:12,color:C.textSec,fontWeight:500}}>คลิกเพื่อเลือกรูป</div>
                    <div style={{fontSize:10,color:C.textMuted}}>PNG, JPG ไม่เกิน 1MB</div>
                  </div>
                  <input type="file" accept="image/*" style={{display:"none"}} onChange={handleImg}/>
                </label>
                {f.img&&(
                  <div style={{marginTop:8,position:"relative",display:"inline-block"}}>
                    <img src={f.img} alt="" style={{maxHeight:100,borderRadius:8,border:`1px solid ${C.line}`}}/>
                    <button onClick={()=>set("img",null)} style={{position:"absolute",top:-6,right:-6,width:20,height:20,background:C.red,border:"none",borderRadius:"50%",color:"#fff",fontSize:11,cursor:"pointer",display:"flex",alignItems:"center",justifyContent:"center"}}>✕</button>
                  </div>
                )}
              </div>
            </div>

            {/* Right column — Plan Grid */}
            <div style={{padding:20,display:"flex",flexDirection:"column",gap:10}}>
              <div>
                <div style={{fontSize:12,fontWeight:700,color:C.textSec,marginBottom:4}}>📅 แผนงาน {CUR_YEAR}</div>
                <div style={{fontSize:11,color:C.textMuted,marginBottom:10}}>เพิ่มรายละเอียดได้หลายแถว · คลิกช่องสีน้ำเงิน = วางแผน · คลิกอีกครั้ง = ยกเลิก</div>
              </div>
              <div style={{border:`1px solid ${C.line}`,borderRadius:8,overflow:"hidden",background:"#fff",padding:12}}>
                <PlanGrid
                  planRows={f.planRows}
                  onChange={rows=>set("planRows",rows)}
                />
              </div>
            </div>
          </div>
        </div>

        {/* Footer */}
        <div style={{padding:"12px 20px",borderTop:`1px solid ${C.line}`,display:"flex",gap:10,background:C.line2,borderRadius:"0 0 10px 10px",flexShrink:0}}>
          <Btn variant="ghost"   onClick={onClose}  style={{flex:1,justifyContent:"center"}}>ยกเลิก</Btn>
          <Btn variant="primary" onClick={()=>{if(f.title.trim()&&f.due)onSave(f);}}
            disabled={!f.title.trim()||!f.due}
            style={{flex:2,justifyContent:"center",opacity:(!f.title.trim()||!f.due)?.4:1}}>
            ✓ สร้าง Issue
          </Btn>
        </div>
      </Panel>
    </div>
  );
}
