<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>FinSight — Demo</title>
<link rel="stylesheet" href="style.css" />
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.4/chart.umd.min.js"></script>
</head>
<body>

<div class="banner">📦 <strong>Demo mode</strong> — front-end only, running in your browser with sample data.</div>

<div class="app-shell">
  <aside class="sidebar">
    <div class="brand"><span class="brand-mark">₹</span><h1>FinSight</h1></div>
    <p class="tag">Smart personal finance tracker</p>
  </aside>
  <main class="content">
    <header class="content-header">
      <h2>Welcome back 👋</h2>
      <p>Here's your financial overview.</p>
    </header>

    <section class="summary-cards">
      <div class="card"><span class="card-label">Total Income</span><span class="card-value income" id="stat-income">₹0</span></div>
      <div class="card"><span class="card-label">Total Expenses</span><span class="card-value expense" id="stat-expense">₹0</span></div>
      <div class="card"><span class="card-label">Net Balance</span><span class="card-value" id="stat-balance">₹0</span></div>
    </section>

    <section class="grid-2">
      <div class="panel">
        <h3>Add Transaction</h3>
        <form id="tx-form">
          <div class="form-row">
            <select id="tx-type"><option value="expense">Expense</option><option value="income">Income</option></select>
            <input id="tx-amount" type="number" step="0.01" min="0.01" placeholder="Amount (₹)" required />
          </div>
          <div class="form-row">
            <select id="tx-category"></select>
            <input id="tx-date" type="date" required />
          </div>
          <input id="tx-desc" type="text" placeholder="Description (optional)" />
          <button type="submit">Add Transaction</button>
        </form>
      </div>
      <div class="panel">
        <h3>Set Budget</h3>
        <form id="budget-form">
          <div class="form-row">
            <select id="budget-category"></select>
            <input id="budget-limit" type="number" step="0.01" min="1" placeholder="Monthly Limit (₹)" required />
          </div>
          <button type="submit">Save Budget</button>
        </form>
        <div id="budget-list" class="budget-list"></div>
      </div>
    </section>

    <section class="grid-2">
      <div class="panel"><h3>Spending by Category</h3><canvas id="categoryChart" height="220"></canvas></div>
      <div class="panel"><h3>Income vs Expense</h3><canvas id="trendChart" height="220"></canvas></div>
    </section>

    <section class="panel">
      <h3>Smart Insights</h3>
      <div id="insights-list" class="insights-list"></div>
    </section>

    <section class="panel">
      <h3>Recent Transactions</h3>
      <table class="tx-table">
        <thead><tr><th>Date</th><th>Type</th><th>Category</th><th>Description</th><th>Amount</th><th></th></tr></thead>
        <tbody id="tx-table-body"></tbody>
      </table>
    </section>
  </main>
</div>

<script src="script.js"></script>
</body>
</html>
