# 003 — Editor de builds

## Problema

Hoje, quem quer compartilhar uma build usa texto solto em fóruns ou vídeos: sem
estrutura, sem versão e sem garantia de que a build é possível no jogo. O
jogador que lê não sabe a ordem exata das habilidades, por que cada item foi
escolhido nem se a build ainda vale no patch atual.

## Objetivos

- O usuário monta uma build completa (habilidades, itens, talentos e textos)
  em poucos minutos.
- Toda build publicada é possível no jogo, na versão a que pertence
  (constituição, princípio III).
- O leitor entende não só o quê, mas o porquê: o autor pode justificar cada item
  e cada habilidade.

## Fora do escopo

- Exportar a build para dentro do jogo, conforme o PRD.
- Coautoria (mais de um autor por build).
- Histórico de versões da mesma build (desfazer edições).

## Requisitos

### Dados básicos (RF01 do PRD)

1. O usuário DEVE escolher o **herói** e a **versão** da build e dar um **título**.
2. A versão atual DEVE vir pré-selecionada. O usuário PODE escolher outra versão
   cadastrada.
3. O usuário PODE informar uma ou mais **posições** (1 a 5) para a build.
4. Um usuário PODE ter várias builds do mesmo herói na mesma versão, sem limite.
5. Depois de criada, a build NÃO DEVE mudar de herói nem de versão
   (constituição, princípio I). Para outra versão, existe a cópia (req. 20).

### Sequência de níveis: habilidades e talentos (RF02 e RF04)

A ordem de habilidades e os talentos são montados numa **grade única**, como no
modelo abaixo (Undying): uma linha por habilidade, uma linha de talentos e uma
coluna por nível. Em cada nível, o ponto vai para **uma** habilidade **ou** para
**um** talento.

```
CEASELESS DIRGE (INATA)   [ícone]                       ← sem casas: não recebe pontos

DECAY         [1][ ][3][ ][ ][ ][ ][ ][ ][ ][ ][12][13][ ][ ][ ][ ][ ]   Q
SOUL RIP      [ ][ ][ ][4][5][ ][7][8][ ][ ][ ][  ][  ][ ][ ][ ][ ][ ]   W
TOMBSTONE     [ ][2][ ][ ][ ][ ][ ][ ][9][10][11][ ][ ][ ][ ][ ][ ][ ]   E
FLESH GOLEM   [ ][ ][ ][ ][ ][6][ ][ ][ ][ ][ ][  ][  ][14][ ][ ][ ][18]  R
TALENTOS      [ ][ ][ ][ ][ ][ ][ ][ ][ ][ ][ ][  ][  ][ ][15][ ][ ][ ]
               1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18
```

6. O editor DEVE mostrar a grade com:
   - uma **linha por habilidade que recebe pontos**, na ordem do jogo, com a
     ultimate por último;
   - uma linha de **talentos**, abaixo das habilidades;
   - uma **coluna por nível** do herói, a partir do nível 1.
6a. A **habilidade inata** DEVE aparecer acima da grade, com ícone e nome e a marca
    "inata", **sem casas de nível**: ela não recebe pontos. Habilidades dadas pelo
    Aghanim's Scepter ou Shard também não recebem pontos e não entram na grade.
6b. O usuário DEVE escolher, em cada nível, **exatamente uma** casa: uma habilidade
    ou a linha de talentos. A casa escolhida mostra o número do nível.
6c. QUANDO o usuário escolher a linha de talentos num nível, ele DEVE escolher
    também **qual talento** (o nível de talento e uma das duas opções da árvore,
    req. 15).
6d. A sequência DEVE ser preenchida do nível 1 em diante, sem pular níveis, até o
    nível que o autor quiser.
6e. Cada linha de habilidade DEVERIA mostrar a tecla padrão do jogo (Q, W, E, R),
    pela posição da habilidade.
7. O sistema DEVE recusar uma sequência que o jogo não permite na versão da build
   (constituição, princípio III):
   - habilidade acima do nível máximo dela;
   - ultimate antes dos níveis em que o jogo permite;
   - talento antes do nível em que aquele nível de talento libera (ex.: talento de
     nível 10 antes do nível 10 do herói);
   - dois talentos do mesmo nível de talento (só uma das duas opções);
   - ponto em nível que não dá ponto ao herói.
   As regras de nível vêm da versão da build (spec 002 e
   [discovery/dados-do-jogo.md](../../discovery/dados-do-jogo.md), pergunta 2).
7a. No editor, as casas que a regra não permite DEVEM aparecer bloqueadas, com o
    motivo ao passar o mouse ou tocar. QUANDO o autor mudar um nível anterior e
    isso tornar inválidos níveis seguintes, o editor DEVE apontar quais.
8. O usuário PODE escrever um texto sobre cada habilidade (inclusive a inata)
   explicando a importância dela na build.

### Grupos de itens (RF03)

9. O usuário DEVE poder criar grupos de itens com **nome livre** (ex.: "Early
   game", "Situacionais") e em qualquer quantidade.
10. O sistema DEVERIA sugerir nomes prontos: Itens iniciais, Early game, Mid game,
    Late game, Situacionais.
11. O usuário DEVE poder adicionar itens a um grupo buscando pelo nome, e
    reordenar itens dentro do grupo e os próprios grupos.
12. O mesmo item PODE aparecer em mais de um grupo.
13. O usuário PODE escrever uma justificativa para cada item de um grupo.
14. Só itens que existem na versão da build DEVEM ser aceitos.
14a. QUANDO a build tiver **Aghanim's Scepter** ou **Aghanim's Shard** num grupo, a
     página da build DEVE mostrar, junto do item, o que ele muda para o herói da
     build naquela versão (dado vindo da importação, spec 002). Isso aparece
     automaticamente, além da justificativa do autor.
14b. A build DEVE ter uma seção separada, **Neutros e bênçãos**, fora dos grupos
     de itens. Nela o autor PODE escolher itens neutros (agrupados pelo tier) e
     bênçãos, com justificativa opcional para cada um. Cada bênção DEVE ser
     escolhida **com o nível** (ex.: Vast nível 2), entre os níveis que existem
     para ela na versão da build, e a página da build DEVE mostrar os bônus
     daquele nível (ex.: +90 de alcance de ataque). Só neutros e bênçãos da
     versão da build DEVEM ser aceitos, e eles NÃO DEVEM ser aceitos nos grupos de
     itens comuns. A seção não é obrigatória para publicar.

### Árvore de talentos (RF04)

15. O sistema DEVE exibir a árvore de talentos do herói na versão da build, com
    as duas opções de cada nível de talento. É dela que o usuário escolhe o
    talento ao marcar a linha de talentos na grade (req. 6c).
16. Na árvore, o talento escolhido DEVE aparecer destacado, com o nível do herói
    em que foi pego. Níveis de talento ainda não pegos na sequência ficam sem
    destaque.

### Descrição geral (RF05)

17. O usuário PODE escrever uma descrição geral da build (estratégia, como jogar),
    com formatação simples (negrito, listas, links).

### Ciclo de vida (RF06)

18. A build DEVE poder ser salva como **rascunho** (só o autor vê) ou **publicada**
    (todos veem).
19. O autor DEVE poder editar e excluir a própria build (constituição, princípio IV).
20. O autor DEVE poder **copiar** a build para outra versão. A cópia é uma nova
    build em rascunho; a original não muda. Itens, habilidades ou talentos que não
    existem na versão de destino DEVEM ser apontados para o autor corrigir antes
    de publicar.
19a. QUANDO o autor excluir uma build, ela DEVE ficar **oculta para todos**,
     inclusive para o próprio autor: some da listagem, da página do herói, do
     ranking e do endereço direto. Votos e comentários ficam guardados, mas também
     ocultos.
21. Para publicar, a build DEVE ter: **título**, **herói**, **versão**, a
    **sequência de níveis** (req. 6–7) sem erros, **preenchida do nível 1 ao nível
    18**, com **pelo menos um talento**, e **pelo menos um grupo de itens com pelo
    menos um item**. O autor PODE continuar a sequência depois do nível 18. O
    rascunho pode ser salvo incompleto.
22. Todos os textos DEVEM ser exibidos sem executar código (constituição,
    princípio VI) e respeitar os tamanhos máximos (padrão inicial, a reavaliar
    depois de testar com builds reais):

    | Texto | Máximo |
    | --- | --- |
    | Título | 80 caracteres |
    | Nome de grupo de itens | 40 caracteres |
    | Justificativa de item | 500 caracteres |
    | Texto sobre uma habilidade | 500 caracteres |
    | Descrição geral | 5.000 caracteres |

22a. QUANDO o autor editar uma build publicada, a data de **última atualização**
     DEVE mudar, e a ordenação por "mais recentes" (spec 004) DEVE usar essa data.
     Os votos da build continuam valendo.

### Página da build

23. A página de uma build publicada DEVE mostrar: herói, título, autor, versão
    (com aviso se desatualizada), posições, a **grade da sequência de níveis**
    igual à do editor (inata à parte, habilidades, talentos e teclas), a árvore de
    talentos com os escolhidos destacados, os textos sobre as habilidades, os
    grupos de itens com justificativas, a descrição geral, a pontuação e a data
    da última atualização.
24. A página DEVE ter um endereço próprio, compartilhável.

## Critérios de aceite

```
Cenário: criar e publicar build
  Dado um usuário logado e a versão atual 7.37d
  Quando ele cria uma build de Invoker com título, ordem de habilidades,
    um grupo "Early game" com 3 itens e talentos
  E publica
  Então a build aparece publicada, vinculada a 7.37d, com ele como autor

Cenário: ordem de habilidades impossível
  Dado uma build em edição
  Quando o usuário coloca a ultimate no nível 2
  Então o sistema recusa e explica a regra

Cenário: grade da sequência
  Dado uma build de Undying
  Quando o autor abre o editor
  Então vê Ceaseless Dirge acima da grade, marcada como inata e sem casas
  E vê as linhas Decay, Soul Rip, Tombstone e Flesh Golem, nessa ordem
  E uma linha de talentos abaixo delas

Cenário: talento na sequência
  Dado uma build de Undying com os níveis 1 a 14 preenchidos
  Quando o autor marca a linha de talentos no nível 15 e escolhe um talento do nível 10
  Então o nível 15 aparece na linha de talentos
  E o talento escolhido fica destacado na árvore, com "nível 15"
  E nenhuma habilidade recebe ponto no nível 15

Cenário: talento antes da hora
  Quando o autor tenta marcar a linha de talentos no nível 8
  Então a casa está bloqueada e o motivo é exibido

Cenário: dois talentos do mesmo nível
  Dado que o autor já pegou um talento do nível 10
  Quando tenta pegar a outra opção do nível 10 em outro nível do herói
  Então o sistema recusa

Cenário: inata não recebe ponto
  Quando a API recebe uma sequência com ponto em Ceaseless Dirge
  Então a build é recusada

Cenário: mudança que invalida níveis seguintes
  Dado uma sequência com a ultimate nos níveis 6 e 14
  Quando o autor troca o nível 6 para outra habilidade
  Então o editor aponta que o nível 14 ficou inválido

Cenário: item de outra versão
  Dado uma build de 7.37d
  Quando a API recebe um item que não existe em 7.37d
  Então a build é recusada

Cenário: grupo com nome livre
  Quando o usuário cria o grupo "Contra ilusões" com 2 itens e uma justificativa
  Então o grupo aparece na página da build com esse nome, os itens na ordem
    escolhida e a justificativa

Cenário: rascunho
  Dado uma build salva como rascunho
  Quando outro usuário tenta abri-la
  Então ela não é encontrada

Cenário: editar build de outro autor
  Dado uma build de "ana"
  Quando "bruno" tenta editá-la pela API
  Então a API recusa

Cenário: copiar para nova versão
  Dado uma build publicada de 7.37c com um item removido em 7.37d
  Quando o autor copia a build para 7.37d
  Então surge uma nova build em rascunho em 7.37d
  E o item removido é apontado para correção
  E a build de 7.37c continua igual

Cenário: texto com código
  Quando o autor escreve "<script>alert(1)</script>" na justificativa de um item
  Então a página exibe o texto como texto, sem executar nada

Cenário: publicar sem chegar ao nível 18
  Dado um rascunho com a sequência preenchida até o nível 16
  Quando o autor tenta publicar
  Então o sistema recusa e informa que a sequência precisa ir até o nível 18

Cenário: publicar sem talento
  Dado um rascunho com a sequência até o nível 18 só com habilidades
  Quando o autor tenta publicar
  Então o sistema recusa e informa que falta pelo menos um talento

Cenário: publicar com sequência inválida
  Dado um rascunho com título e grupos de itens
  E uma sequência com o nível 7 vazio
  Quando o autor tenta publicar
  Então o sistema recusa e aponta o nível que falta
  E o rascunho continua salvo

Cenário: neutros e bênçãos
  Quando o autor adiciona um item neutro de tier 2 e uma bênção à build
  Então eles aparecem na seção "Neutros e bênçãos", separados dos grupos de itens
  E o neutro aparece agrupado pelo tier

Cenário: bênção com nível
  Dado que a bênção Vast tem os níveis 1, 2 e 3 na versão da build
  Quando o autor escolhe Vast nível 2
  Então a página da build mostra Vast nível 2 com os bônus desse nível

Cenário: nível de bênção que não existe
  Quando a API recebe Vast nível 5
  Então a build é recusada

Cenário: neutro num grupo comum
  Quando a API recebe um item neutro dentro do grupo "Early game"
  Então a build é recusada

Cenário: Aghanim's Scepter na build
  Dado uma build de Juggernaut com Aghanim's Scepter no grupo "Late game"
  Quando alguém abre a build
  Então vê, junto do item, o que o Scepter muda para Juggernaut naquela versão

Cenário: excluir build
  Dado uma build publicada de "ana" com votos e comentários
  Quando "ana" exclui a build
  Então ela some da listagem, do ranking e do endereço direto
  E nem "ana" consegue mais abri-la

Cenário: várias builds do mesmo herói
  Dado que "ana" já tem uma build de Pudge em 7.37d
  Quando ela cria outra build de Pudge em 7.37d
  Então a segunda build é criada normalmente

Cenário: editar build publicada
  Dado uma build publicada em 01/11 com 10 votos
  Quando o autor a edita em 20/11
  Então a última atualização passa a ser 20/11
  E os 10 votos continuam valendo

Cenário: texto acima do limite
  Quando o autor escreve um título com 81 caracteres
  Então o sistema recusa e informa o máximo de 80
```

## Casos de borda

- Herói com regras especiais de habilidades (ex.: mais de 4 habilidades evoluíveis,
  ultimate que evolui de outro jeito).
- Grupo de itens vazio no momento de publicar.
- Autor edita uma build publicada: ela continua publicada durante a edição?
- Cópia para uma versão em que o herói não existe mais.
- Herói cujo Aghanim's Scepter ou Shard não muda nada (ou ainda não tem efeito
  coletado).

## Decisões

- **Grupos de itens com nome livre e justificativas opcionais:** conforme o PRD (RF03).
- **Build não muda de versão; para outra versão, cópia:** constituição, princípio I.
- **Sem faceta:** as facetas foram descontinuadas no jogo; o portal só trabalha com
  versões sem facetas.
- **Sequência de níveis em grade única** (habilidades + talentos, inata à parte),
  seguindo o modelo enviado pelo produto (req. 6–7a).
- **Mínimo para publicar:** título, sequência sem erros do nível 1 ao 18 com pelo
  menos um talento, e um grupo com um item (req. 21).
- **Aghanim's Scepter e Shard:** a página mostra o efeito para o herói (req. 14a).
- **Neutros e bênçãos:** seção própria, opcional, fora dos grupos; bênção escolhida
  com o nível (req. 14b).
- **Tamanhos de texto:** padrão inicial do req. 22, a reavaliar.
- **Várias builds do mesmo herói e versão:** permitidas, sem limite (req. 4).
- **Excluir:** oculta para todos, inclusive o autor (req. 19a).
- **Editar publicada:** muda a última atualização; votos continuam (req. 22a).

## Perguntas em aberto

1. A build deve registrar a faceta do herói? Ela é obrigatória quando o herói tem facetas?
   **R:** As facetas foram descontinuadas; as builds são criadas a partir das
   versões novas, sem faceta.
2. Qual o mínimo para publicar: ordem de habilidades completa, pelo menos um grupo
   de itens, talentos? (req. 21)
   **R:** Ordem de habilidades, um grupo de itens, talentos e título.
3. A build deve registrar itens de Aghanim (Scepter/Shard) de forma especial ou
   eles são itens comuns dentro dos grupos?
   **R:** Mostrar junto do item o que ele muda para o herói da build (req. 14a).
4. Qual o tamanho máximo de cada texto (título, justificativa, descrição geral)?
   **R:** Usar um padrão e reavaliar depois (req. 22).
5. Um usuário pode ter várias builds do mesmo herói na mesma versão? Há limite?
   **R:** Sim, sem limite.
6. Ao excluir uma build, ela some de vez ou fica oculta (por causa dos votos e
   comentários)?
   **R:** Fica oculta, inclusive para o dono.
7. Editar uma build publicada altera a data de "última atualização" exibida e afeta
   o ranking?
   **R:** Sim (req. 22a).

### Nova pergunta (surgiu do modelo da grade)

8. Para publicar, até que nível a sequência precisa ir? A resposta 2 pedia ordem
   de habilidades e talentos, e eu tinha registrado "todos os níveis que dão
   ponto e um talento em cada nível de talento". O modelo da grade vai até o
   nível 18 com um talento só. Opções:
   - (a) até o nível 18 (como no modelo), com pelo menos um talento;
   - (b) até o último nível que dá ponto, com os 4 níveis de talento;
   - (c) até onde o autor quiser, desde que comece no nível 1 sem pular níveis.
   **R:** (a) Até o nível 18, com pelo menos um talento (req. 21). 
