# Инструкции по базовому тестированию (Smoke Tests)

После успешного выполнения Ansible-плейбука и развертывания всех сервисов, рекомендуется провести следующие базовые проверки ("smoke tests"), чтобы убедиться в их работоспособности.

## 1. Проверка доступности веб-интерфейсов

Используйте URL-адреса, которые были выведены Ansible или которые вы настроили (в зависимости от выбора домена/IP).

*   **Caddy:** Попробуйте открыть основной домен (если настроен) или IP-адрес сервера. Вы должны увидеть стандартную страницу Caddy или один из сервисов, если он настроен на корневой путь. Проверьте, что HTTPS работает (если настроен домен).
*   **n8n:**
    *   Откройте URL n8n (например, `http://<vps_ip>:{{ caddy_n8n_port }}` или `https://n8n.yourdomain.com`).
    *   Попробуйте создать нового пользователя (если это первый запуск).
    *   Войдите в систему.
    *   Попробуйте создать простой workflow.
*   **OpenWebUI:**
    *   Откройте URL OpenWebUI.
    *   Зарегистрируйте нового пользователя.
    *   Войдите в систему.
    *   Проверьте, видит ли OpenWebUI модели Ollama (в настройках или при выборе модели).
    *   Попробуйте отправить запрос к одной из загруженных моделей Ollama.
*   **Flowise:**
    *   Откройте URL Flowise.
    *   Попробуйте создать простой чат-флоу, использующий Ollama и, возможно, Qdrant.
    *   Протестируйте чат-флоу.
*   **SearXNG:**
    *   Откройте URL SearXNG.
    *   Выполните несколько поисковых запросов.
    *   Проверьте страницу настроек (если доступна).
*   **Supabase Studio:**
    *   Откройте URL Supabase (например, `http://<vps_ip>:{{ caddy_supabase_port }}` или `https://supabase.yourdomain.com`).
    *   Войдите в Supabase Studio, используя учетные данные, которые вы задали (или которые были сгенерированы и должны быть указаны в `.env` или выводе Ansible).
    *   Проверьте доступ к таблицам, SQL Editor, аутентификации.
*   **Langfuse Web UI:**
    *   Откройте URL Langfuse.
    *   Попробуйте войти (если настроена аутентификация) или просто убедитесь, что UI загружается.
    *   Проверьте, видны ли проекты (если были созданы).

## 2. Проверка работы Ollama

*   Подключитесь к VPS по SSH.
*   Выполните команду, чтобы проверить запущенные модели Ollama (если вы знаете имя контейнера Ollama, например `ollama`):
    ```bash
    docker exec ollama ollama list
    ```
*   Попробуйте выполнить запрос к Ollama через `curl` с VPS (или с вашей локальной машины, если порт 11434 проброшен и доступен):
    ```bash
    curl http://localhost:11434/api/generate -d '{
      "model": "имя_загруженной_модели", # например, qwen2.5:7b-instruct-q4_K_M
      "prompt":"Why is the sky blue?"
    }'
    ```

## 3. Проверка взаимодействия сервисов (примеры)

*   **OpenWebUI/Flowise -> Ollama:** Уже проверяется при тестировании UI.
*   **Flowise -> Qdrant:** Если вы создали чат-флоу с использованием векторного хранилища Qdrant, проверьте, что он работает.
*   **n8n -> Supabase/Qdrant/Ollama:** Создайте тестовый workflow в n8n, который взаимодействует с этими сервисами.

## 4. Проверка логов контейнеров

Если какой-то сервис не работает или работает некорректно, проверьте его логи:
```bash
docker logs <имя_контейнера>
# Например:
# docker logs n8n
# docker logs supabase_db
# docker logs langfuse_worker
```
Список имен контейнеров можно посмотреть командой `docker ps`.

## 5. Проверка статуса Docker Compose стека
На VPS выполните:
```bash
cd /opt/ai_stack
docker-compose ps
# или docker compose ps (для V2)
```
Все сервисы должны быть в состоянии `Up` или `running`.

---
Этот список не является исчерпывающим, но покрывает основные моменты для первоначальной проверки. Для более глубокого тестирования потребуется разработка специфичных тестовых сценариев для каждого приложения и их интеграций.