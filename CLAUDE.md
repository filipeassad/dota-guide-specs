# dota-guide-specs

Repositório de produto do Dota Guide: o **quê** e o **porquê**. Não contém código.

@constitution.md

## Conteúdo

- `constitution.md`: princípios que valem para todos os repositórios
- `prd.md`: documento de requisitos do produto (origem das specs)
- `glossario.md`: significado de build, versão, talento, grupo de itens etc.
  Usar sempre esses termos nas specs
- `specs/<nnn>-<feature>/spec.md`: specs de feature (ver `specs/README.md`)
- `discovery/`: investigações para destravar decisões (opções, custos, fontes).
  Uma discovery termina numa decisão registrada na spec ou na constituição

## Regras para specs

- Spec NÃO cita tecnologia (banco, framework, biblioteca). Isso fica no `plan.md`
  de cada projeto (`dota-guide-api`, `dota-guide-web`)
- Seções: Problema, Objetivos, Fora do escopo, Requisitos (EARS),
  Critérios de aceite (Dado/Quando/Então), Casos de borda, Decisões, Perguntas em aberto
- O que não se sabe vai para "Perguntas em aberto", nunca é inventado
- Respostas às perguntas são escritas na própria spec (`**R:** ...`)
