# Design: Sistema de Gestão de Contratos — Araujo Costa Advogados Associados

**Data:** 2026-06-29
**Status:** Aprovado
**Contexto:** Sistema para gerenciar contratos fechados da área de advocacia previdenciária do escritório Araujo Costa Advogados Associados, baseado nos dados da planilha "CONTRATOS FECHADOS MENSAIS" (2025–2026). Entrega como arquivo HTML único com localStorage, inspirado no dashboard financeiro de Pablo Marques.

---

## 1. Contexto e Objetivos

O escritório Araujo Costa Advogados Associados atua principalmente em advocacia previdenciária. Atualmente, os contratos fechados são registrados em uma planilha Excel organizada por mês, com 4 colunas: nome do cliente, data do contrato, tipo de causa e origem de captação. Não há controle financeiro, status de andamento, nem visualizações analíticas.

**Objetivos:**
- Migrar e expandir os dados da planilha para um sistema navegável
- Adicionar controle financeiro (honorários, valor esperado, valor recebido)
- Adicionar status de andamento por contrato
- Oferecer dashboard com KPIs e visualizações
- Insights automáticos via Claude API (Anthropic)
- Arquivo único HTML — sem dependência de servidor ou banco de dados externo

---

## 2. Entrega Técnica

**Arquivo:** `araujo-costa-contratos.html`
**Localização sugerida:** `C:\projetos\araujo-costa-contratos\`
**Tecnologia:** HTML5 + CSS3 + JavaScript puro (vanilla JS)
**Persistência:** `localStorage` (chave principal: `ac_contratos`)
**IA:** Claude API via `fetch` direto do browser (chave configurada em Configurações)
**Autenticação:** `sessionStorage` — expira ao fechar o browser
**Fontes:** Playfair Display + Montserrat (Google Fonts)

---

## 3. Modelo de Dados

### 3.1 Contrato
```json
{
  "id": "uuid-v4",
  "cliente": "Nome Completo do Cliente",
  "data": "DD/MM/AAAA",
  "mes": "Janeiro",
  "ano": 2026,
  "causa": "Planejamento Previdenciário",
  "origem": "Tráfego",
  "status": "Em Andamento",
  "honorarios": 0.00,
  "valor_esperado": 0.00,
  "valor_recebido": 0.00,
  "observacao": ""
}
```

### 3.2 Status possíveis
- Em Andamento
- Ganho
- Perdido
- Aguardando Decisão

### 3.3 Dados de Referência (editáveis em Configurações)
Três listas editáveis salvas em `localStorage`:
- `ac_origens` — origens de captação (padrão: Tráfego, Indicação, CAMIS, BIA, Escritório)
- `ac_causas` — tipos de causa (padrão: Planejamento Previdenciário, Auxílio Doença, Salário Maternidade, Seguro Defeso, BPC, Pensão por Morte, Trabalhista, Execução de Alimentos, Ação de Alimentos, Auxílio Acidente, APSI, Consumidor, Outros)
- `ac_status_lista` — status de contrato (padrão: Em Andamento, Ganho, Perdido, Aguardando Decisão)

### 3.4 Configurações do sistema
Salvas em `localStorage` (chave `ac_config`):
```json
{
  "anthropic_key": "",
  "usuario": "araujo.costa",
  "senha": "advocacia2026",
  "tema": "advocacia"
}
```

### 3.5 Dados pré-carregados
Todos os contratos das abas 2025 e 2026 da planilha serão hardcoded como array JS inicial. O sistema carrega esse array para o `localStorage` apenas se a chave `ac_contratos` ainda não existir (primeira abertura).

---

## 4. Autenticação

- Tela de login exibida antes de qualquer seção
- Sessão mantida em `sessionStorage` — ao fechar o browser, pede login novamente
- Credenciais padrão: usuário `araujo.costa` / senha `advocacia2026`
- Credenciais alteráveis em Configurações
- Botão "Sair" no header (limpa `sessionStorage` e retorna ao login)

---

## 5. Temas Visuais

Quatro temas, selecionáveis em Configurações e persistidos em `localStorage`:

| Tema | Base | Destaque | Observação |
|------|------|----------|-----------|
| **Advocacia** | Verde-escuro profundo `#080E09`/`#0D1210` | Dourado `#C4962A` | Padrão — identidade do site araujocostaadv.pablomarques.com.br |
| **Dark** | Azul-escuro `#0f1117`/`#1a1d27` | Azul `#4a90e2` | Tema escuro genérico |
| **Light** | Cinza-claro `#f0f2f7`/`#ffffff` | Azul `#2563eb` | Tema claro |
| **Neon** | Verde-preto `#09110E` | Verde neon `#00C26F` | Alto contraste |

Fontes do tema Advocacia: Playfair Display (títulos) + Montserrat (corpo).

---

## 6. Módulos

### 6.1 Login
- Logo/nome do escritório centralizado
- Campos: usuário, senha
- Botão "Entrar"
- Mensagem de erro se credenciais incorretas
- Sem cadastro público — apenas alteração em Configurações

### 6.2 Dashboard
Primeira tela após login. Composto por:

**Cards KPI (topo):**
- Total de Contratos
- Contratos Ganhos
- Em Andamento
- Taxa de Sucesso (Ganhos ÷ (Ganhos + Perdidos) × 100)

**Cards Financeiros:**
- Total de Honorários
- Total Recebido
- A Receber (valor_esperado − valor_recebido, somado)

**Gráficos:**
- Barras: contratos fechados por mês (eixo X = meses, eixo Y = quantidade)
- Rosca/donut: distribuição por causa (top 5 + "Outros")
- Barras horizontais: distribuição por origem de captação

**Insights da IA:**
- 4–6 cards gerados pela Claude API
- Exibidos com ícone, título curto e texto explicativo
- Botão "Atualizar Insights" para regenerar

### 6.3 Clientes
**Tabela com:**
- Colunas: Nome · Data · Causa · Origem · Status (badge colorido) · Honorários · Recebido · Ações
- Busca por nome (tempo real)
- Filtros: Mês · Ano · Causa · Origem · Status
- Ordenação por coluna (clique no cabeçalho)
- Novos itens aparecem no topo

**CRUD:**
- Botão "Novo Contrato" abre modal com formulário completo
- Botão editar (ícone lápis) reabre modal preenchido
- Botão excluir (ícone lixeira) com confirmação
- Campos do formulário: Cliente, Data, Causa (dropdown), Origem (dropdown), Status (dropdown), Honorários, Valor Esperado, Valor Recebido, Observação

**Badges de status:**
- Em Andamento → azul
- Ganho → verde
- Perdido → vermelho
- Aguardando Decisão → amarelo/âmbar

### 6.4 Relatórios
**Filtros no topo (afetam toda a visualização em tempo real):**
- Mês (Janeiro–Dezembro · Todos)
- Ano (2025 · 2026 · Todos)
- Causa
- Origem
- Status

**Visualizações:**
- Contratos por mês — gráfico de barras
- Contratos por ano — comparativo 2025 vs 2026
- Receita mensal: honorários vs recebido (barras agrupadas)
- Ranking de causas: quantidade + valor total (tabela ordenável)
- Ranking de origens de captação: quantidade + percentual
- Tabela resumo dos contratos filtrados

### 6.5 Agente de IA
- Botão "Gerar Insights"
- Ao clicar: monta contexto JSON compacto com totais, distribuições e pendências financeiras dos contratos
- Envia para Claude API (`claude-sonnet-4-6` ou configurável) com prompt em português
- Exibe spinner durante geração
- Renderiza 4–6 cards de insight com título, texto e ícone
- Exemplos de insights esperados:
  - "68% dos contratos de março/2026 ainda estão em andamento"
  - "Tráfego gerou 42% dos contratos captados em 2026"
  - "R$ 12.400 em honorários esperados ainda não foram recebidos"
  - "Planejamento Previdenciário é a causa mais frequente (31% dos contratos)"
- Campo para exibir erros de API (chave inválida, limite atingido)

### 6.6 Configurações
Organizada em abas ou seções:

**Acesso:**
- Alterar usuário e senha

**API:**
- Campo para inserir/atualizar a Anthropic API Key
- Salva em `localStorage` — usado pelo Agente de IA
- Modelo Claude (padrão: `claude-sonnet-4-6`)

**Aparência:**
- Seletor de tema: Advocacia · Dark · Light · Neon

**Dados de Referência** (listas editáveis):
- **Origens de captação:** lista com campos para adicionar, editar nome inline e excluir
- **Tipos de causa:** idem
- **Status de contrato:** idem
- Alterações refletem imediatamente nos dropdowns de formulário

**Dados:**
- Exportar todos os contratos como JSON (download)
- Importar contratos de JSON (substituição total, com diálogo de confirmação antes de sobrescrever)

---

## 7. Navegação

Sidebar fixa à esquerda com ícones + labels:
```
📊  Dashboard
👥  Clientes
📈  Relatórios
🤖  Agente de IA
⚙️  Configurações
```
Header: logo/nome do escritório + tema atual + botão "Sair".

---

## 8. Dados Pré-carregados

A planilha contém contratos de **2025** (maio–dezembro) e **2026** (janeiro–março+). Todos serão inseridos como array JS no arquivo HTML. Campos financeiros (honorários, valor_esperado, valor_recebido) serão inicializados com `0` para os registros da planilha, para preenchimento posterior pelo usuário.

Total estimado de registros pré-carregados: ~80–100 contratos.

---

## 9. Segurança e Limitações

- API key armazenada em `localStorage` — aceitável para uso local/pessoal, não deve ser usado em servidor público
- Senha armazenada como string (sem hash criptográfico real) — adequado para uso interno
- Dados ficam no browser da máquina — usar exportação JSON para backup regular
- Sem sincronização entre dispositivos (limitação do localStorage)

---

## 10. Estrutura do Arquivo

```
araujo-costa-contratos.html
├── <head>         — meta, fontes Google, sem dependências externas de CSS
├── <style>        — todos os estilos inline (4 temas em variáveis CSS)
├── <body>
│   ├── #login-screen     — tela de login
│   ├── #app              — container principal (oculto até login)
│   │   ├── #sidebar
│   │   ├── #header
│   │   └── #content      — renderizado dinamicamente por JS
└── <script>       — todo o JS inline
    ├── DADOS_INICIAIS[]  — array com contratos pré-carregados da planilha
    ├── storage.*         — funções localStorage/sessionStorage
    ├── auth.*            — login/logout
    ├── nav.*             — navegação entre seções
    ├── dashboard.*       — KPIs, gráficos (Canvas API nativo)
    ├── clientes.*        — tabela, CRUD, modal, filtros
    ├── relatorios.*      — gráficos com filtros dinâmicos
    ├── ia.*              — chamada Claude API, render de cards
    └── config.*          — configurações, listas editáveis
```
