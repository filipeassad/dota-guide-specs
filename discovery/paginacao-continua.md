# Discovery: carregamento contínuo da listagem de builds

> **Status:** decidida | **Relacionado a:** spec 004 (req. 4,
> 4a, 4b e 8), [ranking-semanal.md](ranking-semanal.md) | **Princípios afetados:**
> IX (a API serve web e mobile), VIII (operação enxuta)
> **Última atualização:** 2026-11-24

## 1. Pergunta que este discovery responde

A listagem de builds carrega **20 builds por vez, ao rolar a página** (spec 004,
pergunta 5). Como a API entrega as builds aos poucos e como o web dispara o
próximo carregamento, sem repetir nem pular builds quando alguém publica uma
build nova no meio da rolagem (req. 4a)?

## 2. Duas partes do problema

1. **API:** como pedir "as próximas 20" (paginação).
2. **Web:** quando pedir, como mostrar e como voltar ao mesmo ponto depois de
   abrir uma build (req. 4b).

## 3. Paginação na API

### Opções

| Opção | Como funciona | Build nova no meio da rolagem | Desempenho em listas longas |
| --- | --- | --- | --- |
| **A. Deslocamento** ("pule 40, traga 20") | A API conta posições a partir do início | Empurra tudo uma posição: a build 40 aparece de novo na página seguinte (**repete**). Uma exclusão faz **pular** uma | Piora quanto mais fundo: o banco percorre e descarta todas as linhas anteriores |
| **B. Cursor** ("traga 20 depois desta build") | A API devolve, junto das 20 builds, um cursor que marca a última build entregue; o próximo pedido continua dali | Não afeta: a build nova entra antes do cursor e não empurra nada | Constante: o banco vai direto ao ponto pelo índice |

**Escolha: B (cursor).** É o único método que atende o req. 4a, e é o padrão de
feeds com rolagem contínua. A desvantagem do cursor (não dá para "pular para a
página 7") não importa aqui, porque a interface não tem páginas numeradas.

### Como o cursor funciona

O cursor guarda os valores de ordenação da última build entregue, sempre
terminando no identificador da build para que nunca existam dois itens iguais:

| Ordenação (spec 004, req. 8) | Chave do cursor (nesta ordem) |
| --- | --- |
| Mais bem ranqueadas | pontuação ↓, última atualização ↓, id ↓ |
| Mais recentes | última atualização ↓, id ↓ |
| Mais votadas na semana | saldo da semana ↓, última atualização ↓, id ↓ |
| Por versão | ordem da versão ↓, pontuação ↓, última atualização ↓, id ↓ |

A "última atualização" já é o desempate pedido no req. 8a; o id só garante a
ordem total.

Contrato proposto (detalhado no plano da API):

```
GET /v1/builds?hero=pudge&version=7.37d&sort=ranking&limit=20
→ { "items": [ ...20 builds... ], "nextCursor": "eyJzIjoxMCwidSI6Ii4uLiJ9" }

GET /v1/builds?hero=pudge&version=7.37d&sort=ranking&limit=20&cursor=eyJzIjoxMCwidSI6Ii4uLiJ9
→ { "items": [ ...próximas 20... ], "nextCursor": null }   ← null = acabou
```

Regras:

- O cursor é **opaco** para o cliente (texto codificado): o cliente só devolve o
  que recebeu. Assim a API pode mudar o formato sem quebrar clientes (princípio IX).
- O cursor carrega também os filtros e a ordenação com que foi criado. Se o pedido
  vier com filtros diferentes, a API recusa e o cliente recomeça do início.
- `limit` padrão de 20, máximo de 50.
- Funciona igual para um futuro app mobile (princípio IX).

### O que o cursor não resolve sozinho

Na ordenação por **pontuação**, um voto pode mudar a posição de uma build entre um
carregamento e outro, e ela pode aparecer de novo ou ser pulada. Mitigações:

- O web **descarta builds repetidas** pelo id antes de mostrar.
- Em **mais votadas na semana**, o saldo é recalculado de hora em hora
  ([ranking-semanal.md](ranking-semanal.md)) e fica parado durante a rolagem.
- Em **mais bem ranqueadas**, uma build que sobe de posição durante a rolagem
  pode ser pulada. É aceitável: ela já estava acima e a pessoa provavelmente a
  viu. Congelar a pontuação total custaria a atualização na hora (spec 005, req. 11).

## 4. Carregamento no web

### Quando carregar

- Um elemento invisível ("sentinela") fica no fim da lista. Quando ele chega a
  **cerca de uma tela de distância** do fim visível, o web pede as próximas 20.
  O navegador avisa isso nativamente (Intersection Observer), sem escutar cada
  movimento de rolagem.
- Só um pedido por vez: enquanto um carregamento está em andamento, a sentinela
  não dispara outro.
- Com `nextCursor` nulo, a lista mostra "Você chegou ao fim" e para de observar.

### Botão "Carregar mais" como reserva

Sempre que o carregamento automático não servir, aparece um botão **"Carregar
mais"** no fim da lista:

- falha de rede: mostra o erro e o botão para tentar de novo;
- navegação por teclado ou leitor de tela, que não "rolam" como o mouse;
- navegador sem suporte ao aviso de rolagem.

### Voltar ao mesmo ponto (req. 4b)

- As builds já carregadas e o último cursor ficam guardados em memória, ligados ao
  endereço da busca.
- Ao voltar de uma build, a listagem é montada com o que já estava carregado e a
  rolagem volta para a posição salva, sem pedir tudo de novo à API.
- O endereço guarda só os filtros e a ordenação (spec 004, req. 9), não o cursor.
  Um link compartilhado sempre abre a busca do início.

### Acessibilidade e rodapé

- Builds novas carregadas são anunciadas a leitores de tela ("20 builds carregadas").
- Com rolagem infinita o rodapé nunca é alcançado; links importantes do rodapé
  também ficam no menu.

### O que fica de fora por enquanto

- **Lista virtualizada** (manter na tela só os cards visíveis): só é necessária
  com centenas de cards abertos ao mesmo tempo. Reavaliar se a listagem ficar
  lenta em celulares.

## 5. Recomendação

1. API com paginação por **cursor opaco**, 20 por vez, ordem total terminando no id.
2. Web com **sentinela no fim da lista** + botão **"Carregar mais"** como reserva.
3. Builds repetidas descartadas pelo id no web.
4. Volta ao mesmo ponto com as builds carregadas guardadas em memória.
5. Custo: zero. Nada de infraestrutura nova (princípio VIII).

## 6. Perguntas em aberto

- A página do herói (spec 004, req. 12) e "Minhas favoritas" (req. 16) também usam
  carregamento contínuo, ou mostram um número fixo de builds com link para a
  listagem completa?
  **R:** Usam um número fixo de builds, as mais bem avaliadas (padrão inicial:
  **10**), com link para a listagem completa, que tem o carregamento contínuo.

## 7. Próximos passos

- [x] Revisar e aprovar a recomendação
- [ ] No `plan.md` da 004 na `dota-guide-api`: parâmetros `cursor`/`limit`, formato
      do cursor, índices de cada ordenação e testes (build nova no meio, fim da lista,
      cursor com filtro diferente)
- [ ] No `plan.md` da 004 no `dota-guide-web`: sentinela, botão de reserva, cache da
      listagem e restauração da rolagem

## Referências

- [MDN — Intersection Observer API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)
- [Use the Index, Luke — Paginação por keyset ("seek method")](https://use-the-index-luke.com/no-offset)
