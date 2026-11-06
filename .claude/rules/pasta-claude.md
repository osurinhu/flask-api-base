---
paths:
  - ".claude/**"
---

# Manutenção da pasta `.claude/`

Mapa, roadmap e registro de ajustes: `.claude/README.md`.

- `documentacao/` é a fonte de verdade: aqui entram processo, ambiente e ponteiros para a seção
  certa. Não copie conteúdo da documentação — ele envelheceria em dois lugares.
- `CLAUDE.md` carrega em toda sessão: mantenha-o curto. O que vale para uma camada vai para
  `rules/` com `paths`; procedimento de vários passos vira skill.
- Toda skill termina em verificação (`/verificar`); fluxo que muda código não trivial termina no
  agente `revisor-arquitetura`.
- Skill nova: `.claude/skills/<nome>/SKILL.md` com `description` dizendo quando usar. Script fica na
  pasta da skill, em LF, chamado por caminho relativo à raiz — é o que a permissão do
  `settings.json` casa.
- "Comandos" são skills (`/nome`): não crie `.claude/commands/`.
- Na versão 2.1.319, skills e agentes novos apareceram sozinhos, na mesma sessão (a doc oficial
  fala em `/reload-skills` e reinício). Se não aparecerem, use um dos dois. `CLAUDE.md` e `rules/`
  novos só valem a partir da sessão seguinte.
- Instrução que se mostrou errada numa sessão: corrija na hora e registre uma linha no
  "Registro de ajustes" do README.
