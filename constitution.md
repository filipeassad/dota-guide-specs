# Constituição do Dota Guide

> Princípios inegociáveis que toda spec, plano e tarefa DEVEM respeitar.
> Base: [PRD](prd.md) (portal de builds de Dota 2 vinculadas a versões do jogo).
> Palavras-chave: **DEVE / NÃO DEVE** = obrigatório (violar exige justificativa
> explícita no plano). **DEVERIA** = recomendado. **(INEGOCIÁVEL)** = sem exceção.

## Princípios

### I. Toda build pertence a uma versão do jogo (INEGOCIÁVEL)

Toda build DEVE estar vinculada a exatamente uma versão do Dota 2, e essa versão
DEVE aparecer em todo lugar onde a build é exibida (listagem, detalhe, página do
herói). Build de versão anterior à versão atual DEVE ser sinalizada como
desatualizada. Uma build NÃO DEVE mudar de versão silenciosamente: levar uma build
para outra versão gera uma nova build (cópia), preservando a original.
**Por quê:** é a proposta de valor do produto. Builds mudam a cada patch, e uma
build sem versão (ou com versão errada) leva o jogador a montar algo que não
funciona mais.

### II. Dados do jogo são versionados e vêm de fonte externa (INEGOCIÁVEL)

Heróis, habilidades, talentos e itens DEVEM ser guardados por versão do
jogo, e uma build DEVE ser validada contra os dados da versão a que pertence. Esses
dados DEVEM vir de uma fonte externa por um processo de importação que pode ser
repetido sem duplicar nem corromper dados. Eles NÃO DEVEM ser cadastrados nem
corrigidos à mão no banco.
**Por quê:** um talento ou item que muda num patch não pode alterar builds antigas,
e manter centenas de habilidades e itens à mão a cada patch é inviável para um
time pequeno e gera erros.

### III. A build respeita as regras do jogo

A API DEVE recusar uma build que o jogo não permite: habilidade evoluída além do
nível máximo, ultimate antes do nível permitido, talento de outro herói ou de
outro nível, item que não existe na versão. As regras DEVEM ser as da versão da
build.
**Por quê:** uma build impossível de executar no jogo destrói a confiança no
portal e polui o ranking.

### IV. Conteúdo é do autor; quem decide é o servidor (INEGOCIÁVEL)

Só o autor DEVE poder editar ou excluir a própria build ou o próprio comentário;
o admin PODE moderar (ocultar ou excluir) qualquer conteúdo de usuário. Notícias e
versões do jogo DEVEM ser criadas só pelo admin ou pela importação automática de
versões (spec 002), nunca por usuários. Toda verificação de permissão
DEVE ser feita na API, nunca só no front-end.
**Por quê:** é a base de confiança de um portal colaborativo. Uma permissão
checada só no front-end é contornada chamando a API direto.

### V. O ranking é justo e calculado no servidor

Cada usuário DEVE ter no máximo um voto por build e NÃO DEVE votar na própria
build. A pontuação DEVE ser calculada só pela API, a partir dos votos registrados.
Votar DEVE ter limite de frequência por usuário.
**Por quê:** o ranking decide quais builds aparecem primeiro. Se puder ser
inflado (voto repetido, autovoto, script), ele deixa de indicar as melhores builds.

### VI. Texto de usuário é dado, nunca código; dados pessoais mínimos

Todo texto escrito por usuários (descrições, justificativas, comentários,
nomes de grupo) DEVE ser exibido de forma que não execute código no navegador
de quem lê, e DEVE ter tamanho máximo. Só DEVEM ser coletados os dados pessoais
necessários para a conta. Senhas, tokens e e-mails NÃO DEVEM aparecer em logs,
URLs ou mensagens de erro.
**Por quê:** conteúdo de usuário exibido para milhares de leitores é o caminho
mais comum para ataques (XSS) e spam; um vazamento de contas acaba com o portal.

### VII. Regras de build e de ranking têm teste automatizado

Toda regra nova ou alterada de validação de build (ordem de habilidades,
talentos, itens, versão) e de ranking/pontuação DEVE ter teste automatizado que
falha se a regra for quebrada.
**Por quê:** são as regras mais fáceis de quebrar sem perceber a cada patch novo e
as que mais afetam o que o usuário vê.

### VIII. Operação enxuta

Todo componente novo de infraestrutura (serviço pago, fila, cache, banco extra,
worker dedicado) DEVE ser justificado no plano com seu custo mensal estimado. A
preferência é por serviços gerenciados com plano gratuito ou de baixo custo e por
resolver no banco/aplicação existentes antes de adicionar peças.
**Por quê:** o projeto é mantido por um time pequeno; cada peça extra é mais
custo fixo e mais uma coisa para manter.

### IX. A API é o produto; os clientes são substituíveis

Toda regra de negócio, validação e autorização DEVE estar na API. Os clientes (o
web app atual e futuros fronts ou app mobile) apenas exibem e coletam dados e NÃO
DEVEM ser a única barreira de nenhuma regra. A API NÃO DEVE depender de nada
específico do cliente web (ex.: respostas em HTML). A API DEVE ter um contrato
documentado e versionado; mudanças incompatíveis DEVEM gerar uma nova versão,
sem quebrar clientes existentes.
**Por quê:** uma regra que só existe no front precisa ser reescrita em cada
cliente e pode ser contornada chamando a API direto.

## Ecossistema

| Repositório | Conteúdo | Dono |
| --- | --- | --- |
| `dota-guide-specs` | Esta constituição, PRD e specs de feature (o quê e por quê) | Produto |
| `dota-guide-api` | API, banco, importação dos dados do jogo, contrato da API, planos e tarefas do back-end | Time de back-end |
| `dota-guide-web` | Web App (portal público e painel admin), planos e tarefas do front-end | Time de front-end |

- Os repositórios DEVEM ser clonados lado a lado na mesma pasta; os `CLAUDE.md`
  de `dota-guide-api` e `dota-guide-web` importam esta constituição por caminho relativo
- Cada projeto é versionado e publicado de forma independente. A stack de cada
  projeto é decidida e registrada no `CLAUDE.md` do próprio repositório, desde
  que respeite os princípios e as restrições abaixo
- Novos clientes (ex.: app mobile) DEVEM ser novos repositórios que seguem as
  mesmas specs e consomem o mesmo contrato da API

## Restrições técnicas (valem para todos os projetos)

- O Web App é responsivo e mobile-first. O painel do admin é uma área do Web App
- Banco de dados relacional. Decisão atual: PostgreSQL
- Páginas públicas de builds, heróis e notícias DEVEM ter URL própria e
  compartilhável, com título e descrição próprios
- Datas DEVEM ser armazenadas e trafegadas em UTC (ISO 8601) e exibidas no fuso
  do usuário
- A importação dos dados do jogo DEVE ficar isolada num único módulo da API, para
  que trocar de fonte não espalhe mudanças pelo código
- Configuração e segredos apenas por variáveis de ambiente, nunca no repositório
- Formatação e lint por ferramenta em cada repositório. Estilo de código não é
  discutido em revisão
- Contrato da API em **OpenAPI gerado a partir do código** da API (schemas de
  validação das rotas), exportado para `openapi.json` no repositório da API.
  Decisão atual (2026-11-24)
- Fonte dos dados do jogo: **site oficial do Dota 2**, por coleta automática do
  datafeed JSON do site. Decisão atual (2026-11-24, spec 002); detalhes em
  `discovery/dados-do-jogo.md`
- As **regras de nível** do jogo (níveis da ultimate, níveis de talento, níveis
  sem ponto), que o site oficial não fornece, ficam escritas no código da API,
  por versão e com teste. Elas não contam como "dados do jogo cadastrados à mão"
  (princípio II): heróis, habilidades, talentos e itens continuam vindo só da
  coleta. Decisão atual (2026-11-24)
- Armazenamento de arquivos (ícones do jogo, GIFs dos comentários, imagens das
  notícias): **Cloudflare R2**, por um adaptador na API. Decisão atual (2026-11-24,
  plano 002 da API)
- **A definir:** hospedagem de cada projeto. Ao decidir, registrar aqui
  (alteração MINOR)

## Fluxo de desenvolvimento

- Mudanças em builds, dados do jogo, ranking, controle de acesso ou moderação
  DEVEM passar pelo fluxo spec → plano → tarefas. Correções pequenas de UI
  podem ser feitas direto
- A spec é escrita e aprovada em `dota-guide-specs`. Cada projeto afetado escreve o
  próprio `plans/<nnn>-<feature>/plan.md` e `tasks.md` no seu repositório, com
  o mesmo número da spec
- Todo plano DEVE ter uma seção **"Conferência da constituição"** listando cada
  princípio e como é atendido (ou a justificativa da exceção)
- Mudanças no contrato da API DEVEM ser propostas no plano da API e revisadas
  pelo time de front-end antes da implementação. O front-end PODE trabalhar em
  paralelo usando o contrato aprovado (mock)
- Uma feature só está pronta quando os testes passam em cada projeto e os
  critérios de aceite da spec funcionam de ponta a ponta

## Governança

- Esta constituição tem precedência sobre qualquer spec, plano, `CLAUDE.md` ou
  instrução de tarefa, em qualquer repositório
- Alterações são feitas em `dota-guide-specs` e seguem versionamento semântico:
  MAJOR para remover ou redefinir um princípio, MINOR para adicionar
  princípio/seção ou registrar decisão, PATCH para ajustes de redação
- Após uma alteração MAJOR, specs e planos em andamento DEVEM ser conferidos de novo
- Toda alteração DEVE atualizar a versão e a data abaixo

**Versão:** 1.4.0 | **Ratificada em:** 2026-11-24 | **Última alteração:** 2026-11-24
