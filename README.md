<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Inventário de Produtos - LBP</title>
<style>
:root{--red:#d71920;--line:#9aa0a6}
*{box-sizing:border-box}
body{margin:0;font-family:Arial,Helvetica,sans-serif;background:#f2f2f2;color:#1a1a1a}
header{background:#fff;padding:10px 16px;display:flex;align-items:center;gap:16px;flex-wrap:wrap;border-bottom:1px solid #ccc}
.logo{font-weight:900;font-size:22px;color:var(--red);letter-spacing:1px}.logo small{color:#555;font-weight:400;font-size:15px;margin-left:8px;border-left:2px solid #999;padding-left:8px}
.title{background:var(--red);color:#fff;text-align:center;font-weight:bold;font-size:20px;padding:8px}
nav{display:flex;gap:4px;padding:8px 16px 0;background:#fff}
nav button{border:1px solid #ccc;border-bottom:none;background:#e6e6e6;padding:8px 18px;cursor:pointer;font-weight:bold;border-radius:6px 6px 0 0}
nav button.on{background:var(--red);color:#fff;border-color:var(--red)}
.bar{display:flex;gap:8px;align-items:center;flex-wrap:wrap;padding:10px 16px;background:#fff}
.bar input,.bar select,.bar button{padding:7px 10px;font-size:14px}
button.b{background:#333;color:#fff;border:0;border-radius:4px;cursor:pointer}
button.b.red{background:var(--red)}button.b.g{background:#777}
#status{font-size:13px;margin-left:auto;text-align:right}.warn{color:#b45309}.ok{color:#15803d}
.wrap{overflow:auto;padding:0 16px 24px;max-height:calc(100vh - 230px)}
table{border-collapse:collapse;background:#fff;min-width:100%;font-size:13px}
th{background:#e3e3e3;border:1px solid var(--line);padding:6px;position:sticky;top:0;z-index:2;white-space:nowrap}
thead tr:nth-child(2) th{top:31px}thead tr:first-child th{height:31px}
td{border:1px solid var(--line);padding:0}
td input{width:100%;border:0;padding:7px 5px;font-size:13px;background:transparent;font-family:inherit}
td input:focus{outline:2px solid var(--red);background:#fffbe6}
td.d{min-width:130px}td.p{min-width:190px}td.q{min-width:70px}td.l{min-width:120px}td.o{min-width:140px}
td.tot{min-width:80px;text-align:center;font-weight:bold;padding:6px}
.prod{font-weight:bold}.sap{color:var(--red);font-size:11px}
.x{border:0;background:none;color:#888;cursor:pointer;font-size:15px}.x:hover{color:var(--red)}
#hist{display:none;padding:0 16px 24px}
#hist td{padding:7px;vertical-align:top}
.tag{border-radius:3px;padding:1px 6px;font-size:11px;color:#fff;background:#666}
.t-edição{background:#2563eb}.t-nova{background:#16a34a}.t-exclusão{background:#dc2626}.t-estrutura{background:#7c3aed}
.ch{margin:2px 0}.old{background:#fee2e2;color:#b91c1c;text-decoration:line-through;padding:0 4px;border-radius:3px}
.new{background:#dcfce7;color:#166534;padding:0 4px;border-radius:3px;font-weight:bold}.nil{color:#999}
#banner{display:none;padding:8px 16px;font-size:13px;background:#fff3cd;color:#664d03}
</style>
</head>
<body>
<header><div class="logo">HALLIBURTON<small>Cimentação</small></div>
<label>Responsável: <input id="user" placeholder="Seu nome"></label></header>
<div id="banner"></div>
<div class="title">INVENTÁRIO DE PRODUTOS - LBP</div>
<nav><button id="t1" class="on">Principal</button><button id="t2">Histórico</button></nav>

<section id="main">
<div class="bar">
<button class="b g" id="addG">+ Adicionar QTD / LOTE / OBSERVAÇÃO</button>
<button class="b g" id="addR">+ Linha</button>
<button class="b red" id="save">Salvar alterações</button>
<span id="status"></span>
</div>
<div class="wrap"><table id="tb"></table></div>
</section>

<section id="hist">
<div class="bar" style="padding-left:0">
<label>Mês: <select id="fm"></select></label>
<input id="fq" placeholder="Buscar produto ou lote..." size="28">
<button class="b g" id="fclr">Limpar filtros</button>
<span id="hcount"></span>
</div>
<div style="overflow:auto"><table><thead><tr><th>Data</th><th>Produto</th><th>Lotes</th><th>Alteração</th><th>Responsável</th><th>Atualizado em</th></tr></thead><tbody id="hb"></tbody></table></div>
</section>

<script src="https://www.gstatic.com/firebasejs/10.12.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.0/firebase-firestore-compat.js"></script>
<script>
// ====== CONFIGURAÇÃO FIREBASE: cole aqui o objeto do seu projeto ======
// Ex.: {apiKey:"...",authDomain:"...",projectId:"...",appId:"..."}
const FIREBASE_CONFIG = null;
// ======================================================================

const $=s=>document.querySelector(s),J=JSON.stringify,clone=o=>JSON.parse(J(o));
const today=()=>{const d=new Date();return new Date(d-d.getTimezoneOffset()*6e4).toISOString().slice(0,10)};
const esc=s=>String(s??"").replace(/&/g,"&amp;").replace(/"/g,"&quot;").replace(/</g,"&lt;");
const fd=d=>d?d.split("-").reverse().join("/"):"";
const E=()=>({q:"",l:"",o:""}),LBL={q:"QTD",l:"LOTE",o:"OBSERVAÇÃO"};
const uid=()=>"r"+Math.random().toString(36).slice(2,9);
const mkRow=(id,n,s,c)=>({id,date:today(),name:n,sap:s,g:Array.from({length:c},E)});
const PRODUCTS=[["GASCON 469","100064127"],["SCR-100L","100012238"],["GASSTOP LIQUID","101242935"],["ECONOLITE LIQUID","101530807"],["CHEM, HR-25L","101630169"],["CLORETO DE CALCIO","100064159"],["FDP-C1410-20","1157591"],["LIQUILITE","1018486"]];
// ids fixos nas linhas padrão: evita duplicar linhas quando vários navegadores iniciam do zero
const initial=()=>({cols:1,rows:[...PRODUCTS.map((p,i)=>mkRow("p"+i,p[0],p[1],1)),...Array.from({length:8},(_,i)=>mkRow("b"+i,"","",1))]});

let state=initial(),saved=J(state),ops=[],pend=[],hist=[],db=null,synced=false,last="";
const dirty=()=>J(state)!==saved||ops.length>0;
const rowOf=id=>state.rows.find(r=>r.id===id);

// ---------- Tabela principal ----------
const total=r=>{const t=r.g.reduce((a,g)=>a+(parseFloat(String(g.q).replace(",","."))||0),0);return t?t.toLocaleString("pt-BR"):"—"};
function render(){
  let h1='<tr><th rowspan="2">DATA</th><th rowspan="2">PRODUTO</th>',h2="<tr>";
  for(let i=0;i<state.cols;i++){
    h1+=`<th colspan="3">Grupo ${i+1} <button class="x" data-dg="${i}" title="Remover Grupo ${i+1}">✕</button></th>`;
    h2+="<th>QTD</th><th>LOTE</th><th>OBSERVAÇÃO</th>";
  }
  h1+='<th rowspan="2">TOTAL (GAL)</th><th rowspan="2"></th></tr>';
  let b="";
  state.rows.forEach(r=>{
    b+=`<tr><td class="d"><input type="date" data-id="${r.id}" data-f="date" value="${esc(r.date)}"></td>
    <td class="p"><input class="prod" data-id="${r.id}" data-f="name" value="${esc(r.name)}" placeholder="Produto (escrever)">
    <input class="sap" data-id="${r.id}" data-f="sap" value="${esc(r.sap)}" placeholder="SAP"></td>`;
    r.g.forEach((g,i)=>{for(const f of"qlo")b+=`<td class="${f}"><input ${f==="q"?'inputmode="decimal"':""} data-id="${r.id}" data-g="${i}" data-f="${f}" value="${esc(g[f])}"></td>`});
    b+=`<td class="tot" id="tot${r.id}">${total(r)}</td><td><button class="x" data-del="${r.id}" title="Excluir linha">✕</button></td></tr>`;
  });
  $("#tb").innerHTML=`<thead>${h1}${h2}</tr></thead><tbody>${b}</tbody>`;
  status();
}
function rerender(){ // re-renderiza mantendo o foco do campo em edição
  const a=document.activeElement;let sel=null,s,e;
  if(a&&a.dataset&&a.dataset.f&&a.dataset.id){sel=`[data-id="${a.dataset.id}"][data-f="${a.dataset.f}"]`+(a.dataset.g!==undefined?`[data-g="${a.dataset.g}"]`:"");try{s=a.selectionStart;e=a.selectionEnd}catch(_){}}
  render();
  if(sel){const n=$(sel);if(n){n.focus();try{n.setSelectionRange(s,e)}catch(_){}}}
}
function status(){
  const s=$("#status");s.className=dirty()?"warn":"ok";
  s.innerHTML=(dirty()?"● Alterações não salvas":"✓ Tudo salvo"+(db?" (compartilhado)":" (somente neste navegador)"))+(last?`<br><small style="color:#555">${esc(last)}</small>`:"");
}
function onEdit(e){ // 'input' e 'change': o calendário de alguns navegadores só dispara 'change'
  const t=e.target,r=t.dataset.id&&rowOf(t.dataset.id);if(!r)return;
  if(t.dataset.g!==undefined)r.g[t.dataset.g][t.dataset.f]=t.value;else r[t.dataset.f]=t.value;
  $("#tot"+r.id).textContent=total(r);status();
}
$("#tb").addEventListener("input",onEdit);$("#tb").addEventListener("change",onEdit);
$("#tb").addEventListener("click",e=>{
  const t=e.target;
  if(t.dataset.del&&confirm("Excluir esta linha? (o histórico será mantido)")){state.rows=state.rows.filter(r=>r.id!==t.dataset.del);render()}
  if(t.dataset.dg!==undefined)delGroup(+t.dataset.dg);
});
$("#addR").onclick=()=>{state.rows.push(mkRow(uid(),"","",state.cols));render();$(".wrap").scrollTop=1e6};
$("#addG").onclick=()=>{
  state.cols++;state.rows.forEach(r=>r.g.push(E()));ops.push({t:"add"});
  pend.push({tipo:"estrutura",produto:"(Estrutura da tabela)",lotes:"",dataRef:today(),changes:[{c:"Estrutura",a:null,b:`Grupo ${state.cols} adicionado`}]});
  render();
};
function delGroup(i){
  if(state.cols<=1)return alert("É preciso manter ao menos 1 grupo.");
  const filled=state.rows.filter(r=>r.g[i]&&(r.g[i].q||r.g[i].l||r.g[i].o)).length;
  if(!confirm(`Remover o Grupo ${i+1} de todas as linhas?`+(filled?`\n${filled} linha(s) têm dados nesse grupo; ficarão registrados no histórico ao salvar.`:"")))return;
  state.rows.forEach(r=>{const g=r.g[i];if(g&&(g.q||g.l||g.o))pend.push(mkEnt("estrutura",r,r,[{c:`Grupo ${i+1} removido`,a:`QTD ${g.q||"∅"} · LOTE ${g.l||"∅"} · OBS ${g.o||"∅"}`,b:null}]))});
  state.rows.forEach(r=>r.g.splice(i,1));state.cols--;ops.push({t:"del",i});render();
}

// ---------- Mesclagem e salvamento ----------
const nonEmpty=r=>r.name||r.g.some(g=>g.q||g.l||g.o);
const lotes=(a,b)=>[...new Set([a,b].filter(Boolean).flatMap(r=>r.g.map(g=>g.l).filter(Boolean)))].join(", ");
const mkEnt=(tipo,a,b,changes)=>{const r=b||a;return{tipo,produto:r.name||"(sem nome)",lotes:lotes(a,b),dataRef:r.date||"",changes}};
function diffList(a,b){
  const o=[];
  for(const[k,n]of[["date","Data"],["name","Produto"],["sap","SAP"]])if((a[k]||"")!==(b[k]||""))o.push({c:n,a:a[k]||"",b:b[k]||""});
  b.g.forEach((g,i)=>{for(const f of"qlo"){const x=a.g[i]?a.g[i][f]:"";if(x!==g[f])o.push({c:`Grupo ${i+1} · ${LBL[f]}`,a:x,b:g[f]})}});
  return o;
}
const fields=l=>{const o=[{c:"Data",a:null,b:l.date}];if(l.name)o.push({c:"Produto",a:null,b:l.name});l.g.forEach((g,i)=>{for(const f of"qlo")if(g[f])o.push({c:`Grupo ${i+1} · ${LBL[f]}`,a:null,b:g[f]})});return o};
const summary=r=>[r.name,...r.g.map((g,i)=>g.q||g.l||g.o?`G${i+1}: ${g.q||"∅"} / ${g.l||"∅"} / ${g.o||"∅"}`:"")].filter(Boolean).join(" | ");
function norm(rows,c){rows.forEach(r=>{while(r.g.length<c)r.g.push(E());r.g.length=c});return rows}
function applyOps(rows){rows.forEach(r=>ops.forEach(o=>o.t==="add"?r.g.push(E()):r.g.splice(o.i,1)));return rows}
function overlay(r,l,b){ // aplica sobre r apenas as células que o usuário alterou (l vs b)
  for(const k of["date","name","sap"])if(l[k]!==b[k])r[k]=l[k];
  l.g.forEach((g,i)=>{if(!r.g[i])r.g[i]=E();for(const f of"qlo")if(!b.g[i]||g[f]!==b.g[i][f])r.g[i][f]=g[f]});
  return r;
}
function plan(remote){ // remote = estado atual do servidor (ou null)
  const base=JSON.parse(saved),baseAfter=applyOps(clone(base.rows)),bm=new Map(baseAfter.map(r=>[r.id,r])),lm=new Map(state.rows.map(r=>[r.id,r]));
  const cols=(remote?remote.cols:base.cols)+ops.reduce((n,o)=>n+(o.t==="add"?1:-1),0);
  const rem=remote?clone(remote):clone(base);applyOps(rem.rows);
  const rm=new Map(rem.rows.map(r=>[r.id,r])),order=rem.rows.map(r=>r.id),ents=[];
  state.rows.forEach(l=>{
    const b=bm.get(l.id),r=rm.get(l.id);
    if(!b){if(!r){rm.set(l.id,clone(l));order.push(l.id)}if(nonEmpty(l))ents.push(mkEnt("nova linha",null,l,fields(l)))}
    else{const d=diffList(b,l);if(d.length){if(r)overlay(r,l,b);else{rm.set(l.id,clone(l));order.push(l.id)}ents.push(mkEnt("edição",b,l,d))}}
  });
  baseAfter.forEach(b=>{if(!lm.has(b.id)){rm.delete(b.id);if(nonEmpty(b))ents.push(mkEnt("exclusão",b,null,[{c:"Linha excluída",a:null,b:summary(b)}]))}});
  return{state:{cols,rows:norm(order.filter(id=>rm.has(id)).map(id=>rm.get(id)),cols)},ents:[...pend.map(clone),...ents]};
}
async function save(){
  let user=$("#user").value.trim();
  if(!user){user=(prompt("Informe seu nome para registrar no histórico:")||"").trim();if(!user)return;$("#user").value=user}
  localStorage.setItem("lbp_user",user);
  const ts=Date.now(),fin=a=>a.map(e=>({...e,ts,user,mes:(e.dataRef||"").slice(0,7)}));let res;
  try{
    if(db){
      const ref=db.collection("lbp").doc("estado");
      await db.runTransaction(async tx=>{
        const s=await tx.get(ref);res=plan(s.exists?JSON.parse(s.data().json):null);res.ents=fin(res.ents);
        tx.set(ref,{json:J(res.state),upd:ts,by:user});
        res.ents.forEach(e=>tx.set(db.collection("historico").doc(),e));
      });
    }else{
      res=plan(null);res.ents=fin(res.ents);
      localStorage.setItem("lbp_estado",J(res.state));hist=[...res.ents.slice().reverse(),...hist];localStorage.setItem("lbp_hist",J(hist));
    }
  }catch(err){alert("Erro ao salvar: "+err.message);return}
  state=res.state;saved=J(state);ops=[];pend=[];rerender();renderHist();
}
$("#save").onclick=save;
window.addEventListener("beforeunload",e=>{if(dirty()){e.preventDefault();e.returnValue=""}});

// ---------- Sincronização em tempo real ----------
function netErr(e){const b=$("#banner");b.style.display="block";b.style.background="#f8d7da";b.style.color="#842029";b.textContent="Erro de conexão com o banco: "+e.message+" (verifique a configuração e as regras do Firestore)"}
function onRemote(d){
  if(!d.exists){synced=true;return status()}
  const data=d.data(),rm=JSON.parse(data.json);
  if(data.by)last=`Última atualização: ${data.by} às ${new Date(data.upd).toLocaleTimeString("pt-BR",{hour:"2-digit",minute:"2-digit"})}`;
  if(ops.length)return status(); // há mudança de estrutura pendente: mescla ao salvar
  const before=J(state);
  if(!synced||!dirty())state=rm;
  else{
    const pm=new Map(JSON.parse(saved).rows.map(r=>[r.id,r])),lm=new Map(state.rows.map(r=>[r.id,r])),rows=[];
    rm.rows.forEach(r=>{const l=lm.get(r.id),p=pm.get(r.id);
      if(l&&p)rows.push(J(l)!==J(p)?overlay(clone(r),l,p):r);
      else if(!p)rows.push(r)});
    state.rows.forEach(l=>{if(!pm.has(l.id)&&!rm.rows.some(r=>r.id===l.id))rows.push(l)});
    state={cols:rm.cols,rows:norm(rows,rm.cols)};
  }
  saved=data.json;synced=true;
  J(state)!==before?rerender():status();
}
function onHist(q){hist=q.docs.map(x=>x.data());renderHist()}

// ---------- Histórico ----------
const MES=m=>{if(!m)return"Sem data";const[y,n]=m.split("-");return new Date(y,n-1,1).toLocaleDateString("pt-BR",{month:"long",year:"numeric"})};
const chg=h=>h.changes?h.changes.map(x=>{
  const f=v=>x.c==="Data"&&v?fd(v):v,o=v=>v?`<span class="old">${esc(f(v))}</span>`:'<i class="nil">vazio</i>',n=v=>v?`<span class="new">${esc(f(v))}</span>`:'<i class="nil">vazio</i>';
  return`<div class="ch"><b>${esc(x.c)}</b>${x.a===null?" · "+esc(f(x.b)):x.b===null?" · "+esc(x.a):`: ${o(x.a)} → ${n(x.b)}`}</div>`;
}).join(""):esc(h.mudanca||"");
function renderHist(){
  const sel=$("#fm"),cur=sel.value,ms=[...new Set(hist.map(h=>h.mes||""))].sort().reverse();
  sel.innerHTML='<option value="*">Todos os meses</option>'+ms.map(m=>`<option value="${m}">${MES(m)}</option>`).join("");
  sel.value=ms.includes(cur)?cur:"*";
  const q=$("#fq").value.trim().toLowerCase();
  const list=hist.filter(h=>(sel.value==="*"||(h.mes||"")===sel.value)&&(!q||(h.produto+" "+h.lotes+" "+(h.mudanca||"")).toLowerCase().includes(q)));
  $("#hb").innerHTML=list.map(h=>`<tr><td>${fd(h.dataRef)||"—"}</td><td><b>${esc(h.produto)}</b><br><span class="tag t-${(h.tipo||"").split(" ").pop()==="linha"?"nova":h.tipo}">${esc(h.tipo)}</span></td>
  <td>${esc(h.lotes)}</td><td>${chg(h)}</td><td>${esc(h.user)}</td><td>${new Date(h.ts).toLocaleString("pt-BR")}</td></tr>`).join("")||'<tr><td colspan="6">Nenhum registro.</td></tr>';
  $("#hcount").textContent=list.length+" registro(s)";
}
$("#fm").addEventListener("change",renderHist);$("#fq").addEventListener("input",renderHist);
$("#fclr").onclick=()=>{$("#fm").value="*";$("#fq").value="";renderHist()};
function tab(n){$("#main").style.display=n==1?"block":"none";$("#hist").style.display=n==2?"block":"none";
  $("#t1").classList.toggle("on",n==1);$("#t2").classList.toggle("on",n==2);if(n==2)renderHist()}
$("#t1").onclick=()=>tab(1);$("#t2").onclick=()=>tab(2);

// ---------- Início ----------
$("#user").value=localStorage.getItem("lbp_user")||"";
if(FIREBASE_CONFIG&&window.firebase){
  try{
    firebase.initializeApp(FIREBASE_CONFIG);db=firebase.firestore();
    db.collection("lbp").doc("estado").onSnapshot(onRemote,netErr);
    db.collection("historico").orderBy("ts","desc").limit(3000).onSnapshot(onHist,netErr);
  }catch(e){db=null;netErr(e)}
}
if(!db){
  const b=$("#banner");if(b.style.display!=="block"){b.style.display="block";b.textContent="Modo local: os dados ficam salvos apenas neste navegador. Configure o Firebase (FIREBASE_CONFIG no código) para compartilhar entre usuários."}
  try{const s=localStorage.getItem("lbp_estado");if(s){state=JSON.parse(s);saved=s}hist=JSON.parse(localStorage.getItem("lbp_hist")||"[]")}catch(e){}
}
render();renderHist();
</script>
</body>
</html>
