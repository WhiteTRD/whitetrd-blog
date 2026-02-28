---
title: "Мінімалістичний блог на Hugo за 30 хвилин"
date: 2026-02-27
description: "Monospace, zero JS, dark mode — створюємо блог в стилі хакерської естетики"
tags: ["hugo", "web", "design"]
---

## Чому Hugo

- Збірка за мілісекунди (Go-based)
- Zero JavaScript у фінальному HTML
- Markdown → HTML без складних пайплайнів
- Безкоштовний хостинг на Cloudflare Pages / Vercel / Netlify

## Дизайн-принципи

Весь стиль — 60 рядків CSS:

```css
body {
  font-family: monospace;
  max-width: 600px;
  margin: 0 auto;
}

a { color: #0000EE; }
a:visited { color: #551A8B; }
```

Monospace шрифт + 600px + сині лінки = ретро-хакерська естетика.

## Dark mode

Одна CSS media query:

```css
@media (prefers-color-scheme: dark) {
  :root {
    --bg: #212121;
    --fg: #d1d5db;
  }
}
```

Браузер автоматично переключає тему.

## Деплой

```bash
# Build
hugo

# Deploy to Cloudflare Pages
git push  # auto-deploy on push
```

Результат: статичний сайт без JavaScript, з dark mode, RSS, SEO — і все це безкоштовно.
