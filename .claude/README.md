# `.claude/` — o Claude Code neste projeto

Esta pasta transforma `documentacao/` em processo de trabalho: o que ler em cada momento, como
verificar e como revisar. Ela **não repete** a documentação. Aponta para a seção certa e
acrescenta só o que a documentação não diz: o ambiente (Windows + Docker), a ordem dos passos e
a verificação.

```
orientar  →  implementar  →  verificar  →  revisar  →  documentar
CLAUDE.md    /nova-entidade   /verificar    revisor-      rules/documentacao.md
+ rules/     /migracao                      arquitetura
```

## Mapa

| Arquivo | Para quê | Carrega quando |
|---|---|---|
| `CLAUDE.md` | fontes de verdade, invariantes, armadilhas do ambiente, fluxo | toda sessão |
| `rules/rotas.md`, `servicos.md`, `repositorios.md`, `schemas.md`, `modelos-e-migracoes.md`, `autorizacao.md`, `extensoes-e-config.md`, `testes.md` | o que cada camada pode e não pode, com a seção da doc que explica o porquê | ao ler ou editar um arquivo da camada (`paths`) |
| `rules/infra.md`, `documentacao.md`, `pasta-claude.md` | Docker, CI e dependências; manutenção da doc; manutenção desta pasta | idem |
| `skills/verificar/` (`SKILL.md` + `verificar.sh`) | `/verificar`: as etapas do CI, no Docker | `/verificar`, ou quando o Claude for testar |
| `skills/migracao/` | `/migracao`: gerar, revisar e validar migração Alembic | `/migracao`, ou ao mudar modelo |
| `skills/nova-entidade/` | `/nova-entidade`: receita 1 ponta a ponta | `/nova-entidade`, ou ao pedir entidade nova |
| `agents/revisor-arquitetura.md` | revisão independente do diff, só leitura | ao fim de mudança não trivial |
| `settings.json` | permissões do projeto (versionado) | toda sessão |

Preferências pessoais vão em `settings.local.json`, que não é versionado.

## Princípios

1. **`documentacao/` é a fonte de verdade.** Aqui entram processo, ambiente e ponteiros; conteúdo
   copiado envelheceria em dois lugares.
2. **Contexto sob demanda.** `CLAUDE.md` curto; regras só carregam com a camada; skills só quando
   invocadas; a revisão roda em contexto isolado.
3. **Toda skill termina em `/verificar`**, e mudança não trivial termina no revisor.
4. **Comandos são skills.** Cada skill já é um `/comando`; não há `.claude/commands/`.
5. **Corrigir no uso.** Instrução errada ou incompleta é corrigida na sessão em que aparece e
   registrada abaixo.

## Roadmap

| Onda | Item | Estado |
|---|---|---|
| 1 | `CLAUDE.md`, 11 regras, `settings.json` (nome em minúsculas, permissões, `.env` protegido), `.gitattributes` | feito em 2026-11-06 |
| 1 | `/verificar` | feito e validado: `tudo` (61 testes), `estatico`, `testes`, falha de teste, falha de formatação com diff, limpeza do banco descartável |
| 1 | `/migracao` | feito; validados o rebuild do `api` e o `alembic check`. Falta exercitar com mudança real de modelo |
| 1 | `/nova-entidade` | feito; validar e ajustar no primeiro uso real |
| 1 | `revisor-arquitetura` | feito e validado num ensaio sobre o molde `Item`: zero bloqueantes; os trechos das regras que gerariam falso positivo foram corrigidos (ver registro) |
| 2 | `/smoke-api`: recria o `api`, cria um usuário por papel (`flask create-user`), roda a matriz 401/403/404/200 com curl e rastreia por `request_id` nos logs — também é a skill que o `/run` embutido procura | a fazer |
| 2 | Hook `SessionStart` com 1–2 linhas de estado do Docker (daemon no ar? containers saudáveis?) | a fazer |
| 2 | `/remover-exemplo`: tirar `Item` com segurança quando houver entidades reais (a migração 0001 cria `users` e `items` juntas; `test_write_cost`, `test_authz` e `test_contracts` usam `Item`) | a fazer |
| 3 | `/medir-consultas` (padrão `sql_log` / `test_write_cost`) antes de otimizar | a fazer |
| 3 | `memory: project` no revisor, para acumular erros recorrentes | a avaliar |
| 3 | Calibrar permissões com `/fewer-permission-prompts` depois de algumas sessões | a fazer |

## Pendências encontradas no projeto

Ficaram fora da onda 1, que só mexeu em `.claude/` e `.gitattributes`. As regras já orientam a
não repetir nenhuma delas; corrigir a origem é decisão do usuário.

| Onde | O quê |
|---|---|
| `tests/test_authz.py` | O 403 de escrita só é testado no POST. Sem teste de PATCH e DELETE com `Role.LEITOR`, remover o `@permission_required(Permission.ITEM_ESCREVER)` desses métodos passaria no CI. |
| `documentacao/03-como-estender.md` §10.1 | O exemplo do GET usa `get_page(..., **query)`: com o filtro `name` da §1.5 dá `TypeError` (500); também falta o `@error_responses`. |
| `documentacao/03-como-estender.md` §1.1 e §1.5 | A receita do `Product` omite o lado inverso do `relationship` em `User` e o validador "ao menos um campo" do `ProductUpdateSchema`. |
| `documentacao/01-arquitetura.md` | Diz "nenhuma extensão importa outra", mas `ext.api` importa `ext.config` e `ext.commands` importa `ext.database` (as regras tratam as duas como exceções aceitas). |
| `app/blueprint/restapi/v1/item.py` (POST) | A nota interna da docstring sai no OpenAPI público como `description`. |

## Registro de ajustes

- 2026-11-06 — Onda 1 criada e validada.
- 2026-11-06 — Ensaio do revisor sobre `Item`. Corrigidos nas regras e no prompt: o 422 no
  `@error_responses`, o piso de leitura × decorator de escrita (com teste de 403 por operação), o
  uso de `http_errors` pelas rotas, a docstring como `summary` público e a precedência do molde
  sobre a doc. O ensaio também afirmou que o `summary` do GET sai cortado; conferido no
  `openapi.json`, é falso — o smorest usa o 1º parágrafo inteiro.
- 2026-11-06 — Skills e agente apareceram na mesma sessão, sem `/reload-skills` nem reinício.
