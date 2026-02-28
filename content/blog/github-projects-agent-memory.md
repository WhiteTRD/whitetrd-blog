---
title: "GitHub Projects як довгострокова пам'ять для AI-агентів"
date: 2026-02-28
description: "Як використовувати GitHub Issues + Projects як persistent memory для Claude Code та інших AI-агентів"
tags: ["ai", "agents", "claude-code", "github"]
---

## Проблема

AI-агенти не пам'ятають контекст між сесіями. Кожна нова розмова — чистий аркуш. Це критично для складних проєктів, де задачі тривають днями.

## Рішення

Використовуємо **GitHub Projects** як зовнішню пам'ять:

- **Issues** — одиниці пам'яті з повним контекстом задачі
- **Comments** — журнал прогресу і точки відновлення
- **Project board** — візуальний огляд стану всіх задач
- **`gh` CLI** — інтерфейс агента для читання/запису

## Як це працює

```bash
# Створити задачу з контекстом
gh issue create --title "Додати dark mode" \
  --body "## Контекст\nПотрібен dark mode через CSS custom properties..."

# Відновити контекст
gh issue view 42 --comments

# Записати прогрес
gh issue comment 42 --body "Реалізував CSS змінні, залишилось toggle"

# Закрити
gh issue close 42 --reason completed
```

## Чому це працює

Агент не зберігає контекст внутрішньо — він надійно звертається до зовнішньої документації. Чим краще написаний issue, тим точніше агент відновлює контекст.

{{< callout type="insight" >}}
Коментарі створюють recovery points — перегляд минулих рішень стає можливим і для людей, і для агентів.
{{< /callout >}}

## Налаштування

1. `gh auth login` + `gh auth refresh -s project`
2. Створити Project: `gh project create --title "AI Agent Memory"`
3. Додати інструкції в `CLAUDE.md`
4. Почати використовувати

Мінімум інфраструктури — максимум користі.
