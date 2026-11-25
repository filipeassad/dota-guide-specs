# Discovery: coleta dos dados do jogo no site oficial

> **Status:** decidida | **Relacionado a:** spec 002 (req. 7–11,
> 15–19), spec 003 (req. 7, 14, 14a, 15) | **Princípios afetados:** II (dados
> versionados de fonte externa), III (build respeita o jogo), VIII (operação enxuta)
> **Última atualização:** 2026-11-24 (endpoints consultados nesta data)

## 1. Pergunta que este discovery responde

A spec 002 decidiu que os dados do jogo vêm do **site oficial do Dota 2**, por
coleta automática. De onde, exatamente, vem cada dado (versões, heróis,
habilidades, talentos, efeitos de Aghanim, itens e ícones), como detectar uma
versão nova e o que fazer quando o site mudar?

## 2. Achado principal: não precisa raspar HTML

As páginas do dota2.com são montadas no navegador a partir de um conjunto de
endereços que devolvem **JSON** (`https://www.dota2.com/datafeed/...`). A coleta
lê esse JSON direto, em vez de interpretar o HTML das páginas:

- os dados já vêm estruturados (ids numéricos, níveis, custos);
- muda muito menos que o layout das páginas;
- são poucos pedidos por importação.

**Ressalva:** esses endereços **não são uma API documentada** pela Valve. Podem
mudar ou sair do ar sem aviso. A coleta precisa detectar isso (seção 7).

## 3. Mapa: de onde vem cada dado

Todos responderam HTTP 200 em 2026-11-24. `language=english` em todos (ver
pergunta 5 sobre português).

| Dado (spec 002, req. 7) | Endereço | Onde está no JSON |
| --- | --- | --- |
| **Versões** | `/datafeed/patchnoteslist?language=english` | `patches[]`: `patch_number` (ex.: "7.42"), `patch_timestamp` (segundos Unix). Última hoje: **7.42**, de ~2026-11-18 |
| **Notas de uma versão** | `/datafeed/patchnotes?version=7.42&language=english` | `general_notes[]`, `items[]` (por `ability_id`), `heroes[]` (por `hero_id`, com `abilities[]` e `talent_notes[]`) |
| **Lista de heróis** | `/datafeed/herolist?language=english` | `result.data.heroes[]`: `id`, `name` (`npc_dota_hero_juggernaut`), `name_loc` ("Juggernaut"), `primary_attr`. **127 heróis** |
| **Herói completo** (habilidades, talentos, Aghanim) | `/datafeed/herodata?language=english&hero_id=8` | `result.data.heroes[0]`: `abilities[]`, `talents[]`, `facets[]` (vazio) |
| **Lista de itens** | `/datafeed/itemlist?language=english` | `result.data.itemabilities[]`: `id`, `name` (`item_blink`), `name_loc`, `recipes`, `neutral_item_tier`. Mistura itens, receitas e entradas internas |
| **Item completo** (custo etc.) | `/datafeed/itemdata?language=english&item_id=108` | `result.data.items[0]`: `item_cost`, `desc_loc`, `item_neutral_tier`... |
| **Ícone de herói** | `cdn.cloudflare.steamstatic.com/apps/dota2/images/dota_react/heroes/juggernaut.png` | Nome = `name` do herói sem `npc_dota_hero_` |
| **Ícone de habilidade** | `.../dota_react/abilities/juggernaut_blade_fury.png` | Nome = `name` da habilidade |
| **Ícone de item** | `.../dota_react/items/ultimate_scepter.png` | Nome = `name` do item sem `item_` |

`/datafeed/abilitylist` existe, mas traz quase só habilidades internas (courier,
Roshan, eventos). As habilidades dos heróis vêm do `herodata`.

### Dentro do `herodata` (exemplo: Juggernaut, id 8)

| Campo | O que é | Uso no portal |
| --- | --- | --- |
| `abilities[].id`, `name`, `name_loc`, `desc_loc` | Identidade e texto da habilidade | Identificador estável (spec 002, req. 9) e exibição |
| `abilities[].max_level` | Nível máximo (ex.: 4) | Validar a ordem de habilidades (spec 003, req. 7) |
| `abilities[].type` | 0 nas habilidades comuns, **1 na Omnislash** (ultimate) | Identificar a ultimate. **Inferido**, a confirmar em mais heróis |
| `abilities[].ability_is_innate` | Habilidade inata (ex.: Bladeform) | **Não recebe pontos**: fica fora da ordem de habilidades |
| `abilities[].ability_has_scepter` / `ability_has_shard` | A habilidade muda com Aghanim | Efeito de Aghanim por herói (spec 003, req. 14a) |
| `abilities[].scepter_loc` / `shard_loc` | Texto do efeito (ex.: "Allows Juggernaut to attack his enemies during Blade Fury even while moving...") | Texto exibido junto do item na build |
| `abilities[].special_values[].values_scepter` / `values_shard` | Números do efeito | Preencher os valores no texto |
| `abilities[].ability_is_granted_by_scepter` / `_by_shard` | Habilidade nova dada pelo Aghanim | Também é efeito de Aghanim; não entra na ordem de habilidades |
| `talents[]` | **8 talentos** com `id`, `name`, `name_loc` (ex.: "-{s:bonus_AbilityCooldown}s Healing Ward Cooldown") | Árvore de talentos (spec 003, req. 15) |
| `facets[]`, `facet_abilities[]` | Vazios | Confirma que as facetas não são mais usadas (spec 003, pergunta 1) |

## 4. Lacunas: o que o site não entrega pronto

| Lacuna | Impacto | Proposta |
| --- | --- | --- |
| **Talentos sem nível.** Os 8 talentos não dizem se são do nível 10, 15, 20 ou 25, nem qual é o par | Sem isso não dá para montar a árvore (spec 003, req. 15) | Ordem do array, **confirmada** (pergunta 1): pares (0,1) = nível 10, (2,3) = 15, (4,5) = 20, (6,7) = 25. Se um herói vier com outra quantidade de talentos, a importação falha (req. 16) em vez de gravar errado |
| **Textos com marcadores.** `name_loc` e `scepter_loc` vêm com marcadores como `{s:bonus_AbilityCooldown}` | Texto ilegível se exibido cru | Substituir pelos valores de `special_values` na importação; se sobrar marcador, a importação falha para aquele herói |
| **Regras de nível do jogo.** Nível máximo do herói, em quais níveis ganha ponto, níveis permitidos da ultimate **não estão** no datafeed | Necessárias para recusar builds impossíveis (princípio III) | Guardar essas regras **por versão** junto com os dados importados, preenchidas a partir de uma tabela mantida no código da importação e revisada a cada versão nova. Ver pergunta 2 (toca o princípio II) |
| **Heróis com regras especiais** (ex.: ultimate que evolui diferente, habilidades sem pontos) | Validação genérica pode recusar builds válidas | Lista de exceções por herói na mesma tabela de regras |
| **Sem histórico.** O datafeed só mostra o estado **atual** do jogo. Não dá para pedir "os dados da 7.41f" depois que a 7.42 saiu | Versões passadas só existem no portal se foram importadas na época | Importar logo que a versão sai (3 verificações por dia, spec 002, req. 15). A primeira carga do portal terá só a versão atual |
| **Hotfix sem número novo.** A Valve às vezes ajusta valores sem mudar o número da versão | Dados da versão atual ficam diferentes do jogo | A cada verificação, comparar uma "impressão digital" (hash) dos dados da versão atual; se mudou, reimportar a versão atual e avisar o admin |

**Atualização (plano 002 da API, T1):** a conferência com os 127 heróis refinou esta
seção. A grade usa também o campo `behavior`. Talentos genéricos precisam de
`/datafeed/abilitydata`. Uns 20 textos não têm o valor na fonte, e o `itemlist` não diz o que
está à venda. Ver a seção 10 do plano 002 da API.

## 5. Como detectar uma versão nova

Nas 3 verificações diárias (spec 002, req. 15):

```
1. Ler /datafeed/patchnoteslist (1 pedido)
2. Pegar o maior patch_number
3. Já existe no portal?
   ├─ não → importar a versão nova (seção 6)
   └─ sim → é a versão atual? comparar o hash dos dados (hotfix, seção 4)
```

**Risco de tempo:** a lista pode mostrar "7.43" minutos antes de os dados dos heróis
serem atualizados. Mitigação: a importação compara a versão nova com a anterior;
se **nada** mudou em heróis e itens, ela não marca a versão como atual e tenta de
novo na próxima verificação (no máximo um dia, avisando o admin se persistir).

## 6. Fluxo de uma importação

```
1. herolist ............................... 1 pedido  → 127 heróis
2. herodata de cada herói ................. 127 pedidos
3. itemlist ............................... 1 pedido  → filtrar itens compráveis
4. itemdata de cada item .................. ~200–300 pedidos (a confirmar)
5. ícones novos ou alterados .............. baixar e guardar no armazenamento do portal
6. validar tudo (seção 7)
7. gravar a versão inteira numa transação → marcar como atual (spec 002, req. 3)
8. criar a notícia da versão nova (spec 002, req. 19) a partir de /datafeed/patchnotes
```

- **Ritmo:** 1 pedido por segundo, com identificação genérica do cliente. Cerca de
  5–8 minutos por importação. Só acontece quando sai versão nova ou hotfix.
- **Ícones copiados, não apontados:** guardar uma cópia no armazenamento do portal
  (o mesmo dos GIFs e das notícias, specs 006 e 007). Se a Valve trocar ou apagar
  um ícone, builds antigas continuam com o ícone da versão delas (princípio I).
- **Identificador estável** (spec 002, req. 9): usar os `id` numéricos do datafeed
  (herói 8, habilidade 5028, item 108). São os mesmos entre versões.
- **Notícia da versão nova:** o `/datafeed/patchnotes` traz as mudanças por herói e
  por item, já ligadas aos mesmos ids. Dá para montar a notícia com heróis e itens
  alterados e o link para as notas oficiais.

## 7. Validação antes de gravar (spec 002, req. 16)

A importação **não grava nada** e avisa o admin se:

- a resposta não for JSON ou faltar um campo esperado (o site mudou o formato);
- algum herói vier sem habilidades, sem ultimate ou sem exatamente 8 talentos;
- algum texto exibido ainda tiver marcadores `{s:...}` não substituídos;
- a quantidade de heróis ou itens cair mais de 5% em relação à versão anterior
  (sinal de resposta parcial);
- uma reimportação remover um talento, habilidade ou item **usado por builds**
  daquela versão (spec 002, req. 10): nesse caso, o item fica marcado como
  "removido" em vez de apagado, e o admin é avisado.

Testes automatizados (princípio VII): guardar respostas reais do datafeed como
exemplo e testar a importação com elas, incluindo respostas com campo faltando.

## 8. Custo e infraestrutura (princípio VIII)

- Datafeed e ícones: gratuitos.
- Agendador: o mesmo das 3 verificações diárias e do recálculo semanal
  ([ranking-semanal.md](ranking-semanal.md)).
- Armazenamento: ícones de ~127 heróis, ~600 habilidades e ~300 itens, poucos KB
  cada (alguns MB por versão). Mesmo armazenamento de arquivos das specs 006 e 007.

## 9. Recomendação

1. Coletar pelo **datafeed JSON** do dota2.com, não pelo HTML das páginas.
2. Detectar versão nova pelo `patchnoteslist` e hotfix pelo hash dos dados.
3. Importar a versão inteira, validar e gravar de uma vez; falhou, não grava nada.
4. Copiar os ícones para o armazenamento do portal.
5. Talentos por ordem do array, em pares de 10 a 25 (confirmado).
6. Regras de nível do jogo escritas no código do portal (pergunta 2b).
7. Itens neutros e bênçãos coletados à parte (seção 10.1).
8. Textos do jogo em inglês, guardados com o idioma (seção 10.2).

## 10. Perguntas em aberto

1. **Ordem dos talentos:** confirmar em 3–4 heróis que os pares do array
   correspondem aos níveis 10, 15, 20 e 25.
   **R:** Confirmado: a árvore começa de baixo para cima, nível 10 com duas
   opções, depois 15, 20 e 25. Os pares do array seguem essa ordem: (0,1) = 10,
   (2,3) = 15, (4,5) = 20, (6,7) = 25. Na tela, a árvore é desenhada com o nível
   10 embaixo.
2. **Regras de nível × princípio II.**
   **R:** "Não entendi." Reescrita abaixo (pergunta 2b).
2b. **Quem escreve as regras de subir de nível?** O site oficial entrega os
    heróis, as habilidades e os talentos, mas **não** entrega as regras do tipo:
    - "a ultimate só pode receber ponto nos níveis 6, 12 e 18";
    - "talento só pode ser pego a partir do nível 10 (depois 15, 20, 25)";
    - "em tal nível o herói não ganha ponto".

    O portal precisa dessas regras para bloquear casas na grade da sequência
    (spec 003, req. 7). Como o site não as fornece, **nós escrevemos essas regras
    no código** do portal e as revisamos quando sair uma versão que mude alguma
    delas. Isso não fere o princípio II (que proíbe cadastrar heróis, habilidades,
    talentos e itens à mão), porque são regras do jogo, não dados de heróis.
    **Pode ser assim?** (sim / não)
    **R:** Sim.
3. **Primeira carga:** o portal começa só com a versão atual (7.42)?
   **R:** Não faz sentido colocar versões anteriores. O portal começa só com a
   versão atual. (Observação: o datafeed só mostra o estado atual do jogo, então
   versões anteriores não poderiam ser coletadas de qualquer forma.)
4. **Quais itens entram:** só os compráveis na loja, ou também itens neutros?
   **R:** Os itens neutros aparecem, e as bênçãos também, numa seção separada da
   build. Ver seção 10.1.
5. **Idioma:** inglês ou português?
   **R:** Inglês por enquanto, com a possibilidade de adicionar português no futuro.
   Ver seção 10.2.

6. **Nível da bênção na build:** cada bênção tem valores diferentes por nível
   (ex.: Vast dá +60, +90 ou +120 de alcance). Na seção "Neutros e bênçãos" da
   build, o autor escolhe **só a bênção**, ou a bênção **e o nível** dela?
   **R:** A bênção com o nível. Os níveis que podem ser escolhidos são os que
   existem nos valores da bênção (ex.: 3 valores = níveis 1, 2 e 3).

### 10.1 Itens neutros e bênçãos na coleta

- O `itemlist` já marca itens neutros com `neutral_item_tier` (−1 = não é neutro;
  0 ou mais = tier do neutro). A importação guarda o tier de cada neutro.
- A build ganha uma seção própria para neutros e bênçãos (spec 003, req. 14b).

**Bênçãos localizadas (consulta de 2026-11-24).** No jogo, os neutros se dividem
em **artefatos** (o item neutro em si) e **encantamentos** (o que chamamos de
bênçãos). No datafeed:

| Pergunta | Achado |
| --- | --- |
| Onde estão | No mesmo `itemlist`, como itens comuns cujo `name` começa com **`item_enhancement_`** (ex.: `item_enhancement_vast`, "Vast"). **27** bênçãos hoje |
| Como separar de artefato | **Só pelo prefixo do nome.** Artefatos têm `neutral_item_tier` de 0 a 3 (cerca de 12 por tier); bênçãos vêm com `neutral_item_tier` −1, igual a itens comuns |
| Detalhes | `itemdata` da bênção (ex.: id 1576) traz `desc_loc` vazio e os bônus em `special_values`, com **um valor por nível** (ex.: alcance de ataque 60 / 90 / 120, dano 6 / 10 / 14, armadura −4 / −6 / −8) |
| Em quais tiers cada bênção existe | **Não vem como campo.** As notas de patch falam disso em texto ("Tiers changed from 1 to 1-2") |
| Ícone | `.../dota_react/items/enhancement_vast.png` (nome sem `item_`): responde PNG |
| Português | `itemlist?language=brazilian` funciona: "Blink Dagger" vira "Adaga de Translocação", "Vast" vira "Grandeza" (útil para a pergunta 5, no futuro) |

Regras para a importação:

- **Artefato:** item com `neutral_item_tier` ≥ 0. O tier do jogo é o valor + 1
  (0 = Tier 1, 3 = Tier 4), a confirmar no plano.
- **Bênção:** item cujo nome começa com `item_enhancement_`.
- A descrição da bênção é montada a partir dos `special_values` (o `desc_loc` vem
  vazio), mostrando os valores de cada nível.
- Se aparecer um item com tier de neutro que não seja artefato nem bênção, ou se
  o prefixo mudar, a importação falha e avisa o admin (seção 7).

**Ressalva:** parte do `itemlist` (~140 mil caracteres) foi lida por resumo, não
na íntegra. A contagem exata de bênçãos e artefatos deve ser conferida na
primeira tarefa do plano 002, com o arquivo completo.

### 10.2 Idioma

- Os textos do jogo (nomes e descrições de heróis, habilidades, talentos e itens)
  são coletados com `language=english`.
- Cada texto é guardado **com o idioma** a que pertence, para que adicionar
  português depois seja só mais uma coleta (`language=brazilian`, a testar), sem
  mudar a estrutura dos dados nem as builds.
- A interface do portal (menus, botões, mensagens) continua em português.

## 11. Próximos passos

- [x] Responder as perguntas da seção 10
- [x] Confirmar a ordem dos talentos
- [x] Localizar as bênçãos no datafeed (seção 10.1)
- [x] Responder a pergunta 6 (nível da bênção na build)
- [x] Registrar as decisões na spec 002 e na constituição
- [ ] No `plan.md` da 002 na `dota-guide-api`: módulo de importação, tabelas por
      versão, idioma dos textos, neutros e bênçãos, regras de nível, agendador,
      hash de hotfix, cópia de ícones e testes com respostas reais

## Referências (consultadas em 2026-11-24)

- https://www.dota2.com/datafeed/patchnoteslist?language=english
- https://www.dota2.com/datafeed/patchnotes?version=7.42&language=english
- https://www.dota2.com/datafeed/herolist?language=english
- https://www.dota2.com/datafeed/herodata?language=english&hero_id=8
- https://www.dota2.com/datafeed/itemlist?language=english
- https://www.dota2.com/datafeed/itemdata?language=english&item_id=108
- https://www.dota2.com/datafeed/abilitylist?language=english
- https://cdn.cloudflare.steamstatic.com/apps/dota2/images/dota_react/heroes/juggernaut.png
