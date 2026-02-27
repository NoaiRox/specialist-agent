# Codex

## Instalação

```bash
cd your-project
npx specialist-agent init
```

## Uso

Referencie os agents nos prompts:

```
Using @builder:
Create a UserProfile component with avatar and bio fields.

Using @tdd:
Implement calculateTax following TDD.

Using @planner:
Plan the checkout flow implementation.
```

## Agents Disponíveis

| Agent | Use Para |
|-------|---------|
| `@builder` | Criar módulos e componentes |
| `@reviewer` | Revisão de código |
| `@doctor` | Depurar problemas |
| `@planner` | Planejar funcionalidades |
| `@tdd` | Desenvolvimento orientado a testes |
| `@api` | Design de APIs |
| `@security` | Autenticação |
| `@finance` | Pagamentos |

## Solução de Problemas

**Agents não encontrados?**
1. Verifique se `.claude/agents/` existe
2. Referencie o caminho completo: `.claude/agents/builder.md`
