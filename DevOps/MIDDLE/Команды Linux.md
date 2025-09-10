# Основные команды ОС Linux (ssh, scp, top, ps, kill, grep, cat, less, tail). Как писать простые bash скрипты

## 1. SSH — подключение к удалённой машине

* **Базово**

  ```bash
  ssh user@host
  ssh -p 2222 user@host             # нестандартный порт
  ssh -i ~/.ssh/id_ed25519 user@host # указать ключ
  ```
* **Переадресация портов**

  ```bash
  ssh -L 8080:localhost:80 user@host   # локально :8080 → host:80
  ssh -R 9000:localhost:3000 user@host # на host откроется :9000 → ваш :3000
  ```
* **Конфиг \~/.ssh/config**

  ```ini
  Host prod
    HostName 203.0.113.10
    User ubuntu
    IdentityFile ~/.ssh/id_ed25519
    Port 22
    ForwardAgent yes
  # подключение: ssh prod
  ```
* **Полезное**

    * `ssh -J jump user@target` — через jump host (ProxyJump).
    * `-A` агент-форвардинг (осторожно с безопасностью).
    * `-o StrictHostKeyChecking=no` — подавляет проверку ключа (не для продакшена).

## 2. SCP — копирование по SSH

```bash
scp file.txt user@host:/tmp/
scp -r dist/ user@host:/var/www/app/     # рекурсивно каталог
scp -P 2222 user@host:/var/log/app.log . # нестандартный порт
```

* Альтернатива — `rsync -avz --progress -e ssh src/ user@host:/dst/` (дозагрузка, диффы).

## 3. top — мониторинг в реальном времени

* **Запуск**: `top`
* **Горячие клавиши**

    * `M` — сортировать по памяти, `P` — по CPU, `1` — показать все CPU.
    * `k` — отправить сигнал (kill) выбранному PID.
    * `c` — полная командная строка процессов.
* **Нюансы**

    * top показывает «снимок»/поток; для удобства часто ставят `htop` (древовидно, кликабельно).

## 4. ps — снимок процессов

* **Классика**

  ```bash
  ps aux            # все процессы (BSD стиль)
  ps -ef            # System V стиль с PPID
  ps aux | grep java
  ps -o pid,ppid,pcpu,pmem,etime,cmd -p 1234
  ps auxf | less    # древовидно (f) + постранично
  ```
* **Частые задачи**

    * Найти процесс по порту:

      ```bash
      sudo lsof -i :8080 -sTCP:LISTEN -nP
      ```
    * Посмотреть потоки процесса: `ps -T -p <PID>`

## 5. kill — сигналы процессам

```bash
kill -TERM <PID>  # мягко завершить (по умолчанию)
kill -KILL <PID>  # немедленно (SIGKILL), без шанса на очистку
kill -HUP <PID>   # часто «перечитать конфиг»
killall <name>    # по имени процесса
```

* **Порядок применения**: `TERM` → (ждать) → `KILL`.
* **Массово по паттерну** (аккуратно!):

  ```bash
  pkill -f 'java.*MyApp'
  ```

## 6. grep — фильтрация текста по шаблону

* **Базово**

  ```bash
  grep "ERROR" app.log
  grep -i "error" app.log          # без учета регистра
  grep -R "TODO" src/              # рекурсивно по каталогу
  grep -n "pattern" file           # номер строки
  grep -E "WARN|ERROR" app.log     # расширенные регэкспы (|, +, ?)
  ```
* **Вывод контекста**

  ```bash
  grep -C2 "fatal" app.log   # 2 строки вокруг
  grep -A3 "start" app.log   # 3 строки после
  grep -B1 "fail" app.log    # 1 строка до
  ```
* **Подсветка и только совпадения**

  ```bash
  grep --color=always -oE "user=[^ ]+" access.log
  ```

## 7. cat — вывод файла(-ов)

```bash
cat file.txt
cat file1 file2 > merged.txt     # конкатенация
tac file.txt                      # наоборот (снизу вверх)
nl -ba file.txt                   # добавить номера строк
```

* Для больших файлов лучше `less`/`tail`.

## 8. less — постраничный просмотр

```bash
less +F app.log    # режим «почти tail -f», выйти Ctrl+C
less +100 file     # открыть с 100-й строки
```

* **Навигация**: `Space`/`b` — страницы; `/pattern` — поиск вперёд; `?pattern` — назад; `n`/`N` — след./пред.
* **Опции**: `-S` — не переносить строки; `-N` — номера строк.

## 9. tail — «хвост» файла

```bash
tail -n 100 app.log      # последние 100 строк
tail -f app.log          # следить за ростом
tail -F app.log          # как -f + handle ротации
tail -n +100 file        # с 100-й строки до конца
```

* Часто вместе с `grep`:

  ```bash
  tail -F app.log | grep -E "ERROR|WARN"
  ```

## 10. Простые Bash-скрипты

### 10.1. Структура файла

```bash
#!/usr/bin/env bash
set -Eeuo pipefail
# -E: ловить ERR в функциях, -e: падать при ошибке,
# -u: ошибка на неустановленной переменной, -o pipefail: ошибка, если ломается любой элемент пайпа

# --- Константы и параметры ---
SCRIPT_DIR="$(cd -- "$(dirname -- "${BASH_SOURCE[0]}")" && pwd)"
: "${ENV:=dev}"           # переменная по умолчанию (если не задана)

# --- Функции ---
log() { printf '[%s] %s\n' "$(date +%H:%M:%S)" "$*"; }

usage() {
  cat <<EOF
Usage: $(basename "$0") [-n NAME] [--dry-run]
Options:
  -n NAME     Greeting name (default: world)
  --dry-run   Do not perform side effects
EOF
}

# --- Парсинг аргументов ---
NAME="world"
DRY_RUN=false
while [[ $# -gt 0 ]]; do
  case "$1" in
    -n) NAME="$2"; shift 2 ;;
    --dry-run) DRY_RUN=true; shift ;;
    -h|--help) usage; exit 0 ;;
    *) echo "Unknown arg: $1"; usage; exit 1 ;;
  esac
done

# --- Основная логика ---
log "ENV=$ENV; NAME=$NAME; DRY_RUN=$DRY_RUN"
$DRY_RUN || echo "Hello, $NAME!"

```

* Сделать исполняемым: `chmod +x script.sh`
* Запуск: `./script.sh -n Alex --dry-run`

### 10.2. Переменные, подстановки, арифметика

```bash
name="world"
echo "Hello, ${name^^}!"     # верхний регистр
n=5; echo $((n * 2 + 1))      # 11
```

### 10.3. Условия и циклы

```bash
if [[ -f "$1" ]]; then
  echo "file exists"
elif [[ -d "$1" ]]; then
  echo "dir exists"
else
  echo "not found"
fi

for f in *.log; do
  [[ -e "$f" ]] || continue   # пропустить, если ничего не нашлось
  grep -q "ERROR" "$f" && echo "Has errors: $f"
done

while read -r line; do
  echo "-> $line"
done < input.txt
```

### 10.4. Функции и коды возврата

```bash
do_step() {
  local arg="$1"
  [[ -n "$arg" ]] || { echo "arg required"; return 2; }
  echo "process $arg"
}

do_step "x" || echo "step failed: $?"
```

### 10.5. Трэпы и очистка

```bash
tmpdir="$(mktemp -d)"
cleanup() { rm -rf "$tmpdir"; }
trap cleanup EXIT INT TERM

# ... используем $tmpdir ...
```

### 10.6. Безопасная работа с файлами и путями

```bash
# цитируйте переменные!
cp "$src_path" "$dst_path"

# читать по NUL-разделителю — безопасно для «странных» имён
find logs -type f -print0 | while IFS= read -r -d '' f; do
  echo "$f"
done
```

### 10.7. Пайпы и обработка ошибок

```bash
# благодаря set -o pipefail ошибка в grep не потеряется
journalctl -u myservice -n 1000 | grep -E "ERROR|WARN"
```

### 10.8. Аргументы по умолчанию и «строгий» режим

```bash
: "${PORT:?PORT is required}"   # упасть, если PORT не задан
: "${ENV:=dev}"                 # задать значение по умолчанию
```

### 10.9. Вставка here-doc / here-string

```bash
cat >config.ini <<'EOF'
[server]
port=8080
EOF

grep foo <<< "bar foo baz"   # here-string
```

### 10.10. Пример: «деплой» артефакта на удалённый хост

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

HOST="prod"
ARTIFACT="app.jar"
REMOTE_DIR="/opt/myapp"
SERVICE="myapp"

scp "$ARTIFACT" "$HOST:$REMOTE_DIR/"
ssh "$HOST" "sudo systemctl restart $SERVICE && systemctl --no-pager status $SERVICE"
```

---

## 11. Частые комбинации (cheat-sheet)

```bash
# Смотреть логи в реальном времени + фильтр по ключевому слову:
journalctl -u myapp -f -n 200 | grep -iE "error|warn"

# Кто слушает порт 5432:
sudo lsof -i :5432 -sTCP:LISTEN -nP

# Самый прожорливый по памяти процесс:
ps aux --sort=-%mem | head -n 10

# Убить все зависшие java-процессы MyApp (осторожно):
pkill -f 'java.*MyApp' || true

# Просмотр огромного файла без развала строк:
less -SN huge.log
```

---

## 12. Выжимка для собеседования

* **SSH**: ключи в `~/.ssh`, агент-форвардинг `-A` (риски), ProxyJump (`-J`), локальный/удалённый туннель (`-L`/`-R`), `~/.ssh/config` для алиасов.
* **SCP vs rsync**: scp просто копирует; rsync — дельты, перезапуски, удалённые shell-опции (`-e ssh`).
* **top/ps**: `top` — потоковый монитор, горячие клавиши `M/P/1/k/c`; `ps aux`, `ps -ef`, древовидно `auxf`, формат `-o`.
* **kill**: сигналы `TERM` (мягко) → `KILL` (жёстко); `HUP` часто = перечитать конфиг; `pkill/killall` — по имени/шаблону.
* **grep**: `-E` для расширенных регэкспов, `-R` рекурсивно, `-n` номера строк, `-C/-A/-B` контекст, `--color` и `-o` только совпадения.
* **cat/less/tail**: `less` для больших файлов (`-S -N`), `tail -F` корректно переживает ротацию, пайп с `grep` для фильтра.
* **Bash-скрипты**: всегда шебанг `#!/usr/bin/env bash` и `set -Eeuo pipefail`, цитировать переменные, `trap` для очистки, разбор аргументов `case`, туториалы here-doc, проверка переменных через `:${VAR:?msg}`.
* **Безопасность**: не отключайте host key checking в проде; ограничивайте права ключей (`chmod 600`), не храните секреты в скриптах, используйте переменные окружения/хранилища секретов.
