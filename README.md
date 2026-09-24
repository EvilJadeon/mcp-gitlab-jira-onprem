# mcp-gitlab-jira-onprem

MCP-сервер для **GitLab** и **Jira Server / Data Center**.

Форк [`mcp-gitlab-jira`](https://www.npmjs.com/package/mcp-gitlab-jira) с правками под on-prem:

| Проблема upstream | Решение в этом пакете |
|---|---|
| Basic Auth `email + apiToken` (Cloud) → **401** на Server/DC | **Bearer** Personal Access Token |
| `Version3Client` (Cloud API v3) ломается on-prem | **`Version2Client`** (REST API v2) |
| Agile-запросы тоже шли через Basic | Bearer |

Подходит для внутренних инстансов вроде `https://jira.example.com` + self-hosted GitLab.

## Требования

- Node.js 18+
- GitLab Personal Access Token (API)
- Jira Server/DC **Personal Access Token**  
  Profile → Personal Access Tokens → Create  
  Проверка:

```bash
curl -sS -o /dev/null -w "%{http_code}\n" \
  -H "Authorization: Bearer $ATLASSIAN_API_TOKEN" \
  -H "Accept: application/json" \
  "$ATLASSIAN_SITE_NAME/rest/api/2/myself"
# ожидаем 200
```

> Cloud API token (`ATATT…`) и Basic `email:token` для on-prem **не подойдут**.

## Установка

### Вариант A — из git

```bash
npm i -g git+ssh://git@YOUR_GITHOST/YOUR_GROUP/mcp-gitlab-jira-onprem.git
# или
npm i -g /path/to/mcp-gitlab-jira-onprem
```

Проверка:

```bash
which mcp-gitlab-jira-onprem
mcp-gitlab-jira-onprem   # должен стартовать и ждать stdio; Ctrl+C
```

### Вариант B — локально без глобальной установки

```bash
git clone <repo-url> ~/tools/mcp-gitlab-jira-onprem
cd ~/tools/mcp-gitlab-jira-onprem
npm install
npm run build
```

В `mcp.json` укажите абсолютный путь к `dist/index.js` (см. ниже).

## Переменные окружения

| Переменная | Обязательна | Пример |
|---|---|---|
| `GITLAB_URL` | да* | `https://gitlab.example.com` |
| `GITLAB_ACCESS_TOKEN` | да* | `glpat-…` |
| `ATLASSIAN_SITE_NAME` | да* | `https://jira.example.com` (**полный URL**, не sitename Cloud) |
| `ATLASSIAN_API_TOKEN` | да* | PAT из Jira Server/DC |
| `ATLASSIAN_USER_EMAIL` | нет | `user@example.com` (для совместимости) |

\*нужны для соответствующей половины инструментов; можно поднять только GitLab или только Jira.

Положите их в `~/.zshrc` / `~/.bashrc` **и** перезапустите Cursor из окружения, где они видны:

```bash
export GITLAB_URL="https://gitlab.example.com"
export GITLAB_ACCESS_TOKEN="glpat-…"
export ATLASSIAN_SITE_NAME="https://jira.example.com"
export ATLASSIAN_API_TOKEN="ваш-pat"
```

Либо пропишите `env` прямо в `mcp.json` (удобно шарить шаблон, секреты — каждый себе).

## Cursor: `~/.cursor/mcp.json`

```json
{
  "mcpServers": {
    "gitlab-jira-mcp": {
      "command": "mcp-gitlab-jira-onprem",
      "args": [],
      "env": {
        "GITLAB_URL": "https://gitlab.example.com",
        "GITLAB_ACCESS_TOKEN": "REPLACE_ME",
        "ATLASSIAN_SITE_NAME": "https://jira.example.com",
        "ATLASSIAN_API_TOKEN": "REPLACE_ME"
      }
    }
  }
}
```

Без глобальной установки:

```json
{
  "mcpServers": {
    "gitlab-jira-mcp": {
      "command": "node",
      "args": ["/home/YOU/tools/mcp-gitlab-jira-onprem/dist/index.js"],
      "env": {
        "GITLAB_URL": "https://gitlab.example.com",
        "GITLAB_ACCESS_TOKEN": "REPLACE_ME",
        "ATLASSIAN_SITE_NAME": "https://jira.example.com",
        "ATLASSIAN_API_TOKEN": "REPLACE_ME"
      }
    }
  }
}
```

После правок: **полностью перезапустите Cursor** (Reload Window часто оставляет старый MCP-процесс).

## Сеть / VPN

Если Jira доступна только в корпоративной сети / без VPN-туннеля:

1. Убедитесь, что `curl …/myself` = 200 **из того же окружения**, что и Cursor.
2. При split-tunnel (Nekoray и т.п.) добавьте `jira.example.com` в **direct/bypass**, иначе TLS/auth могут вести себя иначе, чем в браузере.

## Отличия от upstream

См. [CHANGELOG.md](./CHANGELOG.md). Кратко:

1. Jira auth: Bearer PAT  
2. Jira client: `Version2Client`  
3. Agile REST: Bearer  
4. Search: классический `/rest/api/2/search`  
5. Комментарии: plain string body (не ADF)

GitLab-часть без изменений по смыслу.

Миграция со старых ручных патчей: [docs/MIGRATION.md](./docs/MIGRATION.md).

## Разработка

```bash
npm install
npm run build
npm start   # stdio MCP
```

Исходники — `src/`, артефакт — `dist/`.

## Лицензия

MIT (как у upstream). Upstream: [hainanzhao/mcp-gitlab-jira](https://github.com/hainanzhao/mcp-gitlab-jira) / npm `mcp-gitlab-jira@0.1.9`.
