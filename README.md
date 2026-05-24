🚀 Команды развертывания из GitHub
# 1. Переход в рабочую директорию
cd /home/cubinez85/converted_files/

# 2. Загрузка последней версии кода
git pull origin main   # замените main на master, если нужно

# 3. Создание .env (только при ПЕРВОМ развертывании)
if [ ! -f .env ]; then
    cat > .env << EOF
DEBUG=False
SECRET_KEY=$(openssl rand -base64 32)
ALLOWED_HOSTS=localhost,127.0.0.1,ваш_домен.ru
DB_NAME=converted_files_db
DB_USER=converted_user
DB_PASSWORD=$(openssl rand -base64 24)
EOF
    echo "✅ Файл .env создан. Проверьте и отредактируйте его при необходимости."
fi

# 4. Остановка старых контейнеров (без удаления volumes)
docker compose down

# 5. Сборка и запуск в фоновом режиме
docker compose up -d --build

# 6. Применение миграций (дублирует command в compose, но для надежности)
docker compose exec web python manage.py migrate --noinput

# 7. Сборка статики
docker compose exec web python manage.py collectstatic --noinput

# 8. Создание суперпользователя (только первый раз)
# docker compose exec web python manage.py createsuperuser

# 9. Перезапуск Nginx на хосте
sudo systemctl reload nginx

# 10. Проверка статуса
docker compose ps
curl -I http://localhost:8085/admin/

#Важно!!! 
Добавьте www-data в группу cubinez85
sudo usermod -aG cubinez85 www-data

# 1. Меняем владельца и группу
sudo chown -R cubinez85:www-data /home/cubinez85/docker_converted_files/converted_files/media/

# 2. Устанавливаем права + setgid бит (2), чтобы новые файлы от Docker наследовали группу www-data
sudo find /home/cubinez85/docker_converted_files/converted_files/media/ -type d -exec chmod 2750 {} \;

# 3. Для файлов: владелец rw, группа r, остальные ничего
sudo find /home/cubinez85/docker_converted_files/converted_files/media/ -type f -exec chmod 640 {} \;

# 4. in nginx config меняем путь к папке проекта!

# Тест конфига
sudo nginx -t

# Перезагрузка
sudo systemctl reload nginx

# Тест доступа
curl -I http://converted-files.cubinez.ru/media/conversions/result/2026/05/24/444f9a5e-1dc6-4285-b573-21a77c93025b_df_funnel.pdf
# Должно вернуть: HTTP/1.1 200 OK

1) Django-приложение для автоматической конвертации документов.
Стек: Python, Django, Django Rest Framework, PostgreSQL, Gunicorn, Nginx.
Особенности:
Конвертация между XLSX, CSV, PDF, DOCX и JSON.
Интеграция с pandas и reportlab для обработки данных.
Защищенный REST API и панель администратора для мониторинга конвертаций.
Деплой на Ubuntu через Systemd и Nginx с Basic Auth.

2) Скрипт для автоматической конвертации данных между различными форматами файлов (Excel, CSV, JSON, Word, PDF) с целью исключения
необходимости в ручной конвертации.
Задача:
Скрипт позволяет пользователю загружать файл одного из поддерживаемых форматов 
(Excel .xlsx/.xls, CSV, JSON, Word .docx,, PDF .pdf),
преобразовывает его содержимое — включая как таблицы, так и текст — в другой указанный формат и сохраняет результат
в отдельную папку проекта.
Как устроен / как пользоваться:

Пользователь указывает путь к исходному файлу.
Выбирает целевой формат для конвертации.
Скрипт автоматически анализирует содержимое:
Для Excel/CSV/JSON: работа с таблицами (DataFrame).
Для Word/PDF: извлекаются как таблицы, так и текст.
Производит конвертацию с сохранением логики структуры и содержания:
Таблицы конвертируются в соответствующие структурированные форматы (CSV, JSON, Excel, Word).
Текст конвертируется в текстовые форматы (JSON, Word, PDF).
Результат сохраняется в указанной папке проекта с понятным именем файла.
Требования:
Обработать возможные ошибки (неподдерживаемый формат, битый файл и т.д.).
Для Word и PDF:
При извлечении: если есть и текст, и таблицы, обработать оба компонента.
При конвертации:
Word -> PDF: сохранить текст и таблицы в виде PDF.
PDF -> Word/Excel:
Если есть таблицы — извлечь в табличный формат.
Если есть только текст — сохранить его в Word.
Если есть и текст, и таблицы — приоритет отдать таблицам (для Excel), либо создать Word с обеими частями.
Использовать популярные библиотеки (например, pandas, openpyxl, json, python-docx, pdfplumber, reportlab).
Обеспечить понятный интерфейс (можно через аргументы командной строки)

запуск скрипта:
python converter.py <путь_к_файлу> <целевой_формат> <папка_для_вывода>

Примеры:
python converter.py document.docx pdf ./output (с текстом и таблицами)
python converter.py data_with_text.pdf word ./output (извлекает текст и таблицы в Word)
python converter.py data_with_text.pdf json ./output (сохраняет и текст, и таблицы в JSON)
python converter.py data_only_text.docx pdf ./output (конвертирует и текст, и таблицы  в PDF)

Полный чек-лист действий после клонирования:
1. Создать и активировать виртуальное окружение
python3 -m venv venv
source venv/bin/activate

2. Установить зависимости
pip install -r requirements.txt

3. Настроить файл .env

4. Применить миграции
python manage.py migrate

5. Собрать статические файлы (для админки)
python manage.py collectstatic

6. Создать суперпользователя (для входа в админку)
python manage.py createsuperuser

#или сменить пароль для существующего пользователя
python manage.py changepassword cubinez85
