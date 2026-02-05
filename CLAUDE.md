# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**SendBetter AI** — кроссплатформенное desktop-приложение (macOS + Windows) для улучшения текста через LLM по глобальной горячей клавише. Работает поверх любых приложений (Telegram, Slack, браузер, почта и т.д.).

**Позиционирование:** Your personal AI copywriter everywhere.

## Core UX Flow

1. Пользователь печатает текст в любом приложении
2. Нажимает `Alt+Enter` (настраиваемая горячая клавиша)
3. Открывается popup с двумя полями:
   - **Source Text** — автоматически подхваченный текст из текущего поля ввода
   - **Instruction** — что сделать с текстом (фокус по умолчанию здесь)
4. `Enter` → LLM преобразует текст → результат заменяет исходный текст в приложении

## Architecture

### Key Modules

| Module | Responsibility |
|--------|---------------|
| `HotkeyManager` | Регистрация глобальных горячих клавиш |
| `TextCapture` | Подхват текста из активного поля ввода |
| `PopupUI` | Окно с двумя textarea + prompt picker |
| `PromptManager` | История и pinned промпты |
| `HistoryStore` | Локальное хранение истории запросов (SQLite) |
| `LLMProvider` | Абстракция над провайдерами (OpenAI, Custom HTTP) |
| `BillingManager` | Управление режимами (Trial/AppKey/BYOK) и лимитами |
| `SecureStorage` | Безопасное хранение API ключей (Keychain/DPAPI) |
| `TextInjector` | Возврат фокуса и вставка результата через clipboard |

### LLM Modes (Business Logic)

- **Mode A — BYOK:** Пользователь вводит свой ключ, без лимитов
- **Mode B — App Key Pool:** Используются ключи приложения с лимитами (X req/day)
- **Mode C — Trial:** N бесплатных запросов (например, 20), затем paywall

## Tech Stack (Recommended)

**Tauri (Rust backend + Web UI)** — легковесный, кроссплатформенный, нативные API для hotkey/clipboard.

Альтернативы: Electron (TypeScript), .NET Avalonia.

## Popup Hotkeys

| Key | Action |
|-----|--------|
| `Enter` | Выполнить преобразование |
| `Shift+Enter` | Новая строка |
| `Esc` | Закрыть без изменений |
| `Tab` | Принять placeholder (последняя инструкция) |
| `Arrow Down` | Открыть Prompt Picker |

## Text Replace Algorithm

```
1. Сохранить текущий clipboard
2. Положить result в clipboard
3. Вернуть фокус в исходное окно
4. Симулировать Ctrl/Cmd + A (Select All)
5. Симулировать Ctrl/Cmd + V (Paste)
6. Восстановить предыдущий clipboard
```

Fallback: если вставка не удалась — показать уведомление "Результат скопирован в буфер".

## Local Storage (SQLite)

**Prompt History:**
- `id`, `text`, `created_at`, `last_used_at`, `use_count`

**Request History:**
- `timestamp`, `app_name`, `source_text`, `instruction`, `result_text`, `provider`, `model`, `latency_ms`, `tokens`

## Default Pinned Prompts

1. "Исправь орфографию и пунктуацию, сохрани смысл и тон"
2. "Сделай текст вежливым, ясным и деловым"
3. "Сделай текст дружелюбным и тактичным"

## Settings Structure

- **General:** hotkey, язык, поведение Enter
- **AI/LLM:** режим (Trial/AppKey/BYOK), API key, модель, temperature, max tokens
- **Prompts:** pinned CRUD, история, лимит хранения
- **Billing (задел):** статус плана, usage, upgrade

## Performance Requirements

- Popup открывается < 150ms
- Async запросы к LLM, UI не блокируется
- Ключи хранятся в secure storage платформы

## Development Environment

- Platform: Windows (primary development), macOS (target)
- License: MIT

## Current Status

Project is in initial design phase. Specification complete, implementation not started.
