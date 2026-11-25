# 006 — Comentários em builds

## Problema

O voto diz se a build é boa, mas não diz por quê. Leitores têm dúvidas ("por que
este item antes daquele?") e sugestões ("contra heróis mágicos, troque X por Y"),
e o autor não tem onde responder nem como melhorar a build com a comunidade.

## Objetivos

- Leitores tiram dúvidas e sugerem ajustes diretamente na build.
- O autor responde e melhora a build a partir dos comentários.
- Conteúdo abusivo pode ser removido rapidamente pelo admin, com ajuda das
  denúncias dos usuários.

## Fora do escopo

- Votos em comentários.
- Notificações (e-mail ou no portal) de novos comentários.
- Menções a usuários (@nome).
- Imagens que não sejam GIF (fotos, prints) e vídeos.

## Requisitos

### Comentar (RF14 do PRD)

1. Qualquer visitante DEVE poder ler os comentários de uma build publicada.
2. Só usuários logados, com e-mail confirmado, DEVEM poder comentar (spec 001).
3. O comentário DEVE ser texto, com até **2.000 caracteres**, exibido sem
   executar código (constituição, princípio VI).
3a. Links no texto DEVEM ser clicáveis e abrir em outra aba.
3b. O comentário PODE ter **um GIF**, adicionado de duas formas:
    - colando o **link** de um GIF;
    - **enviando o arquivo** GIF no formulário do comentário (até **5 MB**).
    O GIF DEVE ser exibido como imagem animada dentro do comentário. Só o formato
    GIF DEVE ser aceito.
3c. O GIF exibido NÃO DEVE mudar depois de publicado (ex.: se o site de origem
    trocar o arquivo do link). Como garantir isso fica no plano.
4. O usuário DEVE poder responder a um comentário. Respostas têm **um nível só**:
   não é possível responder a uma resposta criando um novo nível (a resposta vai
   para o mesmo comentário principal).
5. Os comentários DEVEM ser exibidos do mais antigo para o mais recente, com as
   respostas logo abaixo do comentário principal, em páginas de **20 comentários**.
6. Comentários do autor da build DEVEM ter destaque visual.
7. Builds em rascunho NÃO DEVEM receber comentários.
8. A frequência de comentários por usuário DEVE ter limite, para impedir spam.

### Editar, excluir e moderar (RF15)

9. O autor do comentário DEVE poder editá-lo (texto e GIF) **a qualquer momento**,
   sem prazo. Um comentário editado DEVE exibir a marca "editado".
10. O autor do comentário DEVE poder excluí-lo (constituição, princípio IV).
11. **Só o admin** DEVE poder excluir comentários de outras pessoas. O autor da
    build NÃO DEVE poder excluir comentários de outros na própria build.
12. QUANDO um comentário com respostas for excluído, ele DEVE aparecer como
    "comentário removido", mantendo as respostas visíveis. Sem respostas, ele
    some da lista.
13. A quantidade de comentários da build DEVE aparecer na listagem (spec 004).

### Denúncias

14. O usuário logado DEVE poder **denunciar** um comentário de outra pessoa, com
    um motivo opcional (até 500 caracteres).
15. Cada usuário DEVE poder denunciar o mesmo comentário **uma vez só**.
16. O admin DEVE ter uma lista de comentários denunciados, com a quantidade de
    denúncias e os motivos, do mais denunciado para o menos denunciado.
17. Para cada comentário denunciado, o admin DEVE poder **excluir** o comentário
    (req. 11) ou **descartar** as denúncias, mantendo o comentário.
18. Quem denunciou NÃO DEVE ser identificado para o autor do comentário.

## Critérios de aceite

```
Cenário: comentar
  Dado um usuário logado numa build publicada
  Quando ele escreve um comentário
  Então o comentário aparece no fim da lista, com o nome dele e a data

Cenário: visitante tenta comentar
  Dado um visitante sem login
  Quando ele tenta comentar
  Então o sistema pede login

Cenário: responder
  Dado um comentário de "bruno"
  Quando "ana", autora da build, responde
  Então a resposta aparece abaixo do comentário de "bruno"
  E a resposta de "ana" tem o destaque de autora da build

Cenário: resposta a uma resposta
  Dado uma resposta de "ana" a um comentário de "bruno"
  Quando "carla" responde à resposta de "ana"
  Então a resposta de "carla" fica no mesmo comentário principal de "bruno"

Cenário: texto longo
  Quando o usuário envia um comentário com 2.001 caracteres
  Então o sistema recusa e informa o limite

Cenário: editar
  Quando o autor edita o próprio comentário
  Então o texto novo aparece com a marca "editado"

Cenário: excluir comentário com respostas
  Dado um comentário com 2 respostas
  Quando o autor do comentário o exclui
  Então ele aparece como "comentário removido"
  E as 2 respostas continuam visíveis

Cenário: comentário de outro usuário
  Quando "bruno" tenta editar o comentário de "carla" pela API
  Então a API recusa

Cenário: moderação
  Quando o admin exclui um comentário ofensivo
  Então ele deixa de aparecer para todos

Cenário: autor da build tenta excluir comentário
  Dado um comentário de "bruno" na build de "ana"
  Quando "ana" tenta excluir o comentário de "bruno" pela API
  Então a API recusa

Cenário: link clicável
  Quando o usuário comenta com um link para outro site
  Então o link aparece clicável e abre em outra aba

Cenário: GIF por link
  Quando o usuário cola o link de um GIF no comentário
  Então o GIF aparece animado dentro do comentário

Cenário: GIF enviado
  Quando o usuário envia um arquivo GIF de 2 MB no formulário
  Então o GIF aparece animado dentro do comentário

Cenário: arquivo inválido
  Quando o usuário tenta enviar um PNG ou um GIF de 6 MB
  Então o sistema recusa e informa as regras

Cenário: editar muito depois
  Dado um comentário publicado há 3 meses
  Quando o autor o edita
  Então a edição é aceita e aparece a marca "editado"

Cenário: denunciar
  Dado um comentário ofensivo de "carla"
  Quando "bruno" o denuncia com o motivo "ofensivo"
  Então o comentário aparece na lista de denúncias do admin
  E "carla" não sabe quem denunciou

Cenário: denúncia repetida
  Dado que "bruno" já denunciou um comentário
  Quando ele tenta denunciar o mesmo comentário de novo
  Então a denúncia não é registrada outra vez

Cenário: descartar denúncia
  Dado um comentário denunciado que não viola nada
  Quando o admin descarta as denúncias
  Então o comentário continua visível e sai da lista de denúncias
```

## Casos de borda

- Build excluída: os comentários ficam ocultos junto com ela (spec 003, req. 19a).
- Comentário editado depois de receber respostas (as respostas podem perder o sentido).
- Comentário denunciado que é editado pelo autor antes de o admin revisar.
- Link de GIF que não aponta para um GIF (página, outro formato, link quebrado).
- GIF ofensivo: tratado por denúncia e moderação, como o texto.
- Usuário que troca de nome: os comentários antigos mostram o nome novo.
- Comentário só com espaços ou vazio.
- Spam de links: tratado por denúncia, moderação e limite de frequência (req. 8).

## Decisões

- **Comentários no MVP:** incluídos no escopo a pedido do produto (antes estavam
  fora do escopo do PRD).
- **Até 2.000 caracteres, um nível de resposta, do mais antigo para o mais
  recente, 20 por página:** conforme o PRD (RF14).
- **Sem notificação de novos comentários** por enquanto (resposta 1).
- **Só o admin exclui comentários de outros** (req. 11).
- **Denúncias** com lista de revisão para o admin (req. 14–18).
- **Links clicáveis e um GIF por comentário**, por link ou envio de arquivo de até
  5 MB (req. 3a–3c). O limite de 5 MB e "um GIF por comentário" são padrão
  inicial, a reavaliar.
- **Edição sem prazo** (req. 9).

## Perguntas em aberto

1. O autor da build deve ser avisado de novos comentários? Como (e-mail, aviso no portal)?
   **R:** Não, por enquanto.
2. O autor da build pode excluir comentários de outros na própria build, ou só o admin?
   **R:** Só o admin.
3. Usuários podem denunciar um comentário para o admin revisar?
   **R:** Sim (req. 14–18).
4. Links no comentário ficam clicáveis?
   **R:** Sim. Também pode ter GIF, por link ou enviando o arquivo no formulário
   (req. 3a–3c).
5. Há prazo para editar um comentário depois de publicado?
   **R:** Não.
