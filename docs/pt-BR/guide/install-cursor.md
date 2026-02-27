# Cursor

## Instalação

```bash
cd your-project
npx specialist-agent init
```

## Configuração

Adicione ao `.cursorrules`:

```
When you see @agent-name, read and follow .claude/agents/agent-name.md
When you see /skill-name, execute .claude/skills/skill-name/SKILL.md
```

## Uso

```
@builder create a UserProfile component

@reviewer review src/components/

/plan add authentication
```

## Solução de Problemas

**Agents não estão carregando?**
1. Verifique se `.claude/agents/` existe
2. Reinicie o Cursor

**Framework incorreto?**
Crie `.claude/config.json`:
```json
{ "framework": "react" }
```
