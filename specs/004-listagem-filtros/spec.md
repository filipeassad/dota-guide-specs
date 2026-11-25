# 004 — Listagem, filtros e página de herói

## Problema

Com muitas builds publicadas, o jogador precisa achar rápido a build certa: do
herói que vai jogar, na versão atual, na posição dele. Sem filtros, as builds
boas se perdem entre as antigas e as incompletas.

## Objetivos

- O jogador chega à melhor build do herói na versão atual em poucos cliques.
- Logo depois de um patch, a listagem não fica vazia: mostra as builds mais
  novas que existirem.
- Uma busca filtrada pode ser compartilhada por link.
- Cada herói tem uma página própria com as builds dele.
- O usuário guarda as builds de que gostou para achar depois.

## Fora do escopo

- Busca por texto livre dentro das descrições das builds.
- Recomendação personalizada de builds.

## Requisitos

### Listagem (RF07 do PRD)

1. A listagem DEVE mostrar só builds publicadas.
2. Por padrão, a listagem DEVE mostrar builds da **versão atual**, ordenadas pelo
   **ranking** (spec 005).
2a. ENQUANTO a versão atual não tiver nenhuma build publicada, a listagem padrão
    DEVE mostrar as builds da **versão anterior mais recente que tenha builds**,
    com um aviso de que ainda não há builds da versão atual. QUANDO a primeira
    build da versão atual for publicada, a listagem padrão DEVE voltar a mostrar
    a versão atual.
3. Cada item da listagem DEVE mostrar: herói, título, autor, versão (com aviso se
   desatualizada), posições, pontuação, quantidade de comentários (spec 006) e
   data da última atualização.
4. A listagem DEVE carregar **20 builds por vez**, com **carregamento contínuo ao
   rolar a página**: ao chegar perto do fim, as próximas 20 aparecem sozinhas. Como
   fazer está em [discovery/paginacao-continua.md](../../discovery/paginacao-continua.md).
4a. O carregamento contínuo NÃO DEVE repetir nem pular builds quando novas builds
    forem publicadas enquanto a pessoa rola a lista.
4b. QUANDO a pessoa abrir uma build e voltar, a listagem DEVERIA voltar ao mesmo
    ponto da rolagem.

### Filtros e ordenação (RF08)

5. O usuário DEVE poder filtrar por **herói** (busca pelo nome, com ícone).
6. O usuário DEVE poder filtrar por **versão**, com a opção "todas as versões".
7. O usuário DEVE poder filtrar por **posição** (1 a 5).
8. O usuário DEVE poder ordenar por:
   - **mais bem ranqueadas**: maior pontuação primeiro (spec 005);
   - **mais recentes**: última atualização mais recente primeiro (spec 003, req. 22a);
   - **mais votadas na semana**: maior **saldo de votos dos últimos 7 dias**
     primeiro, com a ordem atualizada **de hora em hora** (cálculo em
     [discovery/ranking-semanal.md](../../discovery/ranking-semanal.md)). O card
     continua mostrando a pontuação total, não o saldo da semana;
   - **por versão**: versão mais nova primeiro e, dentro da mesma versão, maior
     pontuação primeiro. É a ordenação padrão quando o filtro é "todas as versões".
8a. Em qualquer ordenação, o **empate** DEVE ser desfeito pela última atualização
    mais recente.
9. Filtros e ordenação DEVEM fazer parte do endereço da página, para que a busca
   possa ser compartilhada e reaberta igual.
10. QUANDO nenhuma build atender aos filtros, o sistema DEVE informar isso e
    sugerir remover um filtro (ex.: ver todas as versões).

### Página de herói (RF10)

11. Cada herói DEVE ter uma página própria, com endereço compartilhável.
12. A página do herói DEVE mostrar as **10 builds mais bem ranqueadas** dele na
    versão atual (com a mesma regra do req. 2a quando não houver builds da versão
    atual), sem carregamento contínuo, e um link para a listagem completa já
    filtrada pelo herói.
13. A página do herói DEVE mostrar a lista de builds por posição, quando houver
    builds para mais de uma posição.

### Navegação

14. O portal DEVE ter uma lista de todos os heróis (com ícone) que leva à página
    de cada herói.

### Favoritas

15. O usuário logado DEVE poder marcar e desmarcar uma build publicada como
    **favorita**.
16. O usuário DEVE ter uma página **"Minhas favoritas"** com as **10 favoritas
    mais bem avaliadas** (maior pontuação primeiro), sem carregamento contínuo, e um
    link "ver todas" que abre a listagem completa só com as favoritas dele, com os
    mesmos filtros, ordenações e carregamento contínuo da listagem.
17. As favoritas de um usuário NÃO DEVEM ser visíveis para outras pessoas e NÃO
    DEVEM afetar a pontuação nem o ranking.
18. QUANDO uma build favorita for excluída ou ocultada pela moderação, ela DEVE
    sumir das favoritas.
19. A build favorita DEVE mostrar a versão com o aviso de desatualizada, como na
    listagem, para o usuário saber que precisa procurar uma build mais nova.

## Critérios de aceite

```
Cenário: listagem padrão
  Dado a versão atual 7.37d
  E builds publicadas em 7.37c e 7.37d
  Quando um visitante abre a listagem sem filtros
  Então vê só as builds de 7.37d, da maior para a menor pontuação

Cenário: versão nova sem builds
  Dado a versão atual 7.37e, sem builds publicadas
  E builds publicadas em 7.37d
  Quando um visitante abre a listagem sem filtros
  Então vê as builds de 7.37d com o aviso de que ainda não há builds da 7.37e
  Quando a primeira build da 7.37e é publicada
  Então a listagem padrão passa a mostrar só as builds da 7.37e

Cenário: ordenação por versão
  Quando o visitante escolhe "todas as versões"
  Então vê primeiro as builds da 7.37d, depois as da 7.37c
  E, dentro de cada versão, da maior para a menor pontuação
  E as de 7.37c aparecem como desatualizadas

Cenário: empate
  Dado duas builds com pontuação 10, uma atualizada ontem e outra hoje
  Quando o visitante ordena por "mais bem ranqueadas"
  Então a atualizada hoje aparece primeiro

Cenário: mais votadas na semana
  Dado uma build com saldo 50 no total, mas 2 nos últimos 7 dias
  E outra build com saldo 10 no total e 8 nos últimos 7 dias
  Quando o visitante ordena por "mais votadas na semana"
  Então a segunda build aparece primeiro

Cenário: carregamento contínuo
  Dado 45 builds que atendem aos filtros
  Quando o visitante abre a listagem
  Então vê 20 builds
  Quando rola até o fim
  Então aparecem mais 20, sem repetir nenhuma
  Quando rola até o fim de novo
  Então aparecem as 5 restantes e a lista indica que acabou

Cenário: build publicada durante a rolagem
  Dado um visitante que já carregou as primeiras 20 builds por "mais recentes"
  Quando outra pessoa publica uma build nova
  E o visitante rola para carregar mais
  Então nenhuma build das 20 primeiras aparece repetida

Cenário: filtro por herói e posição
  Quando o visitante filtra por "Juggernaut" e posição 1
  Então vê só builds de Juggernaut marcadas como posição 1

Cenário: busca compartilhada
  Dado uma busca filtrada por "Pudge", versão 7.37d, ordenada por mais recentes
  Quando o visitante copia o endereço e abre em outro navegador
  Então vê a mesma busca, com os mesmos filtros

Cenário: rascunhos não aparecem
  Dado uma build em rascunho
  Então ela não aparece na listagem nem na página do herói

Cenário: página do herói
  Quando o visitante abre a página de "Invoker"
  Então vê as builds mais bem ranqueadas de Invoker na versão atual

Cenário: sem resultados
  Quando o visitante filtra por um herói sem builds em nenhuma versão
  Então o sistema informa que não há builds

Cenário: favoritar
  Dado um usuário logado numa build publicada
  Quando ele marca a build como favorita
  Então ela aparece em "Minhas favoritas"
  E a pontuação da build não muda

Cenário: favorita excluída
  Dado uma build nas favoritas de "bruno"
  Quando o autor exclui a build
  Então ela some das favoritas de "bruno"

Cenário: visitante tenta favoritar
  Dado um visitante sem login
  Quando ele tenta favoritar uma build
  Então o sistema pede login
```

## Casos de borda

- Herói sem nenhuma build em nenhuma versão.
- Endereço com filtro inválido (herói que não existe, versão que não existe).
- Versão atual sem builds, mas filtro de herói que tem builds só em versões muito
  antigas (req. 2a vale para a listagem padrão; com filtro de herói, usar a mesma
  regra?).
- Build excluída ou ocultada pela moderação enquanto alguém está rolando a listagem.
- Pontuação de uma build muda enquanto a pessoa rola a lista ordenada por ranking.
- Usuário favorita uma build e depois ela fica desatualizada.

## Decisões

- **Padrão da listagem:** versão atual, ordenada pelo ranking (PRD); sem builds da
  versão atual, mostra a versão anterior mais recente com builds (req. 2a).
- **Ordenação por versão:** nova opção, padrão em "todas as versões" (req. 8).
- **Mais votadas na semana:** saldo de votos dos últimos 7 dias (req. 8).
- **Desempate:** última atualização mais recente (req. 8a).
- **Favoritas:** incluídas no MVP, privadas e sem efeito no ranking (req. 15–19).
- **Carregamento contínuo ao rolar, 20 por vez** (req. 4), só na listagem.
- **Página do herói e "Minhas favoritas":** as 10 mais bem avaliadas, com link
  para a listagem completa (req. 12 e 16; discovery `paginacao-continua.md`).
- **Filtros no endereço:** PRD, RF08.

## Perguntas em aberto

1. Na versão atual recém-lançada, a listagem padrão fica vazia. Ela deveria mostrar
   builds da versão anterior até surgirem builds novas?
   **R:** Mostrar a versão anterior até aparecer alguma build da versão nova, com
   uma ordenação por versão (req. 2a e 8).
2. "Mais votadas na semana" considera votos positivos, saldo de votos ou outro critério?
   **R:** Saldo de votos. O cálculo está em `discovery/ranking-semanal.md`.
3. Em caso de empate de pontuação, qual o desempate (mais recente, mais votos)?
   **R:** Mais recente (req. 8a).
4. O usuário pode salvar builds favoritas para achar depois?
   **R:** Sim (req. 15–19).
5. Paginação por páginas numeradas ou carregamento contínuo ao rolar?
   **R:** Contínuo ao rolar. O método está em `discovery/paginacao-continua.md`.
