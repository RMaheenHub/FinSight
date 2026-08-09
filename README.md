const CATEGORIES = ["Food & Dining","Transport","Shopping","Bills & Utilities","Entertainment","Health","Education","Rent","Salary","Investment","Other"];
["tx-category","budget-category"].forEach(id=>{
  const sel = document.getElementById(id);
  CATEGORIES.forEach(c=>{ const o=document.createElement("option"); o.value=c; o.textContent=c; sel.appendChild(o); });
});
document.getElementById("tx-date").valueAsDate = new Date();

let nextId = 1;
let transactions = [];
let budgets = [];

function addTx(type, amount, category, description, dateStr) {
  transactions.push({ id: nextId++, type, amount: Number(amount), category, description, date: dateStr });
}

const today = new Date();
function daysAgo(n){ const d=new Date(today); d.setDate(d.getDate()-n); return d.toISOString().slice(0,10); }

addTx("income", 45000, "Salary", "Monthly salary", daysAgo(20));
addTx("expense", 350, "Food & Dining", "Groceries", daysAgo(18));
addTx("expense", 420, "Food & Dining", "Dinner out", daysAgo(15));
addTx("expense", 3200, "Food & Dining", "Big party order", daysAgo(10));
addTx("expense", 1200, "Transport", "Cab rides", daysAgo(9));
addTx("expense", 2500, "Shopping", "New shoes", daysAgo(7));
addTx("expense", 1800, "Bills & Utilities", "Electricity + Wifi", daysAgo(5));
addTx("expense", 600, "Entertainment", "Movies", daysAgo(3));
budgets.push({ category: "Food & Dining", monthly_limit: 3000 });
budgets.push({ category: "Shopping", monthly_limit: 2000 });

const fmt = n => `₹${Number(n).toLocaleString("en-IN",{maximumFractionDigits:2})}`;

function monthBounds(){
  const start = new Date(today.getFullYear(), today.getMonth(), 1);
  const endLast = new Date(start.getTime() - 1000);
  const startLast = new Date(endLast.getFullYear(), endLast.getMonth(), 1);
  return { start, startLast, endLast };
}

function detectAnomalies(){
  const byCat = {};
  transactions.filter(t=>t.type==="expense").forEach(t=>{ (byCat[t.category] ||= []).push(t); });
  const anomalies = [];
  for (const [cat, txs] of Object.entries(byCat)){
    if (txs.length < 3) continue;
    const amounts = txs.map(t=>t.amount);
    const mean = amounts.reduce((a,b)=>a+b,0)/amounts.length;
    const variance = amounts.reduce((a,b)=>a+(b-mean)**2,0)/amounts.length;
    const std = Math.sqrt(variance) || 1e-6;
    txs.forEach(t=>{
      const z = (t.amount - mean)/std;
      if (z >= 2){
        anomalies.push({ severity:"high", message:`Unusually high ${cat} expense of ${fmt(t.amount)} (category average ${fmt(mean.toFixed(2))})` });
      }
    });
  }
  return anomalies;
}

function budgetStatusList(){
  const { start } = monthBounds();
  const spentByCat = {};
  transactions.filter(t=>t.type==="expense" && new Date(t.date) >= start).forEach(t=>{
    spentByCat[t.category] = (spentByCat[t.category]||0) + t.amount;
  });
  return budgets.map(b=>{
    const spent = spentByCat[b.category] || 0;
    const pct = b.monthly_limit ? (spent/b.monthly_limit*100) : 0;
    return { category:b.category, limit:b.monthly_limit, spent, percent_used: Math.round(pct*10)/10,
      status: spent > b.monthly_limit ? "over" : (pct >= 80 ? "warning" : "ok") };
  });
}

function monthlyTrends(){
  const { start, startLast, endLast } = monthBounds();
  const cur = {}, prev = {};
  transactions.filter(t=>t.type==="expense").forEach(t=>{
    const d = new Date(t.date);
    if (d >= start) cur[t.category] = (cur[t.category]||0) + t.amount;
    else if (d >= startLast && d <= endLast) prev[t.category] = (prev[t.category]||0) + t.amount;
  });
  const cats = new Set([...Object.keys(cur), ...Object.keys(prev)]);
  const trends = [];
  cats.forEach(cat=>{
    const c = cur[cat]||0, p = prev[cat]||0;
    if (c===0 && p===0) return;
    const change = p ? ((c-p)/p*100) : 100;
    trends.push({ category:cat, change_percent: Math.round(change*10)/10, last_month:p });
  });
  return trends;
}

function generateInsights(){
  const insights = [...detectAnomalies()];
  budgetStatusList().forEach(b=>{
    if (b.status==="over") insights.push({severity:"high", message:`You've exceeded your ${b.category} budget: ${fmt(b.spent)} of ${fmt(b.limit)} (${b.percent_used}%)`});
    else if (b.status==="warning") insights.push({severity:"medium", message:`You're at ${b.percent_used}% of your ${b.category} budget for this month`});
  });
  monthlyTrends().forEach(t=>{
    if (t.change_percent >= 30 && t.last_month > 0) insights.push({severity:"medium", message:`${t.category} spending is up ${t.change_percent}% vs last month`});
  });
  const rank = {high:0, medium:1, low:2};
  insights.sort((a,b)=>(rank[a.severity]??3)-(rank[b.severity]??3));
  return insights;
}

let categoryChart, trendChart;

function render(){
  const tbody = document.getElementById("tx-table-body");
  tbody.innerHTML = "";
  let income=0, expense=0;
  [...transactions].sort((a,b)=> new Date(b.date)-new Date(a.date)).forEach(t=>{
    if (t.type==="income") income+=t.amount; else expense+=t.amount;
    const row = document.createElement("tr");
    row.innerHTML = `<td>${t.date}</td><td>${t.type}</td><td>${t.category}</td><td>${t.description||"—"}</td>
      <td class="tx-amount ${t.type}">${t.type==="expense"?"-":"+"}${fmt(t.amount)}</td>
      <td><button class="delete-btn" data-id="${t.id}">Delete</button></td>`;
    tbody.appendChild(row);
  });
  document.getElementById("stat-income").textContent = fmt(income);
  document.getElementById("stat-expense").textContent = fmt(expense);
  document.getElementById("stat-balance").textContent = fmt(income-expense);
  tbody.querySelectorAll(".delete-btn").forEach(btn=>{
    btn.addEventListener("click", ()=>{ transactions = transactions.filter(t=>t.id!=btn.dataset.id); render(); });
  });

  const bl = document.getElementById("budget-list");
  const bstatus = budgetStatusList();
  bl.innerHTML = bstatus.length ? "" : '<p class="empty-state">No budgets set yet.</p>';
  bstatus.forEach(b=>{
    const pct = Math.min(b.percent_used, 100);
    const el = document.createElement("div");
    el.innerHTML = `<div>${b.category}: ${fmt(b.spent)} / ${fmt(b.limit)} (${b.percent_used}%)</div>
      <div class="budget-bar"><div class="budget-bar-fill ${b.status}" style="width:${pct}%"></div></div>`;
    bl.appendChild(el);
  });

  const insights = generateInsights();
  const il = document.getElementById("insights-list");
  il.innerHTML = insights.length ? "" : '<p class="empty-state">No insights yet — add a few transactions to see them here.</p>';
  insights.forEach(i=>{
    const el = document.createElement("div");
    el.className = `insight-item ${i.severity}`;
    el.textContent = i.message;
    il.appendChild(el);
  });

  const catTotals = {};
  transactions.filter(t=>t.type==="expense").forEach(t=>{ catTotals[t.category] = (catTotals[t.category]||0)+t.amount; });
  if (categoryChart) categoryChart.destroy();
  categoryChart = new Chart(document.getElementById("categoryChart"), {
    type:"doughnut",
    data:{ labels:Object.keys(catTotals), datasets:[{ data:Object.values(catTotals),
      backgroundColor:["#5b8def","#f2545b","#3ecf8e","#f2b84b","#7c5cff","#4dd0e1","#ff8a65","#a1887f","#90a4ae","#ec407a","#8d6e63"], borderWidth:0 }] },
    options:{ plugins:{ legend:{ position:"bottom", labels:{ color:"#e8eaf0", boxWidth:12, font:{size:11} } } } }
  });

  if (trendChart) trendChart.destroy();
  trendChart = new Chart(document.getElementById("trendChart"), {
    type:"bar",
    data:{ labels:["This period"], datasets:[
      {label:"Income", data:[income], backgroundColor:"#3ecf8e"},
      {label:"Expense", data:[expense], backgroundColor:"#f2545b"} ] },
    options:{ plugins:{legend:{labels:{color:"#e8eaf0"}}},
      scales:{ x:{ticks:{color:"#9aa1b1"},grid:{color:"#2a2e3a"}}, y:{ticks:{color:"#9aa1b1"},grid:{color:"#2a2e3a"}} } }
  });
}

document.getElementById("tx-form").addEventListener("submit", e=>{
  e.preventDefault();
  addTx(
    document.getElementById("tx-type").value,
    document.getElementById("tx-amount").value,
    document.getElementById("tx-category").value,
    document.getElementById("tx-desc").value,
    document.getElementById("tx-date").value
  );
  e.target.reset();
  document.getElementById("tx-date").valueAsDate = new Date();
  render();
});

document.getElementById("budget-form").addEventListener("submit", e=>{
  e.preventDefault();
  const category = document.getElementById("budget-category").value;
  const limit = Number(document.getElementById("budget-limit").value);
  const existing = budgets.find(b=>b.category===category);
  if (existing) existing.monthly_limit = limit; else budgets.push({category, monthly_limit: limit});
  e.target.reset();
  render();
});

render();
