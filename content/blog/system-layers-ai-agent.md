---
title: "Слої системи: архітектура AI-агента"
date: 2026-03-13
description: "Кожна задача для AI-агента проходить через 8 шарів — від оператора до артефакту. Каркас для побудови будь-якого агентного workflow поверх LLM."
tags: ["AI", "agent", "architecture", "Claude", "multi-agent", "LLM", "orchestrator"]
categories: ["ai"]
---

<style>
/* ─── Layer list ─── */
.layer-list {
  display: flex;
  flex-direction: column;
  gap: 1px;
  margin: 24px 0;
  background: var(--border-primary);
  border: 1px solid var(--border-primary);
  border-radius: 6px;
  overflow: hidden;
}

.layer-item {
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 13px 18px;
  background: var(--bg-secondary);
  cursor: pointer;
  transition: background 0.15s;
  user-select: none;
}
.layer-item:hover { background: var(--bg-tertiary); }
.layer-item.active {
  background: var(--bg-tertiary);
  border-left: 2px solid var(--green);
  padding-left: 16px;
}

.layer-num {
  font-size: 12px;
  color: var(--text-secondary);
  min-width: 24px;
}
.layer-item.active .layer-num { color: var(--green); }

.layer-name {
  font-size: 14px;
  font-weight: 500;
  color: var(--text-primary);
}
.layer-item.active .layer-name { color: var(--text-primary); }

.layer-eng {
  font-size: 11px;
  color: var(--text-secondary);
  margin-left: auto;
}
.layer-item.active .layer-eng { color: var(--accent); }

/* ─── Detail panel ─── */
.detail-panel {
  margin: 0 0 24px;
  border: 1px solid var(--border-primary);
  border-radius: 6px;
  background: var(--bg-secondary);
  overflow: hidden;
  transition: opacity 0.2s;
}

.detail-panel.hidden {
  display: none;
}

.detail-panel-header {
  padding: 16px 20px 12px;
  border-bottom: 1px solid var(--border-primary);
  display: flex;
  align-items: center;
  gap: 10px;
}

.detail-panel-label {
  font-size: 10px;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  color: var(--text-secondary);
}

.detail-panel-name {
  font-size: 14px;
  font-weight: 500;
  color: var(--green);
}

.detail-panel-num {
  font-size: 13px;
  color: var(--text-secondary);
}

.detail-panel-eng {
  font-size: 12px;
  color: var(--accent);
  margin-left: auto;
}

.detail-body {
  padding: 20px;
}

.detail-field {
  margin-bottom: 20px;
}
.detail-field:last-child { margin-bottom: 0; }

.detail-field-label {
  font-size: 10px;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  color: var(--text-secondary);
  margin-bottom: 6px;
}

.detail-field-value {
  font-size: 14px;
  line-height: 1.65;
  color: var(--text-primary);
}
.detail-field-value.highlight { color: var(--text-primary); font-weight: 500; }
.detail-field-value.warning { color: var(--callout-warning); }

.detail-field-desc {
  font-size: 13px;
  line-height: 1.65;
  color: var(--text-primary);
  margin-top: 12px;
  padding-top: 12px;
  border-top: 1px solid var(--border-primary);
}

.detail-code {
  background: var(--bg-primary);
  border: 1px solid var(--border-primary);
  border-radius: 4px;
  padding: 12px 16px;
  margin-top: 10px;
  font-size: 12px;
  line-height: 1.6;
  white-space: pre;
  overflow-x: auto;
}

.detail-tags-list {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
}

.detail-tag {
  font-size: 11px;
  padding: 3px 10px;
  border: 1px solid var(--border-primary);
  border-radius: 3px;
  color: var(--text-secondary);
  background: var(--bg-primary);
  transition: border-color 0.15s, color 0.15s;
}
.detail-tag:hover {
  border-color: var(--text-secondary);
  color: var(--text-primary);
}

/* ─── Code block (colored) ─── */
.code-block {
  background: var(--bg-secondary);
  border: 1px solid var(--border-primary);
  border-radius: 6px;
  padding: 16px 20px;
  margin: 16px 0 20px;
  overflow-x: auto;
  font-size: 13px;
  line-height: 1.7;
  white-space: pre;
}
.code-block .comment { color: var(--text-secondary); }
.code-block .key { color: var(--callout-info); }
.code-block .val { color: var(--green); }
.code-block .arrow { color: var(--callout-warning); }
.code-block .type { color: var(--accent); }
.code-block .str { color: var(--callout-warning); }
.code-block .num { color: var(--callout-info); }

/* ─── Table wrap ─── */
.table-wrap {
  overflow-x: auto;
  margin: 24px 0;
  border: 1px solid var(--border-primary);
  border-radius: 6px;
}
.table-wrap table {
  margin: 0;
  border: none;
}

@media (max-width: 600px) {
  .layer-item { padding: 11px 14px; }
  .layer-eng { display: none; }
  .detail-body { padding: 16px; }
}
</style>

## Як система працює з ШІ

У сучасних agent-системах LLM виконує роль мозку, але навколо нього є інфраструктура:

- планування
- пам'ять
- інструменти
- виконання
- перевірка

Саме **оркестратор** координує всі ці компоненти. Схема з 8 шарів майже ідеально лягає на цю модель.

---

## 8 шарів у AI-системі

Кожен шар переформатований під LLM-архітектуру. Натисни на шар, щоб побачити деталі — **головне питання**, **артефакт**, **точку відмови**, і конкретний приклад реалізації.

<div class="layer-list" id="layerList"></div>
<div class="detail-panel hidden" id="detailPanel"></div>

---

## Як це виглядає повністю

<div class="code-block"><span class="comment">// повна архітектура agent-системи</span>

<span class="key">USER</span>
 <span class="arrow">│</span>
 <span class="arrow">▼</span>
<span class="key">Operator Layer</span>
 <span class="arrow">│</span>
 <span class="arrow">▼</span>
<span class="key">Planner</span> <span class="type">(Claude Opus)</span>
 <span class="arrow">│</span>
 <span class="arrow">▼</span>
<span class="key">Orchestrator</span>
 <span class="arrow">│</span>
 <span class="arrow">├──</span> <span class="val">Memory</span>
 <span class="arrow">│</span>
 <span class="arrow">├──</span> <span class="val">Knowledge base</span>
 <span class="arrow">│</span>
 <span class="arrow">├──</span> <span class="val">Tools</span>
 <span class="arrow">│</span>
 <span class="arrow">└──</span> <span class="val">Agents</span>
        <span class="arrow">│</span>
        <span class="arrow">├</span> <span class="type">Researcher</span>
        <span class="arrow">├</span> <span class="type">Writer</span>
        <span class="arrow">├</span> <span class="type">Critic</span>
        <span class="arrow">└</span> <span class="type">Executor</span>
             <span class="arrow">│</span>
             <span class="arrow">▼</span>
          <span class="key">Workspace</span>
             <span class="arrow">│</span>
             <span class="arrow">▼</span>
          <span class="str">Artifact</span></div>

---

## Як це виглядає з Claude Opus

Claude Opus працює як **planner + reasoning engine**. Він отримує system prompt:

<div class="code-block"><span class="key">system prompt:</span>

<span class="str">You are the strategic planner of the system.</span>
<span class="str">Your task:</span>
<span class="str">- decompose goals</span>
<span class="str">- create plans</span>
<span class="str">- assign tasks to agents</span></div>

Потім orchestrator читає план і запускає агентів:

<div class="code-block"><span class="comment">// orchestrator reads planner output</span>

{
 <span class="key">"task1"</span>: <span class="str">"research aircraft"</span>,
 <span class="key">"task2"</span>: <span class="str">"collect sources"</span>,
 <span class="key">"task3"</span>: <span class="str">"write script"</span>
}</div>

---

## Claude Opus 4.6 coverage

<div class="code-block"><span class="comment">// що Claude закриває нативно (✓) і що потребує infra (⚙)</span>

<span class="val">✓</span> <span class="key">Оператор</span>          <span class="type">— chat, API, Claude Code</span>
<span class="val">✓</span> <span class="key">Стратегічний</span>       <span class="type">— extended thinking, planner agent</span>
<span class="arrow">⚙</span> <span class="key">Довга пам'ять</span>      <span class="type">— Pinecone / Weaviate / Chroma</span>
<span class="arrow">⚙</span> <span class="key">Координація</span>       <span class="type">— LangGraph / CrewAI / AutoGen</span>
<span class="arrow">⚙</span> <span class="key">Істина проєкту</span>    <span class="type">— structured DB / state management</span>
<span class="val">✓</span> <span class="key">Робоча среда</span>      <span class="type">— sandbox, bash, web search, tools</span>
<span class="val">✓</span> <span class="key">Виконання</span>         <span class="type">— multi-agent: researcher, writer, critic</span>
<span class="val">✓</span> <span class="key">Артефакт</span>          <span class="type">— files, code, documents, API</span></div>

**5 з 8 шарів** працюють з коробки. Три шари (пам'ять, координація, істина) — інфраструктура, яку треба будувати. Саме вони відрізняють "чат з AI" від "AI-агента".

---

## Overview

<div class="table-wrap">
  <table>
    <thead>
      <tr><th>#</th><th>Слой</th><th>AI Role</th><th>Реалізація</th></tr>
    </thead>
    <tbody>
      <tr><td>1</td><td>Оператор</td><td>User / Control Layer</td><td>Chat / API / Code</td></tr>
      <tr><td>2</td><td>Стратегічний</td><td>Planner Agent</td><td>Claude Opus + thinking</td></tr>
      <tr><td>3</td><td>Довга пам'ять</td><td>Long-Term Memory</td><td>Pinecone / Weaviate / Chroma</td></tr>
      <tr><td>4</td><td>Координація</td><td>Orchestrator</td><td>LangGraph / CrewAI / AutoGen</td></tr>
      <tr><td>5</td><td>Істина проєкту</td><td>Source of Truth</td><td>Structured DB / state file</td></tr>
      <tr><td>6</td><td>Робоча среда</td><td>Workspace / Tools</td><td>Sandbox / bash / APIs</td></tr>
      <tr><td>7</td><td>Виконання</td><td>Execution Agents</td><td>Researcher / Writer / Critic</td></tr>
      <tr><td>8</td><td>Артефакт</td><td>Output</td><td>Files / commits / responses</td></tr>
    </tbody>
  </table>
</div>

---

## What's next

Ця карта — відправна точка. Кожен шар можна розгорнути в окремий документ з конкретними інструментами та імплементацією. Шари 3–5 — головний виклик: без них агент не масштабується і не вчиться.

<script>
const layers = [
  {
    num: 1, name: "Оператор", eng: "User / Control Layer",
    question: "Хто запускає систему і з яким наміром?",
    produces: "задача, контекст, обмеження, system prompt",
    breaks: "Немає фокусу — система працює вхолосту або на випадкових задачах",
    desc: "Це людина або система, яка ставить задачу. Цей шар приймає input, додає контекст, і передає його стратегічному шару.",
    example: 'Зроби сценарій для відео про Saab 21\nЗроби аналіз коду',
    tags: ["людина в чаті", "API-виклик", "Claude Code", "cron / webhook"]
  },
  {
    num: 2, name: "Стратегічний слой", eng: "Planner Agent",
    question: "Що робити, чому і в якому порядку?",
    produces: "план, декомпозиція задачі, пріоритети",
    breaks: "Рішення змішуються з імпровізацією і випадковими відповідями",
    desc: "Тут працює Claude Opus. Його задача — зрозуміти мету, створити план, розбити задачу. Це типова функція planner agent у multi-agent системах.",
    example: 'Goal: video script Saab 21\n\nPlan:\n1. research aircraft\n2. collect technical data\n3. write narrative structure\n4. produce script',
    tags: ["extended thinking", "planner agent", "goal decomposition"]
  },
  {
    num: 3, name: "Довга пам'ять", eng: "Long-Term Memory",
    question: "Що ми вже знаємо з минулих сесій?",
    produces: "контекст проєкту, уроки, патерни, preferences",
    breaks: "Кожна сесія — чистий аркуш, повторюємо помилки, втрачаємо контекст",
    desc: "Це база знань — vector database, knowledge base, document store. Дозволяє агенту згадувати попередні дані.",
    example: 'memory:\n  user prefers aviation content\n  channel style: analytical\n  last_topic: Saab J-35 Draken',
    tags: ["Pinecone", "Weaviate", "Chroma", "MCP storage"]
  },
  {
    num: 4, name: "Координація", eng: "Orchestrator",
    question: "Хто що робить і як синхронізуватись?",
    produces: "розподіл задач, статуси, залежності між агентами",
    breaks: "Один агент робить все — повільно, не масштабується, bottleneck",
    desc: "Найважливіший шар. Він запускає агентів, розподіляє задачі, контролює порядок. Оркестратор керує workflow.",
    example: 'frameworks:\n  - LangGraph\n  - AutoGen\n  - CrewAI\n  - Semantic Kernel',
    tags: ["LangGraph", "CrewAI", "AutoGen", "Semantic Kernel"]
  },
  {
    num: 5, name: "Істина проєкту", eng: "Source of Truth",
    question: "Який поточний стан і що вже вирішено?",
    produces: "стан проєкту, прийняті рішення, глосарій, обмеження",
    breaks: "Агент галюцинує свою версію реальності, конфлікти між рішеннями",
    desc: "Це фактична база даних стану проєкту. Не пам'ять LLM — це структурована база.",
    example: 'project_state:\n  script_version: v3\n  topic: Saab 21\n  sources: 12\n  length: 12 minutes',
    tags: ["structured state", "Notion / Docs", "project DB", "config files"]
  },
  {
    num: 6, name: "Робоча среда", eng: "Workspace / Tools",
    question: "Де і якими інструментами агент діє?",
    produces: "файли, виконаний код, API-виклики, середовище",
    breaks: "Агент може тільки говорити, але не діяти — zero execution capability",
    desc: "Тут відбувається робота. Це tools layer — LLM тут викликає функції.",
    example: 'search_web()\ngenerate_script()\nrun_python()\nread_file()',
    tags: ["Python sandbox", "bash / terminal", "web search", "API calls"]
  },
  {
    num: 7, name: "Виконання", eng: "Execution Agents",
    question: "Як перетворити план на конкретний результат?",
    produces: "код, текст, аналіз, рефакторинг, тести",
    breaks: "Garbage in — garbage out. Без стратегії генерація хаотична",
    desc: "Це агенти які роблять роботу. Такий підхід називається multi-agent architecture.",
    example: 'agents:\n  - researcher\n  - coder\n  - writer\n  - critic',
    tags: ["researcher", "writer", "coder", "critic", "executor"]
  },
  {
    num: 8, name: "Артефакт", eng: "Output",
    question: "Що отримує оператор як кінцевий результат?",
    produces: "файли, документи, презентації, коміти, задачі",
    breaks: "Результат залишається в чаті і не має цінності поза розмовою",
    desc: "Фінальний результат — текст, код, файл, відео сценарій. Те, що живе поза розмовою.",
    example: 'output:\n  script_saab21.txt\n  research_notes.md\n  sources.json',
    tags: [".docx / .pdf", "git commit", "Jira / Slack", "API response"]
  }
];

const listEl = document.getElementById('layerList');
const panelEl = document.getElementById('detailPanel');
let activeIndex = null;

layers.forEach((layer, i) => {
  const el = document.createElement('div');
  el.className = 'layer-item';
  el.innerHTML = '<span class="layer-num">' + layer.num + '.</span><span class="layer-name">' + layer.name + '</span><span class="layer-eng">' + layer.eng + '</span>';
  el.addEventListener('click', () => selectLayer(i));
  listEl.appendChild(el);
});

function escapeHtml(s) {
  return s.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;');
}

function selectLayer(i) {
  if (activeIndex === i) {
    activeIndex = null;
    panelEl.classList.add('hidden');
    document.querySelectorAll('.layer-item').forEach(el => el.classList.remove('active'));
    return;
  }
  activeIndex = i;
  const l = layers[i];
  document.querySelectorAll('.layer-item').forEach(el => el.classList.remove('active'));
  listEl.children[i].classList.add('active');
  panelEl.classList.remove('hidden');
  panelEl.innerHTML =
    '<div class="detail-panel-header">' +
      '<span class="detail-panel-label">слой</span>' +
      '<span class="detail-panel-num">' + l.num + '.</span>' +
      '<span class="detail-panel-name">' + l.name + '</span>' +
      '<span class="detail-panel-eng">' + l.eng + '</span>' +
    '</div>' +
    '<div class="detail-body">' +
      '<div class="detail-field">' +
        '<div class="detail-field-label">головне питання</div>' +
        '<div class="detail-field-value highlight">' + l.question + '</div>' +
      '</div>' +
      '<div class="detail-field">' +
        '<div class="detail-field-label">що виробляє</div>' +
        '<div class="detail-field-value">' + l.produces + '</div>' +
      '</div>' +
      '<div class="detail-field">' +
        '<div class="detail-field-label">що ломається без слоя</div>' +
        '<div class="detail-field-value warning">' + l.breaks + '</div>' +
      '</div>' +
      '<div class="detail-field">' +
        '<div class="detail-field-label">опис</div>' +
        '<div class="detail-field-value">' + l.desc + '</div>' +
        '<div class="detail-code">' + escapeHtml(l.example) + '</div>' +
      '</div>' +
      '<div class="detail-field">' +
        '<div class="detail-field-label">типові реалізації</div>' +
        '<div class="detail-tags-list">' +
          l.tags.map(t => '<span class="detail-tag">' + t + '</span>').join('') +
        '</div>' +
      '</div>' +
    '</div>';
}

selectLayer(1);
</script>
