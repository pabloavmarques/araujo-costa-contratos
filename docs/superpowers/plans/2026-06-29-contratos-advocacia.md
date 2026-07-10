# Araujo Costa Contratos — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Criar um arquivo HTML único (`araujo-costa-contratos.html`) com sistema completo de gestão de contratos para o escritório Araujo Costa Advogados Associados, com 203 contratos pré-carregados da planilha, 4 temas visuais, CRUD completo, gráficos, insights via Claude API e configurações editáveis.

**Architecture:** Arquivo HTML único com CSS inline (4 temas via variáveis CSS), JavaScript vanilla puro e persistência em `localStorage`. Nenhuma dependência de servidor — abre direto no browser. Autenticação simples via `sessionStorage`. IA via fetch direto à Anthropic API.

**Tech Stack:** HTML5 · CSS3 (variáveis CSS, Grid, Flexbox) · JavaScript ES6+ (vanilla) · Canvas API (gráficos) · Anthropic Claude API (`claude-sonnet-4-6`) · Google Fonts (Playfair Display + Montserrat) · localStorage/sessionStorage

**Output file:** `C:\projetos\araujo-costa-contratos\araujo-costa-contratos.html`

---

## Estrutura de Funções JS

Todas as funções vivem dentro de um único `<script>` no final do `<body>`. Organizadas por namespace:

```
AUTH.*        — login, logout, verificação de sessão
STORAGE.*     — leitura/escrita localStorage
NAV.*         — navegação entre seções
RENDER.*      — renderização de cada seção
CRUD.*        — criar/editar/excluir contratos
CHART.*       — desenhar gráficos com Canvas API
IA.*          — chamar Claude API e renderizar insights
CONFIG.*      — configurações, listas editáveis, export/import
UTILS.*       — formatação, uuid, helpers
```

---

## Task 1: Esqueleto HTML + 4 Temas CSS + Tela de Login

**Arquivo:** `C:\projetos\araujo-costa-contratos\araujo-costa-contratos.html` (criar)

- [ ] **Criar o arquivo HTML com a estrutura base completa**

```html
<!DOCTYPE html>
<html lang="pt-BR" data-tema="advocacia">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Araujo Costa — Gestão de Contratos</title>
<link href="https://fonts.googleapis.com/css2?family=Playfair+Display:wght@400;600;700&family=Montserrat:wght@300;400;500;600;700&display=swap" rel="stylesheet">
<style>
/* ── TEMA ADVOCACIA (padrão) ── */
:root {
  --bg:#080E09;--bg2:#0D1210;--bg3:#131A14;--bg4:#1a2119;
  --border:rgba(196,150,42,.22);--text:#F7F4EE;--muted:#8D887A;
  --shadow:0 2px 16px rgba(0,0,0,.5);
  --gold:#C4962A;--gold-l:#F0D890;--gold-d:#9A7018;
  --green:#22c55e;--green-d:#14532d;
  --red:#ef4444;--red-d:#7f1d1d;
  --blue:#3b82f6;--blue-d:#1e3a8a;
  --yellow:#f59e0b;--yellow-d:#78350f;
  --r:12px;--rs:8px;
  --font-title:'Playfair Display',serif;
  --font-body:'Montserrat',sans-serif;
  --accent:var(--gold);--accent-l:var(--gold-l);
  --sidebar-w:220px;
}
/* ── TEMA DARK ── */
html[data-tema="dark"] {
  --bg:#0f1117;--bg2:#1a1d27;--bg3:#22263a;--bg4:#2a2f45;
  --border:#2e3347;--text:#e8eaf0;--muted:#8890aa;
  --shadow:0 2px 8px rgba(0,0,0,.35);
  --accent:#4a90e2;--accent-l:#93c5fd;
  --font-title:'Montserrat',sans-serif;--font-body:'Montserrat',sans-serif;
}
/* ── TEMA LIGHT ── */
html[data-tema="light"] {
  --bg:#f0f2f7;--bg2:#ffffff;--bg3:#e8ebf2;--bg4:#dde1ec;
  --border:#c8cde0;--text:#1a1d2e;--muted:#6b7280;
  --shadow:0 2px 8px rgba(0,0,0,.10);
  --accent:#2563eb;--accent-l:#93c5fd;
  --font-title:'Montserrat',sans-serif;--font-body:'Montserrat',sans-serif;
}
/* ── TEMA NEON ── */
html[data-tema="neon"] {
  --bg:#09110E;--bg2:#0D1A14;--bg3:#111F18;--bg4:#162619;
  --border:rgba(0,194,111,.22);--text:#F2F2F2;--muted:#6fa882;
  --shadow:0 2px 16px rgba(0,194,111,.12);
  --accent:#00C26F;--accent-l:#86efac;
  --font-title:'Montserrat',sans-serif;--font-body:'Montserrat',sans-serif;
}

/* ── RESET + BASE ── */
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0}
body{font-family:var(--font-body);background:var(--bg);color:var(--text);min-height:100vh;line-height:1.5}
button{cursor:pointer;font-family:var(--font-body)}
input,select,textarea{font-family:var(--font-body)}

/* ── TELA DE LOGIN ── */
#login-screen{
  position:fixed;inset:0;background:var(--bg);
  display:flex;align-items:center;justify-content:center;z-index:9999;
}
.login-box{
  background:var(--bg2);border:1px solid var(--border);border-radius:var(--r);
  padding:48px 40px;width:100%;max-width:380px;
  box-shadow:var(--shadow);text-align:center;
}
.login-logo{
  font-family:var(--font-title);font-size:1.4rem;font-weight:700;
  color:var(--accent);margin-bottom:6px;letter-spacing:-.3px;
}
.login-sub{font-size:.75rem;color:var(--muted);margin-bottom:32px;letter-spacing:.5px}
.login-field{margin-bottom:16px;text-align:left}
.login-field label{display:block;font-size:.72rem;font-weight:600;color:var(--muted);text-transform:uppercase;letter-spacing:.8px;margin-bottom:6px}
.login-field input{
  width:100%;padding:11px 14px;background:var(--bg3);border:1px solid var(--border);
  border-radius:var(--rs);color:var(--text);font-size:.9rem;outline:none;
  transition:border-color .2s;
}
.login-field input:focus{border-color:var(--accent)}
.login-btn{
  width:100%;padding:13px;background:var(--accent);color:#fff;border:none;
  border-radius:var(--rs);font-size:.95rem;font-weight:700;margin-top:8px;
  transition:opacity .2s;letter-spacing:.3px;
}
.login-btn:hover{opacity:.88}
.login-error{color:var(--red);font-size:.8rem;margin-top:12px;min-height:20px}

/* ── APP LAYOUT ── */
#app{display:none;min-height:100vh}
#app.visible{display:flex}

/* ── SIDEBAR ── */
#sidebar{
  width:var(--sidebar-w);background:var(--bg2);border-right:1px solid var(--border);
  display:flex;flex-direction:column;position:fixed;top:0;left:0;bottom:0;z-index:100;
  padding:0;
}
.sidebar-brand{
  padding:24px 20px 20px;border-bottom:1px solid var(--border);
  font-family:var(--font-title);font-size:1rem;font-weight:700;color:var(--accent);
  line-height:1.3;
}
.sidebar-brand span{display:block;font-family:var(--font-body);font-size:.6rem;font-weight:500;color:var(--muted);text-transform:uppercase;letter-spacing:1px;margin-top:2px}
.sidebar-nav{flex:1;padding:16px 10px;display:flex;flex-direction:column;gap:4px}
.nav-item{
  display:flex;align-items:center;gap:12px;padding:11px 14px;
  border-radius:var(--rs);cursor:pointer;transition:background .15s,color .15s;
  color:var(--muted);font-size:.85rem;font-weight:500;border:none;background:none;width:100%;text-align:left;
}
.nav-item:hover{background:var(--bg3);color:var(--text)}
.nav-item.active{background:rgba(196,150,42,.15);color:var(--accent);font-weight:600}
html[data-tema="dark"] .nav-item.active{background:rgba(74,144,226,.15);color:var(--accent)}
html[data-tema="light"] .nav-item.active{background:rgba(37,99,235,.12);color:var(--accent)}
html[data-tema="neon"] .nav-item.active{background:rgba(0,194,111,.12);color:var(--accent)}
.nav-icon{font-size:1.1rem;width:20px;text-align:center}
.sidebar-footer{padding:16px;border-top:1px solid var(--border)}
.btn-sair{
  width:100%;padding:9px;background:rgba(239,68,68,.1);border:1px solid rgba(239,68,68,.3);
  color:var(--red);border-radius:var(--rs);font-size:.8rem;font-weight:600;
  transition:background .15s;
}
.btn-sair:hover{background:rgba(239,68,68,.2)}

/* ── MAIN CONTENT ── */
#main{margin-left:var(--sidebar-w);flex:1;display:flex;flex-direction:column;min-height:100vh}
#header{
  background:var(--bg2);border-bottom:1px solid var(--border);
  padding:16px 28px;display:flex;align-items:center;justify-content:space-between;
  position:sticky;top:0;z-index:50;
}
.header-title{font-family:var(--font-title);font-size:1.15rem;font-weight:700;color:var(--text)}
.header-right{display:flex;align-items:center;gap:12px}
.tema-badge{font-size:.7rem;color:var(--muted);background:var(--bg3);padding:4px 10px;border-radius:20px;border:1px solid var(--border)}
#content{flex:1;padding:28px;overflow-y:auto}

/* ── CARDS ── */
.card{background:var(--bg2);border:1px solid var(--border);border-radius:var(--r);padding:20px;box-shadow:var(--shadow)}
.card-grid{display:grid;gap:16px}
.grid-4{grid-template-columns:repeat(4,1fr)}
.grid-3{grid-template-columns:repeat(3,1fr)}
.grid-2{grid-template-columns:repeat(2,1fr)}
@media(max-width:1100px){.grid-4{grid-template-columns:repeat(2,1fr)}}
@media(max-width:700px){.grid-4,.grid-3,.grid-2{grid-template-columns:1fr}}
.card-label{font-size:.65rem;text-transform:uppercase;letter-spacing:1px;color:var(--muted);font-weight:700;margin-bottom:8px}
.card-value{font-size:1.7rem;font-weight:900;letter-spacing:-.5px;color:var(--text)}
.card-sub{font-size:.75rem;color:var(--muted);margin-top:4px}

/* ── BADGES ── */
.badge{display:inline-block;padding:3px 10px;border-radius:20px;font-size:.7rem;font-weight:700;letter-spacing:.3px}
.badge-em-andamento{background:rgba(59,130,246,.15);color:#60a5fa;border:1px solid rgba(59,130,246,.3)}
.badge-ganho{background:rgba(34,197,94,.15);color:#4ade80;border:1px solid rgba(34,197,94,.3)}
.badge-perdido{background:rgba(239,68,68,.15);color:#f87171;border:1px solid rgba(239,68,68,.3)}
.badge-aguardando{background:rgba(245,158,11,.15);color:#fbbf24;border:1px solid rgba(245,158,11,.3)}

/* ── BOTÕES ── */
.btn{display:inline-flex;align-items:center;gap:6px;padding:9px 18px;border-radius:var(--rs);font-size:.82rem;font-weight:600;border:none;transition:opacity .15s}
.btn:hover{opacity:.85}
.btn-primary{background:var(--accent);color:#fff}
.btn-secondary{background:var(--bg3);color:var(--text);border:1px solid var(--border)}
.btn-danger{background:rgba(239,68,68,.15);color:var(--red);border:1px solid rgba(239,68,68,.3)}
.btn-sm{padding:5px 12px;font-size:.75rem}

/* ── TABELA ── */
.table-wrap{overflow-x:auto;border-radius:var(--rs)}
table{width:100%;border-collapse:collapse;font-size:.83rem}
thead th{padding:10px 14px;text-align:left;font-size:.65rem;text-transform:uppercase;letter-spacing:.8px;color:var(--muted);font-weight:700;border-bottom:1px solid var(--border);background:var(--bg3);cursor:pointer;white-space:nowrap}
thead th:hover{color:var(--text)}
tbody tr{border-bottom:1px solid var(--border);transition:background .12s}
tbody tr:hover{background:var(--bg3)}
tbody td{padding:10px 14px;color:var(--text);vertical-align:middle}

/* ── MODAL ── */
.modal-overlay{position:fixed;inset:0;background:rgba(0,0,0,.65);z-index:1000;display:none;align-items:center;justify-content:center;padding:20px}
.modal-overlay.open{display:flex}
.modal{background:var(--bg2);border:1px solid var(--border);border-radius:var(--r);width:100%;max-width:540px;max-height:90vh;overflow-y:auto;box-shadow:0 8px 40px rgba(0,0,0,.6)}
.modal-header{padding:20px 24px;border-bottom:1px solid var(--border);display:flex;align-items:center;justify-content:space-between}
.modal-title{font-family:var(--font-title);font-size:1.05rem;font-weight:700}
.modal-close{background:none;border:none;color:var(--muted);font-size:1.2rem;cursor:pointer;padding:4px}
.modal-body{padding:24px}
.modal-footer{padding:16px 24px;border-top:1px solid var(--border);display:flex;gap:10px;justify-content:flex-end}
.form-row{margin-bottom:16px}
.form-row label{display:block;font-size:.7rem;font-weight:700;color:var(--muted);text-transform:uppercase;letter-spacing:.7px;margin-bottom:6px}
.form-row input,.form-row select,.form-row textarea{
  width:100%;padding:10px 13px;background:var(--bg3);border:1px solid var(--border);
  border-radius:var(--rs);color:var(--text);font-size:.88rem;outline:none;
  transition:border-color .2s;
}
.form-row input:focus,.form-row select,.form-row textarea:focus{border-color:var(--accent)}
.form-row select{appearance:none;background-image:url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='12' viewBox='0 0 12 12'%3E%3Cpath fill='%238D887A' d='M6 8L1 3h10z'/%3E%3C/svg%3E");background-repeat:no-repeat;background-position:right 12px center}
.form-grid-2{display:grid;grid-template-columns:1fr 1fr;gap:12px}

/* ── FILTROS ── */
.filter-bar{display:flex;gap:10px;flex-wrap:wrap;align-items:center;margin-bottom:20px}
.filter-bar select,.filter-bar input[type=text]{
  padding:8px 12px;background:var(--bg2);border:1px solid var(--border);
  border-radius:var(--rs);color:var(--text);font-size:.82rem;outline:none;min-width:130px;
}
.filter-bar select:focus,.filter-bar input:focus{border-color:var(--accent)}

/* ── GRÁFICOS ── */
.chart-wrap{position:relative;width:100%}
canvas{width:100%!important}

/* ── IA INSIGHTS ── */
.insight-card{
  background:var(--bg3);border:1px solid var(--border);border-left:3px solid var(--accent);
  border-radius:var(--rs);padding:16px 18px;
}
.insight-icon{font-size:1.3rem;margin-bottom:8px}
.insight-title{font-size:.85rem;font-weight:700;color:var(--text);margin-bottom:4px}
.insight-text{font-size:.8rem;color:var(--muted);line-height:1.5}

/* ── SECTION HEADER ── */
.section-header{display:flex;align-items:center;justify-content:space-between;margin-bottom:24px}
.section-title{font-family:var(--font-title);font-size:1.3rem;font-weight:700;color:var(--text)}
.section-sub{font-size:.75rem;color:var(--muted);margin-top:2px}

/* ── CONFIG LISTAS ── */
.ref-list-item{
  display:flex;align-items:center;gap:8px;padding:8px 0;
  border-bottom:1px solid var(--border);
}
.ref-list-item input{flex:1;background:var(--bg3);border:1px solid var(--border);border-radius:6px;padding:6px 10px;color:var(--text);font-size:.85rem;outline:none}
.ref-list-item input:focus{border-color:var(--accent)}

/* ── SPINNER ── */
.spinner{width:32px;height:32px;border:3px solid var(--border);border-top-color:var(--accent);border-radius:50%;animation:spin .7s linear infinite;margin:0 auto}
@keyframes spin{to{transform:rotate(360deg)}}

/* ── UTIL ── */
.t-gold{color:var(--gold)}.t-green{color:var(--green)}.t-red{color:var(--red)}.t-muted{color:var(--muted)}
.mt-4{margin-top:16px}.mb-4{margin-bottom:16px}.gap-4{gap:16px}
.empty-state{text-align:center;padding:48px 20px;color:var(--muted);font-size:.9rem}
</style>
</head>
<body>

<!-- LOGIN -->
<div id="login-screen">
  <div class="login-box">
    <div class="login-logo">Araujo Costa</div>
    <div class="login-sub">Advogados Associados · Gestão de Contratos</div>
    <div class="login-field">
      <label>Usuário</label>
      <input type="text" id="login-user" placeholder="araujo.costa" autocomplete="username">
    </div>
    <div class="login-field">
      <label>Senha</label>
      <input type="password" id="login-pass" placeholder="••••••••" autocomplete="current-password">
    </div>
    <button class="login-btn" onclick="AUTH.login()">Entrar</button>
    <div class="login-error" id="login-error"></div>
  </div>
</div>

<!-- APP -->
<div id="app">
  <nav id="sidebar">
    <div class="sidebar-brand">
      Araujo Costa
      <span>Gestão de Contratos</span>
    </div>
    <div class="sidebar-nav">
      <button class="nav-item active" data-section="dashboard" onclick="NAV.ir('dashboard')">
        <span class="nav-icon">📊</span> Dashboard
      </button>
      <button class="nav-item" data-section="clientes" onclick="NAV.ir('clientes')">
        <span class="nav-icon">👥</span> Clientes
      </button>
      <button class="nav-item" data-section="relatorios" onclick="NAV.ir('relatorios')">
        <span class="nav-icon">📈</span> Relatórios
      </button>
      <button class="nav-item" data-section="ia" onclick="NAV.ir('ia')">
        <span class="nav-icon">🤖</span> Agente de IA
      </button>
      <button class="nav-item" data-section="config" onclick="NAV.ir('config')">
        <span class="nav-icon">⚙️</span> Configurações
      </button>
    </div>
    <div class="sidebar-footer">
      <button class="btn-sair" onclick="AUTH.logout()">↩ Sair</button>
    </div>
  </nav>
  <div id="main">
    <header id="header">
      <div>
        <div class="header-title" id="header-title">Dashboard</div>
      </div>
      <div class="header-right">
        <span class="tema-badge" id="tema-badge">Advocacia</span>
      </div>
    </header>
    <div id="content"></div>
  </div>
</div>

<!-- MODAL CONTRATO -->
<div class="modal-overlay" id="modal-contrato">
  <div class="modal">
    <div class="modal-header">
      <div class="modal-title" id="modal-titulo">Novo Contrato</div>
      <button class="modal-close" onclick="CRUD.fecharModal()">✕</button>
    </div>
    <div class="modal-body">
      <input type="hidden" id="form-id">
      <div class="form-row">
        <label>Nome do Cliente</label>
        <input type="text" id="form-cliente" placeholder="Nome completo">
      </div>
      <div class="form-grid-2">
        <div class="form-row">
          <label>Data do Contrato</label>
          <input type="text" id="form-data" placeholder="DD/MM/AAAA">
        </div>
        <div class="form-row">
          <label>Status</label>
          <select id="form-status"></select>
        </div>
      </div>
      <div class="form-grid-2">
        <div class="form-row">
          <label>Causa</label>
          <select id="form-causa"></select>
        </div>
        <div class="form-row">
          <label>Origem</label>
          <select id="form-origem"></select>
        </div>
      </div>
      <div class="form-grid-2">
        <div class="form-row">
          <label>Honorários (R$)</label>
          <input type="number" id="form-honorarios" placeholder="0,00" min="0" step="0.01">
        </div>
        <div class="form-row">
          <label>Valor Esperado (R$)</label>
          <input type="number" id="form-esperado" placeholder="0,00" min="0" step="0.01">
        </div>
      </div>
      <div class="form-row">
        <label>Valor Recebido (R$)</label>
        <input type="number" id="form-recebido" placeholder="0,00" min="0" step="0.01">
      </div>
      <div class="form-row">
        <label>Observação</label>
        <textarea id="form-obs" rows="2" placeholder="Observações adicionais"></textarea>
      </div>
    </div>
    <div class="modal-footer">
      <button class="btn btn-secondary" onclick="CRUD.fecharModal()">Cancelar</button>
      <button class="btn btn-primary" onclick="CRUD.salvar()">Salvar</button>
    </div>
  </div>
</div>

<!-- MODAL CONFIRM DELETE -->
<div class="modal-overlay" id="modal-confirm">
  <div class="modal" style="max-width:380px">
    <div class="modal-header">
      <div class="modal-title">Confirmar exclusão</div>
      <button class="modal-close" onclick="document.getElementById('modal-confirm').classList.remove('open')">✕</button>
    </div>
    <div class="modal-body">
      <p style="font-size:.9rem;color:var(--muted)">Tem certeza que deseja excluir este contrato? Esta ação não pode ser desfeita.</p>
    </div>
    <div class="modal-footer">
      <button class="btn btn-secondary" onclick="document.getElementById('modal-confirm').classList.remove('open')">Cancelar</button>
      <button class="btn btn-danger" id="btn-confirm-del">Excluir</button>
    </div>
  </div>
</div>

<script>
'use strict';
</script>
</body>
</html>
```

- [ ] **Verificar:** Abrir o arquivo no browser. Deve aparecer a tela de login com o visual escuro dourado (tema Advocacia). Não deve haver erros no console.

- [ ] **Commit**
```bash
cd C:\projetos\araujo-costa-contratos
git init
git add araujo-costa-contratos.html
git commit -m "feat: estrutura HTML, 4 temas CSS, tela de login e modais"
```

---

## Task 2: Dados + Storage + Auth + Navegação

**Arquivo:** `araujo-costa-contratos.html` (dentro do `<script>`)

- [ ] **Adicionar DADOS_INICIAIS e funções UTILS/STORAGE/AUTH/NAV** (substituir `'use strict';` no `<script>`):

```javascript
'use strict';

// ═══════════════════════════════════════════════
// UTILS
// ═══════════════════════════════════════════════
const UTILS = {
  uuid() {
    return Date.now().toString(36) + Math.random().toString(36).slice(2);
  },
  fmtMoeda(v) {
    return 'R$ ' + (parseFloat(v)||0).toLocaleString('pt-BR',{minimumFractionDigits:2,maximumFractionDigits:2});
  },
  fmtNum(v) {
    return (parseInt(v)||0).toLocaleString('pt-BR');
  },
  mesToNum(mes) {
    return ['Janeiro','Fevereiro','Março','Abril','Maio','Junho',
            'Julho','Agosto','Setembro','Outubro','Novembro','Dezembro'].indexOf(mes)+1;
  },
  badgeStatus(status) {
    const map = {
      'Em Andamento':'badge-em-andamento',
      'Ganho':'badge-ganho',
      'Perdido':'badge-perdido',
      'Aguardando Decisão':'badge-aguardando'
    };
    return `<span class="badge ${map[status]||'badge-em-andamento'}">${status}</span>`;
  },
  escapeHtml(s) {
    return String(s||'').replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;');
  }
};

// ═══════════════════════════════════════════════
// DADOS INICIAIS (203 contratos da planilha)
// ═══════════════════════════════════════════════
// IMPORTANTE: Cole aqui o conteúdo do arquivo:
// C:\projetos\araujo-costa-contratos\dados_iniciais.json
// Envolto em: const DADOS_INICIAIS = [ ... ];
// O arquivo JSON foi gerado automaticamente e contém todos os
// 203 contratos normalizados de 2025 e 2026.
// Veja Task 2b abaixo para o comando de geração.
const DADOS_INICIAIS = []; // substituir conforme Task 2b

// ═══════════════════════════════════════════════
// DEFAULTS de referência
// ═══════════════════════════════════════════════
const DEFAULT_CAUSAS = [
  'Planejamento Previdenciário','Auxílio Doença','Auxílio Doença Rural','Auxílio Doença Urbano',
  'Salário Maternidade','Salário Maternidade Rural','Salário Maternidade Urbano',
  'Seguro Defeso','BPC','Pensão por Morte','Pensão Alimentícia','Trabalhista',
  'Execução de Alimentos','Ação de Alimentos','Ação de Cobrança','Auxílio Acidente',
  'Auxílio Maternidade','Aposentadoria por Invalidez','Aposentadoria por Idade Rural',
  'Aposentadoria Rural','APSI','APTC','Consumidor','Outros'
];
const DEFAULT_ORIGENS = ['Tráfego','Tráfego V4','Tráfego - Cartilha','Indicação','Indicação A','Indicação B','CAMIS','BIA','Escritório','Pro Bono','RUI'];
const DEFAULT_STATUS  = ['Em Andamento','Ganho','Perdido','Aguardando Decisão'];

// ═══════════════════════════════════════════════
// STORAGE
// ═══════════════════════════════════════════════
const STORAGE = {
  get(k, def=null) {
    try { const v = localStorage.getItem(k); return v !== null ? JSON.parse(v) : def; } catch(e){ return def; }
  },
  set(k, v) { try { localStorage.setItem(k, JSON.stringify(v)); } catch(e){} },

  getContratos()  { return STORAGE.get('ac_contratos', null); },
  setContratos(v) { STORAGE.set('ac_contratos', v); },

  getCausas()   { return STORAGE.get('ac_causas', DEFAULT_CAUSAS); },
  getOrigens()  { return STORAGE.get('ac_origens', DEFAULT_ORIGENS); },
  getStatusList(){ return STORAGE.get('ac_status_lista', DEFAULT_STATUS); },
  setCausas(v)  { STORAGE.set('ac_causas', v); },
  setOrigens(v) { STORAGE.set('ac_origens', v); },
  setStatusList(v){ STORAGE.set('ac_status_lista', v); },

  getConfig()   { return STORAGE.get('ac_config', {usuario:'araujo.costa',senha:'advocacia2026',tema:'advocacia',modelo:'claude-sonnet-4-6',api_key:''}); },
  setConfig(v)  { STORAGE.set('ac_config', v); },

  init() {
    if (STORAGE.getContratos() === null) {
      // Primeiro acesso: carrega dados da planilha
      const com_ids = DADOS_INICIAIS.map((c,i) => ({...c, id: UTILS.uuid()}));
      STORAGE.setContratos(com_ids);
    }
    // Aplica tema salvo
    const cfg = STORAGE.getConfig();
    document.documentElement.setAttribute('data-tema', cfg.tema || 'advocacia');
    const nomes = {advocacia:'Advocacia',dark:'Dark',light:'Light',neon:'Neon'};
    const badge = document.getElementById('tema-badge');
    if(badge) badge.textContent = nomes[cfg.tema] || 'Advocacia';
  }
};

// ═══════════════════════════════════════════════
// AUTH
// ═══════════════════════════════════════════════
const AUTH = {
  login() {
    const user = document.getElementById('login-user').value.trim();
    const pass = document.getElementById('login-pass').value;
    const cfg  = STORAGE.getConfig();
    if (user === cfg.usuario && pass === cfg.senha) {
      sessionStorage.setItem('ac_session', '1');
      document.getElementById('login-screen').style.display = 'none';
      document.getElementById('app').classList.add('visible');
      STORAGE.init();
      NAV.ir('dashboard');
    } else {
      document.getElementById('login-error').textContent = 'Usuário ou senha incorretos.';
    }
  },
  logout() {
    sessionStorage.removeItem('ac_session');
    document.getElementById('app').classList.remove('visible');
    document.getElementById('login-screen').style.display = 'flex';
    document.getElementById('login-user').value = '';
    document.getElementById('login-pass').value = '';
    document.getElementById('login-error').textContent = '';
  },
  check() {
    if (sessionStorage.getItem('ac_session') === '1') {
      document.getElementById('login-screen').style.display = 'none';
      document.getElementById('app').classList.add('visible');
      STORAGE.init();
      NAV.ir('dashboard');
    }
  }
};

// Enviar com Enter no login
document.addEventListener('keydown', e => {
  if (e.key === 'Enter' && document.getElementById('login-screen').style.display !== 'none') AUTH.login();
});

// ═══════════════════════════════════════════════
// NAV
// ═══════════════════════════════════════════════
const NAV = {
  atual: null,
  titulos: {
    dashboard: 'Dashboard',
    clientes: 'Clientes',
    relatorios: 'Relatórios',
    ia: 'Agente de IA',
    config: 'Configurações'
  },
  ir(secao) {
    NAV.atual = secao;
    document.querySelectorAll('.nav-item').forEach(b => {
      b.classList.toggle('active', b.dataset.section === secao);
    });
    document.getElementById('header-title').textContent = NAV.titulos[secao] || secao;
    const renders = {
      dashboard: RENDER.dashboard,
      clientes:  RENDER.clientes,
      relatorios:RENDER.relatorios,
      ia:        RENDER.ia,
      config:    RENDER.config
    };
    if (renders[secao]) renders[secao]();
  }
};

// Boot
window.addEventListener('load', () => AUTH.check());
```

- [ ] **Task 2b: Injetar DADOS_INICIAIS com os 203 contratos**

  Ler o arquivo JSON gerado e substituir `const DADOS_INICIAIS = [];` pelo conteúdo real.
  O arquivo está em: `C:\projetos\araujo-costa-contratos\dados_iniciais.json`

  Abrir o arquivo dados_iniciais.json, copiar o array JSON e substituir no HTML:
  ```javascript
  const DADOS_INICIAIS = [ /* colar aqui o conteúdo do dados_iniciais.json */ ];
  ```

- [ ] **Verificar:** Login com `araujo.costa` / `advocacia2026`. Deve entrar no app sem erros. Logout deve voltar para o login. Abrir console e rodar `STORAGE.getContratos().length` — deve retornar 203.

- [ ] **Commit**
```bash
git add araujo-costa-contratos.html
git commit -m "feat: storage, auth, navegação e 203 contratos pré-carregados"
```

---

## Task 3: Dashboard

**Arquivo:** `araujo-costa-contratos.html` — adicionar objeto `RENDER` e funções `CHART` antes do `// Boot`

- [ ] **Adicionar CHART + RENDER.dashboard**

```javascript
// ═══════════════════════════════════════════════
// CHART — gráficos com Canvas API
// ═══════════════════════════════════════════════
const CHART = {
  cores: ['#C4962A','#3b82f6','#22c55e','#ef4444','#f59e0b','#8b5cf6','#06b6d4','#ec4899'],

  barras(canvasId, labels, dados, cor='#C4962A', titulo='') {
    const canvas = document.getElementById(canvasId);
    if (!canvas) return;
    const ctx = canvas.getContext('2d');
    const W = canvas.width = canvas.offsetWidth;
    const H = canvas.height = 200;
    ctx.clearRect(0,0,W,H);
    const max = Math.max(...dados, 1);
    const pad = {l:40,r:10,t:10,b:40};
    const areaW = W - pad.l - pad.r;
    const areaH = H - pad.t - pad.b;
    const barW = Math.max(8, (areaW / labels.length) * 0.6);
    const gap  = areaW / labels.length;
    // grade
    const style = getComputedStyle(document.documentElement);
    const borderColor = style.getPropertyValue('--border').trim() || 'rgba(255,255,255,.1)';
    const textColor   = style.getPropertyValue('--muted').trim() || '#888';
    ctx.strokeStyle = borderColor; ctx.lineWidth = 1;
    for (let i=0; i<=4; i++) {
      const y = pad.t + areaH - (areaH * i/4);
      ctx.beginPath(); ctx.moveTo(pad.l,y); ctx.lineTo(W-pad.r,y); ctx.stroke();
      ctx.fillStyle = textColor; ctx.font = '10px Montserrat,sans-serif';
      ctx.fillText(Math.round(max*i/4), 2, y+4);
    }
    dados.forEach((v,i) => {
      const x = pad.l + gap*i + gap/2 - barW/2;
      const h = (v/max) * areaH;
      const y = pad.t + areaH - h;
      ctx.fillStyle = cor;
      ctx.beginPath();
      ctx.roundRect ? ctx.roundRect(x,y,barW,h,4) : ctx.rect(x,y,barW,h);
      ctx.fill();
      // label eixo X
      ctx.fillStyle = textColor; ctx.font = '9px Montserrat,sans-serif';
      ctx.textAlign='center';
      ctx.fillText(labels[i].slice(0,3), pad.l + gap*i + gap/2, H-pad.b+14);
      // valor acima
      if (v>0) {
        ctx.fillStyle = cor; ctx.font = 'bold 10px Montserrat,sans-serif';
        ctx.fillText(v, pad.l + gap*i + gap/2, y-4);
      }
    });
  },

  donut(canvasId, labels, dados) {
    const canvas = document.getElementById(canvasId);
    if (!canvas) return;
    const ctx = canvas.getContext('2d');
    const SIZE = 160;
    canvas.width = SIZE; canvas.height = SIZE;
    ctx.clearRect(0,0,SIZE,SIZE);
    const total = dados.reduce((a,b)=>a+b,0);
    if (!total) return;
    let ang = -Math.PI/2;
    dados.forEach((v,i) => {
      const slice = (v/total) * 2 * Math.PI;
      ctx.beginPath();
      ctx.moveTo(SIZE/2,SIZE/2);
      ctx.arc(SIZE/2,SIZE/2,SIZE/2-6,ang,ang+slice);
      ctx.closePath();
      ctx.fillStyle = CHART.cores[i % CHART.cores.length];
      ctx.fill();
      ang += slice;
    });
    // buraco central
    const style = getComputedStyle(document.documentElement);
    ctx.beginPath();
    ctx.arc(SIZE/2,SIZE/2,SIZE/2-30,0,2*Math.PI);
    ctx.fillStyle = style.getPropertyValue('--bg2').trim() || '#0D1210';
    ctx.fill();
    // total no centro
    ctx.fillStyle = style.getPropertyValue('--text').trim() || '#F7F4EE';
    ctx.font = 'bold 18px Montserrat,sans-serif';
    ctx.textAlign='center'; ctx.textBaseline='middle';
    ctx.fillText(total, SIZE/2, SIZE/2);
  },

  barrasH(canvasId, labels, dados) {
    const canvas = document.getElementById(canvasId);
    if (!canvas) return;
    const ctx = canvas.getContext('2d');
    const W = canvas.width = canvas.offsetWidth;
    const H = canvas.height = labels.length * 36 + 20;
    ctx.clearRect(0,0,W,H);
    const max = Math.max(...dados,1);
    const style = getComputedStyle(document.documentElement);
    const textColor = style.getPropertyValue('--muted').trim() || '#888';
    const padL = 140;
    labels.forEach((lbl,i) => {
      const y = 20 + i*36;
      const barW = ((dados[i]||0)/max) * (W - padL - 60);
      ctx.fillStyle = textColor; ctx.font = '11px Montserrat,sans-serif';
      ctx.textAlign='right'; ctx.fillText(lbl, padL-8, y+14);
      ctx.fillStyle = CHART.cores[i % CHART.cores.length];
      ctx.beginPath();
      ctx.roundRect ? ctx.roundRect(padL,y,Math.max(barW,2),24,4) : ctx.rect(padL,y,Math.max(barW,2),24);
      ctx.fill();
      ctx.fillStyle = style.getPropertyValue('--text').trim() || '#F7F4EE';
      ctx.font = 'bold 11px Montserrat,sans-serif'; ctx.textAlign='left';
      ctx.fillText(dados[i], padL + barW + 8, y+15);
    });
  }
};

// ═══════════════════════════════════════════════
// RENDER
// ═══════════════════════════════════════════════
const RENDER = {
  dashboard() {
    const contratos = STORAGE.getContratos() || [];
    const total = contratos.length;
    const ganhos = contratos.filter(c=>c.status==='Ganho').length;
    const andamento = contratos.filter(c=>c.status==='Em Andamento').length;
    const perdidos = contratos.filter(c=>c.status==='Perdido').length;
    const taxaSucesso = (ganhos+perdidos) > 0 ? Math.round((ganhos/(ganhos+perdidos))*100) : 0;
    const totalHon = contratos.reduce((s,c)=>s+(parseFloat(c.honorarios)||0),0);
    const totalRec = contratos.reduce((s,c)=>s+(parseFloat(c.valor_recebido)||0),0);
    const totalEsp = contratos.reduce((s,c)=>s+(parseFloat(c.valor_esperado)||0),0);
    const aReceber = Math.max(0, totalEsp - totalRec);

    // Contratos por mês (ano corrente ou últimos 12)
    const MESES = ['Janeiro','Fevereiro','Março','Abril','Maio','Junho','Julho','Agosto','Setembro','Outubro','Novembro','Dezembro'];
    const porMes = MESES.map(m => contratos.filter(c=>c.mes===m).length);

    // Por causa (top 5)
    const causaCount = {};
    contratos.forEach(c=>{ if(c.causa) causaCount[c.causa]=(causaCount[c.causa]||0)+1; });
    const causaTop = Object.entries(causaCount).sort((a,b)=>b[1]-a[1]).slice(0,5);
    const outrosCausa = total - causaTop.reduce((s,[,v])=>s+v,0);
    const causaLabels = [...causaTop.map(([k])=>k.slice(0,22)), ...(outrosCausa>0?['Outros']:[])];
    const causaDados  = [...causaTop.map(([,v])=>v), ...(outrosCausa>0?[outrosCausa]:[])];

    // Por origem
    const origCount = {};
    contratos.forEach(c=>{ if(c.origem) origCount[c.origem]=(origCount[c.origem]||0)+1; });
    const origEntries = Object.entries(origCount).sort((a,b)=>b[1]-a[1]).slice(0,7);

    document.getElementById('content').innerHTML = `
      <div class="card-grid grid-4 mb-4">
        <div class="card">
          <div class="card-label">Total de Contratos</div>
          <div class="card-value">${total}</div>
          <div class="card-sub">2025 + 2026</div>
        </div>
        <div class="card">
          <div class="card-label">Contratos Ganhos</div>
          <div class="card-value t-green">${ganhos}</div>
          <div class="card-sub">${Math.round((ganhos/Math.max(total,1))*100)}% do total</div>
        </div>
        <div class="card">
          <div class="card-label">Em Andamento</div>
          <div class="card-value" style="color:var(--accent)">${andamento}</div>
          <div class="card-sub">${Math.round((andamento/Math.max(total,1))*100)}% do total</div>
        </div>
        <div class="card">
          <div class="card-label">Taxa de Sucesso</div>
          <div class="card-value">${taxaSucesso}%</div>
          <div class="card-sub">${ganhos} ganhos / ${perdidos} perdidos</div>
        </div>
      </div>

      <div class="card-grid grid-3 mb-4">
        <div class="card">
          <div class="card-label">Total de Honorários</div>
          <div class="card-value" style="font-size:1.3rem">${UTILS.fmtMoeda(totalHon)}</div>
        </div>
        <div class="card">
          <div class="card-label">Total Recebido</div>
          <div class="card-value t-green" style="font-size:1.3rem">${UTILS.fmtMoeda(totalRec)}</div>
        </div>
        <div class="card">
          <div class="card-label">A Receber</div>
          <div class="card-value t-gold" style="font-size:1.3rem">${UTILS.fmtMoeda(aReceber)}</div>
        </div>
      </div>

      <div class="card-grid grid-2 mb-4">
        <div class="card">
          <div class="card-label" style="margin-bottom:16px">Contratos por Mês</div>
          <div class="chart-wrap">
            <canvas id="chart-mes" style="height:200px"></canvas>
          </div>
        </div>
        <div class="card">
          <div class="card-label" style="margin-bottom:16px">Distribuição por Origem</div>
          <div class="chart-wrap">
            <canvas id="chart-origem" style="height:${origEntries.length*36+20}px"></canvas>
          </div>
        </div>
      </div>

      <div class="card-grid grid-2 mb-4">
        <div class="card">
          <div class="card-label" style="margin-bottom:16px">Causas Mais Frequentes</div>
          <div style="display:flex;align-items:center;gap:20px">
            <canvas id="chart-causa" style="width:160px;height:160px;flex-shrink:0"></canvas>
            <div style="flex:1">
              ${causaLabels.map((l,i)=>`
                <div style="display:flex;align-items:center;gap:8px;margin-bottom:6px">
                  <span style="width:10px;height:10px;border-radius:50%;background:${CHART.cores[i]};flex-shrink:0"></span>
                  <span style="font-size:.75rem;color:var(--muted);flex:1">${l}</span>
                  <span style="font-size:.8rem;font-weight:700;color:var(--text)">${causaDados[i]}</span>
                </div>`).join('')}
            </div>
          </div>
        </div>
        <div class="card" id="ia-dashboard">
          <div class="card-label" style="margin-bottom:12px">🤖 Insights da IA</div>
          <div style="color:var(--muted);font-size:.82rem">
            Acesse a seção <strong>Agente de IA</strong> para gerar insights automáticos sobre os contratos.
          </div>
        </div>
      </div>
    `;

    // Render charts after DOM update
    requestAnimationFrame(() => {
      CHART.barras('chart-mes', MESES.map(m=>m.slice(0,3)), porMes);
      CHART.donut('chart-causa', causaLabels, causaDados);
      CHART.barrasH('chart-origem', origEntries.map(([k])=>k), origEntries.map(([,v])=>v));
    });
  },

  // Placeholders para tasks seguintes
  clientes()  { document.getElementById('content').innerHTML = '<div class="empty-state">Carregando...</div>'; RENDER._clientes(); },
  relatorios(){ document.getElementById('content').innerHTML = '<div class="empty-state">Carregando...</div>'; RENDER._relatorios(); },
  ia()        { document.getElementById('content').innerHTML = '<div class="empty-state">Carregando...</div>'; RENDER._ia(); },
  config()    { document.getElementById('content').innerHTML = '<div class="empty-state">Carregando...</div>'; RENDER._config(); },
  _clientes()  {},
  _relatorios(){},
  _ia()        {},
  _config()    {}
};
```

- [ ] **Verificar:** Fazer login, ver Dashboard. Os 4 cards KPI devem mostrar 203 total. Os 3 cards financeiros devem mostrar R$ 0,00 (dados sem valores ainda). Os gráficos devem renderizar sem erros.

- [ ] **Commit**
```bash
git add araujo-costa-contratos.html
git commit -m "feat: dashboard com KPIs, cards financeiros e gráficos"
```

---

## Task 4: Seção Clientes (Tabela + CRUD)

**Arquivo:** `araujo-costa-contratos.html` — substituir `_clientes(){}` e adicionar `CRUD`

- [ ] **Substituir `_clientes(){}` por implementação completa**

```javascript
  _clientes() {
    const contratos = STORAGE.getContratos() || [];
    const causas = STORAGE.getCausas();
    const origens = STORAGE.getOrigens();
    const statusList = STORAGE.getStatusList();

    document.getElementById('content').innerHTML = `
      <div class="section-header">
        <div>
          <div class="section-title">Clientes</div>
          <div class="section-sub">${contratos.length} contratos cadastrados</div>
        </div>
        <button class="btn btn-primary" onclick="CRUD.abrirNovo()">+ Novo Contrato</button>
      </div>
      <div class="filter-bar">
        <input type="text" id="f-busca" placeholder="🔍 Buscar cliente..." oninput="RENDER._tabela()" style="min-width:200px">
        <select id="f-mes" onchange="RENDER._tabela()">
          <option value="">Todos os meses</option>
          ${['Janeiro','Fevereiro','Março','Abril','Maio','Junho','Julho','Agosto','Setembro','Outubro','Novembro','Dezembro'].map(m=>`<option>${m}</option>`).join('')}
        </select>
        <select id="f-ano" onchange="RENDER._tabela()">
          <option value="">Todos os anos</option>
          <option>2025</option><option>2026</option>
        </select>
        <select id="f-causa" onchange="RENDER._tabela()">
          <option value="">Todas as causas</option>
          ${causas.map(c=>`<option>${UTILS.escapeHtml(c)}</option>`).join('')}
        </select>
        <select id="f-origem" onchange="RENDER._tabela()">
          <option value="">Todas as origens</option>
          ${origens.map(o=>`<option>${UTILS.escapeHtml(o)}</option>`).join('')}
        </select>
        <select id="f-status" onchange="RENDER._tabela()">
          <option value="">Todos os status</option>
          ${statusList.map(s=>`<option>${UTILS.escapeHtml(s)}</option>`).join('')}
        </select>
      </div>
      <div class="card">
        <div class="table-wrap">
          <table>
            <thead>
              <tr>
                <th onclick="CRUD.ordenar('cliente')">Cliente ↕</th>
                <th onclick="CRUD.ordenar('data')">Data ↕</th>
                <th onclick="CRUD.ordenar('causa')">Causa ↕</th>
                <th onclick="CRUD.ordenar('origem')">Origem ↕</th>
                <th onclick="CRUD.ordenar('status')">Status ↕</th>
                <th onclick="CRUD.ordenar('honorarios')" style="text-align:right">Honorários ↕</th>
                <th onclick="CRUD.ordenar('valor_recebido')" style="text-align:right">Recebido ↕</th>
                <th>Ações</th>
              </tr>
            </thead>
            <tbody id="tbody-contratos"></tbody>
          </table>
        </div>
        <div id="tabela-info" style="padding:12px 14px;font-size:.75rem;color:var(--muted)"></div>
      </div>
    `;
    RENDER._tabela();
  },

  _tabela() {
    const contratos = STORAGE.getContratos() || [];
    const busca  = (document.getElementById('f-busca')?.value||'').toLowerCase();
    const mes    = document.getElementById('f-mes')?.value||'';
    const ano    = document.getElementById('f-ano')?.value||'';
    const causa  = document.getElementById('f-causa')?.value||'';
    const origem = document.getElementById('f-origem')?.value||'';
    const status = document.getElementById('f-status')?.value||'';

    let filtrados = contratos.filter(c => {
      if (busca  && !c.cliente.toLowerCase().includes(busca)) return false;
      if (mes    && c.mes !== mes) return false;
      if (ano    && String(c.ano) !== ano) return false;
      if (causa  && c.causa !== causa) return false;
      if (origem && c.origem !== origem) return false;
      if (status && c.status !== status) return false;
      return true;
    });

    // Ordenação
    const ord = CRUD._ord || {campo:'id',asc:false};
    filtrados.sort((a,b) => {
      let va = a[ord.campo]||'', vb = b[ord.campo]||'';
      if (typeof va === 'number') return ord.asc ? va-vb : vb-va;
      return ord.asc ? String(va).localeCompare(String(vb),'pt-BR') : String(vb).localeCompare(String(va),'pt-BR');
    });

    const tbody = document.getElementById('tbody-contratos');
    if (!tbody) return;
    if (!filtrados.length) {
      tbody.innerHTML = `<tr><td colspan="8" style="text-align:center;padding:32px;color:var(--muted)">Nenhum contrato encontrado</td></tr>`;
    } else {
      tbody.innerHTML = filtrados.map(c => `
        <tr>
          <td style="font-weight:600">${UTILS.escapeHtml(c.cliente)}</td>
          <td style="white-space:nowrap">${UTILS.escapeHtml(c.data)}</td>
          <td style="max-width:160px;overflow:hidden;text-overflow:ellipsis;white-space:nowrap" title="${UTILS.escapeHtml(c.causa)}">${UTILS.escapeHtml(c.causa)}</td>
          <td>${UTILS.escapeHtml(c.origem)}</td>
          <td>${UTILS.badgeStatus(c.status)}</td>
          <td style="text-align:right;white-space:nowrap">${UTILS.fmtMoeda(c.honorarios)}</td>
          <td style="text-align:right;white-space:nowrap">${UTILS.fmtMoeda(c.valor_recebido)}</td>
          <td style="white-space:nowrap">
            <button class="btn btn-sm btn-secondary" onclick="CRUD.editar('${c.id}')" title="Editar">✏️</button>
            <button class="btn btn-sm btn-danger" onclick="CRUD.confirmarExcluir('${c.id}')" title="Excluir">🗑️</button>
          </td>
        </tr>`).join('');
    }
    const info = document.getElementById('tabela-info');
    if (info) info.textContent = `Exibindo ${filtrados.length} de ${contratos.length} contratos`;
  },
```

- [ ] **Adicionar objeto CRUD antes do `// Boot`**

```javascript
// ═══════════════════════════════════════════════
// CRUD
// ═══════════════════════════════════════════════
const CRUD = {
  _ord: {campo:'id', asc:false},

  ordenar(campo) {
    if (CRUD._ord.campo === campo) CRUD._ord.asc = !CRUD._ord.asc;
    else { CRUD._ord.campo = campo; CRUD._ord.asc = true; }
    RENDER._tabela();
  },

  _popularSelects() {
    const causas  = STORAGE.getCausas();
    const origens = STORAGE.getOrigens();
    const statusL = STORAGE.getStatusList();
    const sel = (id, arr, val) => {
      const el = document.getElementById(id);
      if (!el) return;
      el.innerHTML = arr.map(v=>`<option value="${UTILS.escapeHtml(v)}" ${v===val?'selected':''}>${UTILS.escapeHtml(v)}</option>`).join('');
    };
    sel('form-causa',  causas,  '');
    sel('form-origem', origens, '');
    sel('form-status', statusL, 'Em Andamento');
  },

  abrirNovo() {
    document.getElementById('modal-titulo').textContent = 'Novo Contrato';
    document.getElementById('form-id').value = '';
    ['form-cliente','form-data','form-honorarios','form-esperado','form-recebido','form-obs'].forEach(id=>{
      document.getElementById(id).value = '';
    });
    CRUD._popularSelects();
    document.getElementById('modal-contrato').classList.add('open');
  },

  editar(id) {
    const contratos = STORAGE.getContratos() || [];
    const c = contratos.find(x=>x.id===id);
    if (!c) return;
    document.getElementById('modal-titulo').textContent = 'Editar Contrato';
    document.getElementById('form-id').value = c.id;
    document.getElementById('form-cliente').value = c.cliente;
    document.getElementById('form-data').value = c.data;
    document.getElementById('form-honorarios').value = c.honorarios||0;
    document.getElementById('form-esperado').value = c.valor_esperado||0;
    document.getElementById('form-recebido').value = c.valor_recebido||0;
    document.getElementById('form-obs').value = c.observacao||'';
    CRUD._popularSelects();
    document.getElementById('form-causa').value = c.causa;
    document.getElementById('form-origem').value = c.origem;
    document.getElementById('form-status').value = c.status;
    document.getElementById('modal-contrato').classList.add('open');
  },

  fecharModal() {
    document.getElementById('modal-contrato').classList.remove('open');
  },

  salvar() {
    const id       = document.getElementById('form-id').value;
    const cliente  = document.getElementById('form-cliente').value.trim();
    const dataStr  = document.getElementById('form-data').value.trim();
    if (!cliente) { alert('Informe o nome do cliente.'); return; }
    const causa    = document.getElementById('form-causa').value;
    const origem   = document.getElementById('form-origem').value;
    const status   = document.getElementById('form-status').value;
    const honorarios    = parseFloat(document.getElementById('form-honorarios').value)||0;
    const valor_esperado= parseFloat(document.getElementById('form-esperado').value)||0;
    const valor_recebido= parseFloat(document.getElementById('form-recebido').value)||0;
    const observacao    = document.getElementById('form-obs').value.trim();

    // Extrair mês/ano da data
    let mes = '', ano = new Date().getFullYear();
    const partes = dataStr.split('/');
    if (partes.length >= 2) {
      const MESES = ['','Janeiro','Fevereiro','Março','Abril','Maio','Junho','Julho','Agosto','Setembro','Outubro','Novembro','Dezembro'];
      mes = MESES[parseInt(partes[1])||0] || '';
      ano = parseInt(partes[2]) || ano;
    }

    let contratos = STORAGE.getContratos() || [];
    if (id) {
      // Editar
      const idx = contratos.findIndex(c=>c.id===id);
      if (idx>=0) contratos[idx] = {...contratos[idx], cliente, data:dataStr, mes, ano, causa, origem, status, honorarios, valor_esperado, valor_recebido, observacao};
    } else {
      // Novo — vai ao topo
      contratos.unshift({ id:UTILS.uuid(), cliente, data:dataStr, mes, ano, causa, origem, status, honorarios, valor_esperado, valor_recebido, observacao });
    }
    STORAGE.setContratos(contratos);
    CRUD.fecharModal();
    RENDER._clientes();
  },

  confirmarExcluir(id) {
    document.getElementById('modal-confirm').classList.add('open');
    document.getElementById('btn-confirm-del').onclick = () => {
      let contratos = (STORAGE.getContratos()||[]).filter(c=>c.id!==id);
      STORAGE.setContratos(contratos);
      document.getElementById('modal-confirm').classList.remove('open');
      RENDER._clientes();
    };
  }
};
```

- [ ] **Verificar:** Ir para Clientes. Tabela deve mostrar 203 linhas. Filtros devem funcionar em tempo real. Criar novo contrato, salvar — aparece no topo. Editar — dados preenchidos. Excluir com confirmação.

- [ ] **Commit**
```bash
git add araujo-costa-contratos.html
git commit -m "feat: seção Clientes com tabela, filtros, ordenação e CRUD completo"
```

---

## Task 5: Seção Relatórios

**Arquivo:** `araujo-costa-contratos.html` — substituir `_relatorios(){}`

- [ ] **Substituir `_relatorios(){}` por implementação completa**

```javascript
  _relatorios() {
    const MESES_LIST = ['Janeiro','Fevereiro','Março','Abril','Maio','Junho','Julho','Agosto','Setembro','Outubro','Novembro','Dezembro'];
    const causas  = STORAGE.getCausas();
    const origens = STORAGE.getOrigens();
    const statusList = STORAGE.getStatusList();

    document.getElementById('content').innerHTML = `
      <div class="section-header">
        <div>
          <div class="section-title">Relatórios</div>
          <div class="section-sub">Visualizações com filtros dinâmicos</div>
        </div>
      </div>
      <div class="filter-bar" style="margin-bottom:24px">
        <select id="r-mes" onchange="RENDER._calcRelatorios()">
          <option value="">Todos os meses</option>
          ${MESES_LIST.map(m=>`<option>${m}</option>`).join('')}
        </select>
        <select id="r-ano" onchange="RENDER._calcRelatorios()">
          <option value="">Todos os anos</option>
          <option>2025</option><option>2026</option>
        </select>
        <select id="r-causa" onchange="RENDER._calcRelatorios()">
          <option value="">Todas as causas</option>
          ${causas.map(c=>`<option>${UTILS.escapeHtml(c)}</option>`).join('')}
        </select>
        <select id="r-origem" onchange="RENDER._calcRelatorios()">
          <option value="">Todas as origens</option>
          ${origens.map(o=>`<option>${UTILS.escapeHtml(o)}</option>`).join('')}
        </select>
        <select id="r-status" onchange="RENDER._calcRelatorios()">
          <option value="">Todos os status</option>
          ${statusList.map(s=>`<option>${UTILS.escapeHtml(s)}</option>`).join('')}
        </select>
      </div>
      <div id="relatorio-content"></div>
    `;
    RENDER._calcRelatorios();
  },

  _calcRelatorios() {
    const MESES_LIST = ['Janeiro','Fevereiro','Março','Abril','Maio','Junho','Julho','Agosto','Setembro','Outubro','Novembro','Dezembro'];
    const allContratos = STORAGE.getContratos() || [];
    const fMes    = document.getElementById('r-mes')?.value||'';
    const fAno    = document.getElementById('r-ano')?.value||'';
    const fCausa  = document.getElementById('r-causa')?.value||'';
    const fOrigem = document.getElementById('r-origem')?.value||'';
    const fStatus = document.getElementById('r-status')?.value||'';

    const contratos = allContratos.filter(c => {
      if (fMes    && c.mes !== fMes) return false;
      if (fAno    && String(c.ano) !== fAno) return false;
      if (fCausa  && c.causa !== fCausa) return false;
      if (fOrigem && c.origem !== fOrigem) return false;
      if (fStatus && c.status !== fStatus) return false;
      return true;
    });

    // Por mês
    const porMes = MESES_LIST.map(m => contratos.filter(c=>c.mes===m).length);
    // Por ano
    const anos2025 = MESES_LIST.map(m => contratos.filter(c=>c.mes===m&&c.ano===2025).length);
    const anos2026 = MESES_LIST.map(m => contratos.filter(c=>c.mes===m&&c.ano===2026).length);
    // Receita mensal
    const honMes = MESES_LIST.map(m => contratos.filter(c=>c.mes===m).reduce((s,c)=>s+(parseFloat(c.honorarios)||0),0));
    const recMes = MESES_LIST.map(m => contratos.filter(c=>c.mes===m).reduce((s,c)=>s+(parseFloat(c.valor_recebido)||0),0));

    // Ranking causas
    const causaMap = {};
    contratos.forEach(c=>{ causaMap[c.causa]=(causaMap[c.causa]||{n:0,v:0}); causaMap[c.causa].n++; causaMap[c.causa].v+=parseFloat(c.honorarios)||0; });
    const causaRank = Object.entries(causaMap).sort((a,b)=>b[1].n-a[1].n);

    // Ranking origens
    const origMap = {};
    contratos.forEach(c=>{ origMap[c.origem]=(origMap[c.origem]||0)+1; });
    const origRank = Object.entries(origMap).sort((a,b)=>b[1]-a[1]);
    const totalOrig = contratos.length;

    document.getElementById('relatorio-content').innerHTML = `
      <div class="card-grid grid-2 mb-4">
        <div class="card">
          <div class="card-label mb-4">Contratos por Mês (todos os anos)</div>
          <div class="chart-wrap"><canvas id="r-chart-mes" style="height:200px"></canvas></div>
        </div>
        <div class="card">
          <div class="card-label mb-4">Comparativo Anual: 2025 vs 2026</div>
          <div class="chart-wrap"><canvas id="r-chart-anual" style="height:200px"></canvas></div>
        </div>
      </div>
      <div class="card-grid grid-2 mb-4">
        <div class="card">
          <div class="card-label mb-4">Ranking de Causas</div>
          <div class="table-wrap">
            <table>
              <thead><tr><th>Causa</th><th style="text-align:right">Qtd</th><th style="text-align:right">Honorários</th></tr></thead>
              <tbody>
                ${causaRank.map(([k,v])=>`
                  <tr>
                    <td>${UTILS.escapeHtml(k||'N/A')}</td>
                    <td style="text-align:right;font-weight:700">${v.n}</td>
                    <td style="text-align:right">${UTILS.fmtMoeda(v.v)}</td>
                  </tr>`).join('')}
              </tbody>
            </table>
          </div>
        </div>
        <div class="card">
          <div class="card-label mb-4">Ranking de Origens de Captação</div>
          <div class="table-wrap">
            <table>
              <thead><tr><th>Origem</th><th style="text-align:right">Qtd</th><th style="text-align:right">%</th></tr></thead>
              <tbody>
                ${origRank.map(([k,v])=>`
                  <tr>
                    <td>${UTILS.escapeHtml(k||'N/A')}</td>
                    <td style="text-align:right;font-weight:700">${v}</td>
                    <td style="text-align:right">${totalOrig>0?Math.round((v/totalOrig)*100):0}%</td>
                  </tr>`).join('')}
              </tbody>
            </table>
          </div>
        </div>
      </div>
      <div class="card mb-4">
        <div class="card-label mb-4">Receita Mensal: Honorários Contratados vs Recebidos</div>
        <div class="chart-wrap"><canvas id="r-chart-receita" style="height:200px"></canvas></div>
      </div>
      <div class="card">
        <div class="card-label mb-4">Contratos Filtrados (${contratos.length})</div>
        <div class="table-wrap">
          <table>
            <thead><tr><th>Cliente</th><th>Data</th><th>Causa</th><th>Origem</th><th>Status</th><th style="text-align:right">Honorários</th><th style="text-align:right">Recebido</th></tr></thead>
            <tbody>
              ${contratos.slice(0,50).map(c=>`
                <tr>
                  <td style="font-weight:600">${UTILS.escapeHtml(c.cliente)}</td>
                  <td>${UTILS.escapeHtml(c.data)}</td>
                  <td style="max-width:140px;overflow:hidden;text-overflow:ellipsis;white-space:nowrap">${UTILS.escapeHtml(c.causa)}</td>
                  <td>${UTILS.escapeHtml(c.origem)}</td>
                  <td>${UTILS.badgeStatus(c.status)}</td>
                  <td style="text-align:right">${UTILS.fmtMoeda(c.honorarios)}</td>
                  <td style="text-align:right">${UTILS.fmtMoeda(c.valor_recebido)}</td>
                </tr>`).join('')}
              ${contratos.length>50?`<tr><td colspan="7" style="text-align:center;padding:12px;color:var(--muted)">... e mais ${contratos.length-50} contratos</td></tr>`:''}
            </tbody>
          </table>
        </div>
      </div>
    `;

    requestAnimationFrame(() => {
      CHART.barras('r-chart-mes', MESES_LIST.map(m=>m.slice(0,3)), porMes);
      // Gráfico comparativo 2025 vs 2026 (barras agrupadas — simplificado)
      CHART.barras('r-chart-anual', MESES_LIST.map(m=>m.slice(0,3)), anos2026, '#C4962A');
      CHART.barras('r-chart-receita', MESES_LIST.map(m=>m.slice(0,3)), honMes, '#3b82f6');
    });
  },
```

- [ ] **Verificar:** Ir para Relatórios. Todos os filtros devem atualizar os gráficos em tempo real. Tabela de contratos filtrados deve aparecer no fundo.

- [ ] **Commit**
```bash
git add araujo-costa-contratos.html
git commit -m "feat: seção Relatórios com filtros dinâmicos, gráficos e rankings"
```

---

## Task 6: Agente de IA

**Arquivo:** `araujo-costa-contratos.html` — substituir `_ia(){}` e adicionar objeto `IA`

- [ ] **Substituir `_ia(){}` por implementação completa**

```javascript
  _ia() {
    document.getElementById('content').innerHTML = `
      <div class="section-header">
        <div>
          <div class="section-title">Agente de IA</div>
          <div class="section-sub">Insights automáticos gerados por Claude (Anthropic)</div>
        </div>
        <button class="btn btn-primary" onclick="IA.gerar()" id="btn-gerar-ia">✨ Gerar Insights</button>
      </div>
      <div id="ia-status" style="margin-bottom:20px"></div>
      <div id="ia-insights" class="card-grid grid-2"></div>
    `;
  },
```

- [ ] **Adicionar objeto IA antes do `// Boot`**

```javascript
// ═══════════════════════════════════════════════
// IA
// ═══════════════════════════════════════════════
const IA = {
  async gerar() {
    const cfg = STORAGE.getConfig();
    if (!cfg.api_key) {
      document.getElementById('ia-status').innerHTML = `
        <div class="card" style="border-color:rgba(239,68,68,.4);background:rgba(239,68,68,.08)">
          <span style="color:var(--red)">⚠️ Configure sua Anthropic API Key em <strong>Configurações → API</strong> antes de gerar insights.</span>
        </div>`;
      return;
    }

    const btn = document.getElementById('btn-gerar-ia');
    if (btn) btn.disabled = true;
    document.getElementById('ia-status').innerHTML = `
      <div style="text-align:center;padding:32px">
        <div class="spinner"></div>
        <div style="margin-top:16px;color:var(--muted);font-size:.85rem">Analisando contratos com Claude...</div>
      </div>`;
    document.getElementById('ia-insights').innerHTML = '';

    const contratos = STORAGE.getContratos() || [];
    const total = contratos.length;
    const ganhos = contratos.filter(c=>c.status==='Ganho').length;
    const perdidos = contratos.filter(c=>c.status==='Perdido').length;
    const andamento = contratos.filter(c=>c.status==='Em Andamento').length;
    const aguardando = contratos.filter(c=>c.status==='Aguardando Decisão').length;
    const taxaSucesso = (ganhos+perdidos)>0 ? Math.round((ganhos/(ganhos+perdidos))*100) : 0;
    const totalHon = contratos.reduce((s,c)=>s+(parseFloat(c.honorarios)||0),0);
    const totalRec = contratos.reduce((s,c)=>s+(parseFloat(c.valor_recebido)||0),0);
    const totalEsp = contratos.reduce((s,c)=>s+(parseFloat(c.valor_esperado)||0),0);
    const aReceber = Math.max(0,totalEsp-totalRec);

    // Contagens por causa
    const causaCount = {};
    contratos.forEach(c=>{ causaCount[c.causa]=(causaCount[c.causa]||0)+1; });
    const causaTop3 = Object.entries(causaCount).sort((a,b)=>b[1]-a[1]).slice(0,3).map(([k,v])=>`${k}(${v})`).join(', ');

    // Contagens por origem
    const origCount = {};
    contratos.forEach(c=>{ origCount[c.origem]=(origCount[c.origem]||0)+1; });
    const origTop3 = Object.entries(origCount).sort((a,b)=>b[1]-a[1]).slice(0,3).map(([k,v])=>`${k}(${v})`).join(', ');

    // Por mês (últimos 6)
    const MESES = ['Janeiro','Fevereiro','Março','Abril','Maio','Junho','Julho','Agosto','Setembro','Outubro','Novembro','Dezembro'];
    const porMes = MESES.map(m=>({mes:m, n:contratos.filter(c=>c.mes===m).length})).filter(x=>x.n>0).slice(-6);

    const contexto = `
Escritório: Araujo Costa Advogados Associados (advocacia previdenciária)
Total de contratos: ${total}
Status: Em Andamento(${andamento}), Ganho(${ganhos}), Perdido(${perdidos}), Aguardando Decisão(${aguardando})
Taxa de sucesso: ${taxaSucesso}%
Financeiro: Honorários contratados=R$${totalHon.toFixed(2)}, Recebido=R$${totalRec.toFixed(2)}, A receber=R$${aReceber.toFixed(2)}
Top causas: ${causaTop3}
Top origens: ${origTop3}
Contratos por mês recentes: ${porMes.map(x=>`${x.mes}(${x.n})`).join(', ')}
`.trim();

    const prompt = `Você é um assistente jurídico especializado em advocacia previdenciária. Analise os dados abaixo do escritório e gere exatamente 6 insights objetivos e práticos em português brasileiro.

Dados:
${contexto}

Retorne SOMENTE um JSON válido no formato:
[
  {"icone":"📊","titulo":"Título curto","texto":"Insight detalhado e prático em 1-2 frases."},
  ...
]
Seja específico com os números dos dados fornecidos. Foque em oportunidades, riscos e observações relevantes para a gestão do escritório.`;

    try {
      const resp = await fetch('https://api.anthropic.com/v1/messages', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'x-api-key': cfg.api_key,
          'anthropic-version': '2023-06-01',
          'anthropic-dangerous-direct-browser-access': 'true'
        },
        body: JSON.stringify({
          model: cfg.modelo || 'claude-sonnet-4-6',
          max_tokens: 1024,
          messages: [{role:'user', content: prompt}]
        })
      });

      if (!resp.ok) {
        const err = await resp.json().catch(()=>({error:{message:`HTTP ${resp.status}`}}));
        throw new Error(err.error?.message || `HTTP ${resp.status}`);
      }

      const data = await resp.json();
      const texto = data.content?.[0]?.text || '';
      const match = texto.match(/\[[\s\S]*\]/);
      if (!match) throw new Error('Resposta da IA não contém JSON válido.');
      const insights = JSON.parse(match[0]);

      document.getElementById('ia-status').innerHTML = `
        <div style="color:var(--muted);font-size:.8rem;margin-bottom:8px">
          ✅ ${insights.length} insights gerados com ${cfg.modelo||'claude-sonnet-4-6'}
        </div>`;
      document.getElementById('ia-insights').innerHTML = insights.map(ins=>`
        <div class="insight-card">
          <div class="insight-icon">${UTILS.escapeHtml(ins.icone||'💡')}</div>
          <div class="insight-title">${UTILS.escapeHtml(ins.titulo)}</div>
          <div class="insight-text">${UTILS.escapeHtml(ins.texto)}</div>
        </div>`).join('');

    } catch(e) {
      document.getElementById('ia-status').innerHTML = `
        <div class="card" style="border-color:rgba(239,68,68,.4);background:rgba(239,68,68,.08)">
          <span style="color:var(--red)">❌ Erro: ${UTILS.escapeHtml(e.message)}</span>
        </div>`;
    } finally {
      if (btn) btn.disabled = false;
    }
  }
};
```

- [ ] **Verificar:** Ir para Agente de IA. Sem API key, deve mostrar mensagem de erro orientando para Configurações. Com API key válida (configurar na próxima task), deve mostrar 6 cards de insight após alguns segundos.

- [ ] **Commit**
```bash
git add araujo-costa-contratos.html
git commit -m "feat: Agente de IA com Claude API e 6 cards de insights automáticos"
```

---

## Task 7: Configurações

**Arquivo:** `araujo-costa-contratos.html` — substituir `_config(){}`

- [ ] **Substituir `_config(){}` por implementação completa**

```javascript
  _config() {
    const cfg = STORAGE.getConfig();
    document.getElementById('content').innerHTML = `
      <div class="section-title" style="margin-bottom:24px">Configurações</div>

      <!-- ACESSO -->
      <div class="card mb-4">
        <div class="card-label" style="margin-bottom:16px;font-size:.8rem">🔐 Acesso</div>
        <div class="form-grid-2">
          <div class="form-row">
            <label>Usuário</label>
            <input type="text" id="cfg-usuario" value="${UTILS.escapeHtml(cfg.usuario||'')}">
          </div>
          <div class="form-row">
            <label>Nova Senha</label>
            <input type="password" id="cfg-senha" placeholder="Deixe em branco para manter">
          </div>
        </div>
        <button class="btn btn-primary" onclick="CONFIG.salvarAcesso()">Salvar Acesso</button>
      </div>

      <!-- API -->
      <div class="card mb-4">
        <div class="card-label" style="margin-bottom:16px;font-size:.8rem">🤖 Anthropic API</div>
        <div class="form-row">
          <label>API Key</label>
          <input type="password" id="cfg-apikey" value="${UTILS.escapeHtml(cfg.api_key||'')}" placeholder="sk-ant-...">
        </div>
        <div class="form-row">
          <label>Modelo Claude</label>
          <select id="cfg-modelo">
            <option value="claude-sonnet-4-6" ${(cfg.modelo||'claude-sonnet-4-6')==='claude-sonnet-4-6'?'selected':''}>claude-sonnet-4-6 (recomendado)</option>
            <option value="claude-opus-4-6" ${cfg.modelo==='claude-opus-4-6'?'selected':''}>claude-opus-4-6 (mais poderoso)</option>
            <option value="claude-haiku-4-5-20251001" ${cfg.modelo==='claude-haiku-4-5-20251001'?'selected':''}>claude-haiku-4-5 (mais rápido)</option>
          </select>
        </div>
        <button class="btn btn-primary" onclick="CONFIG.salvarApi()">Salvar API</button>
      </div>

      <!-- TEMA -->
      <div class="card mb-4">
        <div class="card-label" style="margin-bottom:16px;font-size:.8rem">🎨 Aparência</div>
        <div style="display:flex;gap:12px;flex-wrap:wrap">
          ${[['advocacia','Advocacia','#C4962A'],['dark','Dark','#4a90e2'],['light','Light','#2563eb'],['neon','Neon','#00C26F']].map(([k,n,c])=>`
            <button onclick="CONFIG.setTema('${k}')" style="
              padding:12px 20px;border-radius:var(--rs);border:2px solid ${(cfg.tema||'advocacia')===k?c:'var(--border)'};
              background:var(--bg3);color:${(cfg.tema||'advocacia')===k?c:'var(--muted)'};
              font-weight:${(cfg.tema||'advocacia')===k?'700':'500'};font-size:.85rem;cursor:pointer;
              transition:all .15s;" id="tema-btn-${k}">
              ${n}
            </button>`).join('')}
        </div>
      </div>

      <!-- DADOS DE REFERÊNCIA -->
      <div class="card mb-4">
        <div class="card-label" style="margin-bottom:16px;font-size:.8rem">📋 Origens de Captação</div>
        <div id="lista-origens"></div>
        <button class="btn btn-secondary btn-sm mt-4" onclick="CONFIG.addItem('origens')">+ Adicionar</button>
      </div>

      <div class="card mb-4">
        <div class="card-label" style="margin-bottom:16px;font-size:.8rem">⚖️ Tipos de Causa</div>
        <div id="lista-causas"></div>
        <button class="btn btn-secondary btn-sm mt-4" onclick="CONFIG.addItem('causas')">+ Adicionar</button>
      </div>

      <div class="card mb-4">
        <div class="card-label" style="margin-bottom:16px;font-size:.8rem">🔖 Status de Contrato</div>
        <div id="lista-status"></div>
        <button class="btn btn-secondary btn-sm mt-4" onclick="CONFIG.addItem('status')">+ Adicionar</button>
      </div>

      <!-- DADOS -->
      <div class="card">
        <div class="card-label" style="margin-bottom:16px;font-size:.8rem">💾 Dados</div>
        <div style="display:flex;gap:12px;flex-wrap:wrap;align-items:center">
          <button class="btn btn-secondary" onclick="CONFIG.exportar()">⬇ Exportar JSON</button>
          <label class="btn btn-secondary" style="cursor:pointer">
            ⬆ Importar JSON
            <input type="file" accept=".json" onchange="CONFIG.importar(this)" style="display:none">
          </label>
          <span style="font-size:.75rem;color:var(--muted)">Importar substitui todos os dados atuais</span>
        </div>
      </div>
    `;
    CONFIG.renderListas();
  },
```

- [ ] **Adicionar objeto CONFIG antes do `// Boot`**

```javascript
// ═══════════════════════════════════════════════
// CONFIG
// ═══════════════════════════════════════════════
const CONFIG = {
  salvarAcesso() {
    const cfg = STORAGE.getConfig();
    cfg.usuario = document.getElementById('cfg-usuario').value.trim() || cfg.usuario;
    const novaSenha = document.getElementById('cfg-senha').value;
    if (novaSenha) cfg.senha = novaSenha;
    STORAGE.setConfig(cfg);
    alert('Acesso atualizado! Use as novas credenciais no próximo login.');
  },

  salvarApi() {
    const cfg = STORAGE.getConfig();
    cfg.api_key = document.getElementById('cfg-apikey').value.trim();
    cfg.modelo  = document.getElementById('cfg-modelo').value;
    STORAGE.setConfig(cfg);
    alert('Configurações de API salvas!');
  },

  setTema(tema) {
    const cfg = STORAGE.getConfig();
    cfg.tema = tema;
    STORAGE.setConfig(cfg);
    document.documentElement.setAttribute('data-tema', tema);
    const nomes = {advocacia:'Advocacia',dark:'Dark',light:'Light',neon:'Neon'};
    const badge = document.getElementById('tema-badge');
    if (badge) badge.textContent = nomes[tema] || tema;
    RENDER.config(); // re-render para atualizar botões ativos
  },

  renderListas() {
    CONFIG._renderLista('origens', STORAGE.getOrigens(), 'lista-origens');
    CONFIG._renderLista('causas',  STORAGE.getCausas(),  'lista-causas');
    CONFIG._renderLista('status',  STORAGE.getStatusList(), 'lista-status');
  },

  _renderLista(tipo, arr, containerId) {
    const el = document.getElementById(containerId);
    if (!el) return;
    el.innerHTML = arr.map((item,i) => `
      <div class="ref-list-item">
        <input type="text" value="${UTILS.escapeHtml(item)}"
          onchange="CONFIG.editItem('${tipo}',${i},this.value)"
          onblur="CONFIG.editItem('${tipo}',${i},this.value)">
        <button class="btn btn-sm btn-danger" onclick="CONFIG.removeItem('${tipo}',${i})">✕</button>
      </div>`).join('');
  },

  _getArr(tipo) {
    if (tipo==='origens') return STORAGE.getOrigens();
    if (tipo==='causas')  return STORAGE.getCausas();
    if (tipo==='status')  return STORAGE.getStatusList();
    return [];
  },
  _setArr(tipo, arr) {
    if (tipo==='origens') STORAGE.setOrigens(arr);
    if (tipo==='causas')  STORAGE.setCausas(arr);
    if (tipo==='status')  STORAGE.setStatusList(arr);
  },

  addItem(tipo) {
    const arr = CONFIG._getArr(tipo);
    arr.push('Novo item');
    CONFIG._setArr(tipo, arr);
    CONFIG.renderListas();
  },

  editItem(tipo, idx, val) {
    const arr = CONFIG._getArr(tipo);
    arr[idx] = val.trim();
    CONFIG._setArr(tipo, arr);
  },

  removeItem(tipo, idx) {
    const arr = CONFIG._getArr(tipo);
    arr.splice(idx, 1);
    CONFIG._setArr(tipo, arr);
    CONFIG.renderListas();
  },

  exportar() {
    const contratos = STORAGE.getContratos() || [];
    const blob = new Blob([JSON.stringify(contratos, null, 2)], {type:'application/json'});
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `araujo-costa-contratos-${new Date().toISOString().slice(0,10)}.json`;
    a.click();
    URL.revokeObjectURL(url);
  },

  importar(input) {
    const file = input.files?.[0];
    if (!file) return;
    if (!confirm('Importar este JSON irá SUBSTITUIR todos os contratos atuais. Confirma?')) {
      input.value = '';
      return;
    }
    const reader = new FileReader();
    reader.onload = e => {
      try {
        const dados = JSON.parse(e.target.result);
        if (!Array.isArray(dados)) throw new Error('O arquivo deve conter um array JSON.');
        STORAGE.setContratos(dados);
        alert(`${dados.length} contratos importados com sucesso!`);
        RENDER.config();
      } catch(err) {
        alert('Erro ao importar: ' + err.message);
      }
      input.value = '';
    };
    reader.readAsText(file);
  }
};
```

- [ ] **Verificar:** Ir para Configurações. Testar troca de tema (visual deve mudar imediatamente). Adicionar/editar/remover itens nas listas. Exportar JSON (download deve iniciar). Configurar API key e salvar.

- [ ] **Commit**
```bash
git add araujo-costa-contratos.html
git commit -m "feat: Configurações com temas, listas editáveis, API key e export/import"
```

---

## Task 8: Polimentos Finais + Teste Completo

**Arquivo:** `araujo-costa-contratos.html`

- [ ] **Adicionar atalho Enter para login (já incluído em Task 2) e verificar responsividade**

  Redimensionar o browser para mobile. Verificar que:
  - Cards KPI quebram para 2 colunas em telas menores
  - Tabela tem scroll horizontal
  - Sidebar fica fixa (em mobile pode ser ocultada — opcional para v1)

- [ ] **Testar fluxo completo**

  1. Abrir o arquivo no browser (file:// ou servidor local)
  2. Login com `araujo.costa` / `advocacia2026` ✓
  3. Dashboard mostra 203 contratos e gráficos ✓
  4. Clientes: filtrar por "TRÁFEGO" no campo origem → deve filtrar ✓
  5. Adicionar novo contrato com valores financeiros ✓
  6. Editar contrato existente ✓
  7. Excluir contrato com confirmação ✓
  8. Relatórios: aplicar filtro por mês e ano → gráficos atualizam ✓
  9. Configurações: trocar tema para Light e voltar para Advocacia ✓
  10. Configurações: adicionar item em "Origens" → aparece nos dropdowns de formulário ✓
  11. Configurações: exportar JSON → download do arquivo ✓
  12. Agente de IA: com API key → gerar insights ✓
  13. Logout → tela de login ✓
  14. Fechar e reabrir browser → deve pedir login novamente (sessionStorage) ✓
  15. Abrir nova aba do mesmo arquivo → dados ainda presentes (localStorage) ✓

- [ ] **Commit final**
```bash
git add araujo-costa-contratos.html
git commit -m "feat: sistema completo araujo-costa-contratos v1.0"
```

---

## Resultado Final

Arquivo único `araujo-costa-contratos.html` (~2.000 linhas) com:
- ✅ Tela de login com credenciais configuráveis
- ✅ 4 temas visuais (Advocacia / Dark / Light / Neon)
- ✅ 203 contratos pré-carregados da planilha (2025–2026)
- ✅ Dashboard com KPIs, cards financeiros e 3 gráficos
- ✅ Clientes com tabela, busca, 5 filtros, ordenação e CRUD completo
- ✅ Relatórios com filtros dinâmicos em tempo real, rankings e comparativo anual
- ✅ Agente de IA com insights automáticos via Claude API
- ✅ Configurações com listas editáveis, troca de tema, API key e export/import JSON
