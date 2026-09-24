# Changelog

## 0.2.0 — on-prem fork

Based on `mcp-gitlab-jira@0.1.9`.

### Fixed (Jira Server / Data Center)

- **Auth**: use Bearer Personal Access Token instead of Cloud Basic `email:apiToken`
  (`jira.js` `authentication.oauth2.accessToken`).
- **API version**: use `Version2Client` instead of `Version3Client` (Cloud API v3
  returns unusable/HTML payloads on many Server/DC instances).
- **Agile REST**: send `Authorization: Bearer <PAT>` instead of Basic.

### Changed

- Package / binary renamed to `mcp-gitlab-jira-onprem`.
- `ATLASSIAN_USER_EMAIL` is optional (not used for auth on Server/DC).
- `ATLASSIAN_SITE_NAME` must be a full base URL (`https://jira.example.com`).

### Unchanged

- GitLab tools and tool names (Cursor configs stay compatible if command is updated).
