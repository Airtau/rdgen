## Для полного самостоятельного размещения генератора клиента вам потребуется следующее:

<ol>
    <li>Учетная запись на Github с форком этого репозитория</li>
    <li>Детализированный токен доступа Github с разрешениями для вашего репозитория rdgen
        <ul>
            <li>Войдите в свою учетную запись Github</li>
            <li>Нажмите на вашу фотографию профиля в правом верхнем углу, выберите "Настройки"</li>
            <li>В нижней части левой панели нажмите "Настройки разработчика"</li>
            <li>Нажмите "Личные токены доступа"</li>
            <li>Нажмите "Детализированные токены"</li>
            <li>Нажмите "Создать новый токен"</li>
            <li>Дайте имя токену, измените срок действия по желанию</li>
            <li>В разделе "Доступ к репозиториям" выберите "Только выбранные репозитории", затем выберите ваш репозиторий rdgen</li>
            <li>Предоставьте права на чтение и запись для Actions и Workflows</li>
        </ul>
    </li>
    <li>Настройте переменные окружения / секреты:
        <ul>
            <li>Переменные окружения на сервере, где запущен rdgen:
                <ul>
                <li>GHUSER="ваше_имя_пользователя_github"</li>
                <li>GHBEARER="ваш_детализированный_токен_доступа"</li>
                </ul>
            </li>
            <li>Секреты Github (настройте в вашей учетной записи Github для вашего репозитория rdgen):
                <ul>
                <li>GENURL="example.com:8000"</li>
                * это домен и порт, на котором работает ваш rdgen, должен быть доступен в интернете, в зависимости от настройки порт может не потребоваться
                </ul>
            </li>
            <li>Опциональные секреты Github (для подписи кода):
                <ul>
                <li>WINDOWS_PFX_BASE64</li>
                <li>WINDOWS_PFX_PASSWORD</li>
                <li>WINDOWS_PFX_SHA1_THUMBPRINT</li>
                </ul>
            </li>
        </ul>
    </li>
</ol>

## Для запуска rdgen на вашем сервере:

### Перейдите в каталог, где хотите установить rdgen (измените /opt на нужный путь)

> cd /opt

### Клонируйте ваш репозиторий rdgen, замените bryangerlach на ваше имя пользователя Github

> git clone https://github.com/bryangerlach/rdgen.git

### Перейдите в каталог rdgen

> cd rdgen

### Создайте виртуальное окружение Python под названием rdgen

> python -m venv rdgen

### Активируйте виртуальное окружение Python

> source rdgen/bin/activate

### Установите зависимости Python

> pip install -r requirements.txt

### Настройте базу данных

> python manage.py migrate

### Запустите сервер, замените 8000 на нужный вам порт

> python manage.py runserver 0.0.0.0:8000

### Откройте веб-браузер по адресу ваш_домен:8000

### Используйте nginx, caddy, traefik и т.д. для SSL обратного прокси

## Несколько примечаний:

<ul>
    <li>Если вы измените имя репозитория, убедитесь, что изменили URL в строках 161-168 файла views.py</li>
    <li>Если вы работаете по http вместо https, обязательно внесите изменение в строке 70 файла views.py</li>
</ul>

## Для автоматического запуска сервера при загрузке системы вы можете настроить службу systemd под названием rdgen.service

Замените пользователя, группу и порт при необходимости  
Замените /opt на каталог, где установлен rdgen  
Сохраните следующий файл как /etc/systemd/system/rdgen.service и не забудьте изменить GHUSER, GHBEARER
```
[Unit]
Description=Rustdesk Client Generator
[Service]
Type=simple
LimitNOFILE=1000000
Environment="GHUSER=ваше_имя_github"
Environment="GHBEARER=ваш_токен_github"
PassEnvironment=GHUSER GHBEARER
ExecStart=/opt/rdgen/rdgen/bin/python3 /opt/rdgen/manage.py runserver 0.0.0.0:8000
WorkingDirectory=/opt/rdgen/
User=root
Group=root
Restart=always
StandardOutput=file:/var/log/rdgen.log
StandardError=file:/var/log/rdgen.error
# Перезапуск службы через 10 секунд в случае сбоя
RestartSec=10
[Install]
WantedBy=multi-user.target
```

Затем выполните эти команды для включения автозапуска службы при загрузке и запустите ее вручную на этот раз:
```
sudo systemctl enable rdgen.service
sudo systemctl start rdgen.service
```
Для проверки статуса сервера выполните:
```
sudo systemctl status rdgen.service
```