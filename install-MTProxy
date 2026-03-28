#!/bin/bash

set -e

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

log()  { echo -e "${GREEN}[+]${NC} $1"; }
warn() { echo -e "${YELLOW}[!]${NC} $1"; }
err()  { echo -e "${RED}[-]${NC} $1"; exit 1; }

# --- Проверка root ---
[ "$EUID" -ne 0 ] && err "Запусти скрипт от root"

# --- Установка Docker ---
if ! command -v docker &>/dev/null; then
    log "Docker не найден, устанавливаю..."
    apt-get update -qq
    apt-get install -y -qq ca-certificates curl gnupg
    install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    chmod a+r /etc/apt/keyrings/docker.gpg
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/debian $(. /etc/os-release && echo "$VERSION_CODENAME") stable" \
    > /etc/apt/sources.list.d/docker.list
    apt-get update -qq
    apt-get install -y -qq docker-ce docker-ce-cli containerd.io docker-compose-plugin
    systemctl enable --now docker
    log "Docker установлен"
else
    log "Docker уже установлен: $(docker --version)"
fi

# --- Проверка Docker Compose ---
if ! docker compose version &>/dev/null; then
    log "Docker Compose plugin не найден, устанавливаю..."
    apt-get install -y -qq docker-compose-plugin
    log "Docker Compose установлен"
else
    log "Docker Compose уже установлен: $(docker compose version --short)"
fi

# --- Ввод порта ---
echo ""
read -p "Введи порт для прокси [по умолчанию: 8443]: " USER_PORT
PORT=${USER_PORT:-8443}

# Проверка что порт — число в допустимом диапазоне
if ! [[ "$PORT" =~ ^[0-9]+$ ]] || [ "$PORT" -lt 1 ] || [ "$PORT" -gt 65535 ]; then
    err "Некорректный порт: $PORT"
fi

# --- Ввод домена ---
read -p "Введи домен для маскировки трафика [по умолчанию: www.google.com]: " USER_DOMAIN
DOMAIN=${USER_DOMAIN:-www.google.com}

# --- Генерация секрета ---
SECRET=$(openssl rand -hex 16)
log "Секрет сгенерирован: $SECRET"

# --- Создание директорий ---
INSTALL_DIR="/opt/telemt"
mkdir -p "$INSTALL_DIR/telemt-config"
cd "$INSTALL_DIR"
log "Рабочая директория: $INSTALL_DIR"

# --- Создание telemt.toml ---
cat > "$INSTALL_DIR/telemt-config/telemt.toml" <<EOF
[general]
use_middle_proxy = true
log_level = "normal"

[general.modes]
classic = false
secure = false
tls = true

[general.links]
show = "*"
public_port = $PORT

[server]
port = $PORT

[server.api]
enabled = true
listen = "0.0.0.0:9091"

[[server.listeners]]
ip = "0.0.0.0"

[censorship]
tls_domain = "$DOMAIN"
mask = true

[access.users]
default = "$SECRET"

[network]
ipv6 = false
EOF
log "Конфиг создан: $INSTALL_DIR/telemt-config/telemt.toml"

# --- Создание docker-compose.yml ---
cat > "$INSTALL_DIR/docker-compose.yml" <<EOF
services:
  telemt:
    image: whn0thacked/telemt-docker:latest
    container_name: telemt
    restart: unless-stopped
    user: "root"
    environment:
      RUST_LOG: "info"
    command: ["/etc/telemt/telemt.toml"]
    volumes:
      - ./telemt-config:/etc/telemt
    network_mode: host
    ulimits:
      nofile:
        soft: 65536
        hard: 65536
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
EOF
log "docker-compose.yml создан"

# --- Файрвол ---
if command -v ufw &>/dev/null && ufw status | grep -q "Status: active"; then
    if ! ufw status | grep -q "$PORT"; then
        ufw allow "$PORT/tcp" > /dev/null
        log "Порт $PORT открыт в ufw"
    else
        log "Порт $PORT уже открыт в ufw"
    fi
elif command -v iptables &>/dev/null; then
    if ! iptables -C INPUT -p tcp --dport "$PORT" -j ACCEPT 2>/dev/null; then
        iptables -I INPUT -p tcp --dport "$PORT" -j ACCEPT
        log "Порт $PORT открыт в iptables"
    else
        log "Порт $PORT уже открыт в iptables"
    fi
else
    warn "Файрвол не обнаружен, пропускаю"
fi

# --- Запуск ---
log "Запускаю контейнер..."
docker compose down 2>/dev/null || true
docker compose up -d

# --- Ожидание старта ---
log "Жду запуска (35 сек)..."
sleep 35

# --- Получение ссылки ---
LINK=$(docker compose logs --since 40s 2>/dev/null | grep "EE-TLS" | tail -1 | grep -oP 'tg://proxy\S+')

echo ""
if [ -n "$LINK" ]; then
    echo -e "${GREEN}========================================${NC}"
    echo -e "${GREEN}  Telemt MTProxy успешно запущен!${NC}"
    echo -e "${GREEN}========================================${NC}"
    echo ""
    echo -e "  Ссылка для подключения:"
    echo -e "  ${YELLOW}$LINK${NC}"
    echo ""
    echo -e "  Порт:   ${YELLOW}$PORT${NC}"
    echo -e "  Домен:  ${YELLOW}$DOMAIN${NC}"
    echo -e "  Секрет: ${YELLOW}$SECRET${NC}"
    echo -e "${GREEN}========================================${NC}"
else
    warn "Ссылка не найдена, проверь логи:"
    echo "  docker compose -f $INSTALL_DIR/docker-compose.yml logs --tail=30"
fi
