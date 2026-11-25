# Specs do MVP

Cada pasta é uma feature com `spec.md` (o quê e por quê, sem tecnologia). Toda
spec e todo plano respeitam a [constituição](../constitution.md). Termos como
build, versão e talento estão no [glossário](../glossario.md).

As specs descrevem o **produto** e valem para qualquer cliente (web hoje,
mobile amanhã). Por isso existe uma spec por feature, não uma por projeto. Os
planos técnicos ficam no repositório de cada projeto, com o mesmo número:

```
dota-guide-specs/specs/001-contas-acesso/spec.md   ← o quê (este repo)
dota-guide-api/plans/001-contas-acesso/plan.md     ← como, no back-end
dota-guide-web/plans/001-contas-acesso/plan.md     ← como, no front-end
```

## Ordem de implementação

| #   | Feature                                                                | Depende de | Spec     | API | Web |
| --- | ---------------------------------------------------------------------- | ---------- | -------- | --- | --- |
| 001 | [Contas e controle de acesso](001-contas-acesso/spec.md)               | —          | Aprovada | Implementada (0.1.0) | —   |
| 002 | [Versões e dados do jogo](002-versoes-dados-jogo/spec.md)              | 001        | Aprovada | Implementada (0.2.0) | —   |
| 003 | [Editor de builds](003-editor-builds/spec.md)                          | 001, 002   | Revisada | —   | —   |
| 004 | [Listagem, filtros e página de herói](004-listagem-filtros/spec.md)    | 003        | Revisada | —   | —   |
| 005 | [Votos e ranking](005-votos-ranking/spec.md)                           | 003, 004   | Revisada | —   | —   |
| 006 | [Comentários em builds](006-comentarios/spec.md)                       | 003        | Revisada | —   | —   |
| 007 | [Notícias](007-noticias/spec.md)                                       | 001, 002   | Revisada | —   | —   |

- Spec: Rascunho → Revisada (perguntas respondidas) → Aprovada
- API / Web: — → Planejada → Em andamento → Implementada (com a versão publicada)

## Antes do primeiro plano

- [x] Responder as perguntas em aberto da spec 001
- [ ] Confirmar a stack da API e do Web App (ver `CLAUDE.md` de cada repositório)
- [x] Decidir a fonte dos dados do jogo: site oficial (spec 002)
- [x] Mapear as fontes no site oficial ([discovery/dados-do-jogo.md](../discovery/dados-do-jogo.md))
- [x] Responder as perguntas da discovery `dados-do-jogo.md`
- [x] Localizar as bênçãos no datafeed (discovery `dados-do-jogo.md`, seção 10.1)
- [x] Decidir o armazenamento de arquivos: Cloudflare R2 (constituição 1.4.0)
- [ ] Decidir hospedagem

## Fluxo por feature

1. **dota-guide-specs:** revisar `spec.md`, responder as perguntas em aberto na
   própria spec e marcar como Aprovada
2. **dota-guide-api:** escrever `plans/<nnn>-<feature>/plan.md` (decisões citando o
   requisito que atendem, mudanças no contrato da API, "Conferência da
   constituição") e `tasks.md` (cada tarefa com verificação)
3. **dota-guide-web:** revisar a mudança de contrato proposta pela API e escrever o
   próprio `plan.md` e `tasks.md`. Pode implementar em paralelo usando o
   contrato aprovado como mock
4. Cada time implementa uma tarefa por vez no seu repositório
5. Validar todos os critérios de aceite da spec de ponta a ponta e atualizar a
   tabela acima
