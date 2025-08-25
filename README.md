# Test App

Простое веб-приложение на Nginx с автоматическим деплоем в Kubernetes.

## Настройка GitHub Actions

Для работы автоматического деплоя необходимо настроить следующие секреты в репозитории:

### 1. Секреты для Docker Registry

- `REGISTRY_URL` — адрес реестра (например: `cr.yandex/<registry_id>`)
- `YC_OAUTH_TOKEN` — токен (используется как пароль при логине в реестр с пользователем `oauth`)

### 2. Секреты для доступа к Yandex Managed Kubernetes

- `YC_OAUTH_TOKEN` — OAuth-токен сервисного/пользовательского аккаунта в Yandex Cloud
- `YC_CLOUD_ID` — ID облака
- `YC_FOLDER_ID` — ID каталога
- `YC_CLUSTER_ID` — ID кластера Managed Kubernetes

## Структура проекта

- `Dockerfile` - инструкции для сборки Docker образа
- `nginx.conf` - конфигурация Nginx
- `static/` - статические файлы приложения
- `.github/workflows/deploy.yml` - GitHub Actions workflow

## Kubernetes Deployment

Workflow автоматически обновляет deployment с именем `test-app`. Убедитесь, что в вашем кластере существует deployment с таким именем.

Примечания:
- Аутентификация и получение kubeconfig выполняются командой `yc managed-kubernetes cluster get-credentials --id <CLUSTER_ID> --external`.
- Для логина в реестр используется `username: oauth` и `password: YC_OAUTH_TOKEN`.


## Триггеры workflow

Workflow автоматически запускается при:

### 🔄 **При каждом коммите:**
- Push в ветки `main` или `master`
- Создании Pull Request в эти ветки
- **Результат:** Только сборка и загрузка Docker образа в registry

### 🏷️ **При создании тега (например, v1.0.0):**
- Создание тега: `git tag v1.0.0 && git push origin v1.0.0`
- **Результат:** Сборка, загрузка образа с тегом версии + деплой в Kubernetes кластер

### 🚀 **Ручной запуск:**
- Через GitHub Actions → Run workflow

## Примеры использования

### 📝 **Обычная разработка (только сборка):**
```bash
git add .
git commit -m "Add new feature"
git push origin main
# → Собирается образ с тегами: latest, main-abc123, 123-abc123
```

### 🚀 **Релиз версии (сборка + деплой):**
```bash
git tag v1.2.0
git push origin v1.2.0
# → Собирается образ с тегами: v1.2.0, v1.2, v1, latest
# → Образ автоматически деплоится в Kubernetes кластер
```
