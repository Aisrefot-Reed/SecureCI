```
███████╗███████╗ ██████╗██╗   ██╗██████╗ ███████╗ ██████╗██╗    
██╔════╝██╔════╝██╔════╝██║   ██║██╔══██╗██╔════╝██╔════╝██║    
███████╗█████╗  ██║     ██║   ██║██████╔╝█████╗  ██║     ██║    
╚════██║██╔══╝  ██║     ██║   ██║██╔══██╗██╔══╝  ██║     ██║    
███████║███████╗╚██████╗╚██████╔╝██║  ██║███████╗╚██████╗██║    
╚══════╝╚══════╝ ╚═════╝ ╚═════╝ ╚═╝  ╚═╝╚══════╝ ╚═════╝╚═╝
```

**Автоматический сканер безопасности для Pull Request'ов**

Мы начали строить SecureCI из личной боли: работаем в команде без Security Engineer'а, и каждый раз когда PR мерджится, никто не проверяет код на уязвимости.

SecureCI запускает 4 сканера параллельно: SAST (Semgrep), проверка зависимостей (CVE), AI-анализ (Claude), OWASP чеклист. Через 60 секунд — отчет с находками.

**Без Security Engineer'а. Без $25K/год на Snyk.**

Связанный проект: [ChakLoad-CLI](https://github.com/Aisrefot-Reed/ChakLoad-CLI) - нагрузочное тестирование.

---

## 🚀 Быстрый старт

### GitHub Actions (рекомендуется)

```yaml
# .github/workflows/secureci.yml
name: Security Scan
on: [pull_request]

permissions:
  contents: read
  pull-requests: write
  security-events: write

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: your-org/secureci@v1
        with:
          fail-on: 'high'
```

[Полная документация →](GITHUB_ACTIONS.md)

### CLI (локально)

```bash
npm install -g secureci

# Интерактивное меню
secure-ci

# Прямое сканирование
secure-ci scan ./src --fail-on high

# SARIF для GitHub Security
secure-ci scan --sarif
```

## 📋 Команды

```bash
secure-ci              # Интерактивное меню (локально)
secure-ci scan         # Сканирование (CI/CD)
secure-ci scan --json  # JSON вывод
secure-ci scan --sarif # SARIF для GitHub Security
secure-ci scan --pr    # Пост комментария в PR
secure-ci install      # Установка Semgrep/OSV
secure-ci init         # Создать .secureci.yml
```

## 📊 Exit коды

- `0` - Чисто, нет проблем
- `1` - Найдены проблемы

CI системы используют это для определения статуса.

## 📁 Структура

```
secureci/
├── src/
│   ├── index.ts              # CLI entry point
│   ├── modules/              # SAST, AI, Checklist
│   └── rules/                # Security rules (YAML)
├── tests/
│   ├── fixtures/             # Test data
│   └── e2e/                  # E2E tests
└── apps/dashboard/           # Next.js dashboard
```

## 🔒 Детектируемые уязвимости

- Захардкоженные пароли, API ключи, JWT секреты
- SQL injection паттерны
- Использование eval()
- Небезопасные HTTP эндпоинты
- Логирование чувствительных данных
- .env файлы в коммитах

## 🤝 Обратная связь

Если вы разработчик или DevOps в небольшой команде — дайте знать, хотелось бы поговорить о вашем опыте с безопасностью в CI/CD.

**#devsecops #secureci #devtools #opensource #chakbild**

## 📄 Лицензия

MIT
