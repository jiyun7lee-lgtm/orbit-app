# orbit-app

import { useState, useEffect, useRef } from "react";

// ══════════════════════════════════════════════════
//  CONSTANTS
// ══════════════════════════════════════════════════
const ORBIT_R_BASE = [112, 182, 252, 322];
const ORBIT_SPEEDS = [0.13, 0.09, 0.065, 0.045];
const DEFAULT_ORBIT_NAMES = ["가장 가까운 곳", "소중한 인연", "느슨한 연결", "기억 속 어딘가"];
const PASTEL = ["#FFB3BA","#FFCBA4","#FFF4B3","#BAFFC9","#BAE1FF","#D4BAFF","#FFB3E6","#E0D4FF","#B3FFF0","#FFD4E8","#C8F0FF","#F0E8FF"];

const INITIAL_FRIENDS = [
  { id:1, name:"민지", affiliation:"대학 친구", mbti:"ENFP", orbit:0, color:"#FFB3BA" },
  { id:2, name:"준호", affiliation:"직장 동료",  mbti:"INTJ", orbit:1, color:"#BAE1FF" },
  { id:3, name:"서연", affiliation:"고등학교 친구", mbti:"ISFJ", orbit:2, color:"#BAFFC9" },
  { id:4, name:"다은", affiliation:"온라인 지인", mbti:"ENTP", orbit:3, color:"#FFCBA4" },
  { id:5, name:"현우", affiliation:"동네 친구",  mbti:"INTP", orbit:1, color:"#D4BAFF" },
];

// ══════════════════════════════════════════════════
//  UTILS
// ══════════════════════════════════════════════════
const dk = (hex, amt = 55) => {
  const n = parseInt(hex.replace("#",""), 16);
  const c = (v) => Math.max(0, v - amt).toString(16).padStart(2,"0");
  return `#${c(n>>16)}${c((n>>8)&0xff)}${c(n&0xff)}`;
};
const rad = (d) => d * Math.PI / 180;

// ══════════════════════════════════════════════════
//  PLANET SVG
// ══════════════════════════════════════════════════
function Planet({ mbti="INFP", color="#D4BAFF", size=80, glow=false, pid="p" }) {
  const m = (mbti||"INFP").toUpperCase().padEnd(4,"X");
  const E=m[0]==="E", N=m[1]==="N", F=m[2]==="F", J=m[3]==="J";
  const dark = dk(color);
  const gid=`g${pid}`, cid=`c${pid}`;
  return (
    <svg width={size} height={size} viewBox="0 0 100 100" style={{display:"block",
      filter: glow
        ? `drop-shadow(0 0 22px ${color}) drop-shadow(0 0 8px ${color})`
        : `drop-shadow(0 0 7px ${color}99)`}}>
      <defs>
        <radialGradient id={gid} cx="38%" cy="32%" r="65%">
          <stop offset="0%" stopColor="#fff" stopOpacity="0.68"/>
          <stop offset="50%" stopColor={color}/>
          <stop offset="100%" stopColor={dark}/>
        </radialGradient>
        <clipPath id={cid}><circle cx="50" cy="50" r="38"/></clipPath>
      </defs>
      {/* Atmosphere halo */}
      <circle cx="50" cy="50" r="47" fill={E ? color : "#9ab0ff"} opacity={E ? 0.15 : 0.08}/>
      {/* Body */}
      <circle cx="50" cy="50" r="38" fill={`url(#${gid})`}/>
      <g clipPath={`url(#${cid})`}>
        {/* S/N: dormant vs active volcano */}
        {N ? (<>
          <polygon points="50,13 42,33 58,33" fill="#6b4030" opacity="0.74"/>
          <polygon points="50,13 46,22 54,22" fill="#503020" opacity="0.6"/>
          <circle cx="50" cy="12" r="4" fill="#ff7244" opacity="0.92"/>
          <ellipse cx="50" cy="10" rx="6.5" ry="3" fill="#ffaa66" opacity="0.55"/>
        </>) : (<>
          <polygon points="50,17 43,33 57,33" fill="#888080" opacity="0.44"/>
          <polygon points="50,17 47,25 53,25" fill="#686060" opacity="0.36"/>
        </>)}
        {/* F/T: clouds vs mist */}
        {F ? (<>
          <ellipse cx="26" cy="53" rx="10" ry="6" fill="white" opacity="0.42"/>
          <ellipse cx="30" cy="50" rx="8" ry="5" fill="white" opacity="0.36"/>
          <ellipse cx="74" cy="61" rx="9" ry="5.5" fill="white" opacity="0.3"/>
          <ellipse cx="70" cy="58" rx="7" ry="4" fill="white" opacity="0.26"/>
        </>) : (<>
          <ellipse cx="50" cy="55" rx="36" ry="9" fill="none" stroke="rgba(160,205,255,0.3)" strokeWidth="1.5"/>
          <ellipse cx="50" cy="62" rx="31" ry="7" fill="none" stroke="rgba(160,205,255,0.2)" strokeWidth="1"/>
          <ellipse cx="50" cy="68" rx="24" ry="5" fill="none" stroke="rgba(160,205,255,0.12)" strokeWidth="0.8"/>
        </>)}
        {/* J/P: bricks vs grass */}
        {J ? (
          [16,26,36,55,65].map(x=>(
            <g key={x}>
              <rect x={x} y={66} width={9} height={4} rx="1.5" fill="#8a7050" opacity="0.4"/>
              <rect x={x+0.5} y={71} width={8} height={3.5} rx="1" fill="#7a6040" opacity="0.3"/>
            </g>))
        ) : (
          [19,25,31,57,63,69].map(x=>(
            <g key={x}>
              <line x1={x} y1="68" x2={x-2} y2="60" stroke="#7ec8a0" strokeWidth="1.5" opacity="0.55"/>
              <line x1={x+3} y1="68" x2={x+5} y2="59" stroke="#60c090" strokeWidth="1.5" opacity="0.44"/>
            </g>))
        )}
      </g>
      {/* Specular highlights */}
      <ellipse cx="37" cy="37" rx="9" ry="6" fill="white" opacity="0.26" transform="rotate(-15 37 37)"/>
      <ellipse cx="34" cy="34" rx="4.5" ry="3" fill="white" opacity="0.44" transform="rotate(-15 34 34)"/>
    </svg>
  );
}

// ══════════════════════════════════════════════════
//  ASTRONAUT
// ══════════════════════════════════════════════════
function Astronaut({ size=60 }) {
  return (
    <svg width={size} height={size} viewBox="0 0 60 60" style={{display:"block"}}>
      <ellipse cx="30" cy="43" rx="14" ry="14" fill="#ddddf2"/>
      <ellipse cx="30" cy="43" rx="10" ry="10" fill="#c8c8e2" opacity="0.7"/>
      <circle cx="30" cy="21" r="14" fill="#c0c0e0"/>
      <circle cx="30" cy="21" r="11" fill="#6890b8" opacity="0.7"/>
      <circle cx="30" cy="21" r="9" fill="#88aacc" opacity="0.5"/>
      <ellipse cx="26" cy="17" rx="4" ry="3" fill="white" opacity="0.4" transform="rotate(-20 26 17)"/>
      <ellipse cx="17" cy="41" rx="5.5" ry="11" fill="#cccce2" transform="rotate(-18 17 41)"/>
      <ellipse cx="43" cy="41" rx="5.5" ry="11" fill="#cccce2" transform="rotate(18 43 41)"/>
      <ellipse cx="25" cy="57" rx="5.5" ry="6.5" fill="#b8b8d2"/>
      <ellipse cx="35" cy="57" rx="5.5" ry="6.5" fill="#b8b8d2"/>
      <rect x="25.5" y="38" width="9" height="7" rx="2" fill="#9090b8" opacity="0.55"/>
    </svg>
  );
}

// ══════════════════════════════════════════════════
//  STAR FIELD
// ══════════════════════════════════════════════════
function StarField() {
  const stars = useRef(Array.from({length:110},()=>({
    x:`${(Math.random()*100).toFixed(2)}%`, y:`${(Math.random()*100).toFixed(2)}%`,
    r:(Math.random()*1.7+0.3).toFixed(2),
    op:(Math.random()*0.5+0.2).toFixed(2),
    dur:`${(Math.random()*3+2).toFixed(1)}s`,
    del:`${(Math.random()*6).toFixed(1)}s`,
  }))).current;
  return (
    <svg style={{position:"absolute",inset:0,width:"100%",height:"100%",pointerEvents:"none",zIndex:0}}>
      <defs>
        <radialGradient id="neb1" cx="25%" cy="35%"><stop offset="0%" stopColor="#3a1a6e" stopOpacity="0.45"/><stop offset="100%" stopColor="transparent"/></radialGradient>
        <radialGradient id="neb2" cx="75%" cy="65%"><stop offset="0%" stopColor="#1a3a5e" stopOpacity="0.35"/><stop offset="100%" stopColor="transparent"/></radialGradient>
        <radialGradient id="neb3" cx="60%" cy="20%"><stop offset="0%" stopColor="#4a1a4e" stopOpacity="0.2"/><stop offset="100%" stopColor="transparent"/></radialGradient>
      </defs>
      <ellipse cx="25%" cy="35%" rx="420" ry="320" fill="url(#neb1)"/>
      <ellipse cx="75%" cy="65%" rx="360" ry="270" fill="url(#neb2)"/>
      <ellipse cx="60%" cy="20%" rx="280" ry="200" fill="url(#neb3)"/>
      {stars.map((s,i)=>(
        <circle key={i} cx={s.x} cy={s.y} r={s.r} fill="white" opacity={s.op}>
          <animate attributeName="opacity" values={`${s.op};${(s.op*0.2).toFixed(2)};${s.op}`} dur={s.dur} repeatCount="indefinite" begin={s.del}/>
        </circle>
      ))}
    </svg>
  );
}

// ══════════════════════════════════════════════════
//  BACK BUTTON (shared)
// ══════════════════════════════════════════════════
const BackBtn = ({onClick}) => (
  <button onClick={onClick} style={{background:"rgba(255,255,255,0.07)",border:"1px solid rgba(255,255,255,0.13)",borderRadius:11,padding:"7px 15px",color:"rgba(255,255,255,0.65)",cursor:"pointer",fontSize:13,fontFamily:"inherit"}}>← 홈</button>
);

// ══════════════════════════════════════════════════
//  HOME SCREEN
// ══════════════════════════════════════════════════
function HomeScreen({ user, friends, visitingFriend, onNavigate }) {
  return (
    <div style={{width:"100%",height:"100%",display:"flex",flexDirection:"column",alignItems:"center",justifyContent:"center",position:"relative",zIndex:1}}>
      {/* Planet hero */}
      <div style={{position:"relative",marginBottom:28}}>
        <div style={{animation:"floatPlanet 4s ease-in-out infinite"}}>
          <Planet mbti={user.mbti} color={user.color} size={158} glow pid="home-user"/>
        </div>
        <div style={{position:"absolute",bottom:"86%",left:"50%",marginLeft:-27,animation:"floatAstro 3.5s ease-in-out infinite 0.7s"}}>
          <Astronaut size={54}/>
        </div>
        {visitingFriend && (
          <div style={{position:"absolute",top:8,right:-74,animation:"slideIn 0.5s ease-out forwards"}}>
            <Planet mbti={visitingFriend.mbti} color={visitingFriend.color} size={52} pid={`vis-${visitingFriend.id}`}/>
            <div style={{textAlign:"center",color:"rgba(255,255,255,0.5)",fontSize:10,marginTop:4,letterSpacing:"0.3px"}}>{visitingFriend.name}</div>
          </div>
        )}
      </div>

      <h1 style={{color:"white",fontSize:22,fontWeight:700,margin:"0 0 5px",letterSpacing:"-0.5px"}}>나의 우주</h1>
      <p style={{color:"rgba(255,255,255,0.28)",fontSize:12,margin:"0 0 32px",letterSpacing:"2px"}}>{user.mbti}</p>

      <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10,width:272}}>
        {[
          {icon:"🌌",label:"궤도 보기",sub:"관계의 지도",s:"orbit"},
          {icon:"📓",label:"일기 쓰기",sub:"오늘의 기록",s:"diary"},
          {icon:"📚",label:"일기 목록",sub:"지난 기억들",s:"diaryList"},
          {icon:"🪐",label:"행성 추가",sub:"새로운 인연",s:"addFriend"},
        ].map(item=>(
          <button key={item.s} onClick={()=>onNavigate(item.s)} className="menu-card" style={{
            background:"rgba(255,255,255,0.055)",border:"1px solid rgba(255,255,255,0.1)",
            borderRadius:16,padding:"16px 14px",color:"white",cursor:"pointer",
            textAlign:"left",backdropFilter:"blur(14px)",fontFamily:"inherit",
            transition:"background 0.22s, border-color 0.22s, transform 0.22s",
          }}
          onMouseEnter={e=>{e.currentTarget.style.background="rgba(255,255,255,0.1)";e.currentTarget.style.borderColor="rgba(255,255,255,0.22)";e.currentTarget.style.transform="translateY(-2px)";}}
          onMouseLeave={e=>{e.currentTarget.style.background="rgba(255,255,255,0.055)";e.currentTarget.style.borderColor="rgba(255,255,255,0.1)";e.currentTarget.style.transform="translateY(0)";}}>
            <div style={{fontSize:22,marginBottom:8}}>{item.icon}</div>
            <div style={{fontSize:13,fontWeight:600,marginBottom:3}}>{item.label}</div>
            <div style={{fontSize:11,opacity:0.35}}>{item.sub}</div>
          </button>
        ))}
      </div>
    </div>
  );
}

// ══════════════════════════════════════════════════
//  ORBIT SCREEN
// ══════════════════════════════════════════════════
function OrbitScreen({ user, friends, orbitNames, onPush, onBack, selectedFriend, setSelectedFriend }) {
  const PLANET_SIZE = 42;
  const HALF = PLANET_SIZE / 2;

  // Compute responsive orbit radii
  const [orbitR, setOrbitR] = useState(ORBIT_R_BASE);
  useEffect(()=>{
    const upd=()=>{
      const minDim = Math.min(window.innerWidth, window.innerHeight);
      const max = minDim * 0.44;
      const f = Math.min(1, max / ORBIT_R_BASE[3]);
      setOrbitR(ORBIT_R_BASE.map(r=>r*f));
    };
    upd();
    window.addEventListener("resize",upd);
    return ()=>window.removeEventListener("resize",upd);
  },[]);

  // Initialize angles evenly within each orbit
  const [angles, setAngles] = useState(()=>{
    const tot={0:0,1:0,2:0,3:0}, cnt={0:0,1:0,2:0,3:0};
    friends.forEach(f=>{tot[f.orbit]++;});
    const a={};
    friends.forEach(f=>{a[f.id]=(cnt[f.orbit]/Math.max(tot[f.orbit],1))*360; cnt[f.orbit]++;});
    return a;
  });

  // Animate
  useEffect(()=>{
    let frame;
    const tick=()=>{
      setAngles(prev=>{
        const next={};
        Object.entries(prev).forEach(([id,ang])=>{
          const f=friends.find(x=>x.id===parseInt(id));
          next[id]=(ang+(f?ORBIT_SPEEDS[f.orbit]:0.05))%360;
        });
        return next;
      });
      frame=requestAnimationFrame(tick);
    };
    frame=requestAnimationFrame(tick);
    return ()=>cancelAnimationFrame(frame);
  },[friends]);

  return (
    <div style={{width:"100%",height:"100%",position:"relative",overflow:"hidden"}}
      onClick={()=>setSelectedFriend(null)}>

      {/* Header */}
      <div style={{position:"absolute",top:0,left:0,right:0,padding:"18px 20px",display:"flex",alignItems:"center",zIndex:100}}>
        <BackBtn onClick={e=>{e.stopPropagation();onBack();}}/>
        <div style={{flex:1,textAlign:"center",color:"rgba(255,255,255,0.35)",fontSize:13,letterSpacing:"0.8px"}}>궤도 지도</div>
        <div style={{width:64}}/>
      </div>

      {/* Orbital system */}
      <div style={{position:"absolute",inset:0,display:"flex",alignItems:"center",justifyContent:"center"}}>

        {/* Rings */}
        {orbitR.map((r,i)=>(
          <div key={i} style={{
            position:"absolute",width:r*2,height:r*2,
            left:"50%",top:"50%",transform:"translate(-50%,-50%)",
            borderRadius:"50%",
            border:"1px solid rgba(180,200,255,0.12)",
            boxShadow:"0 0 14px rgba(180,200,255,0.03)",
            pointerEvents:"none",
          }}>
            <div style={{position:"absolute",top:-20,left:"50%",transform:"translateX(-50%)",
              color:"rgba(255,255,255,0.2)",fontSize:10,whiteSpace:"nowrap",letterSpacing:"0.8px"}}>
              {orbitNames[i]}
            </div>
          </div>
        ))}

        {/* User planet (center) */}
        <div style={{position:"absolute",left:"50%",top:"50%",transform:"translate(-50%,-50%)",zIndex:10}}>
          <div style={{animation:"floatPlanet 4s ease-in-out infinite"}}>
            <Planet mbti={user.mbti} color={user.color} size={74} glow pid="orbit-user"/>
          </div>
        </div>

        {/* Friend planets */}
        {friends.map(f=>{
          const r = orbitR[f.orbit] ?? 112;
          const angle = rad(angles[f.id]??0);
          const x = r * Math.cos(angle);
          const y = r * Math.sin(angle);
          const isSel = selectedFriend?.id === f.id;
          const showBelow = y < 20; // popup below if planet in upper half

          return (
            <div key={f.id} style={{
              position:"absolute",
              left:`calc(50% + ${x}px - ${HALF}px)`,
              top:`calc(50% + ${y}px - ${HALF}px)`,
              zIndex:isSel?50:5,
            }}>
              <div onClick={e=>{e.stopPropagation();setSelectedFriend(isSel?null:f);}}
                style={{cursor:"pointer",display:"flex",flexDirection:"column",alignItems:"center"}}>
                <Planet mbti={f.mbti} color={f.color} size={PLANET_SIZE} glow={isSel} pid={`f${f.id}`}/>
                <div style={{color:"rgba(255,255,255,0.52)",fontSize:9,marginTop:3,whiteSpace:"nowrap",letterSpacing:"0.3px"}}>{f.name}</div>
              </div>

              {isSel && (
                <div onClick={e=>e.stopPropagation()} style={{
                  position:"absolute",
                  [showBelow?"top":"bottom"]:"108%",
                  left:"50%",transform:"translateX(-50%)",
                  background:"rgba(9,7,21,0.94)",backdropFilter:"blur(22px)",
                  border:"1px solid rgba(255,255,255,0.12)",borderRadius:18,
                  padding:"16px 18px",minWidth:172,zIndex:60,
                  animation:"fadeUp 0.2s ease-out",
                }}>
                  <div style={{display:"flex",alignItems:"center",gap:10,marginBottom:12}}>
                    <Planet mbti={f.mbti} color={f.color} size={32} pid={`pop-${f.id}`}/>
                    <div>
                      <div style={{color:"white",fontWeight:700,fontSize:15}}>{f.name}</div>
                      <div style={{color:"rgba(255,255,255,0.38)",fontSize:11,marginTop:1}}>{f.affiliation}</div>
                    </div>
                  </div>
                  <div style={{display:"flex",justifyContent:"space-between",marginBottom:14}}>
                    <span style={{color:f.color,fontSize:12,fontWeight:600}}>{f.mbti}</span>
                    <span style={{color:"rgba(255,255,255,0.28)",fontSize:11}}>{orbitNames[f.orbit]}</span>
                  </div>
                  <div style={{display:"flex",gap:8}}>
                    {[{label:"← 가까이",dir:"in",dis:f.orbit===0},{label:"멀리 →",dir:"out",dis:f.orbit===3}].map(btn=>(
                      <button key={btn.dir} disabled={btn.dis}
                        onClick={()=>{onPush(f.id,btn.dir);setSelectedFriend(null);}}
                        style={{flex:1,padding:"9px 0",
                          background:"rgba(255,255,255,0.07)",
                          border:`1px solid ${btn.dis?"rgba(255,255,255,0.06)":"rgba(255,255,255,0.16)"}`,
                          borderRadius:10,color:btn.dis?"rgba(255,255,255,0.18)":"white",
                          fontSize:12,cursor:btn.dis?"default":"pointer",fontFamily:"inherit"}}>
                        {btn.label}
                      </button>
                    ))}
                  </div>
                </div>
              )}
            </div>
          );
        })}
      </div>
    </div>
  );
}

// ══════════════════════════════════════════════════
//  DIARY SCREEN
// ══════════════════════════════════════════════════
function DiaryScreen({ friends, onSave, onBack }) {
  const [content,setContent] = useState("");
  const [tagged,setTagged] = useState([]);
  const [status,setStatus] = useState("idle"); // idle | thinking | done
  const [result,setResult] = useState(null);

  const toggleTag = id => setTagged(p=>p.includes(id)?p.filter(x=>x!==id):[...p,id]);

  const analyze = async () => {
    if (!content.trim() || status==="thinking") return;
    setStatus("thinking");
    try {
      const names = tagged.map(id=>friends.find(f=>f.id===id)?.name).filter(Boolean).join(", ")||"없음";
      const res = await fetch("https://api.anthropic.com/v1/messages",{
        method:"POST",
        headers:{"Content-Type":"application/json"},
        body:JSON.stringify({
          model:"claude-sonnet-4-20250514",
          max_tokens:300,
          messages:[{role:"user",content:
`다음 일기의 감정을 분석해주세요. 태그된 사람: ${names}

일기:
"${content}"

마크다운 없이 JSON만 응답:
{"sentiment":"positive"|"negative"|"neutral","positiveKeywords":["단어"],"negativeKeywords":["단어"],"summary":"한 줄 감정 요약 (한국어, 20자 이내)"}`}]
        })
      });
      const data = await res.json();
      const text = data.content?.find(c=>c.type==="text")?.text||"{}";
      const parsed = JSON.parse(text.replace(/```[\w]*/g,"").replace(/```/g,"").trim());
      setResult(parsed);
      setStatus("done");
    } catch(_) {
      setResult({sentiment:"neutral",positiveKeywords:[],negativeKeywords:[],summary:"분석을 완료했어요"});
      setStatus("done");
    }
  };

  const sentCol = {positive:"#BAFFC9",negative:"#FFB3BA",neutral:"#BAE1FF"};
  const sentEmo  = {positive:"✨ 따뜻한 감정",negative:"💭 무거운 감정",neutral:"🌙 차분한 감정"};

  return (
    <div style={{width:"100%",height:"100%",display:"flex",flexDirection:"column",alignItems:"center",padding:"20px 20px 0",zIndex:1}}>
      <div style={{width:"100%",maxWidth:560,display:"flex",alignItems:"center",marginBottom:22}}>
        <BackBtn onClick={onBack}/>
        <div style={{flex:1,textAlign:"center",color:"rgba(255,255,255,0.4)",fontSize:14}}>오늘의 일기</div>
        <div style={{width:60}}/>
      </div>
      <div style={{width:"100%",maxWidth:560,flex:1,overflowY:"auto",display:"flex",flexDirection:"column",gap:14,paddingBottom:28}}>

        <textarea value={content} onChange={e=>setContent(e.target.value)}
          placeholder="오늘 어떤 하루였나요? 어떤 생각이 들었나요..."
          style={{width:"100%",minHeight:180,padding:"18px",
            background:"rgba(255,255,255,0.05)",border:"1px solid rgba(255,255,255,0.1)",
            borderRadius:18,color:"white",fontSize:14,lineHeight:"1.85",
            resize:"vertical",fontFamily:"inherit",outline:"none"}}/>

        {/* Tag friends */}
        <div style={{background:"rgba(255,255,255,0.04)",border:"1px solid rgba(255,255,255,0.08)",borderRadius:16,padding:"15px"}}>
          <div style={{color:"rgba(255,255,255,0.35)",fontSize:11,marginBottom:11,letterSpacing:"0.8px"}}>태그할 사람</div>
          <div style={{display:"flex",flexWrap:"wrap",gap:8}}>
            {friends.map(f=>(
              <button key={f.id} onClick={()=>toggleTag(f.id)} style={{
                padding:"7px 13px",borderRadius:20,cursor:"pointer",fontFamily:"inherit",fontSize:12,
                background:tagged.includes(f.id)?f.color:"rgba(255,255,255,0.07)",
                border:`1px solid ${tagged.includes(f.id)?f.color:"rgba(255,255,255,0.12)"}`,
                color:tagged.includes(f.id)?"#1a1530":"rgba(255,255,255,0.6)",
                fontWeight:tagged.includes(f.id)?600:400,transition:"all 0.15s",
              }}>
                {f.name} <span style={{opacity:0.6,fontSize:10}}>{f.mbti}</span>
              </button>
            ))}
          </div>
        </div>

        {/* Analyze */}
        {status !== "done" && (
          <button onClick={analyze} disabled={!content.trim()||status==="thinking"} style={{
            padding:"14px",borderRadius:14,fontFamily:"inherit",fontSize:14,fontWeight:600,
            cursor:content.trim()&&status!=="thinking"?"pointer":"default",transition:"all 0.2s",
            background:content.trim()&&status!=="thinking"?"rgba(180,160,255,0.22)":"rgba(255,255,255,0.04)",
            border:`1px solid ${content.trim()&&status!=="thinking"?"rgba(180,160,255,0.4)":"rgba(255,255,255,0.07)"}`,
            color:content.trim()&&status!=="thinking"?"#d4baff":"rgba(255,255,255,0.22)",
          }}>
            {status==="thinking" ? "✨ AI가 감정을 읽는 중..." : "🔍 감정 분석하기"}
          </button>
        )}

        {/* Result */}
        {status==="done" && result && (
          <div style={{background:"rgba(255,255,255,0.04)",border:`1px solid ${sentCol[result.sentiment]||"#BAE1FF"}33`,borderRadius:16,padding:"18px",animation:"fadeUp 0.3s ease-out"}}>
            <div style={{color:sentCol[result.sentiment],fontSize:13,fontWeight:600,marginBottom:9}}>
              {sentEmo[result.sentiment]}
            </div>
            <div style={{color:"rgba(255,255,255,0.58)",fontSize:13,lineHeight:"1.75",marginBottom:13}}>{result.summary}</div>
            <div style={{display:"flex",flexWrap:"wrap",gap:6,marginBottom:result.negativeKeywords?.length?10:0}}>
              {result.positiveKeywords?.map(k=><span key={k} style={{padding:"4px 10px",background:"rgba(186,255,201,0.14)",borderRadius:10,color:"#baffc9",fontSize:11}}>+{k}</span>)}
            </div>
            <div style={{display:"flex",flexWrap:"wrap",gap:6}}>
              {result.negativeKeywords?.map(k=><span key={k} style={{padding:"4px 10px",background:"rgba(255,179,186,0.14)",borderRadius:10,color:"#ffb3ba",fontSize:11}}>−{k}</span>)}
            </div>
            {tagged.length>0&&(
              <div style={{marginTop:13,color:"rgba(255,255,255,0.3)",fontSize:12}}>
                → {tagged.map(id=>friends.find(f=>f.id===id)?.name).filter(Boolean).join(", ")}의 궤도가&nbsp;
                {result.sentiment==="positive"?"조금 가까워집니다":result.sentiment==="negative"?"조금 멀어집니다":"유지됩니다"}
              </div>
            )}
          </div>
        )}

        <button onClick={()=>content.trim()&&onSave(content,tagged,result?.sentiment||"neutral")}
          disabled={!content.trim()} style={{
            padding:"15px",borderRadius:14,fontFamily:"inherit",fontSize:15,fontWeight:700,
            cursor:content.trim()?"pointer":"default",transition:"all 0.2s",
            background:content.trim()?"rgba(212,186,255,0.2)":"rgba(255,255,255,0.04)",
            border:`1px solid ${content.trim()?"rgba(212,186,255,0.4)":"rgba(255,255,255,0.07)"}`,
            color:content.trim()?"white":"rgba(255,255,255,0.2)",
          }}>
          저장하기
        </button>
      </div>
    </div>
  );
}

// ══════════════════════════════════════════════════
//  ADD FRIEND SCREEN
// ══════════════════════════════════════════════════
function AddFriendScreen({ orbitNames, onAdd, onBack }) {
  const [form,setForm] = useState({name:"",affiliation:"",mbti:"INFP",orbit:0,color:PASTEL[0]});
  const set = (k,v) => setForm(p=>({...p,[k]:v}));

  const mbtiPairs = [["E","I"],["S","N"],["F","T"],["J","P"]];

  return (
    <div style={{width:"100%",height:"100%",display:"flex",flexDirection:"column",alignItems:"center",padding:"20px",overflow:"auto",zIndex:1}}>
      <div style={{width:"100%",maxWidth:480}}>
        <div style={{display:"flex",alignItems:"center",marginBottom:26}}>
          <BackBtn onClick={onBack}/>
          <div style={{flex:1,textAlign:"center",color:"rgba(255,255,255,0.4)",fontSize:14}}>새 행성 등록</div>
          <div style={{width:60}}/>
        </div>

        {/* Live preview */}
        <div style={{display:"flex",justifyContent:"center",marginBottom:26}}>
          <div style={{display:"flex",flexDirection:"column",alignItems:"center",gap:10}}>
            <div style={{animation:"floatPlanet 3.5s ease-in-out infinite"}}>
              <Planet mbti={form.mbti} color={form.color} size={90} glow pid="preview"/>
            </div>
            <div style={{color:"rgba(255,255,255,0.45)",fontSize:12}}>{form.name||"이름을 입력하세요"}</div>
          </div>
        </div>

        <div style={{display:"flex",flexDirection:"column",gap:14}}>
          {/* Name */}
          <div>
            <label style={{color:"rgba(255,255,255,0.35)",fontSize:11,display:"block",marginBottom:8,letterSpacing:"0.8px"}}>이름</label>
            <input value={form.name} onChange={e=>set("name",e.target.value)} placeholder="이름을 입력하세요"
              style={{width:"100%",padding:"13px 16px",background:"rgba(255,255,255,0.05)",border:"1px solid rgba(255,255,255,0.1)",borderRadius:13,color:"white",fontSize:14,fontFamily:"inherit",outline:"none",boxSizing:"border-box"}}/>
          </div>
          {/* Affiliation */}
          <div>
            <label style={{color:"rgba(255,255,255,0.35)",fontSize:11,display:"block",marginBottom:8,letterSpacing:"0.8px"}}>소속</label>
            <input value={form.affiliation} onChange={e=>set("affiliation",e.target.value)} placeholder="예: 대학 친구, 직장 동료..."
              style={{width:"100%",padding:"13px 16px",background:"rgba(255,255,255,0.05)",border:"1px solid rgba(255,255,255,0.1)",borderRadius:13,color:"white",fontSize:14,fontFamily:"inherit",outline:"none",boxSizing:"border-box"}}/>
          </div>

          {/* MBTI selector */}
          <div>
            <label style={{color:"rgba(255,255,255,0.35)",fontSize:11,display:"block",marginBottom:8,letterSpacing:"0.8px"}}>MBTI</label>
            <div style={{display:"grid",gridTemplateColumns:"1fr 1fr 1fr 1fr",gap:8}}>
              {mbtiPairs.map((pair,i)=>(
                <div key={i} style={{display:"flex",borderRadius:10,overflow:"hidden",border:"1px solid rgba(255,255,255,0.12)"}}>
                  {pair.map(ch=>(
                    <button key={ch} onClick={()=>{const m=form.mbti.split("");m[i]=ch;set("mbti",m.join(""));}} style={{
                      flex:1,padding:"10px 0",border:"none",fontSize:13,cursor:"pointer",fontFamily:"inherit",transition:"all 0.15s",
                      background:form.mbti[i]===ch?"rgba(212,186,255,0.28)":"rgba(255,255,255,0.04)",
                      color:form.mbti[i]===ch?"white":"rgba(255,255,255,0.4)",
                      fontWeight:form.mbti[i]===ch?700:400,
                    }}>{ch}</button>
                  ))}
                </div>
              ))}
            </div>
          </div>

          {/* Color picker */}
          <div>
            <label style={{color:"rgba(255,255,255,0.35)",fontSize:11,display:"block",marginBottom:8,letterSpacing:"0.8px"}}>행성 색상</label>
            <div style={{display:"flex",gap:8,flexWrap:"wrap"}}>
              {PASTEL.map(c=>(
                <button key={c} onClick={()=>set("color",c)} style={{
                  width:30,height:30,borderRadius:"50%",background:c,
                  border:form.color===c?"3px solid white":"3px solid transparent",
                  cursor:"pointer",transition:"border 0.15s",padding:0,
                }}/>
              ))}
            </div>
          </div>

          {/* Starting orbit */}
          <div>
            <label style={{color:"rgba(255,255,255,0.35)",fontSize:11,display:"block",marginBottom:8,letterSpacing:"0.8px"}}>시작 궤도</label>
            <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:8}}>
              {orbitNames.map((name,i)=>(
                <button key={i} onClick={()=>set("orbit",i)} style={{
                  padding:"10px 8px",borderRadius:10,fontFamily:"inherit",fontSize:11,cursor:"pointer",transition:"all 0.15s",textAlign:"center",
                  background:form.orbit===i?"rgba(212,186,255,0.22)":"rgba(255,255,255,0.04)",
                  border:`1px solid ${form.orbit===i?"rgba(212,186,255,0.4)":"rgba(255,255,255,0.1)"}`,
                  color:form.orbit===i?"white":"rgba(255,255,255,0.4)",
                }}>{name}</button>
              ))}
            </div>
          </div>

          <button onClick={()=>form.name.trim()&&onAdd(form)} disabled={!form.name.trim()} style={{
            marginTop:8,padding:"15px",borderRadius:14,fontFamily:"inherit",fontSize:15,fontWeight:700,
            cursor:form.name.trim()?"pointer":"default",transition:"all 0.2s",
            background:form.name.trim()?"rgba(212,186,255,0.22)":"rgba(255,255,255,0.04)",
            border:`1px solid ${form.name.trim()?"rgba(212,186,255,0.4)":"rgba(255,255,255,0.07)"}`,
            color:form.name.trim()?"white":"rgba(255,255,255,0.22)",
          }}>
            🪐 궤도에 올리기
          </button>
        </div>
      </div>
    </div>
  );
}

// ══════════════════════════════════════════════════
//  DIARY LIST SCREEN
// ══════════════════════════════════════════════════
function DiaryListScreen({ diaries, friends, onBack }) {
  const sentCol = {positive:"#BAFFC9",negative:"#FFB3BA",neutral:"#BAE1FF"};
  const sentEmo  = {positive:"✨",negative:"💭",neutral:"🌙"};

  return (
    <div style={{width:"100%",height:"100%",display:"flex",flexDirection:"column",alignItems:"center",padding:"20px",zIndex:1}}>
      <div style={{width:"100%",maxWidth:520}}>
        <div style={{display:"flex",alignItems:"center",marginBottom:22}}>
          <BackBtn onClick={onBack}/>
          <div style={{flex:1,textAlign:"center",color:"rgba(255,255,255,0.4)",fontSize:14}}>일기 목록</div>
          <div style={{width:60}}/>
        </div>
        <div style={{overflowY:"auto",maxHeight:"calc(100vh - 100px)",display:"flex",flexDirection:"column",gap:12,paddingBottom:28}}>
          {diaries.length===0 ? (
            <div style={{textAlign:"center",color:"rgba(255,255,255,0.22)",fontSize:14,marginTop:60,lineHeight:"2"}}>
              아직 작성된 일기가 없어요<br/>🌙
            </div>
          ) : diaries.map(d=>(
            <div key={d.id} style={{background:"rgba(255,255,255,0.04)",border:`1px solid ${sentCol[d.sentiment]||"#BAE1FF"}30`,borderRadius:16,padding:"16px 18px"}}>
              <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:10}}>
                <span style={{color:sentCol[d.sentiment],fontSize:13}}>{sentEmo[d.sentiment]} {d.date}</span>
                <div style={{display:"flex",gap:5,flexWrap:"wrap",justifyContent:"flex-end"}}>
                  {d.tags.map(id=>{const f=friends.find(x=>x.id===id);return f?<span key={id} style={{padding:"3px 8px",background:f.color+"20",borderRadius:8,color:f.color,fontSize:10}}>{f.name}</span>:null;})}
                </div>
              </div>
              <p style={{color:"rgba(255,255,255,0.55)",fontSize:13,lineHeight:"1.75",margin:0,
                overflow:"hidden",display:"-webkit-box",WebkitLineClamp:3,WebkitBoxOrient:"vertical"}}>
                {d.content}
              </p>
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}

// ══════════════════════════════════════════════════
//  APP ROOT
// ══════════════════════════════════════════════════
export default function App() {
  const [screen,setScreen] = useState("home");
  const [user] = useState({name:"나",mbti:"INFP",color:"#D4BAFF"});
  const [friends,setFriends] = useState(INITIAL_FRIENDS);
  const [orbitNames] = useState(DEFAULT_ORBIT_NAMES);
  const [diaries,setDiaries] = useState([]);
  const [selectedFriend,setSelectedFriend] = useState(null);
  const [visitingFriend,setVisitingFriend] = useState(null);

  // Visiting friend (home only)
  useEffect(()=>{
    if (screen!=="home") return;
    const cores=friends.filter(f=>f.orbit===0);
    if (!cores.length) return;
    const t=setTimeout(()=>{
      setVisitingFriend(cores[Math.floor(Math.random()*cores.length)]);
      setTimeout(()=>setVisitingFriend(null),6000);
    },2200);
    return ()=>clearTimeout(t);
  },[screen,friends]);

  const handlePush = (id,dir) => setFriends(p=>p.map(f=>
    f.id!==id?f:{...f,orbit:dir==="in"?Math.max(0,f.orbit-1):Math.min(3,f.orbit+1),lastInteraction:Date.now()}
  ));

  const handleAddFriend = (data) => {
    setFriends(p=>[...p,{id:Date.now(),...data,lastInteraction:Date.now()}]);
    setScreen("home");
  };

  const handleDiarySave = (content,taggedIds,sentiment) => {
    setDiaries(p=>[{id:Date.now(),date:new Date().toLocaleDateString("ko-KR"),content,tags:taggedIds,sentiment},...p]);
    setFriends(p=>p.map(f=>{
      if (!taggedIds.includes(f.id)) return f;
      const delta=sentiment==="positive"?-1:sentiment==="negative"?1:0;
      return {...f,orbit:Math.max(0,Math.min(3,f.orbit+delta)),lastInteraction:Date.now()};
    }));
    setScreen("home");
  };

  return (
    <div style={{
      width:"100vw",height:"100vh",overflow:"hidden",position:"relative",
      background:"radial-gradient(ellipse at 28% 38%, #1d1642 0%, #0f0c25 45%, #070518 100%)",
      fontFamily:"'Apple SD Gothic Neo','Pretendard','Noto Sans KR',sans-serif",
    }}>
      <StarField/>
      {screen==="home"      && <HomeScreen user={user} friends={friends} visitingFriend={visitingFriend} onNavigate={setScreen}/>}
      {screen==="orbit"     && <OrbitScreen user={user} friends={friends} orbitNames={orbitNames} onPush={handlePush} onBack={()=>setScreen("home")} selectedFriend={selectedFriend} setSelectedFriend={setSelectedFriend}/>}
      {screen==="diary"     && <DiaryScreen friends={friends} onSave={handleDiarySave} onBack={()=>setScreen("home")}/>}
      {screen==="addFriend" && <AddFriendScreen orbitNames={orbitNames} onAdd={handleAddFriend} onBack={()=>setScreen("home")}/>}
      {screen==="diaryList" && <DiaryListScreen diaries={diaries} friends={friends} onBack={()=>setScreen("home")}/>}

      <style>{`
        *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
        @keyframes floatPlanet {
          0%,100% { transform: translateY(0); }
          50%      { transform: translateY(-14px); }
        }
        @keyframes floatAstro {
          0%,100% { transform: translateY(0); }
          50%      { transform: translateY(-9px); }
        }
        @keyframes slideIn {
          from { opacity:0; transform: translateX(20px); }
          to   { opacity:1; transform: translateX(0); }
        }
        @keyframes fadeUp {
          from { opacity:0; transform: translateY(6px); }
          to   { opacity:1; transform: translateY(0); }
        }
        input::placeholder, textarea::placeholder { color: rgba(255,255,255,0.18); }
        input:focus, textarea:focus { border-color: rgba(212,186,255,0.35) !important; }
        ::-webkit-scrollbar { width: 4px; }
        ::-webkit-scrollbar-track { background: transparent; }
        ::-webkit-scrollbar-thumb { background: rgba(255,255,255,0.14); border-radius: 2px; }
      `}</style>