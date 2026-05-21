# Инструкция по запуску: reusable workflow

## Архитектура

```
amihailov76/docs-pipeline  (сервисный репо, публичный)
├── .github/workflows/docs-review.yml   ← reusable workflow
├── .github/scripts/                    ← ru_linter.py, review-agent.py
├── styles/Russian/                     ← правила линтера
├── styles/Microsoft/, styles/proselint/
├── style_guide/
├── .vale-en.ini, .vale-ru.ini
└── mcp_server/

amihailov76/doc-reviewer   (контентный репо)
└── .github/workflows/docs-review.yml  ← caller workflow (5 строк)
```

Раннер зарегистрирован **только в docs-pipeline**. Он обрабатывает задания от
doc-reviewer, потому что reusable workflow запускается на раннере вызываемого репо.

---

## Шаг 1. Создать репо docs-pipeline на GitHub

1. Откройте https://github.com/new
2. Repository name: `docs-pipeline`
3. **Public** (обязательно — иначе checkout в workflow упадёт без PAT)
4. Не инициализировать (без README, без .gitignore)
5. Нажмите Create repository

---

## Шаг 2. Инициализировать локальный репо и запушить

Все файлы уже подготовлены в `C:\docuz-test\docs-pipeline-export\`.

```powershell
# Переходим в папку экспорта
cd C:\docuz-test\docs-pipeline-export

# Инициализируем git
git init
git add .
git commit -m "feat: initial docs-pipeline setup"

# Связываем с GitHub и пушим
git remote add origin https://github.com/amihailov76/docs-pipeline.git
git branch -M main
git push -u origin main
```

---

## Шаг 3. Зарегистрировать self-hosted раннер в docs-pipeline

Раннер нужен именно здесь — reusable workflow берёт раннер из вызываемого репо.

1. GitHub → `amihailov76/docs-pipeline` → Settings → Actions → Runners
2. Нажмите **New self-hosted runner**
3. Выберите Windows, скопируйте команды
4. Запустите на той же машине, где уже запущен раннер для docuz-test
   (машина может быть зарегистрирована в нескольких репо одновременно)
5. Убедитесь, что раннер онлайн: в списке Runners появился статус **Idle**

---

## Шаг 4. Добавить секреты в doc-reviewer

GitHub → `amihailov76/doc-reviewer` → Settings → Secrets and variables

**Secrets (Actions secrets):**
- `LLM_API_KEY` — ключ API (OpenAI или совместимый)
- `MCP_API_KEY` — ключ MCP-сервера (если используется)

**Variables (Actions variables):**
- `LLM_BASE_URL` — базовый URL API (если не api.openai.com)
- `LLM_MODEL` — название модели (например, `gpt-4o-mini`)
- `MCP_SERVER_URL` — URL MCP-сервера (если используется)

Если MCP не нужен — `MCP_API_KEY` и `MCP_SERVER_URL` можно не задавать.

---

## Шаг 5. Добавить caller workflow в doc-reviewer

Скопируйте файл `caller-workflow-for-doc-reviewer.yml` в doc-reviewer:

```powershell
# Создаём папку для workflow (если нет)
mkdir -p C:\Projects\doc-reviewer\.github\workflows

# Копируем файл
copy C:\docuz-test\docs-pipeline-export\caller-workflow-for-doc-reviewer.yml `
     C:\Projects\doc-reviewer\.github\workflows\docs-review.yml

# Коммитим и пушим
cd C:\Projects\doc-reviewer
git add .github/workflows/docs-review.yml
git commit -m "feat: add docs-review caller workflow"
git push origin main
```

---

## Шаг 6. Тест

1. Создайте ветку в doc-reviewer и измените любой `.mdx` файл в `docs/ru/` или `docs/en/`
2. Откройте Pull Request
3. Навесьте лейбл `docs-review` на PR
4. В GitHub → Actions проверьте, что запустился workflow `Docs Review`
5. Он должен появиться в разделе Actions репо **doc-reviewer**,
   но выполняться на раннере из **docs-pipeline**

---

## Диагностика

**Workflow не запускается:**
- Проверьте, что лейбл называется именно `docs-review`
- Проверьте, что в doc-reviewer есть workflow в `.github/workflows/docs-review.yml`

**Ошибка "Resource not accessible by integration":**
- Проверьте, что в doc-reviewer → Settings → Actions → General включено
  "Allow GitHub Actions to create and approve pull requests"

**Ошибка при checkout pipeline:**
- Убедитесь, что docs-pipeline — **публичный** репо

**Раннер не подхватывает задания:**
- Убедитесь, что раннер зарегистрирован в **docs-pipeline** (не только в docuz-test)
- Проверьте статус: docs-pipeline → Settings → Actions → Runners → должен быть Idle
