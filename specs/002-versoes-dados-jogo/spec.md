# 002 — Versões e dados do jogo

## Problema

Uma build só faz sentido dentro de uma versão do Dota 2: habilidades, talentos e
itens mudam a cada patch. Para montar e validar builds, o portal precisa saber
quais versões existem, qual é a atual e quais heróis, habilidades,
talentos e itens existiam em cada uma.

## Objetivos

- Cada versão nova do jogo entra no portal sozinha, com os dados coletados do
  site oficial do Dota 2 (constituição, princípio II).
- Uma versão nova vira uma notícia sem o admin precisar escrevê-la do zero.
- Builds antigas continuam mostrando os dados da versão delas, mesmo depois de
  um patch mudar um talento ou remover um item.

## Fora do escopo

- Estatísticas de partidas (win rate, popularidade de itens), conforme o PRD.
- Comparação automática entre versões ("o que mudou neste patch").
- Cadastro ou correção manual de heróis, habilidades, talentos e itens
  (constituição, princípio II). Correções são feitas por reimportação (req. 17).

## Requisitos

### Versões

1. Cada versão DEVE ter: número oficial (ex.: 7.37d), data de lançamento e link
   para as notas do patch. Patches "letra" (7.37c, 7.37d) são **versões
   separadas**.
1a. As versões DEVEM ser criadas pela importação automática (req. 15). O admin
    TAMBÉM DEVE poder cadastrar uma versão (ex.: quando a coleta falhar).
2. O número da versão DEVE ser único.
3. QUANDO a importação de uma versão nova terminar com sucesso, ela DEVE ser
   marcada como **atual** automaticamente. Só uma versão DEVE ser a atual por vez;
   marcar uma nova desmarca a anterior. O admin DEVE poder marcar outra versão
   como atual (ex.: para desfazer um erro).
4. QUANDO a versão atual mudar, todas as builds de versões anteriores DEVEM passar
   a ser exibidas como **desatualizadas** (constituição, princípio I).
5. Uma versão NÃO DEVE poder ser marcada como atual enquanto os dados do jogo
   dela não estiverem importados.
6. Uma versão que já tem builds NÃO DEVE poder ser excluída.

### Dados do jogo

7. Para cada versão, o sistema DEVE guardar: heróis, habilidades de cada herói
   (indicando qual é a ultimate e o nível máximo de cada uma), talentos de cada
   herói (nível e as duas opções), o que **Aghanim's Scepter** e **Aghanim's
   Shard** mudam em cada herói (spec 003, req. 14a) e itens, incluindo **itens
   neutros** (com o tier) e **bênçãos** (spec 003, req. 14b).
7a. A ordem dos talentos DEVE seguir a árvore do jogo, de baixo para cima: nível 10
    (duas opções), depois 15, 20 e 25.
7b. Os textos do jogo (nomes e descrições) DEVEM ser coletados em **inglês** e
    guardados com o idioma, para que o português possa ser adicionado no futuro
    sem mudar as builds. A interface do portal continua em português.
7c. A primeira carga do portal DEVE ter **só a versão atual** do jogo. Versões
    anteriores não são importadas.
7d. As **regras de nível** que o site oficial não fornece (em quais níveis a
    ultimate recebe ponto, a partir de que nível cada nível de talento libera, em
    quais níveis o herói não ganha ponto, exceções de heróis específicos) DEVEM
    ser escritas no código do portal, por versão, com teste automatizado, e
    revisadas quando sair uma versão que mude alguma delas (constituição,
    princípios II e III).
8. Os dados DEVEM ser coletados do **site oficial do Dota 2**, por um processo
   automático de coleta. A importação DEVE poder ser repetida para a mesma versão
   sem duplicar dados.
9. Cada herói, habilidade, talento e item DEVE ter um identificador
   estável entre versões, para que o sistema saiba que é "o mesmo item" em 7.37c e
   7.37d.
10. Os dados de uma versão já usada por builds NÃO DEVEM ser alterados por uma
    importação de outra versão.
11. Imagens (ícones de heróis, habilidades e itens) DEVEM estar disponíveis para
    exibição nas telas de builds.

### Consulta

12. Qualquer visitante DEVE poder consultar a lista de versões e a versão atual.
13. Qualquer visitante DEVE poder consultar os heróis de uma versão e, de cada
    herói, as habilidades e talentos daquela versão.
14. Qualquer visitante DEVE poder consultar os itens de uma versão.

### Importação automática

15. O sistema DEVE verificar no site oficial se saiu uma versão nova **3 vezes por
    dia: de manhã, à tarde e à noite** (horários exatos definidos no plano). QUANDO encontrar uma versão que não existe no portal, ele DEVE criá-la
    e importar os dados do jogo dela, sem ação do admin.
16. SE a coleta falhar ou vier incompleta (ex.: herói sem habilidades, talento
    sem as duas opções), ENTÃO a importação NÃO DEVE gravar dados parciais e o
    admin DEVE ser avisado.
17. O admin DEVE poder **reimportar** uma versão a qualquer momento, para corrigir
    dados coletados errado. A reimportação NÃO DEVE duplicar dados (req. 8).
18. O admin DEVE poder ver o histórico de importações: quando rodou, de qual
    versão, se deu certo e o motivo da falha.

### Notícia da versão nova

19. QUANDO uma versão nova for importada com sucesso, o sistema DEVE criar e
    **publicar** automaticamente uma notícia com os dados da atualização (número,
    data, link das notas do patch e heróis/itens novos ou removidos). O admin PODE
    editar ou despublicar essa notícia como qualquer outra (spec 007;
    constituição, princípio IV).

## Critérios de aceite

```
Cenário: nova versão atual
  Dado a versão atual 7.37c com builds publicadas
  Quando a importação da 7.37d termina com sucesso
  Então a 7.37d passa a ser a atual e a 7.37c deixa de ser
  E as builds de 7.37c aparecem como desatualizadas

Cenário: versão sem dados não vira atual
  Dado a versão 7.38 cadastrada sem dados importados
  Quando o admin tenta marcá-la como atual
  Então o sistema recusa e informa que faltam os dados do jogo

Cenário: importação repetida
  Dado a versão 7.37d com dados importados
  Quando a importação de 7.37d roda de novo
  Então nenhum herói, habilidade, talento ou item fica duplicado

Cenário: talento muda no patch
  Dado uma build de 7.37c que escolheu um talento do herói
  E na versão 7.37d esse talento foi trocado
  Quando alguém abre a build de 7.37c
  Então ela mostra o talento como era em 7.37c

Cenário: versão com builds
  Dado a versão 7.37c com builds
  Quando o admin tenta excluí-la
  Então o sistema recusa

Cenário: versão nova detectada
  Dado a última versão no portal 7.37d
  Quando o site oficial publica a 7.37e
  Então a verificação seguinte cria a versão 7.37e e importa os dados dela
  E a 7.37e passa a ser a versão atual
  E uma notícia sobre a 7.37e é publicada na página de notícias

Cenário: admin desfaz versão atual
  Dado a 7.37e marcada como atual por engano
  Quando o admin marca a 7.37d como atual
  Então a 7.37d volta a ser a atual

Cenário: coleta incompleta
  Quando a coleta da 7.37e traz um herói sem habilidades
  Então nada da 7.37e é gravado
  E o admin é avisado da falha

Cenário: reimportar
  Dado a versão 7.37e com um item coletado errado
  Quando o admin reimporta a 7.37e
  Então o item fica igual ao do site oficial
  E nada fica duplicado
```

## Casos de borda

- Herói novo lançado num patch: existe só a partir daquela versão.
- Item removido do jogo: some das versões novas, continua nas builds antigas.
- Herói com mais habilidades que o normal ou com regras especiais de nível
  (ex.: heróis que evoluem a ultimate de outra forma).
- Patch "letra" (7.37c → 7.37d) com poucas mudanças: os dados são quase iguais.
- O site oficial fica fora do ar, atrasa os dados de um patch novo ou muda o
  formato das páginas (a coleta quebra).
- O site publica o patch em etapas (notas antes dos dados atualizados).
- O admin marca a versão errada como atual e precisa desfazer.
- Reimportação de uma versão que já tem builds remove um talento ou item usado
  por elas (req. 10).

## Decisões

- **Dados do jogo por versão e importados:** constituição, princípio II.
- **Versão atual:** uma por vez, marcada automaticamente ao fim de uma importação
  com sucesso; o admin pode trocar (req. 3).
- **Sem cadastro manual de dados do jogo:** princípio II mantido; correções por
  reimportação (req. 17). O admin só cadastra versões à mão (req. 1a).
- **Frequência:** 3 verificações por dia, manhã, tarde e noite (req. 15).
- **Termos de uso:** o site oficial permite a coleta e o uso dos ícones (pergunta 9).
- **Coleta pelo datafeed JSON do site oficial** e decisões de talentos, idioma,
  primeira carga, neutros/bênçãos e regras de nível: ver a discovery
  `dados-do-jogo.md` (req. 7a–7d).
- **Fonte:** site oficial do Dota 2, por coleta automática (req. 8).
- **Patches "letra":** versões separadas (req. 1).
- **Importação:** automática ao detectar versão nova, com reimportação pelo admin
  (req. 15–17).
- **Notícia:** versão nova gera notícia publicada automaticamente (req. 19).

## Discovery

- Fontes, campos e regras da coleta: [discovery/dados-do-jogo.md](../../discovery/dados-do-jogo.md).

## Perguntas em aberto

1. Qual a fonte dos dados do jogo (heróis, habilidades, facetas, talentos,
   itens, ícones)? Precisa de uma discovery: opções, custo, licença de uso das
   imagens e com que rapidez a fonte atualiza após um patch.
   **R:** Do próprio site oficial do Dota 2, com um script de coleta (scraping).
2. Cada patch "letra" (7.37c, 7.37d) é uma versão separada no portal, ou só os
   patches "número" (7.37, 7.38)?
   **R:** São versões separadas.
3. A importação é disparada pelo admin (botão no painel) ou roda sozinha quando
   sai um patch?
   **R:** Automática, e gera uma notícia do admin com os dados da atualização.
4. O admin precisa de uma forma de corrigir um dado importado errado (sem editar à
   mão, ex.: reimportar)?
   **R:** Sim (reimportar), e também de cadastrar manualmente.

### Novas perguntas (surgiram das respostas)

5. **Cadastro manual × constituição.** O princípio II (INEGOCIÁVEL) diz que dados do
   jogo NÃO DEVEM ser cadastrados nem corrigidos à mão, para não divergir do jogo.
   A resposta 4 pede cadastro manual. Opções:
   - (a) manter a constituição: o admin só reimporta, e cadastra **versões** à mão,
     mas não heróis, habilidades, talentos ou itens;
   - (b) alterar a constituição (MAJOR): o admin pode cadastrar e corrigir dados à
     mão, com registro de quem alterou, e a reimportação avisa antes de
     sobrescrever uma correção manual.
   **R:** (a) Manter o princípio II como inegociável.
6. Quem marca a versão nova como **atual**: a importação automática, ao terminar com
   sucesso, ou o admin, depois de conferir os dados?
   **R:** Automático.
7. A notícia da versão nova é criada como **rascunho** para o admin revisar e
   publicar, ou já é publicada sozinha?
   **R:** Publicada sozinha.
8. De quanto em quanto tempo o sistema verifica se saiu versão nova (ex.: a cada
   hora, uma vez por dia)?
   **R:** 3 vezes por dia: manhã, tarde e noite.
9. Coleta no site oficial: os termos de uso do site permitem a coleta e o uso dos
   ícones num portal público?
   **R:** Sim. A discovery `discovery/dados-do-jogo.md` continua necessária para
   mapear de quais páginas vem cada dado (entra no plano 002 da API). 
