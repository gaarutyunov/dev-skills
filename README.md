# dev-skills

Personal collection of Claude Code skills for infrastructure tools - CLIs, APIs, and SDKs.

## Plugins

| Plugin | Description |
|--------|-------------|
| [gitea](./gitea) | Skills for Gitea: tea CLI operations and Go SDK development |

## Installation

```bash
# Add the marketplace
claude mcp add-json dev-skills '{"type":"url","url":"https://raw.githubusercontent.com/germanarutyunov/dev-skills/main/marketplace.json"}'

# Install a plugin
/plugin install gitea@dev-skills
```

## Adding New Plugins

Each plugin lives in its own directory with the structure:

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   └── skill-name/
│       ├── SKILL.md
│       └── references/
├── README.md
└── LICENSE
```

Then add the plugin to `marketplace.json`.

## License

MIT
