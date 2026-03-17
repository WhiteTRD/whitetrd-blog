---
title: "Пам'ять для Claude Code: D1 + KV + Vectorize на практиці"
date: 2026-03-17
description: "Як я побудував persistent memory на Cloudflare Worker з D1, KV та Vectorize: семантичний пошук, проектний контекст і синхронізація з GitHub Issues."
tags: ["claude-code", "cloudflare", "ai-agent", "memory", "d1", "vectorize"]
---

<style>
/* Pattern 1: Clickable List + Detail Panel */
.cf-list { display:flex; flex-direction:column; gap:1px; margin:24px 0; background:var(--border-primary); border:1px solid var(--border-primary); border-radius:6px; overflow:hidden; }
.cf-list-item { display:flex; align-items:center; gap:16px; padding:13px 18px; background:var(--bg-secondary); cursor:pointer; transition:background 0.15s; user-select:none; }
.cf-list-item:hover { background:var(--bg-tertiary); }
.cf-list-item.active { background:var(--bg-tertiary); border-left:2px solid var(--green); padding-left:16px; }
.cf-list-num { font-size:12px; color:var(--text-secondary); min-width:24px; }
.cf-list-item.active .cf-list-num { color:var(--green); }
.cf-list-name { font-size:14px; font-weight:500; color:var(--text-primary); }
.cf-list-sub { font-size:11px; color:var(--text-secondary); margin-left:auto; }
.cf-list-item.active .cf-list-sub { color:var(--accent); }
.cf-detail-panel { margin:0 0 24px; border:1px solid var(--border-primary); border-radius:6px; background:var(--bg-secondary); overflow:hidden; transition:opacity 0.2s; }
.cf-hidden { display:none; }
.cf-detail-header { padding:16px 20px 12px; border-bottom:1px solid var(--border-primary); display:flex; align-items:center; gap:10px; }
.cf-detail-label { font-size:10px; text-transform:uppercase; letter-spacing:0.1em; color:var(--text-secondary); }
.cf-detail-name { font-size:14px; font-weight:500; color:var(--green); }
.cf-detail-body { padding:20px; }
.cf-detail-field { margin-bottom:20px; }
.cf-detail-field:last-child { margin-bottom:0; }
.cf-detail-field-label { font-size:10px; text-transform:uppercase; letter-spacing:0.05em; color:var(--text-secondary); margin-bottom:6px; }
.cf-detail-field-value { font-size:14px; line-height:1.65; color:var(--text-primary); }
.cf-detail-field-value.cf-highlight { color:var(--text-primary); font-weight:500; }
.cf-detail-field-value.cf-warning { color:var(--callout-warning); }
.cf-detail-code { background:var(--bg-primary); border:1px solid var(--border-primary); border-radius:4px; padding:12px 16px; margin-top:10px; font-size:12px; line-height:1.6; white-space:pre; overflow-x:auto; }
.cf-detail-tags { display:flex; flex-wrap:wrap; gap:6px; }
.cf-detail-tag { font-size:11px; padding:3px 10px; border:1px solid var(--border-primary); border-radius:3px; color:var(--text-secondary); background:var(--bg-primary); transition:border-color 0.15s,color 0.15s; }
.cf-detail-tag:hover { border-color:var(--text-secondary); color:var(--text-primary); }

/* Pattern 2: Colored Code Block */
.cf-code-block { background:var(--bg-secondary); border:1px solid var(--border-primary); border-radius:6px; padding:16px 20px; margin:16px 0 20px; overflow-x:auto; font-size:13px; line-height:1.7; white-space:pre; }
.cf-code-block .cf-comment { color:var(--text-secondary); }
.cf-code-block .cf-key { color:var(--callout-info); }
.cf-code-block .cf-val { color:var(--green); }
.cf-code-block .cf-arrow { color:var(--callout-warning); }
.cf-code-block .cf-type { color:var(--accent); }
.cf-code-block .cf-str { color:var(--callout-warning); }
.cf-code-block .cf-num { color:var(--callout-info); }

/* Pattern 3: Themed Table */
.cf-table-wrap { overflow-x:auto; margin:24px 0; border:1px solid var(--border-primary); border-radius:6px; }
.cf-table-wrap table { margin:0; border:none; width:100%; border-collapse:collapse; }
.cf-table-wrap th { text-transform:uppercase; font-size:11px; letter-spacing:0.05em; padding:12px 16px; text-align:left; border-bottom:1px solid var(--border-primary); color:var(--text-secondary); background:var(--bg-secondary); }
.cf-table-wrap td { padding:10px 16px; border-bottom:1px solid var(--border-primary); font-size:14px; color:var(--text-primary); }
.cf-table-wrap tr:last-child td { border-bottom:none; }
.cf-table-wrap td:first-child { color:var(--green); font-weight:500; }

/* Responsive */
@media (max-width: 600px) {
  .cf-list-item { padding:10px 14px; gap:10px; }
  .cf-list-item.active { padding-left:12px; }
  .cf-list-sub { display:none; }
  .cf-list-name { font-size:13px; }
  .cf-detail-body { padding:14px; }
  .cf-detail-header { padding:12px 14px 10px; }
  .cf-detail-code { padding:10px 12px; font-size:11px; }
  .cf-code-block { padding:12px 14px; font-size:12px; line-height:1.6; }
  .cf-table-wrap th, .cf-table-wrap td { padding:8px 10px; font-size:12px; }
}
</style>

Персональна пам'ять для Claude Code: Cloudflare Worker із D1, KV та Vectorize, який замінює ручне ведення MEMORY.md і дає агентам семантичний пошук, проектний контекст і синхронізацію з GitHub Issues.

Щоранку одне й те саме. Відкриваю Claude Code, вставляю три абзаци контексту про поточний проект. Які ендпоінти є, що деплоїв минулого разу, яка структура бази. Через годину нова сесія, копіюю знову. На п'ятий раз я зрозумів: агент, який не пам'ятає мій проект, змушує мене бути його пам'яттю.

## Чому MEMORY.md не вистачає

MEMORY.md це файл на 200 рядків максимум. Плоский текст без структури. Зв'язків між проектами нуль, а замість пошуку за змістом — grep по точних словах. Для одного pet project вистачає, для трьох паралельних — ні.

Я [вже пробував GitHub Projects як пам'ять](/blog/github-projects-pamyat-agenta/) для AI-агента. Працювало краще за голий файл, але семантичного пошуку там немає. Хочеш знайти "як я налаштовував авторизацію"? Шукай точні слова. Записав "Supabase JWT setup", а питаєш "як працює auth" — результатів нуль.

Між сесіями зв'язок губиться. Агент не знає, що вчора ти фіксив баг у тому ж модулі, що сьогодні хочеш розширити. Кожна сесія як чистий аркуш.

## Модель даних: 5 сутностей

Я зупинився на п'яти сутностях. Не три, не десять. П'ять — я перевірив.

### Сутності

**Memories**: ядро системи. Текстовий запис з importance від 1 до 5, теги, категорія. TTL залежить від importance: факт з importance 1 живе тиждень, критичне рішення з importance 5 зберігається назавжди. Кожен memory прив'язаний до проекту.

**Project state**: goals, поточний контекст як JSON, активні рішення. Агент бере стейт на старті сесії і одразу розуміє, де ми зупинились.

**Tasks**: я зробив їх дзеркалом GitHub Issues. Єдине джерело правди для того, що потрібно зробити. Створюю task в пам'яті — з'являється issue. Закриваю issue — task оновлюється.

**Artifacts**: посилання на конкретні файли, конфіги, сніпети коду, URL-и. Щоб агент міг знайти "той Dockerfile, який я правив для Railway" без перебору всього репозиторію.

**Sessions і events**: лог того, що відбувалось. Яка сесія, які дії, які результати. Для розуміння контексту, не для аудиту.

<div class="cf-list" id="dataModelList"></div>
<div class="cf-detail-panel cf-hidden" id="dataModelPanel"></div>

## Інфраструктура: один Worker, zero-ops

Чому Cloudflare, а не VPS? Бо я не хочу думати про сервер. Worker на Hono запускається за мілісекунди, масштабується сам, коштує копійки на моїх обсягах.

**D1**: SQLite на edge. П'ять таблиць, прості JOIN-и, все працює. Для персонального використання більше не потрібно.

**KV**: кеш перед D1. Memories кешуються на 5 хвилин, projects на 2 хвилини. Більшість запитів агента ("дай контекст проекту") йдуть з кешу, а не з бази.

**Vectorize**: векторний індекс для семантичного пошуку. Я генерую ембедінги через Gemini API. Записую memory, створюю ембедінг, зберігаю у Vectorize. Питаю "як налаштовував auth" і отримую записи про JWT, Supabase, JWKS — навіть коли слова "auth" ніде немає.

І весь цей стек — один Worker, один `wrangler deploy`. Без Docker і без systemd.

## Skill: тригери і оркестрація

Skill активується на конкретні фрази. "Запам'ятай" означає збережи. "Згадай" означає знайди. "Контекст проекту" завантажує стейт.

{{< callout type="insight" >}}
"запам'ятай [щось]" → збережи як memory з importance 3, прив'яжи до проекту, додай теги.
"згадай [щось]" → semantic search, топ-3 релевантних memories.
{{< /callout >}}

Під капотом три ролі агентів і шість фаз оркестрації. [Архітектура шарів AI-агента](/blog/system-layers-ai-agent/) працює і тут, тільки шар один — пам'ять.

**Scout**: read-only агент на Explore (opus). Збирає контекст: які memories є, який стейт проекту, що змінилось.

**Worker**: general-purpose агент (opus), який робить реальну роботу. Створює записи, оновлює стейт, синхронізує tasks.

**Analyst**: general-purpose агент (sonnet). Перевіряє: чи не дублюється memory, чи правильний importance, чи актуальний стейт. Як у [консиліумі з трьома агентами](/blog/konsylium-nezalezhni-dumky-agentiv/) — різні ролі дають різні перспективи.

Шість фаз оркестрації:

<div class="cf-code-block"><span class="cf-comment">// Orchestration Flow: 6 phases, 3 agent roles</span>
<span class="cf-comment">// Scout = blue(info) | Worker = green | Analyst = yellow(warning)</span>

<span class="cf-key">1. Context</span>    <span class="cf-arrow">→</span> <span class="cf-num">Scout</span>     збирає memories, project state, останні events
<span class="cf-key">2. Plan</span>       <span class="cf-arrow">→</span> <span class="cf-val">Worker</span>    аналізує контекст, визначає дії
<span class="cf-key">3. Delegate</span>   <span class="cf-arrow">→</span> <span class="cf-val">Worker</span>    розподіляє підзадачі між агентами
               <span class="cf-arrow">→</span> <span class="cf-str">Analyst</span>   отримує задачу на валідацію
<span class="cf-key">4. Observe</span>    <span class="cf-arrow">→</span> <span class="cf-str">Analyst</span>   перевіряє: дублі? importance? актуальність?
<span class="cf-key">5. Adapt</span>      <span class="cf-arrow">→</span> <span class="cf-val">Worker</span>    записує в <span class="cf-type">D1</span> + оновлює <span class="cf-type">KV</span> кеш + <span class="cf-type">Vectorize</span> ембедінг
   <span class="cf-comment">Persist</span>              якщо Analyst знайшов проблему → Worker адаптує
<span class="cf-key">6. Report</span>     <span class="cf-arrow">→</span> <span class="cf-val">Worker</span>    формує відповідь користувачу

<span class="cf-comment">// Потік даних:</span>
<span class="cf-num">Scout</span> <span class="cf-arrow">──→</span> <span class="cf-val">Worker</span> <span class="cf-arrow">──→</span> <span class="cf-str">Analyst</span> <span class="cf-arrow">──→</span> <span class="cf-val">Worker</span> <span class="cf-arrow">──→</span> <span class="cf-type">D1/KV/Vectorize</span></div>

## GitHub sync: bidirectional webhook

Я налаштував webhook в обидва боки.

Створюю task через агента → Worker відправляє запит на GitHub API → з'являється issue. Хтось коментує або закриває issue → webhook приходить на Worker → task оновлюється в D1. Навіщо дублювати вручну?

Я вже використовував [GitHub Issues як контекст проекту](/blog/github-issues-kontekst-proektu/). Тепер issues не єдине джерело контексту, а частина ширшої системи. Tasks + memories + project state разом дають повну картину.

Єдиний source of truth. Я більше не синхронізую вручну.

## Semantic search vs. grep

MEMORY.md + grep: шукаю "авторизація", знаходжу тільки рядки зі словом "авторизація". Записав як "Supabase JWT"? Не знайду.

Vectorize + Gemini embeddings: шукаю "як працює auth", отримую записи про JWT setup, JWKS конфіг, Supabase tokens. Пошук за змістом, не за буквами.

Я обрав Gemini, бо у free tier достатньо для персонального використання. Хочеш інший провайдер — заміни, інтерфейс один: текст → вектор.

<div class="cf-table-wrap">
  <table>
    <thead>
      <tr>
        <th>Підхід</th>
        <th>Пошук</th>
        <th>Cross-project</th>
        <th>GitHub sync</th>
        <th>Self-hosted</th>
        <th>Setup</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>MEMORY.md</td>
        <td>grep (точний збіг)</td>
        <td>Ні</td>
        <td>Ні</td>
        <td>Файл в репо</td>
        <td>Нульова</td>
      </tr>
      <tr>
        <td>GitHub Projects</td>
        <td>API search (keyword)</td>
        <td>Через organization</td>
        <td>Нативний</td>
        <td>GitHub SaaS</td>
        <td>Низька</td>
      </tr>
      <tr>
        <td>claude-mem</td>
        <td>SQLite FTS</td>
        <td>Так</td>
        <td>Ні</td>
        <td>Локальний SQLite</td>
        <td>Низька</td>
      </tr>
      <tr>
        <td>Mem0</td>
        <td>Семантичний (vector)</td>
        <td>Так</td>
        <td>Ні</td>
        <td>Cloud / self-host</td>
        <td>Середня</td>
      </tr>
      <tr>
        <td>agent-memory (CF)</td>
        <td>Семантичний (Vectorize)</td>
        <td>Так</td>
        <td>Bidirectional webhook</td>
        <td>Cloudflare Worker</td>
        <td>Середня</td>
      </tr>
    </tbody>
  </table>
</div>

```
Запит: "як я налаштовував деплой"

MEMORY.md grep:
  "деплой" → 2 рядки (якщо слово є)

Semantic search:
  → Railway конфіг для backend
  → Dockerfile зміни від 12 березня
  → Environment variables для Supabase
  → Проблема з портом, яку фіксив тиждень тому
```

Чотири релевантних результати замість двох рядків з точним збігом.

## Коли НЕ потрібно

Ця система це overhead для простих випадків.

**Рідкісне використання Claude Code на одному проекті.** Відкриваєш раз на тиждень для одного репозиторію? MEMORY.md вистачить. Серйозно. Три хвилини на копіпаст контексту не варті окремого Worker.

**Команда більше однієї людини.** Зараз це single-user рішення. Немає namespace-ів, немає access control. Для команди потрібен інший підхід.

**Статичний контекст.** Проект не змінюється тижнями — semantic search не дасть переваги над простим файлом. Пам'ять корисна, коли контекст рухається.

## Що далі

**MCP-сервер.** Зараз skill працює через HTTP запити до Worker. MCP дасть нативну інтеграцію з Claude Code, менше glue code, простіший setup.

**Compression sessions.** Старі memories з низьким importance можна стискати: десять записів → один summary. Менше шуму при пошуку.

**Multi-user namespace.** Коли це стане корисним комусь ще — додати tenant isolation. Різні D1 databases per user, той самий Worker.

Я зрозумів: пам'ять — не фіча. Мінімальна вимога для того, щоб агент працював як помічник, а не як калькулятор із щоразу чистою стрічкою.

## FAQ

**Скільки коштує Cloudflare для цього?**
На персональних обсягах безкоштовно. Free tier D1: 5 GB storage, 5M reads/day. Free tier KV: 100K reads/day. Vectorize: 5M stored vectors. Я використовую менше 1% від лімітів.

**Чи можна без Gemini для ембедінгів?**
Так. Gemini можна замінити на OpenAI embeddings, Cohere, або Cloudflare Workers AI (модель bge-base-en). Інтерфейс один: текст → вектор. Провайдер це деталь реалізації.

**Як мігрувати з MEMORY.md?**
Парсиш файл, розбиваєш на логічні блоки, кожен блок стає memory із відповідним importance і тегами. Скрипт на 50 рядків. Або починаєш з нуля — я так і зробив.

**Що буде, якщо Worker впаде?**
Claude Code продовжить працювати, просто без пам'яті. Як раніше. Дані не втрачуються. D1 бекапиться автоматично, KV працює з eventual consistency, але для кешу це ок.

---

*Читайте також: [GitHub Projects як пам'ять для AI-агента](/blog/github-projects-pamyat-agenta/), [Контекст проєкту в GitHub Issues](/blog/github-issues-kontekst-proektu/)*

<script>
(function() {
  /* Data Model Explorer — Pattern 1: Clickable List + Detail Panel */
  var items = [
    {
      num: 1,
      name: 'Memories',
      sub: 'D1 + Vectorize',
      description: 'Ядро системи. Текстовий запис з метаданими, який індексується для семантичного пошуку. Кожен memory прив\'язаний до проекту і має TTL залежно від importance.',
      fields: 'id, project_id, content, category, importance (1-5), tags[], source, created_at, expires_at',
      ttl: 'importance 1 = 7 днів, importance 2 = 30 днів, importance 3 = 90 днів, importance 4 = 365 днів, importance 5 = безстроково',
      example: '{ content: "Railway деплой потребує PORT=8080 в env", category: "config", importance: 4, tags: ["railway", "deploy", "env"] }',
      tags: ['semantic-search', 'auto-ttl', 'embedding', 'per-project']
    },
    {
      num: 2,
      name: 'Project State',
      sub: 'D1 + KV cache',
      description: 'Знімок поточного стану проекту. Агент завантажує його на старті сесії і одразу розуміє контекст: цілі, активні рішення, стек технологій.',
      fields: 'id, project_name, repo_url, goals[], current_context (JSON), active_decisions[], tech_stack, updated_at',
      ttl: 'Без TTL. Оновлюється при кожній зміні. KV кеш: 2 хвилини.',
      example: '{ project_name: "ai_video_editor", goals: ["додати переклад субтитрів"], tech_stack: "Flask + React + Supabase", active_decisions: ["Supabase JWT для auth"] }',
      tags: ['session-start', 'kv-cached', 'json-context', 'single-per-project']
    },
    {
      num: 3,
      name: 'Tasks',
      sub: 'D1 + GitHub Issues',
      description: 'Bidirectional sync з GitHub Issues. Створюєш task через агента — з\'являється issue. Закриваєш issue — task оновлюється. Єдине джерело правди для TODO.',
      fields: 'id, project_id, github_issue_id, title, description, status (open/closed), priority, assignee, synced_at',
      ttl: 'Без TTL. Живе поки issue існує. Закриті tasks архівуються через 30 днів.',
      example: '{ title: "Додати rate limiting на /api/translate", status: "open", priority: "high", github_issue_id: 42 }',
      tags: ['github-sync', 'webhook', 'bidirectional', 'source-of-truth']
    },
    {
      num: 4,
      name: 'Artifacts',
      sub: 'D1',
      description: 'Посилання на конкретні файли, конфіги, сніпети. Щоб агент знайшов "той Dockerfile для Railway" без перебору репозиторію. Не зберігає вміст — тільки шлях і опис.',
      fields: 'id, project_id, type (file/config/snippet/url), path, description, content_hash, last_referenced, created_at',
      ttl: 'Без TTL. Видаляється вручну або коли файл більше не існує в репо.',
      example: '{ type: "file", path: "Dockerfile", description: "Multi-stage build для Railway, порт 8080" }',
      tags: ['file-reference', 'no-content-stored', 'quick-lookup', 'per-project']
    },
    {
      num: 5,
      name: 'Sessions + Events',
      sub: 'D1',
      description: 'Лог сесій і подій. Яка сесія, коли почалась, які дії виконувались, які результати. Для розуміння контексту між сесіями, не для аудиту.',
      fields: 'session: id, project_id, started_at, ended_at, summary\nevent: id, session_id, type, action, details (JSON), timestamp',
      ttl: 'Sessions: 90 днів. Events: 30 днів. Summary зберігається довше за окремі events.',
      example: '{ session: { summary: "Фіксив JWT validation + додав rate limiting" }, event: { type: "memory_created", action: "store", details: { importance: 3 } } }',
      tags: ['session-log', 'auto-cleanup', 'context-chain', 'non-audit']
    }
  ];

  var listEl = document.getElementById('dataModelList');
  var panelEl = document.getElementById('dataModelPanel');
  var activeIdx = null;

  items.forEach(function(item, i) {
    var el = document.createElement('div');
    el.className = 'cf-list-item';
    el.innerHTML = '<span class="cf-list-num">' + item.num + '.</span><span class="cf-list-name">' + item.name + '</span><span class="cf-list-sub">' + item.sub + '</span>';
    el.addEventListener('click', function() { toggleItem(i); });
    listEl.appendChild(el);
  });

  function toggleItem(i) {
    var allItems = listEl.querySelectorAll('.cf-list-item');
    if (activeIdx === i) {
      activeIdx = null;
      panelEl.classList.add('cf-hidden');
      allItems.forEach(function(el) { el.classList.remove('active'); });
      return;
    }
    activeIdx = i;
    var d = items[i];
    allItems.forEach(function(el) { el.classList.remove('active'); });
    allItems[i].classList.add('active');
    panelEl.classList.remove('cf-hidden');

    var tagsHtml = '';
    if (d.tags && d.tags.length) {
      tagsHtml = '<div class="cf-detail-field"><div class="cf-detail-field-label">Характеристики</div><div class="cf-detail-tags">';
      d.tags.forEach(function(t) { tagsHtml += '<span class="cf-detail-tag">' + t + '</span>'; });
      tagsHtml += '</div></div>';
    }

    panelEl.innerHTML =
      '<div class="cf-detail-header">' +
        '<span class="cf-detail-label">Entity</span>' +
        '<span class="cf-detail-name">' + d.name + '</span>' +
      '</div>' +
      '<div class="cf-detail-body">' +
        '<div class="cf-detail-field">' +
          '<div class="cf-detail-field-label">Опис</div>' +
          '<div class="cf-detail-field-value">' + d.description + '</div>' +
        '</div>' +
        '<div class="cf-detail-field">' +
          '<div class="cf-detail-field-label">Поля</div>' +
          '<div class="cf-detail-code">' + d.fields + '</div>' +
        '</div>' +
        '<div class="cf-detail-field">' +
          '<div class="cf-detail-field-label">TTL / Lifecycle</div>' +
          '<div class="cf-detail-field-value cf-highlight">' + d.ttl + '</div>' +
        '</div>' +
        '<div class="cf-detail-field">' +
          '<div class="cf-detail-field-label">Приклад</div>' +
          '<div class="cf-detail-code">' + d.example + '</div>' +
        '</div>' +
        tagsHtml +
      '</div>';
  }

  toggleItem(0);
})();
</script>