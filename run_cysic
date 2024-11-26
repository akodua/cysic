#!/bin/bash

# Функция для обработки ошибок
function error_exit {
    echo "Ошибка: $1" >&2
    exit 1
}

# 1. Вывод "ABZALLIANCE" большими буквами
echo -e "\e[1mABZALLIANCE\e[0m"

# 2. Обновление списка пакетов и установка обновлений
echo "Обновление списка пакетов..."
sudo apt update -y || error_exit "Не удалось выполнить 'sudo apt update'"

echo "Установка обновлений..."
sudo apt upgrade -y || error_exit "Не удалось выполнить 'sudo apt upgrade'"

# 3. Установка необходимых зависимостей
echo "Установка необходимых пакетов..."
sudo apt install -y curl docker.io docker-compose git || error_exit "Не удалось установить необходимые пакеты"

# Запуск и настройка Docker
echo "Настройка Docker..."
sudo systemctl start docker || error_exit "Не удалось запустить Docker"
sudo systemctl enable docker || error_exit "Не удалось настроить автозапуск Docker"

# Добавление текущего пользователя в группу docker для выполнения команд без sudo
sudo usermod -aG docker "$USER" || error_exit "Не удалось добавить пользователя в группу docker"

# Применение изменений группы без перезагрузки (требует повторного входа в систему)
newgrp docker <<EONG
echo "Пользователь добавлен в группу docker."
EONG

# 4. Загрузка и запуск setup_linux.sh
echo "Загрузка скрипта setup_linux.sh..."
curl -L https://github.com/cysic-labs/phase2_libs/releases/download/v1.0.0/setup_linux.sh -o ~/setup_linux.sh || error_exit "Не удалось загрузить setup_linux.sh"

chmod +x ~/setup_linux.sh || error_exit "Не удалось сделать setup_linux.sh исполняемым"

# Запрос адреса EVM кошелька у пользователя
read -p "Введите адрес вашего EVM кошелька: " EVM_ADDRESS

# Проверка ввода адреса
if [[ -z "$EVM_ADDRESS" ]]; then
    error_exit "Адрес EVM кошелька не может быть пустым."
fi

# Запуск setup_linux.sh с передачей адреса кошелька
echo "Запуск setup_linux.sh..."
echo "$EVM_ADDRESS" | bash ~/setup_linux.sh || error_exit "Не удалось выполнить setup_linux.sh"

# 5. Клонирование репозитория cysic-verifier (если еще не клонирован)
if [ ! -d "$HOME/cysic-verifier" ]; then
    echo "Клонирование репозитория cysic-verifier..."
    git clone https://github.com/cysic-labs/cysic-verifier.git ~/cysic-verifier || error_exit "Не удалось клонировать репозиторий cysic-verifier"
fi

# Переход в каталог cysic-verifier
cd ~/cysic-verifier/ || error_exit "Каталог ~/cysic-verifier/ не найден"

# Проверка наличия Dockerfile или docker-compose.yml и запуск соответствующим образом
if [ -f Dockerfile ]; then
    echo "Найден Dockerfile. Сборка Docker-образа..."
    docker build -t cysic-verifier . || error_exit "Не удалось собрать Docker-образ"
    echo "Запуск Docker-контейнера в фоне..."
    docker run -d --name cysic-verifier-container cysic-verifier || error_exit "Не удалось запустить Docker-контейнер"
elif [ -f docker-compose.yml ]; then
    echo "Найден docker-compose.yml. Запуск сервисов с помощью Docker Compose..."
    docker-compose up -d || error_exit "Не удалось запустить сервисы с помощью Docker Compose"
else
    echo "Dockerfile и docker-compose.yml не найдены. Запуск start.sh напрямую в фоне..."
    bash start.sh & || error_exit "Не удалось запустить start.sh в фоне"
fi

echo "Установка и запуск завершены успешно."
echo "Пожалуйста, перезайдите в систему, чтобы изменения групп пользователей вступили в силу."

# Предложение перезагрузки системы
echo "Если вы не видите ошибок, скрипт завершен. Рекомендуется перезагрузить систему для применения всех изменений."
echo "Вы хотите перезагрузить систему сейчас? (y/n)"
read -p "Введите y для перезагрузки или любую другую клавишу для выхода: " REBOOT_CHOICE

if [[ "$REBOOT_CHOICE" == "y" || "$REBOOT_CHOICE" == "Y" ]]; then
    sudo reboot
else
    echo "Перезагрузка пропущена. Не забудьте перезагрузить систему вручную, чтобы изменения вступили в силу."
fi
