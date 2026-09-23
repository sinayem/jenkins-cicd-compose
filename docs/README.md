# Documentation

Evidence for the assessment report goes here.

| Folder / file | Contents |
|---|---|
| `screenshots/` | Jenkins UI, pipeline runs, console logs, Docker Hub |
| `logs/` | Exported console logs and the audit trail log |

Suggested screenshots:

1. Both GitHub repositories (structure and commits on `main`)
2. `docker compose ps` and `docker compose exec jenkins docker version`
3. Jenkins login page (anonymous access blocked) and the matrix-permissions page
4. Pipeline job configuration (SCM and triggers)
5. Stage view with all stages green, plus the console log of each stage
6. A run failing at the security gate, then the fixed run passing
7. Archived artifacts (`npm-audit.json`, test report)
8. Pushed image tags on Docker Hub
9. Audit Trail log
