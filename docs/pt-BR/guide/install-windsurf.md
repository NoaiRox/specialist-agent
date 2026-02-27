# Windsurf

## Instalação

```bash
cd your-project
npx specialist-agent init
```

## Configuração

Adicione ao `.windsurf/rules.md`:

```
When you see @agent-name, read .claude/agents/agent-name.md and follow it.
When you see /skill-name, execute .claude/skills/skill-name/SKILL.md.
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
2. Reinicie o Windsurf
