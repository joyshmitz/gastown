# Робочий процес: future-org з Gas Town

## Інфраструктура

```
┌─────────────────────────────────────────────────────────────────┐
│                        Твоя інфраструктура                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Mac (локальний)              gt Server (Ubuntu 25.04)         │
│   ┌──────────────┐             ┌──────────────────────┐         │
│   │ Claude Code  │────SSH──────│ ~/gt/                │         │
│   │ локальна     │             │ ├── mayor/           │         │
│   │ розробка     │             │ ├── deacon/          │         │
│   └──────────────┘             │ ├── futureorg/       │         │
│          │                     │ │   ├── crew/serhii/ │         │
│          │                     │ │   ├── polecats/    │         │
│          │                     │ │   └── refinery/    │         │
│   Tailscale                    │ └── gastown/         │         │
│   100.109.77.103               └──────────────────────┘         │
│          │                              │                        │
│          └──────Dashboard───────────────┘                        │
│            http://100.109.77.103:8080                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Проєкти на сервері

| Rig | Prefix | Repo | Призначення |
|-----|--------|------|-------------|
| `futureorg` | `fu-` | joyshmitz/future-org | Основний проєкт |
| `gastown` | `gt-` | joyshmitz/gastown | gt tooling |

## Варіанти роботи

### Варіант A: Робота через SSH з Mac (рекомендовано)

Ти на Mac, запускаєш Claude Code локально, але працюєш з кодом на сервері.

```bash
# З Mac підключитись до сервера
ssh gt

# Перейти в crew workspace
cd ~/gt/futureorg/crew/serhii/rig

# Запустити Claude Code
claude

# Працювати з кодом напряму
```

**Переваги:**
- Весь контекст gt доступний
- Polecats можуть spawn-итись автоматично
- Beads синхронізуються
- Dashboard показує реальний стан

### Варіант B: Локальна розробка + gt для координації

Ти працюєш локально на Mac, а gt server використовуєш для:
- Координації через Mayor
- Spawn polecats для паралельних задач
- Tracking через convoys

```bash
# Локально на Mac
cd ~/projects/future-org
claude

# На сервері (в іншому терміналі)
ssh gt
cd ~/gt
gt mayor attach  # Координація
```

### Варіант C: Повністю на сервері через tmux

```bash
ssh gt
tmux new -s work
cd ~/gt/futureorg/crew/serhii/rig
claude
```

## Щоденний робочий процес

### Початок дня

```bash
# 1. Перевірити стан (з Mac або через SSH)
ssh gt "cd ~/gt && gt status"

# 2. Перевірити inbox
ssh gt "cd ~/gt && gt mail inbox"

# 3. Перевірити convoys
ssh gt "cd ~/gt && gt convoy list"

# 4. Відкрити dashboard в браузері
open http://100.109.77.103:8080
```

### Створення задач

```bash
# SSH на сервер
ssh gt
cd ~/gt/futureorg/mayor/rig

# Створити задачу
bd new "Implement user authentication"
# → fu-abc12

# Створити convoy якщо кілька задач
gt convoy create "Auth Feature" fu-abc12 fu-def34
```

### Робота над задачею

**Варіант 1: Сам працюєш (crew)**
```bash
ssh gt
cd ~/gt/futureorg/crew/serhii/rig
claude
# Працюєш, комітиш, пушиш
```

**Варіант 2: Делегуєш polecat**
```bash
ssh gt
cd ~/gt
gt sling fu-abc12 futureorg
# Polecat spawn-иться і працює автономно
```

**Варіант 3: Mayor координує**
```bash
ssh gt
cd ~/gt
gt mayor attach
# Розповідаєш Mayor що потрібно, він організовує
```

### Моніторинг

```bash
# Статус всього
gt status

# Convoy progress
gt convoy list
gt convoy show 1

# Polecat health
gt peek futureorg/polecats/alpha

# Real-time feed
gt feed
```

### Кінець дня

```bash
# Перевірити що все закомічено
ssh gt "cd ~/gt/futureorg/crew/serhii/rig && git status"

# Sync beads
ssh gt "cd ~/gt/futureorg/mayor/rig && bd sync"

# Перевірити convoy progress
ssh gt "cd ~/gt && gt convoy list"
```

## Beads (задачі)

### Префікси

| Prefix | Rig | Приклад |
|--------|-----|---------|
| `fu-` | futureorg | `fu-abc12` |
| `gt-` | gastown | `gt-xyz99` |
| `hq-` | town-level | `hq-cv-001` (convoy) |

### Команди

```bash
# Створити
bd new "Task title"

# Список
bd list
bd list --status=open

# Деталі
bd show fu-abc12

# Оновити
bd update fu-abc12 --status=in_progress

# Закрити
bd close fu-abc12
```

## Convoy (групи задач)

```bash
# Створити
gt convoy create "Feature Name" fu-abc12 fu-def34

# Список
gt convoy list

# Деталі
gt convoy show 1

# Статус
gt convoy status hq-cv-xxxxx
```

## Корисні alias (додай в ~/.zshrc на Mac)

```bash
# SSH shortcuts
alias gts="ssh gt"
alias gtw="ssh gt 'cd ~/gt/futureorg/crew/serhii/rig && pwd'"

# gt commands через SSH
alias gt-status="ssh gt 'cd ~/gt && gt status'"
alias gt-convoy="ssh gt 'cd ~/gt && gt convoy list'"
alias gt-dash="open http://100.109.77.103:8080"

# Quick work start
alias gt-work="ssh -t gt 'cd ~/gt/futureorg/crew/serhii/rig && exec \$SHELL -l'"
```

## Dashboard

**URL:** http://100.109.77.103:8080

Показує:
- Активні convoys
- Стан агентів (mayor, witness, refinery, polecats)
- Progress по задачах

## Troubleshooting

### gt dashboard не працює

```bash
ssh gt "cd ~/gt && gt dashboard --port 8080 &"
```

### Polecat застряг

```bash
ssh gt "cd ~/gt && gt peek futureorg/polecats/alpha"
ssh gt "cd ~/gt && gt nudge futureorg/polecats/alpha 'Continue work'"
```

### Beads не синхронізуються

```bash
ssh gt "cd ~/gt/futureorg/mayor/rig && bd sync"
```

### Перевірка здоров'я

```bash
ssh gt "cd ~/gt && gt doctor"
ssh gt "cd ~/gt && gt doctor --fix"
```

## Структура futureorg на сервері

```
~/gt/futureorg/
├── config.json           # Rig configuration
├── .repo.git/            # Shared bare repo
├── .beads/               # Beads database (prefix: fu)
├── plugins/              # Rig-level plugins
├── settings/             # Rig settings
├── mayor/rig/            # Mayor's clone (canonical beads)
├── refinery/rig/         # Merge queue worktree
├── witness/              # Polecat monitor
├── crew/
│   └── serhii/rig/       # Твій workspace
└── polecats/             # Worker worktrees (spawn on demand)
```

---

**Автор:** @joyshmitz
**Дата:** 2026-01-20
