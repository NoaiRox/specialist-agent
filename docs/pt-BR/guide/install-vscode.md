# VS Code

## Instalação

```bash
cd your-project
npx specialist-agent init
```

## Configuração

Para o **GitHub Copilot**, adicione ao `.github/copilot-instructions.md`:

```
When asked to use @agent-name, read .claude/agents/agent-name.md and follow it.
When asked to run /skill-name, execute .claude/skills/skill-name/SKILL.md.
```

Para o **Continue**, adicione comandos customizados em `.continue/config.json`.

## Uso

```
Use @builder to create a products module

Read .claude/agents/reviewer.md and review src/components/
```

## Solução de Problemas

**A IA não está seguindo as instruções?**

Seja explícito:
```
Read .claude/agents/builder.md completely, then follow its workflow.
```
