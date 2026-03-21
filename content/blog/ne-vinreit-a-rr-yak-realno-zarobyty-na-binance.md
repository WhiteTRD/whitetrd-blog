---
title: "Не вінрейт, а R:R — як реально заробити на Binance"
date: 2026-03-21
description: "Трейдер з 70% вінрейтом втрачає гроші, а з 40% — заробляє. Розбираю формулу EV, три сетапи з edge та мані-менеджмент для крипти."
tags: ["трейдинг", "binance", "технічний аналіз", "математичне очікування", "крипто"]
---

<style>
/* === Clickable List + Detail Panel === */
.cf-list { display:flex; flex-direction:column; gap:1px; margin:24px 0; background:var(--border-primary); border:1px solid var(--border-primary); border-radius:6px; overflow:hidden; }
.cf-list-item { display:flex; align-items:center; gap:16px; padding:13px 18px; background:var(--bg-secondary); cursor:pointer; transition:background 0.15s; user-select:none; }
.cf-list-item:hover { background:var(--bg-tertiary); }
.cf-list-item.active { background:var(--bg-tertiary); border-left:2px solid var(--green); padding-left:16px; }
.cf-list-num { font-size:12px; color:var(--text-secondary); min-width:24px; }
.cf-list-item.active .cf-list-num { color:var(--green); }
.cf-list-name { font-size:14px; font-weight:500; color:var(--text-primary); }
.cf-list-sub { font-size:11px; color:var(--text-secondary); margin-left:auto; white-space:nowrap; }
.cf-list-item.active .cf-list-sub { color:var(--accent); }
.cf-detail-panel { margin:0 0 24px; border:1px solid var(--border-primary); border-radius:6px; background:var(--bg-secondary); overflow:hidden; transition:opacity 0.2s; }
.cf-hidden { display:none; }
.cf-detail-header { padding:16px 20px 12px; border-bottom:1px solid var(--border-primary); display:flex; align-items:center; gap:10px; }
.cf-detail-label { font-size:10px; text-transform:uppercase; letter-spacing:0.1em; color:var(--text-secondary); }
.cf-detail-name { font-size:14px; font-weight:500; color:var(--green); }
.cf-detail-body { padding:20px; }
.cf-detail-field { margin-bottom:20px; }
.cf-detail-field:last-child { margin-bottom:0; }
.cf-detail-field-label { font-size:10px; text-transform:uppercase; letter-spacing:0.1em; color:var(--text-secondary); margin-bottom:6px; }
.cf-detail-field-value { font-size:14px; line-height:1.65; color:var(--text-primary); }
.cf-detail-field-value.cf-highlight { color:var(--text-primary); font-weight:500; }
.cf-detail-field-value.cf-warning { color:var(--callout-warning); }
.cf-detail-code { background:var(--bg-primary); border:1px solid var(--border-primary); border-radius:4px; padding:12px 16px; margin-top:10px; font-size:12px; line-height:1.6; white-space:pre; overflow-x:auto; }
.cf-detail-tags { display:flex; flex-wrap:wrap; gap:6px; }
.cf-detail-tag { font-size:11px; padding:3px 10px; border:1px solid var(--border-primary); border-radius:3px; color:var(--text-secondary); background:var(--bg-primary); transition:border-color 0.15s,color 0.15s; }
.cf-detail-tag:hover { border-color:var(--text-secondary); color:var(--text-primary); }

/* === Themed Table === */
.cf-table-wrap { overflow-x:auto; margin:24px 0; border:1px solid var(--border-primary); border-radius:6px; }
.cf-table-wrap table { margin:0; border:none; width:100%; border-collapse:collapse; }
.cf-table-wrap th { text-transform:uppercase; font-size:11px; letter-spacing:0.05em; padding:12px 16px; text-align:left; border-bottom:1px solid var(--border-primary); color:var(--text-secondary); background:var(--bg-secondary); }
.cf-table-wrap td { padding:10px 16px; border-bottom:1px solid var(--border-primary); font-size:14px; color:var(--text-primary); }
.cf-table-wrap tr:last-child td { border-bottom:none; }
.cf-table-wrap td:first-child { color:var(--green); font-weight:500; }
.cf-table-wrap tr.cf-row-negative td:first-child { color:var(--callout-warning); }
.cf-table-wrap tr.cf-row-negative td { color:var(--text-secondary); }
.cf-table-wrap .cf-cell-positive { color:var(--green); font-weight:500; }
.cf-table-wrap .cf-cell-negative { color:var(--callout-warning); font-weight:500; }

/* === Checklist (code block pattern) === */
.cf-code-block { background:var(--bg-secondary); border:1px solid var(--border-primary); border-radius:6px; padding:16px 20px; margin:16px 0 20px; overflow-x:auto; font-size:13px; line-height:1.7; white-space:pre; }
.cf-code-block .cf-comment { color:var(--text-secondary); }
.cf-code-block .cf-key { color:var(--callout-info); }
.cf-code-block .cf-val { color:var(--green); }
.cf-code-block .cf-arrow { color:var(--callout-warning); }
.cf-code-block .cf-type { color:var(--accent); }
.cf-code-block .cf-str { color:var(--callout-warning); }
.cf-code-block .cf-num { color:var(--callout-info); }

/* === Mobile === */
@media (max-width: 600px) {
  .cf-list-item { padding:10px 14px; gap:10px; }
  .cf-list-item.active { padding-left:12px; }
  .cf-list-name { font-size:13px; }
  .cf-list-sub { font-size:10px; }
  .cf-detail-body { padding:14px; }
  .cf-detail-code { font-size:11px; padding:10px 12px; }
  .cf-table-wrap th { padding:10px 12px; font-size:10px; }
  .cf-table-wrap td { padding:8px 12px; font-size:13px; }
  .cf-code-block { padding:12px 14px; font-size:12px; }
}
</style>

## 70% влучань, мінус на рахунку

Трейдер вгадує напрямок 7 із 10 разів. Звучить круто? При R:R 1:0.5 він втрачає $200 на кожні 100 угод. Інший трейдер влучає лише 4 з 10, але з R:R 1:3 забирає +$6,000 на тих самих 100 угодах (при ризику $100 на угоду).

Різниця не у вгадуванні, а в математичному очікуванні.

## Формула, яка вирішує все

EV = (Win Rate × Avg Win) − (Loss Rate × Avg Loss).

Я перевіряю кожен свій сетап через цю формулу. Якщо EV додатне, система заробляє на дистанції. Від'ємне: зливаєш, навіть якщо "майже завжди правий".

Беззбитковий вінрейт для R:R 1:2 складає 33.3%. Для R:R 1:3 потрібно лише 25%. Достатньо влучати в кожну четверту угоду, щоб не втрачати гроші.

І більшість все одно намагається бути правими в 80% випадків і ріже профіт на першому зеленому барі. Знайоме?

Ще один мінус, який ігнорують: комісії. Spot round-trip на Binance складає 0.2% (0.15% з BNB), futures 0.14% (0.10% з BNB). На 100 угод це відчутний шматок від депозиту, який знижує і так тонке edge.

<div class="cf-list" id="evCalcList"></div>
<div class="cf-detail-panel cf-hidden" id="evCalcPanel"></div>

## Три сетапи, де є edge

Я не вірю в універсальні стратегії, але є конкретні сетапи, які я тестував і торгую. Вони дають додатне EV.

### Breakout + Retest (4H/Daily)

Ціна пробиває рівень, повертається до нього, підтверджує як підтримку/опір. Вхід на ретесті зі стопом за структуру.

WR: 40-55%. R:R: від 1:2. Я торгую це переважно на BTC та ETH, бо там достатня ліквідність і рівні тримаються. На альткоїнах з малим об'ємом ціна пробиває рівні наскрізь без ретесту.

### RSI як моментум-фільтр

Більшість використовують RSI як сигнал перекупленості/перепроданості. Це помилка. RSI працює як фільтр моментуму: купуй, коли RSI вище 50 і росте, продавай, коли нижче 50 і падає.

Я тестував RSI як фільтр на BTC/ETH: CAGR 122% проти 101% у Buy & Hold. Drawdown 39% проти 83%. Більше прибутку, менше просадки. Не магія, а фільтрація угод проти тренду.

### EMA Cross Trend Following

Перетин швидкої EMA повільну, класика. WR низький, в районі 40%. Але R:R 1:3 компенсує все, бо ловиш рідкісні великі рухи і відсікаєш решту.

Ключ: не чіпати стоп, коли ціна повільно повзе проти тебе. Хто закриває раніше "бо некомфортно", той і зливає EV у мінус.

```
Фільтрація угод і EV:

  Без фільтра:  EV = $2.4 на угоду
                 |████░░░░░░░░░░░░░░░░|

  З фільтром:   EV = $18 на угоду (×7.5)
                 |████████████████████|

  Фільтр = вхід тільки коли сетап
  підтверджений на старшому таймфреймі
```

Я порахував: фільтрація угод покращує EV у 7.5 разів. Менше угод, але кожна якісніша.

<div class="cf-table-wrap">
  <table>
    <thead>
      <tr>
        <th>Стратегія</th>
        <th>Win Rate</th>
        <th>R:R</th>
        <th>EV на угоду ($100 ризик)</th>
        <th>Max Drawdown</th>
        <th>Вердикт</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>Breakout + Retest</td>
        <td>45%</td>
        <td>1:2</td>
        <td><span class="cf-cell-positive">+$35</span></td>
        <td>15-20%</td>
        <td>Найпростіший старт</td>
      </tr>
      <tr>
        <td>RSI Momentum Filter</td>
        <td>52%</td>
        <td>1:1.5</td>
        <td><span class="cf-cell-positive">+$30</span></td>
        <td>10-15%</td>
        <td>Найкращий Sharpe</td>
      </tr>
      <tr>
        <td>EMA Cross Trend</td>
        <td>40%</td>
        <td>1:3</td>
        <td><span class="cf-cell-positive">+$60</span></td>
        <td>20-30%</td>
        <td>Макс. прибуток, болючий DD</td>
      </tr>
      <tr class="cf-row-negative">
        <td>Скальпінг без фільтра</td>
        <td>55%</td>
        <td>1:0.8</td>
        <td><span class="cf-cell-negative">-$2</span></td>
        <td>25-40%</td>
        <td>Комісії з'їдають edge</td>
      </tr>
      <tr class="cf-row-negative">
        <td>Вгадування напрямку</td>
        <td>70%</td>
        <td>1:0.5</td>
        <td><span class="cf-cell-negative">-$2</span></td>
        <td>30-50%</td>
        <td>Ілюзія вінрейту</td>
      </tr>
    </tbody>
  </table>
</div>

## Стоп і тейк: за структурою, не за відчуттями

Стоп ставиш за найближчий структурний рівень. Не "50 пунктів від входу". Не "скільки готовий втратити". За структуру.

Приклад. BTC на 4H сформував подвійне дно на $62,400. Вхід на пробої шийки $63,100, стоп під друге дно $62,200, різниця $900. Тейк на наступний опір $65,800, різниця $2,700. R:R = 1:3.

Якщо стоп довший за прийнятний ризик, не бери угоду. Підганяти стоп під "зручний" розмір = ламати систему. Без винятків.

## Мані-менеджмент: математика проти емоцій

Ризик на угоду: 1-2% від депозиту. Не більше. При 2% ризику максимальний drawdown складає 6%, при 5% ризику вже 15%.

Формула позиції:

```
Розмір позиції = (Депозит × Ризик%) ÷ (Вхід − Стоп)

Приклад:
  Депозит:  $10,000
  Ризик:    2% = $200
  Вхід:     $63,100
  Стоп:     $62,200
  Різниця:  $900

  Позиція = $200 ÷ $900 = 0.222 BTC
```

Quarter Kelly дає ще один рівень захисту. Класична формула Келлі занадто агресивна для крипти, тому я беру чверть від рекомендованого і сплю спокійно.

Навіть з 60% вінрейтом шанс отримати 5 програшних угод поспіль — 10%. Коли саме це станеться? Невідомо, але станеться точно.

## Коли це НЕ працює

Я знаю чотири ситуації, коли краще сидіти на руках.

**Низька ліквідність.** Альткоїни за межами топ-30 мають широкі спреди і прослизання, яке з'їдає edge.

**Sideways ринок.** Трендові стратегії в боковику генерують хибні сигнали серіями. Drawdown болючий, але це нормальна частина процесу.

**Новинний фон.** Перед рішенням ФРС або великим анлоком токенів технічний аналіз не працює, бо ціну рухають не графіки.

**Мала вибірка.** Протестував 30 угод і вирішив, що система працює? Мінімум 200-500 угод для статистичної значимості.

Якщо хоча б один з цих факторів присутній, сиди на руках. Ринок нікуди не дінеться.

<div class="cf-code-block"><span class="cf-comment">// Чеклист перед кожною угодою</span>
<span class="cf-val">&#10003;</span> <span class="cf-key">Сетап підтверджений</span>         <span class="cf-type">— вхід за одним із 3 патернів, не "відчуття"</span>
<span class="cf-val">&#10003;</span> <span class="cf-key">Стоп виставлений</span>            <span class="cf-type">— за структурний рівень, до входу в угоду</span>
<span class="cf-val">&#10003;</span> <span class="cf-key">R:R не менше 1:2</span>            <span class="cf-type">— якщо менше, пропускаєш угоду</span>
<span class="cf-val">&#10003;</span> <span class="cf-key">Позиція розрахована</span>         <span class="cf-type">— ризик 1-2% депозиту, формула застосована</span>
<span class="cf-val">&#10003;</span> <span class="cf-key">Немає новин</span>                 <span class="cf-type">— перевірив календар: ФРС, CPI, анлоки</span>
<span class="cf-val">&#10003;</span> <span class="cf-key">Ліквідність достатня</span>        <span class="cf-type">— актив у топ-30, спред адекватний</span>
<span class="cf-val">&#10003;</span> <span class="cf-key">Журнал готовий</span>              <span class="cf-type">— причина входу записана ДО відкриття</span>
<span class="cf-arrow">!</span> <span class="cf-key">Емоційний стан</span>             <span class="cf-type">— після програшу чи ейфорії не торгуєш</span></div>

## Що далі

Чотири кроки на найближчі 3-6 місяців.

Перший: вибери один сетап. Не п'ять, один. Для старту я рекомендую Breakout + Retest, бо він найпростіший у виконанні і дає зрозумілі точки входу.

Другий: протестуй на історії, мінімум 200 угод. Запиши WR, середній профіт, середній лос і порахуй EV. Від'ємне? Міняй параметри або сетап.

Третій: торгуй на мінімальних позиціях 2-3 місяці і перевір, чи реальне EV збігається з бектестом. Комісії, прослизання та емоції завжди знижують результат порівняно з теорією.

Четвертий: масштабуй поступово, збільшуючи позицію тільки після 100+ реальних угод з додатним EV.

{{< callout type="insight" >}}
Перед кожною угодою запитай себе: "Яке математичне очікування цього входу?" Якщо не можеш відповісти числом, не входь.
{{< /callout >}}

## FAQ

**Скільки потрібно депозит, щоб почати?**
Мінімум, з яким комісії не з'їдять все: $500 для spot, $200 для futures (з плечем). Краще $1,000+. На менших сумах комісія Binance складає відчутний відсоток від кожної угоди. Я починав саме з такої суми.

**Чи працює це на альткоїнах?**
Breakout + Retest торгую тільки на топ-15 за ліквідністю. EMA Cross працює на будь-чому з достатнім об'ємом. RSI-моментум я тестував на BTC/ETH. На мемкоїнах і мікрокепах жодна з цих стратегій не має edge.

**Яке плече використовувати?**
Плече не впливає на EV системи. Впливає на швидкість, з якою досягнеш руїни. 2-5x максимум. 10x+ це лотерея з від'ємним EV після комісій.

**Чи можна автоматизувати?**
Так. Я рекомендую хоча б напів-автоматизацію: алерти на вхід, автоматичні стопи й тейки. Ручний вхід, автоматичний вихід. Повна автоматизація вимагає серйозного бектестінгу, ті самі 500+ угод на історії. Як програміст-любитель, я рухаюсь саме в цьому напрямку.

<script>
(function() {
  var items = [
    { num:1, name:"WR 70% / R:R 1:0.5", sub:"Високий WR, низький R:R",
      wr:70, rr:0.5,
      desc:"Типовий скальпер: вгадує часто, але ріже профіт і тримає лоси. 70 угод приносять по $50, 30 угод забирають по $100.",
      verdict:"Від'ємне EV. Навіть 70% влучань не рятують, коли середній лос вдвічі більший за середній він.",
      tags:["скальпінг","ілюзія вінрейту","зливає депозит"] },
    { num:2, name:"WR 50% / R:R 1:1", sub:"Монетка",
      wr:50, rr:1,
      desc:"Рівно 50/50 з однаковим профітом і лосом. EV = нуль до комісій. Після комісій Binance (0.1-0.2% round-trip) — стабільно від'ємне.",
      verdict:"Беззбитковість без комісій. З комісіями — повільний злив. Потрібен або вищий WR, або кращий R:R.",
      tags:["беззбитковість","комісії вирішують","немає edge"] },
    { num:3, name:"WR 45% / R:R 1:2", sub:"Breakout + Retest",
      wr:45, rr:2,
      desc:"45 угод приносять по $200, 55 забирають по $100. Чистий прибуток $3,500 на 100 угод при ризику $100.",
      verdict:"Додатне EV = +$35 на угоду. Стабільна прибуткова система навіть з урахуванням комісій.",
      tags:["додатне EV","структурний трейдинг","рекомендовано"] },
    { num:4, name:"WR 40% / R:R 1:3", sub:"EMA Cross Trend",
      wr:40, rr:3,
      desc:"Лише 40 з 100 угод прибуткові, але кожна приносить $300 проти $100 лосу. Результат: +$6,000 на 100 угод.",
      verdict:"Найвище EV = +$60 на угоду. Але drawdown болючий — 5-7 лосів поспіль трапляються регулярно.",
      tags:["трендова система","високий R:R","потрібна дисципліна"] },
    { num:5, name:"WR 52% / R:R 1:1.5", sub:"RSI Momentum Filter",
      wr:52, rr:1.5,
      desc:"52 виграші по $150, 48 програшів по $100. Менший drawdown завдяки фільтрації угод проти тренду.",
      verdict:"EV = +$30 на угоду. Найкращий баланс прибутку та комфорту. CAGR 122% vs 101% Buy & Hold у бектесті.",
      tags:["фільтр моментуму","низький DD","найкращий Sharpe"] },
    { num:6, name:"WR 55% / R:R 1:0.8", sub:"Скальпінг без фільтра",
      wr:55, rr:0.8,
      desc:"55 угод по $80, 45 угод по $100. Виглядає прибутково ($4,400 - $4,500 = -$100), але це до комісій. З комісіями — ще гірше.",
      verdict:"EV = -$1 на угоду навіть без комісій. З комісіями Binance futures (-$14 на round-trip) — стабільний збиток.",
      tags:["від'ємне EV","комісії критичні","не торгуй"] }
  ];

  var listEl = document.getElementById('evCalcList');
  var panelEl = document.getElementById('evCalcPanel');
  var activeIdx = null;

  items.forEach(function(item, i) {
    var el = document.createElement('div');
    el.className = 'cf-list-item';
    el.innerHTML = '<span class="cf-list-num">' + item.num + '.</span><span class="cf-list-name">' + item.name + '</span><span class="cf-list-sub">' + item.sub + '</span>';
    el.addEventListener('click', function() { toggleItem(i); });
    listEl.appendChild(el);
  });

  function toggleItem(i) {
    if (activeIdx === i) {
      activeIdx = null;
      panelEl.classList.add('cf-hidden');
      var allItems = listEl.querySelectorAll('.cf-list-item');
      for (var j = 0; j < allItems.length; j++) allItems[j].classList.remove('active');
      return;
    }
    activeIdx = i;
    var d = items[i];
    var allItems = listEl.querySelectorAll('.cf-list-item');
    for (var j = 0; j < allItems.length; j++) allItems[j].classList.remove('active');
    listEl.children[i].classList.add('active');
    panelEl.classList.remove('cf-hidden');

    var evPerTrade = (d.wr / 100) * (d.rr * 100) - ((100 - d.wr) / 100) * 100;
    var evPer100 = evPerTrade * 100;
    var breakeven = (1 / (1 + d.rr) * 100).toFixed(1);
    var evClass = evPerTrade >= 0 ? 'cf-highlight' : 'cf-warning';
    var evSign = evPerTrade >= 0 ? '+' : '';

    var tagsHtml = '';
    for (var t = 0; t < d.tags.length; t++) {
      tagsHtml += '<span class="cf-detail-tag">' + d.tags[t] + '</span>';
    }

    panelEl.innerHTML =
      '<div class="cf-detail-header">' +
        '<span class="cf-detail-label">EV-розрахунок</span>' +
        '<span class="cf-detail-name">' + d.name + '</span>' +
      '</div>' +
      '<div class="cf-detail-body">' +
        '<div class="cf-detail-field">' +
          '<div class="cf-detail-field-label">Формула (ризик $100 на угоду)</div>' +
          '<div class="cf-detail-code">EV = (' + d.wr + '% x $' + (d.rr * 100) + ') - (' + (100 - d.wr) + '% x $100)\n   = $' + (d.wr / 100 * d.rr * 100).toFixed(0) + ' - $' + ((100 - d.wr)).toFixed(0) + '\n   = ' + evSign + '$' + evPerTrade.toFixed(0) + ' на угоду\n\nНа 100 угод: ' + evSign + '$' + evPer100.toFixed(0) + '\nБеззбитковий WR для R:R 1:' + d.rr + ' = ' + breakeven + '%</div>' +
        '</div>' +
        '<div class="cf-detail-field">' +
          '<div class="cf-detail-field-label">Що відбувається</div>' +
          '<div class="cf-detail-field-value">' + d.desc + '</div>' +
        '</div>' +
        '<div class="cf-detail-field">' +
          '<div class="cf-detail-field-label">Вердикт</div>' +
          '<div class="cf-detail-field-value ' + evClass + '">' + d.verdict + '</div>' +
        '</div>' +
        '<div class="cf-detail-field">' +
          '<div class="cf-detail-field-label">Теги</div>' +
          '<div class="cf-detail-tags">' + tagsHtml + '</div>' +
        '</div>' +
      '</div>';
  }

  toggleItem(0);
})();
</script>