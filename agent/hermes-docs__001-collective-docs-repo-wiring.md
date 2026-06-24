hermes-docs__001-collective-docs-repo-wiring
tags: hermes, docs, github, windows, codex, claude, opencode

collective-doc-library is catalog/search repo, not doc payload
collective-docs-1 and collective-docs-2 are doc shard repos
docs layout is <source>/agent/*.md and <source>/human/*.md
Hermes primary lookup path: collective-doc-library/search/search.py -> collective-docs-*/hermes/agent/*.md
Codex reads AGENTS.md
Codex quick reference: CODEX.md
Codex searchable source: codex/agent/guide.md
Claude reads CLAUDE.md
OpenCode uses same local search convention until an opencode-specific adapter exists
no MCP server in current setup

cd ../collective-doc-library && python search/search.py --hybrid "Hermes Docker config volume" --source hermes
cd ../collective-doc-library && python search/search.py --hybrid "AGENTS.md local docs" --source codex
python search/search.py --keyword "your query"
python search/search.py --semantic "your query"
rg -i "your query" */agent/
