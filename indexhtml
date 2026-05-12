import { useState, useEffect, useCallback } from "react";

// ── 定数 ──────────────────────────────────────────────────────
const FISCAL_MONTHS = [
  { label: "2026年6月",  year: 2026, month: 6 },
  { label: "2026年7月",  year: 2026, month: 7 },
  { label: "2026年8月",  year: 2026, month: 8 },
  { label: "2026年9月",  year: 2026, month: 9 },
  { label: "2026年10月", year: 2026, month: 10 },
  { label: "2026年11月", year: 2026, month: 11 },
  { label: "2026年12月", year: 2026, month: 12 },
  { label: "2027年1月",  year: 2027, month: 1 },
  { label: "2027年2月",  year: 2027, month: 2 },
  { label: "2027年3月",  year: 2027, month: 3 },
  { label: "2027年4月",  year: 2027, month: 4 },
  { label: "2027年5月",  year: 2027, month: 5 },
];
const STATUS_OPTIONS = ["見積", "受注", "売上", "失注"];
const RANK_OPTIONS   = ["A", "B", "C"];
const STAFF_OPTIONS  = ["福田", "原田"];
const EMPTY_FORM = {
  date: "", status: "見積", name: "", sales: "", purchase: "", rank: "A", staff: "福田",
};
const EMPTY_GAS_FORM = { monthKey: "2026-06", period: "前期", forecast: "" };

// ── ユーティリティ ────────────────────────────────────────────
const fmt = (n) =>
  n == null || n === "" ? "—" : Number(n).toLocaleString("ja-JP") + "円";
const fmtPct = (n) =>
  n == null || isNaN(n) ? "—" : Number(n).toFixed(1) + "%";
const calcProfit = (sales, purchase) => {
  const s = parseFloat(sales) || 0;
  const p = parseFloat(purchase) || 0;
  return s - p;
};
const calcRate = (sales, purchase) => {
  const s = parseFloat(sales) || 0;
  if (s === 0) return 0;
  return ((s - (parseFloat(purchase) || 0)) / s) * 100;
};
const monthKey = (y, m) => `${y}-${String(m).padStart(2, "0")}`;
const dateToMonthKey = (dateStr) => {
  if (!dateStr) return null;
  const d = new Date(dateStr);
  return monthKey(d.getFullYear(), d.getMonth() + 1);
};

// ── スタイル定義 ──────────────────────────────────────────────
const css = `
  @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@300;400;500;700;900&family=DM+Mono:wght@400;500&display=swap');

  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  :root {
    --navy:   #0b1426;
    --navy2:  #111e35;
    --navy3:  #162240;
    --panel:  #1a2a4a;
    --border: #243558;
    --gold:   #e8a020;
    --gold2:  #f5c060;
    --green:  #2ecc8a;
    --red:    #e85050;
    --text:   #dce8ff;
    --muted:  #7a90b8;
    --white:  #ffffff;
    --radius: 10px;
  }

  body { background: var(--navy); color: var(--text); font-family: 'Noto Sans JP', sans-serif; }

  .app {
    min-height: 100vh;
    background: linear-gradient(135deg, #0b1426 0%, #0e1d3a 50%, #0b1426 100%);
    padding: 0 0 60px;
  }

  /* ─ Header ─ */
  .header {
    background: linear-gradient(90deg, #0b1426 0%, #162952 50%, #0b1426 100%);
    border-bottom: 2px solid var(--gold);
    padding: 22px 36px;
    display: flex; align-items: center; gap: 18px;
  }
  .header-icon { font-size: 28px; }
  .header-title { font-size: 20px; font-weight: 700; color: var(--white); letter-spacing: .05em; }
  .header-sub   { font-size: 12px; color: var(--muted); margin-top: 2px; }
  .header-badge {
    margin-left: auto; background: var(--gold); color: #0b1426;
    font-size: 11px; font-weight: 700; padding: 4px 12px; border-radius: 20px;
    letter-spacing: .08em;
  }

  /* ─ Tabs ─ */
  .tabs {
    display: flex; gap: 0; padding: 0 36px;
    background: var(--navy2); border-bottom: 1px solid var(--border);
  }
  .tab {
    padding: 14px 28px; font-size: 13px; font-weight: 500; cursor: pointer;
    color: var(--muted); border-bottom: 3px solid transparent; transition: all .2s;
    border: none; background: none; font-family: inherit;
  }
  .tab:hover { color: var(--text); }
  .tab.active { color: var(--gold); border-bottom-color: var(--gold); font-weight: 700; }

  /* ─ Content ─ */
  .content { padding: 32px 36px; }

  /* ─ Section Title ─ */
  .section-title {
    font-size: 15px; font-weight: 700; color: var(--gold); letter-spacing: .06em;
    margin-bottom: 20px; display: flex; align-items: center; gap: 10px;
  }
  .section-title::after {
    content: ''; flex: 1; height: 1px; background: linear-gradient(90deg, var(--border), transparent);
  }

  /* ─ Form ─ */
  .form-card {
    background: var(--panel); border: 1px solid var(--border);
    border-radius: var(--radius); padding: 28px; margin-bottom: 32px;
  }
  .form-grid {
    display: grid; grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); gap: 16px;
  }
  .form-group label {
    display: block; font-size: 11px; font-weight: 700; color: var(--muted);
    letter-spacing: .08em; margin-bottom: 6px; text-transform: uppercase;
  }
  .form-group input, .form-group select {
    width: 100%; padding: 9px 12px; border-radius: 6px;
    background: var(--navy3); border: 1px solid var(--border);
    color: var(--text); font-size: 13px; font-family: inherit; outline: none;
    transition: border-color .2s;
  }
  .form-group input:focus, .form-group select:focus { border-color: var(--gold); }
  .form-group select option { background: var(--navy2); }
  .form-group.profit-display input { color: var(--green); font-weight: 700; }
  .form-group.rate-display  input { color: var(--gold2); font-weight: 700; }

  .btn {
    padding: 10px 24px; border-radius: 6px; font-size: 13px; font-weight: 700;
    cursor: pointer; border: none; font-family: inherit; transition: all .18s;
    letter-spacing: .04em;
  }
  .btn-primary { background: var(--gold); color: #0b1426; }
  .btn-primary:hover { background: var(--gold2); transform: translateY(-1px); }
  .btn-secondary { background: transparent; color: var(--muted); border: 1px solid var(--border); }
  .btn-secondary:hover { color: var(--text); border-color: var(--text); }
  .btn-danger { background: transparent; color: var(--red); border: 1px solid var(--red); font-size: 11px; padding: 6px 12px; }
  .btn-danger:hover { background: var(--red); color: #fff; }
  .btn-edit { background: transparent; color: var(--gold); border: 1px solid var(--gold); font-size: 11px; padding: 6px 12px; }
  .btn-edit:hover { background: var(--gold); color: #0b1426; }
  .form-actions { margin-top: 20px; display: flex; gap: 12px; justify-content: flex-end; }

  /* ─ Table ─ */
  .table-wrap { overflow-x: auto; border-radius: var(--radius); border: 1px solid var(--border); }
  table { width: 100%; border-collapse: collapse; font-size: 13px; }
  thead tr { background: var(--navy3); }
  thead th {
    padding: 12px 14px; text-align: left; font-size: 11px; font-weight: 700;
    color: var(--muted); letter-spacing: .07em; text-transform: uppercase;
    white-space: nowrap; border-bottom: 1px solid var(--border);
  }
  tbody tr { border-bottom: 1px solid var(--border); transition: background .15s; }
  tbody tr:hover { background: rgba(232,160,32,.06); }
  tbody tr:last-child { border-bottom: none; }
  tbody td { padding: 11px 14px; vertical-align: middle; white-space: nowrap; }

  .rank-badge {
    display: inline-block; width: 24px; height: 24px; border-radius: 50%;
    text-align: center; line-height: 24px; font-size: 11px; font-weight: 900;
  }
  .rank-A { background: #e8505022; color: #e85050; border: 1px solid #e85050; }
  .rank-B { background: #e8a02022; color: var(--gold); border: 1px solid var(--gold); }
  .rank-C { background: #2ecc8a22; color: var(--green); border: 1px solid var(--green); }

  .status-badge {
    display: inline-block; padding: 3px 10px; border-radius: 20px;
    font-size: 11px; font-weight: 700;
  }
  .status-失注 { background: rgba(150,100,200,.2); color: #b08ad0; }
  .status-受注 { background: rgba(232,160,32,.2); color: var(--gold); }
  .status-売上 { background: rgba(46,204,138,.2); color: var(--green); }

  .num { font-family: 'DM Mono', monospace; font-size: 12px; text-align: right; }
  .num.profit { color: var(--green); }
  .num.loss    { color: var(--red); }
  .num.gold    { color: var(--gold2); }

  /* ─ Budget Table ─ */
  .budget-table th.sub { color: #5a7aaa; font-size: 10px; }
  .diff-pos { color: var(--green); font-family: 'DM Mono', monospace; font-size: 12px; }
  .diff-neg { color: var(--red);   font-family: 'DM Mono', monospace; font-size: 12px; }
  .diff-zero{ color: var(--muted); font-family: 'DM Mono', monospace; font-size: 12px; }

  .total-row { background: linear-gradient(90deg, #1a3060, #162240) !important; }
  .total-row td { font-weight: 700; font-size: 14px; padding: 16px 14px; }
  .total-row .total-label { color: var(--gold); font-size: 13px; letter-spacing: .06em; }

  .budget-input {
    width: 120px; padding: 6px 10px; border-radius: 5px; text-align: right;
    background: var(--navy); border: 1px solid var(--border);
    color: var(--text); font-size: 12px; font-family: 'DM Mono', monospace;
    outline: none;
  }
  .budget-input:focus { border-color: var(--gold); }

  /* ─ Modal ─ */
  .modal-overlay {
    position: fixed; inset: 0; background: rgba(0,0,0,.7); backdrop-filter: blur(4px);
    display: flex; align-items: center; justify-content: center; z-index: 100;
  }
  .modal {
    background: var(--panel); border: 1px solid var(--gold); border-radius: var(--radius);
    padding: 32px; width: 90%; max-width: 640px; max-height: 90vh; overflow-y: auto;
  }
  .modal-title { font-size: 16px; font-weight: 700; color: var(--gold); margin-bottom: 24px; }

  /* ─ Delete confirm ─ */
  .delete-confirm {
    background: #200a0a; border: 1px solid var(--red); border-radius: 8px;
    padding: 16px; margin-top: 12px; font-size: 13px;
  }
  .delete-confirm p { color: #ff9090; margin-bottom: 12px; line-height: 1.6; }
  .delete-confirm input {
    width: 100%; padding: 8px 12px; border-radius: 5px;
    background: #300; border: 1px solid var(--red); color: #ff9090;
    font-size: 13px; font-family: 'DM Mono', monospace; outline: none; margin-bottom: 10px;
  }

  /* ─ Stats bar ─ */
  .stats-bar { display: flex; gap: 16px; margin-bottom: 24px; flex-wrap: wrap; }
  .stat-card {
    background: var(--panel); border: 1px solid var(--border); border-radius: 8px;
    padding: 14px 20px; min-width: 150px; flex: 1;
  }
  .stat-label { font-size: 10px; font-weight: 700; color: var(--muted); letter-spacing: .08em; text-transform: uppercase; }
  .stat-value { font-size: 20px; font-weight: 700; color: var(--white); margin-top: 4px; font-family: 'DM Mono', monospace; }
  .stat-value.gold { color: var(--gold); }
  .stat-value.green { color: var(--green); }

  /* ─ Filter bar ─ */
  .filter-bar { display: flex; gap: 10px; margin-bottom: 16px; flex-wrap: wrap; }
  .filter-btn {
    padding: 6px 14px; border-radius: 20px; font-size: 11px; font-weight: 700;
    cursor: pointer; border: 1px solid var(--border); background: transparent;
    color: var(--muted); font-family: inherit; transition: all .15s;
  }
  .filter-btn.active { border-color: var(--gold); color: var(--gold); background: rgba(232,160,32,.1); }

  .empty-state { text-align: center; padding: 60px 20px; color: var(--muted); font-size: 14px; }
  .empty-state .icon { font-size: 40px; margin-bottom: 12px; }

  .save-status { font-size: 11px; color: var(--green); margin-left: 10px; opacity: 0; transition: opacity .3s; }
  .save-status.show { opacity: 1; }

  .annual-highlight {
    background: linear-gradient(90deg, #1e3a1e, #162240) !important;
    border-top: 2px solid var(--green) !important;
  }
  .annual-highlight td { padding: 18px 14px !important; font-size: 15px !important; font-weight: 900 !important; }

  /* ─ Gas ─ */
  .period-btn {
    flex: 1; padding: "9px 0"; border-radius: 6px; font-size: 13px; font-weight: 700;
    cursor: pointer; border: none; font-family: inherit; transition: all .15s;
  }
  .gas-actual-input {
    width: 130px; padding: 6px 10px; border-radius: 5px; text-align: right;
    background: var(--navy); border: 1px solid var(--border);
    color: var(--text); font-size: 12px; font-family: 'DM Mono', monospace; outline: none;
  }
  .gas-actual-input:focus { border-color: var(--green); }
  .gas-section-header {
    background: #0e2020; border-top: 2px solid #2ecc8a44;
  }
  .gas-section-header td {
    padding: 8px 14px; font-size: 11px; font-weight: 700; color: var(--green);
    letter-spacing: .08em; text-transform: uppercase;
  }
`;

// ── メインコンポーネント ───────────────────────────────────────
export default function App() {
  const [tab, setTab] = useState("input");
  const [properties, setProperties] = useState([]);
  const [budgets, setBudgets] = useState({});
  const [gasEntries, setGasEntries] = useState([]);
  const [gasForm, setGasForm] = useState(EMPTY_GAS_FORM);
  const [gasEditId, setGasEditId] = useState(null);
  const [gasActualInputs, setGasActualInputs] = useState({});
  const [form, setForm] = useState(EMPTY_FORM);
  const [filter, setFilter] = useState("all");
  const [staffFilter, setStaffFilter] = useState("all");
  const [editId, setEditId] = useState(null);
  const [editForm, setEditForm] = useState(null);
  const [deleteTarget, setDeleteTarget] = useState(null);
  const [deleteInput, setDeleteInput] = useState("");
  const [saveMsg, setSaveMsg] = useState(false);
  const [loaded, setLoaded] = useState(false);

  // ── ストレージ読み込み ──────────────────────────────────────
  useEffect(() => {
    (async () => {
      try {
        const p = await window.storage.get("he_properties");
        if (p) setProperties(JSON.parse(p.value));
      } catch {}
      try {
        const b = await window.storage.get("he_budgets");
        if (b) setBudgets(JSON.parse(b.value));
      } catch {}
      try {
        const g = await window.storage.get("he_gas");
        if (g) setGasEntries(JSON.parse(g.value));
      } catch {}
      setLoaded(true);
    })();
  }, []);

  // ── ストレージ保存 ──────────────────────────────────────────
  const save = useCallback(async (props, bdgs, gas) => {
    try {
      await window.storage.set("he_properties", JSON.stringify(props));
      await window.storage.set("he_budgets", JSON.stringify(bdgs));
      await window.storage.set("he_gas", JSON.stringify(gas));
      setSaveMsg(true);
      setTimeout(() => setSaveMsg(false), 2000);
    } catch {}
  }, []);

  const saveGas = useCallback(async (gas) => {
    try {
      await window.storage.set("he_gas", JSON.stringify(gas));
      setSaveMsg(true);
      setTimeout(() => setSaveMsg(false), 2000);
    } catch {}
  }, []);

  // ── 物件追加 ───────────────────────────────────────────────
  const handleAdd = () => {
    if (!form.date || !form.name) return alert("売上予定日と物件名は必須です");
    const newProp = {
      id: Date.now().toString(),
      ...form,
      sales:    parseFloat(form.sales)    || 0,
      purchase: parseFloat(form.purchase) || 0,
      createdAt: new Date().toISOString(),
    };
    const next = [...properties, newProp];
    setProperties(next);
    save(next, budgets, gasEntries);
    setForm(EMPTY_FORM);
  };

  // ── 物件編集保存 ───────────────────────────────────────────
  const handleEditSave = () => {
    const updated = properties.map(p =>
      p.id === editId
        ? { ...p, ...editForm, sales: parseFloat(editForm.sales) || 0, purchase: parseFloat(editForm.purchase) || 0 }
        : p
    );
    setProperties(updated);
    save(updated, budgets, gasEntries);
    setEditId(null);
  };

  // ── 物件削除 ───────────────────────────────────────────────
  const handleDelete = () => {
    if (deleteInput !== "削除確認") return alert('「削除確認」と入力してください');
    const next = properties.filter(p => p.id !== deleteTarget.id);
    setProperties(next);
    save(next, budgets, gasEntries);
    setDeleteTarget(null);
    setDeleteInput("");
    setEditId(null);
  };

  // ── 予算変更 ───────────────────────────────────────────────
  const handleBudget = (key, val) => {
    const next = { ...budgets, [key]: parseFloat(val) || 0 };
    setBudgets(next);
    save(properties, next, gasEntries);
  };

  // ── ガス登録 ───────────────────────────────────────────────
  const handleGasAdd = () => {
    if (!gasForm.forecast) return alert("見込金額を入力してください");
    const dup = gasEntries.find(g => g.monthKey === gasForm.monthKey && g.period === gasForm.period);
    if (dup) return alert("同じ月・期がすでに登録されています");
    const entry = {
      id: Date.now().toString(),
      monthKey: gasForm.monthKey,
      period: gasForm.period,
      forecast: parseFloat(gasForm.forecast) || 0,
      actual: null,
      isActual: false,
      createdAt: new Date().toISOString(),
    };
    const next = [...gasEntries, entry];
    setGasEntries(next);
    saveGas(next);
    setGasForm(EMPTY_GAS_FORM);
  };

  // ── ガス実績切替 ──────────────────────────────────────────
  const handleGasToggleActual = (id) => {
    const next = gasEntries.map(g =>
      g.id === id ? { ...g, isActual: !g.isActual } : g
    );
    setGasEntries(next);
    saveGas(next);
  };

  // ── ガス実績金額保存 ──────────────────────────────────────
  const handleGasActualSave = (id) => {
    const val = parseFloat(gasActualInputs[id]);
    if (isNaN(val)) return;
    const next = gasEntries.map(g =>
      g.id === id ? { ...g, actual: val, isActual: true } : g
    );
    setGasEntries(next);
    saveGas(next);
  };
  const filtered = (() => {
    let list = filter === "all" ? properties : properties.filter(p => p.status === filter);
    if (staffFilter !== "all") list = list.filter(p => p.staff === staffFilter);
    return list;
  })();

  // ── 月別集計 ───────────────────────────────────────────────
  const monthlyActual = FISCAL_MONTHS.reduce((acc, { year, month }) => {
    const key = monthKey(year, month);
    const propActual = properties
      .filter(p => p.status === "売上" && dateToMonthKey(p.date) === key)
      .reduce((s, p) => s + p.sales, 0);
    const gasActual = gasEntries
      .filter(g => g.monthKey === key && g.isActual && g.actual != null)
      .reduce((s, g) => s + g.actual, 0);
    acc[key] = propActual + gasActual;
    return acc;
  }, {});

  const monthlyForecast = FISCAL_MONTHS.reduce((acc, { year, month }) => {
    const key = monthKey(year, month);
    const propForecast = properties
      .filter(p => ["見積","受注","売上"].includes(p.status) && dateToMonthKey(p.date) === key)
      .reduce((s, p) => s + p.sales, 0);
    const gasForecast = gasEntries
      .filter(g => g.monthKey === key)
      .reduce((s, g) => s + (g.forecast || 0), 0);
    acc[key] = propForecast + gasForecast;
    return acc;
  }, {});

  const totalBudget   = FISCAL_MONTHS.reduce((s, { year, month }) => s + (budgets[monthKey(year, month)] || 0), 0);
  const totalActual   = Object.values(monthlyActual).reduce((a, b) => a + b, 0);
  const totalForecast = Object.values(monthlyForecast).reduce((a, b) => a + b, 0);
  const totalDiff     = totalActual - totalBudget;

  // ── ガス月別集計 ──────────────────────────────────────────
  const gasMonthly = FISCAL_MONTHS.reduce((acc, { year, month }) => {
    const key = monthKey(year, month);
    const entries = gasEntries.filter(g => g.monthKey === key);
    acc[key] = {
      forecast: entries.reduce((s, g) => s + (g.forecast || 0), 0),
      actual:   entries.filter(g => g.isActual && g.actual != null).reduce((s, g) => s + g.actual, 0),
      hasActual: entries.some(g => g.isActual && g.actual != null),
    };
    return acc;
  }, {});
  const gasTotalForecast = Object.values(gasMonthly).reduce((s, v) => s + v.forecast, 0);
  const gasTotalActual   = Object.values(gasMonthly).reduce((s, v) => s + v.actual, 0);

  if (!loaded) return <div style={{ color: "#7a90b8", padding: 40, fontFamily: "sans-serif" }}>読み込み中...</div>;

  return (
    <>
      <style>{css}</style>
      <div className="app">
        {/* ─ Header ─ */}
        <div className="header">
          <div className="header-icon">⚡</div>
          <div>
            <div className="header-title">ホームエネルギー部 売上物件管理</div>
            <div className="header-sub">2026年度（2026/6/1 〜 2027/5/31）</div>
          </div>
          <div className="header-badge">FY2026</div>
          <span className={`save-status ${saveMsg ? "show" : ""}`}>✓ 保存済み</span>
        </div>

        {/* ─ Tabs ─ */}
        <div className="tabs">
          {[
            { id: "input",  label: "📝 物件入力" },
            { id: "list",   label: "📋 物件一覧" },
            { id: "gas",    label: "🔥 ガス" },
            { id: "budget", label: "📊 売上管理表" },
          ].map(t => (
            <button key={t.id} className={`tab ${tab === t.id ? "active" : ""}`} onClick={() => setTab(t.id)}>
              {t.label}
            </button>
          ))}
        </div>

        <div className="content">
          {/* ══════════════════════════════════════
              TAB: 物件入力
          ══════════════════════════════════════ */}
          {tab === "input" && (
            <>
              <div className="section-title">新規物件登録</div>
              <div className="form-card">
                <div className="form-grid">
                  <div className="form-group">
                    <label>売上予定日 ✱</label>
                    <input type="date" value={form.date}
                      onChange={e => setForm({ ...form, date: e.target.value })} />
                  </div>
                  <div className="form-group">
                    <label>ステータス</label>
                    <select value={form.status} onChange={e => setForm({ ...form, status: e.target.value })}>
                      {STATUS_OPTIONS.map(s => <option key={s}>{s}</option>)}
                    </select>
                  </div>
                  <div className="form-group" style={{ gridColumn: "span 2" }}>
                    <label>物件名（顧客名含む） ✱</label>
                    <input type="text" placeholder="例）〇〇邸 太陽光設置工事（山田太郎様）" value={form.name}
                      onChange={e => setForm({ ...form, name: e.target.value })} />
                  </div>
                  <div className="form-group">
                    <label>売上金額（円）</label>
                    <input type="number" placeholder="0" value={form.sales}
                      onChange={e => setForm({ ...form, sales: e.target.value })} />
                  </div>
                  <div className="form-group">
                    <label>仕入金額（円）</label>
                    <input type="number" placeholder="0" value={form.purchase}
                      onChange={e => setForm({ ...form, purchase: e.target.value })} />
                  </div>
                  <div className="form-group profit-display">
                    <label>利益（自動計算）</label>
                    <input readOnly value={
                      form.sales || form.purchase
                        ? calcProfit(form.sales, form.purchase).toLocaleString("ja-JP")
                        : ""
                    } placeholder="—" />
                  </div>
                  <div className="form-group rate-display">
                    <label>利益率（自動計算）</label>
                    <input readOnly value={
                      form.sales
                        ? calcRate(form.sales, form.purchase).toFixed(1) + "%"
                        : ""
                    } placeholder="—" />
                  </div>
                  <div className="form-group">
                    <label>担当者</label>
                    <div style={{ display: "flex", gap: 8, marginTop: 2 }}>
                      {STAFF_OPTIONS.map(s => (
                        <button key={s} type="button"
                          onClick={() => setForm({ ...form, staff: s })}
                          style={{
                            flex: 1, padding: "8px 0", borderRadius: 6, fontSize: 13, fontWeight: 700,
                            cursor: "pointer", fontFamily: "inherit", transition: "all .15s",
                            background: form.staff === s ? "var(--gold)" : "var(--navy3)",
                            color: form.staff === s ? "#0b1426" : "var(--muted)",
                            border: form.staff === s ? "1px solid var(--gold)" : "1px solid var(--border)",
                          }}>{s}</button>
                      ))}
                    </div>
                  </div>
                  <div className="form-group">
                    <label>受注ランク</label>
                    <select value={form.rank} onChange={e => setForm({ ...form, rank: e.target.value })}>
                      {RANK_OPTIONS.map(r => <option key={r}>{r}</option>)}
                    </select>
                  </div>
                </div>
                <div className="form-actions">
                  <button className="btn btn-secondary" onClick={() => setForm(EMPTY_FORM)}>クリア</button>
                  <button className="btn btn-primary" onClick={handleAdd}>＋ 物件を登録</button>
                </div>
              </div>

              {/* 直近5件プレビュー */}
              {properties.length > 0 && (
                <>
                  <div className="section-title">最近の登録物件</div>
                  <div className="table-wrap">
                    <table>
                      <thead><tr>
                        <th>売上予定日</th><th>ステータス</th><th>物件名</th><th>担当者</th>
                        <th>売上金額</th><th>利益率</th><th>ランク</th>
                      </tr></thead>
                      <tbody>
                        {[...properties].reverse().slice(0, 5).map(p => (
                          <tr key={p.id}>
                            <td>{p.date}</td>
                            <td><span className={`status-badge status-${p.status}`}>{p.status}</span></td>
                            <td>{p.name}</td>
                            <td>{p.staff || "—"}</td>
                            <td className="num">{fmt(p.sales)}</td>
                            <td className={`num ${calcProfit(p.sales, p.purchase) >= 0 ? "profit" : "loss"}`}>
                              {fmtPct(calcRate(p.sales, p.purchase))}
                            </td>
                            <td><span className={`rank-badge rank-${p.rank}`}>{p.rank}</span></td>
                          </tr>
                        ))}
                      </tbody>
                    </table>
                  </div>
                </>
              )}
            </>
          )}

          {/* ══════════════════════════════════════
              TAB: 物件一覧
          ══════════════════════════════════════ */}
          {tab === "list" && (
            <>
              {/* Stats */}
              <div className="stats-bar">
                <div className="stat-card">
                  <div className="stat-label">登録物件数</div>
                  <div className="stat-value">{properties.length}<span style={{ fontSize: 13, color: "#7a90b8" }}>件</span></div>
                </div>
                <div className="stat-card">
                  <div className="stat-label">売上済み合計</div>
                  <div className="stat-value green" style={{ fontSize: 16 }}>
                    {properties.filter(p => p.status === "売上").reduce((s, p) => s + p.sales, 0).toLocaleString()}円
                  </div>
                </div>
                <div className="stat-card">
                  <div className="stat-label">受注・見込み合計</div>
                  <div className="stat-value gold" style={{ fontSize: 16 }}>
                    {properties.filter(p => p.status !== "売上").reduce((s, p) => s + p.sales, 0).toLocaleString()}円
                  </div>
                </div>
                <div className="stat-card">
                  <div className="stat-label">Aランク件数</div>
                  <div className="stat-value" style={{ color: "#e85050" }}>
                    {properties.filter(p => p.rank === "A").length}<span style={{ fontSize: 13, color: "#7a90b8" }}>件</span>
                  </div>
                </div>
              </div>

              {/* Filter */}
              <div className="filter-bar">
                {[
                  { id: "all", label: "すべて" },
                  { id: "見積", label: "見積" },
                  { id: "受注", label: "受注" },
                  { id: "失注", label: "失注" },
                ].map(f => (
                  <button key={f.id} className={`filter-btn ${filter === f.id ? "active" : ""}`}
                    onClick={() => setFilter(f.id)}>{f.label}</button>
                ))}
              </div>
              <div className="filter-bar">
                {[
                  { id: "all", label: "👤 全担当者" },
                  { id: "福田", label: "福田" },
                  { id: "原田", label: "原田" },
                ].map(f => (
                  <button key={f.id} className={`filter-btn ${staffFilter === f.id ? "active" : ""}`}
                    onClick={() => setStaffFilter(f.id)}>{f.label}</button>
                ))}
              </div>

              {filtered.length === 0 ? (
                <div className="empty-state"><div className="icon">📭</div>物件が登録されていません</div>
              ) : (
                <div className="table-wrap">
                  <table>
                    <thead><tr>
                      <th>売上予定日</th><th>ステータス</th><th style={{ minWidth: 240 }}>物件名</th><th>担当者</th>
                      <th>売上金額</th><th>仕入金額</th><th>利益</th><th>利益率</th><th>ランク</th><th>操作</th>
                    </tr></thead>
                    <tbody>
                      {filtered.map(p => {
                        const profit = calcProfit(p.sales, p.purchase);
                        const rate   = calcRate(p.sales, p.purchase);
                        return (
                          <tr key={p.id}>
                            <td style={{ fontSize: 12, color: "#7a90b8" }}>{p.date}</td>
                            <td><span className={`status-badge status-${p.status}`}>{p.status}</span></td>
                            <td style={{ maxWidth: 280, overflow: "hidden", textOverflow: "ellipsis" }}>{p.name}</td>
                            <td style={{ fontWeight: 600 }}>{p.staff || "—"}</td>
                            <td className="num">{fmt(p.sales)}</td>
                            <td className="num" style={{ color: "#7a90b8" }}>{fmt(p.purchase)}</td>
                            <td className={`num ${profit >= 0 ? "profit" : "loss"}`}>{fmt(profit)}</td>
                            <td className={`num ${profit >= 0 ? "gold" : "loss"}`}>{fmtPct(rate)}</td>
                            <td><span className={`rank-badge rank-${p.rank}`}>{p.rank}</span></td>
                            <td>
                              <button className="btn btn-edit" onClick={() => { setEditId(p.id); setEditForm({ ...p }); }}>
                                編集
                              </button>
                            </td>
                          </tr>
                        );
                      })}
                    </tbody>
                  </table>
                </div>
              )}
            </>
          )}

          {/* ══════════════════════════════════════
              TAB: ガス入力・一覧
          ══════════════════════════════════════ */}
          {tab === "gas" && (
            <>
              <div className="section-title">ガス売上 入力</div>
              <div className="form-card">
                <div className="form-grid">
                  {/* 月選択 */}
                  <div className="form-group">
                    <label>月</label>
                    <select value={gasForm.monthKey}
                      onChange={e => setGasForm({ ...gasForm, monthKey: e.target.value })}>
                      {FISCAL_MONTHS.map(({ label, year, month }) => (
                        <option key={monthKey(year, month)} value={monthKey(year, month)}>{label}</option>
                      ))}
                    </select>
                  </div>
                  {/* 前期・後期 */}
                  <div className="form-group">
                    <label>期</label>
                    <div style={{ display: "flex", gap: 8, marginTop: 2 }}>
                      {[{ id: "前期", sub: "1〜15日" }, { id: "後期", sub: "16〜末日" }].map(p => (
                        <button key={p.id} type="button"
                          onClick={() => setGasForm({ ...gasForm, period: p.id })}
                          style={{
                            flex: 1, padding: "8px 4px", borderRadius: 6, fontSize: 12, fontWeight: 700,
                            cursor: "pointer", fontFamily: "inherit", transition: "all .15s", lineHeight: 1.4,
                            background: gasForm.period === p.id ? "var(--gold)" : "var(--navy3)",
                            color: gasForm.period === p.id ? "#0b1426" : "var(--muted)",
                            border: gasForm.period === p.id ? "1px solid var(--gold)" : "1px solid var(--border)",
                          }}>
                          {p.id}<br /><span style={{ fontSize: 10, fontWeight: 400 }}>{p.sub}</span>
                        </button>
                      ))}
                    </div>
                  </div>
                  {/* 見込金額 */}
                  <div className="form-group">
                    <label>見込金額（円）</label>
                    <input type="number" placeholder="0" value={gasForm.forecast}
                      onChange={e => setGasForm({ ...gasForm, forecast: e.target.value })} />
                  </div>
                </div>
                <div className="form-actions">
                  <button className="btn btn-secondary" onClick={() => setGasForm(EMPTY_GAS_FORM)}>クリア</button>
                  <button className="btn btn-primary" onClick={handleGasAdd}>＋ 登録</button>
                </div>
              </div>

              {/* ガス一覧 */}
              <div className="section-title">ガス売上 一覧</div>
              {gasEntries.length === 0 ? (
                <div className="empty-state"><div className="icon">🔥</div>ガス売上が登録されていません</div>
              ) : (
                <div className="table-wrap">
                  <table>
                    <thead><tr>
                      <th>月</th><th>期</th><th>見込金額</th>
                      <th>実績金額</th><th>差額</th><th>ステータス</th>
                    </tr></thead>
                    <tbody>
                      {[...gasEntries]
                        .sort((a, b) => a.monthKey.localeCompare(b.monthKey) || a.period.localeCompare(b.period))
                        .map(g => {
                          const mLabel = FISCAL_MONTHS.find(m => monthKey(m.year, m.month) === g.monthKey)?.label || g.monthKey;
                          const diff = g.isActual && g.actual != null ? g.actual - g.forecast : null;
                          const inputVal = gasActualInputs[g.id] != null ? gasActualInputs[g.id] : (g.actual != null ? g.actual : "");
                          return (
                            <tr key={g.id}>
                              <td style={{ fontWeight: 600 }}>{mLabel}</td>
                              <td>
                                <span style={{
                                  display: "inline-block", padding: "2px 10px", borderRadius: 20,
                                  fontSize: 11, fontWeight: 700,
                                  background: g.period === "前期" ? "rgba(232,160,32,.15)" : "rgba(46,204,138,.15)",
                                  color: g.period === "前期" ? "var(--gold)" : "var(--green)",
                                  border: `1px solid ${g.period === "前期" ? "var(--gold)" : "var(--green)"}`,
                                }}>{g.period}</span>
                              </td>
                              <td className="num gold">{fmt(g.forecast)}</td>
                              <td>
                                {g.isActual ? (
                                  <div style={{ display: "flex", gap: 6, alignItems: "center" }}>
                                    <input className="gas-actual-input"
                                      type="number"
                                      value={inputVal}
                                      onChange={e => setGasActualInputs({ ...gasActualInputs, [g.id]: e.target.value })}
                                      onBlur={() => handleGasActualSave(g.id)}
                                      placeholder="実績入力"
                                    />
                                  </div>
                                ) : (
                                  <span style={{ color: "var(--muted)", fontSize: 12 }}>—</span>
                                )}
                              </td>
                              <td>
                                {diff != null ? (
                                  <span className={diff >= 0 ? "diff-pos" : "diff-neg"}>
                                    {diff >= 0 ? "▲ " : "▼ "}{Math.abs(diff).toLocaleString()}円
                                  </span>
                                ) : <span style={{ color: "#243558" }}>—</span>}
                              </td>
                              <td>
                                <button className="btn"
                                  onClick={() => handleGasToggleActual(g.id)}
                                  style={{
                                    padding: "5px 14px", fontSize: 11, fontWeight: 700,
                                    background: g.isActual ? "rgba(46,204,138,.15)" : "rgba(232,160,32,.15)",
                                    color: g.isActual ? "var(--green)" : "var(--gold)",
                                    border: `1px solid ${g.isActual ? "var(--green)" : "var(--gold)"}`,
                                    borderRadius: 20,
                                  }}>
                                  {g.isActual ? "✓ 実績" : "見込"}
                                </button>
                              </td>
                            </tr>
                          );
                        })}
                    </tbody>
                  </table>
                </div>
              )}
            </>
          )}

          {/* ══════════════════════════════════════
              TAB: 売上管理表
          ══════════════════════════════════════ */}
          {tab === "budget" && (
            <>
              <div className="section-title">月別 予算対比表（2026年度）</div>
              <div className="table-wrap">
                <table className="budget-table">
                  <thead>
                    <tr>
                      <th rowSpan={2}>月</th>
                      <th rowSpan={2}>予算<br /><span className="sub">（入力可）</span></th>
                      <th colSpan={3} style={{ textAlign: "center", borderBottom: "1px solid var(--border)", color: "var(--green)" }}>実　績</th>
                      <th colSpan={3} style={{ textAlign: "center", borderBottom: "1px solid var(--border)", color: "var(--gold)" }}>売上見込</th>
                      <th rowSpan={2}>予算比<br /><span className="sub">（実績合計）</span></th>
                      <th rowSpan={2}>達成率</th>
                    </tr>
                    <tr>
                      <th className="sub" style={{ color: "var(--green)" }}>物件</th>
                      <th className="sub" style={{ color: "#2ecc8a99" }}>ガス</th>
                      <th className="sub" style={{ color: "var(--green)", fontWeight: 900 }}>合計</th>
                      <th className="sub" style={{ color: "var(--gold)" }}>物件</th>
                      <th className="sub" style={{ color: "#e8a02099" }}>ガス</th>
                      <th className="sub" style={{ color: "var(--gold)", fontWeight: 900 }}>合計</th>
                    </tr>
                  </thead>
                  <tbody>
                    {FISCAL_MONTHS.map(({ label, year, month }) => {
                      const key = monthKey(year, month);
                      const budget = budgets[key] || 0;
                      const gd = gasMonthly[key];

                      const propActual = properties
                        .filter(p => p.status === "売上" && dateToMonthKey(p.date) === key)
                        .reduce((s, p) => s + p.sales, 0);
                      const gasActual = gd.hasActual ? gd.actual : 0;
                      const totalActualRow = propActual + gasActual;

                      const propForecast = properties
                        .filter(p => ["見積","受注","売上"].includes(p.status) && dateToMonthKey(p.date) === key)
                        .reduce((s, p) => s + p.sales, 0);
                      const gasForecast = gd.forecast || 0;
                      const totalForecastRow = propForecast + gasForecast;

                      const diff = totalActualRow - budget;
                      const rate = budget > 0 ? (totalActualRow / budget) * 100 : null;

                      return (
                        <tr key={key}>
                          <td style={{ fontWeight: 600 }}>{label}</td>
                          <td>
                            <input className="budget-input" type="number"
                              value={budgets[key] || ""}
                              placeholder="予算入力"
                              onChange={e => handleBudget(key, e.target.value)} />
                          </td>
                          {/* 実績 物件 */}
                          <td className="num" style={{ color: propActual > 0 ? "var(--green)" : "#243558" }}>
                            {propActual > 0 ? propActual.toLocaleString() + "円" : "—"}
                          </td>
                          {/* 実績 ガス */}
                          <td className="num" style={{ color: gasActual > 0 ? "#2ecc8a99" : "#243558", fontSize: 11 }}>
                            {gasActual > 0 ? gasActual.toLocaleString() + "円" : "—"}
                          </td>
                          {/* 実績 合計 */}
                          <td className="num" style={{
                            color: totalActualRow > 0 ? "var(--green)" : "#243558",
                            fontWeight: 700, borderRight: "2px solid var(--border)"
                          }}>
                            {totalActualRow > 0 ? totalActualRow.toLocaleString() + "円" : "—"}
                          </td>
                          {/* 見込 物件 */}
                          <td className="num" style={{ color: propForecast > 0 ? "var(--gold)" : "#243558" }}>
                            {propForecast > 0 ? propForecast.toLocaleString() + "円" : "—"}
                          </td>
                          {/* 見込 ガス */}
                          <td className="num" style={{ color: gasForecast > 0 ? "#e8a02099" : "#243558", fontSize: 11 }}>
                            {gasForecast > 0 ? gasForecast.toLocaleString() + "円" : "—"}
                          </td>
                          {/* 見込 合計 */}
                          <td className="num" style={{
                            color: totalForecastRow > 0 ? "var(--gold2)" : "#243558",
                            fontWeight: 700, borderRight: "2px solid var(--border)"
                          }}>
                            {totalForecastRow > 0 ? totalForecastRow.toLocaleString() + "円" : "—"}
                          </td>
                          {/* 予算比 */}
                          <td className={diff > 0 ? "diff-pos" : diff < 0 ? "diff-neg" : "diff-zero"}>
                            {budget > 0 ? (diff > 0 ? "▲ " : diff < 0 ? "▼ " : "") + Math.abs(diff).toLocaleString() + "円" : "—"}
                          </td>
                          {/* 達成率 */}
                          <td>
                            {rate !== null ? (
                              <span style={{
                                color: rate >= 100 ? "#2ecc8a" : rate >= 80 ? "#e8a020" : "#e85050",
                                fontFamily: "'DM Mono', monospace", fontSize: 12, fontWeight: 700
                              }}>{rate.toFixed(1)}%</span>
                            ) : "—"}
                          </td>
                        </tr>
                      );
                    })}

                    {/* ─ 年度合計行 ─ */}
                    <tr className="annual-highlight">
                      <td className="total-label">🏆 2026年度 合計</td>
                      <td>
                        <span style={{ fontFamily: "'DM Mono', monospace", color: "#e8a020" }}>
                          {totalBudget > 0 ? totalBudget.toLocaleString() + "円" : "—"}
                        </span>
                      </td>
                      {/* 実績 物件合計 */}
                      <td><span style={{ fontFamily: "'DM Mono', monospace", color: "var(--green)", fontSize: 13 }}>
                        {properties.filter(p => p.status === "売上").reduce((s, p) => s + p.sales, 0) > 0
                          ? properties.filter(p => p.status === "売上").reduce((s, p) => s + p.sales, 0).toLocaleString() + "円" : "—"}
                      </span></td>
                      {/* 実績 ガス合計 */}
                      <td><span style={{ fontFamily: "'DM Mono', monospace", color: "#2ecc8a99", fontSize: 13 }}>
                        {gasTotalActual > 0 ? gasTotalActual.toLocaleString() + "円" : "—"}
                      </span></td>
                      {/* 実績 合計 */}
                      <td style={{ borderRight: "2px solid var(--border)" }}>
                        <span style={{ fontFamily: "'DM Mono', monospace", color: "#2ecc8a", fontSize: 16, fontWeight: 900 }}>
                          {totalActual > 0 ? totalActual.toLocaleString() + "円" : "—"}
                        </span>
                      </td>
                      {/* 見込 物件合計 */}
                      <td><span style={{ fontFamily: "'DM Mono', monospace", color: "var(--gold)", fontSize: 13 }}>
                        {properties.filter(p => ["見積","受注","売上"].includes(p.status)).reduce((s, p) => s + p.sales, 0) > 0
                          ? properties.filter(p => ["見積","受注","売上"].includes(p.status)).reduce((s, p) => s + p.sales, 0).toLocaleString() + "円" : "—"}
                      </span></td>
                      {/* 見込 ガス合計 */}
                      <td><span style={{ fontFamily: "'DM Mono', monospace", color: "#e8a02099", fontSize: 13 }}>
                        {gasTotalForecast > 0 ? gasTotalForecast.toLocaleString() + "円" : "—"}
                      </span></td>
                      {/* 見込 合計 */}
                      <td style={{ borderRight: "2px solid var(--border)" }}>
                        <span style={{ fontFamily: "'DM Mono', monospace", color: "#f5c060", fontSize: 16, fontWeight: 900 }}>
                          {totalForecast > 0 ? totalForecast.toLocaleString() + "円" : "—"}
                        </span>
                      </td>
                      {/* 予算比 */}
                      <td>
                        <span className={totalDiff > 0 ? "diff-pos" : totalDiff < 0 ? "diff-neg" : "diff-zero"} style={{ fontSize: 15 }}>
                          {totalBudget > 0
                            ? (totalDiff > 0 ? "▲ " : totalDiff < 0 ? "▼ " : "±") + Math.abs(totalDiff).toLocaleString() + "円"
                            : "—"}
                        </span>
                      </td>
                      {/* 達成率 */}
                      <td>
                        {totalBudget > 0 ? (
                          <span style={{
                            fontFamily: "'DM Mono', monospace", fontSize: 16, fontWeight: 900,
                            color: totalActual / totalBudget >= 1 ? "#2ecc8a" : totalActual / totalBudget >= 0.8 ? "#e8a020" : "#e85050"
                          }}>
                            {((totalActual / totalBudget) * 100).toFixed(1)}%
                          </span>
                        ) : "—"}
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>

              {/* 物件サマリー */}
              <div style={{ marginTop: 36 }}>
                <div className="section-title">物件一覧サマリー</div>
                <div className="table-wrap">
                  <table>
                    <thead><tr>
                      <th>売上予定日</th><th>ステータス</th><th>物件名</th><th>担当者</th>
                      <th>売上金額</th><th>利益</th><th>利益率</th><th>ランク</th>
                    </tr></thead>
                    <tbody>
                      {properties.length === 0 ? (
                        <tr><td colSpan={7} className="empty-state">物件なし</td></tr>
                      ) : (
                        [...properties].sort((a, b) => new Date(a.date) - new Date(b.date)).map(p => {
                          const profit = calcProfit(p.sales, p.purchase);
                          const rate   = calcRate(p.sales, p.purchase);
                          return (
                            <tr key={p.id}>
                              <td style={{ fontSize: 12, color: "#7a90b8" }}>{p.date}</td>
                              <td><span className={`status-badge status-${p.status}`}>{p.status}</span></td>
                              <td>{p.name}</td>
                              <td style={{ fontWeight: 600 }}>{p.staff || "—"}</td>
                              <td className="num">{fmt(p.sales)}</td>
                              <td className={`num ${profit >= 0 ? "profit" : "loss"}`}>{fmt(profit)}</td>
                              <td className={`num ${profit >= 0 ? "gold" : "loss"}`}>{fmtPct(rate)}</td>
                              <td><span className={`rank-badge rank-${p.rank}`}>{p.rank}</span></td>
                            </tr>
                          );
                        })
                      )}
                    </tbody>
                  </table>
                </div>
              </div>
            </>
          )}
        </div>
      </div>

      {/* ══════════════════════════════════════
          編集モーダル
      ══════════════════════════════════════ */}
      {editId && editForm && (
        <div className="modal-overlay" onClick={e => { if (e.target === e.currentTarget) { setEditId(null); setDeleteTarget(null); } }}>
          <div className="modal">
            <div className="modal-title">✏️ 物件情報を編集</div>
            <div className="form-grid">
              <div className="form-group">
                <label>売上予定日</label>
                <input type="date" value={editForm.date}
                  onChange={e => setEditForm({ ...editForm, date: e.target.value })} />
              </div>
              <div className="form-group">
                <label>ステータス</label>
                <select value={editForm.status} onChange={e => setEditForm({ ...editForm, status: e.target.value })}>
                  {STATUS_OPTIONS.map(s => <option key={s}>{s}</option>)}
                </select>
              </div>
              <div className="form-group" style={{ gridColumn: "span 2" }}>
                <label>物件名</label>
                <input type="text" value={editForm.name}
                  onChange={e => setEditForm({ ...editForm, name: e.target.value })} />
              </div>
              <div className="form-group">
                <label>売上金額</label>
                <input type="number" value={editForm.sales}
                  onChange={e => setEditForm({ ...editForm, sales: e.target.value })} />
              </div>
              <div className="form-group">
                <label>仕入金額</label>
                <input type="number" value={editForm.purchase}
                  onChange={e => setEditForm({ ...editForm, purchase: e.target.value })} />
              </div>
              <div className="form-group profit-display">
                <label>利益（自動）</label>
                <input readOnly value={calcProfit(editForm.sales, editForm.purchase).toLocaleString("ja-JP")} />
              </div>
              <div className="form-group rate-display">
                <label>利益率（自動）</label>
                <input readOnly value={fmtPct(calcRate(editForm.sales, editForm.purchase))} />
              </div>
              <div className="form-group">
                <label>担当者</label>
                <div style={{ display: "flex", gap: 8, marginTop: 2 }}>
                  {STAFF_OPTIONS.map(s => (
                    <button key={s} type="button"
                      onClick={() => setEditForm({ ...editForm, staff: s })}
                      style={{
                        flex: 1, padding: "8px 0", borderRadius: 6, fontSize: 13, fontWeight: 700,
                        cursor: "pointer", fontFamily: "inherit", transition: "all .15s",
                        background: (editForm.staff || "福田") === s ? "var(--gold)" : "var(--navy3)",
                        color: (editForm.staff || "福田") === s ? "#0b1426" : "var(--muted)",
                        border: (editForm.staff || "福田") === s ? "1px solid var(--gold)" : "1px solid var(--border)",
                      }}>{s}</button>
                  ))}
                </div>
              </div>
              <div className="form-group">
                <label>受注ランク</label>
                <select value={editForm.rank} onChange={e => setEditForm({ ...editForm, rank: e.target.value })}>
                  {RANK_OPTIONS.map(r => <option key={r}>{r}</option>)}
                </select>
              </div>
            </div>

            <div style={{ display: "flex", justifyContent: "space-between", marginTop: 24, alignItems: "flex-start" }}>
              <button className="btn btn-danger" onClick={() => setDeleteTarget(editForm)}>
                🗑 削除
              </button>
              <div style={{ display: "flex", gap: 10 }}>
                <button className="btn btn-secondary" onClick={() => { setEditId(null); setDeleteTarget(null); }}>キャンセル</button>
                <button className="btn btn-primary" onClick={handleEditSave}>保存</button>
              </div>
            </div>

            {/* 削除確認 */}
            {deleteTarget && (
              <div className="delete-confirm">
                <p>⚠️ 削除すると売上見込みから除外されます。<br />
                  本当に削除する場合は下に <strong>「削除確認」</strong> と入力してください。</p>
                <input
                  type="text" placeholder="削除確認" value={deleteInput}
                  onChange={e => setDeleteInput(e.target.value)} />
                <div style={{ display: "flex", gap: 10 }}>
                  <button className="btn btn-secondary" onClick={() => { setDeleteTarget(null); setDeleteInput(""); }}>
                    やめる
                  </button>
                  <button className="btn btn-danger" onClick={handleDelete}>
                    完全に削除する
                  </button>
                </div>
              </div>
            )}
          </div>
        </div>
      )}
    </>
  );
}
