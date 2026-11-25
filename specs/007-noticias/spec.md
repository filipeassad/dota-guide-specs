# 007 — Notícias

## Problema

Builds dependem do que muda no jogo: patches, heróis novos, mudanças no meta. O
jogador busca essas notícias em vários lugares diferentes. Reunir as principais
notícias no portal mantém o jogador informado e dá motivo para voltar.

## Objetivos

- O admin publica notícias relevantes do Dota 2 de forma simples.
- O visitante acompanha as notícias mais recentes na home e numa página própria,
  e filtra pelo assunto que interessa (patch, torneio, comunidade).
- O admin prepara notícias com antecedência e agenda a publicação.
- Notícias de patch levam o jogador às builds da versão nova.

## Fora do escopo

- Notícias escritas por usuários (só o admin publica, conforme o PRD).
- Comentários em notícias.
- Importação automática de notícias de outros sites (a única notícia gerada pelo
  sistema é a de versão nova, req. 6a).
- Newsletter por e-mail.

## Requisitos

### Cadastro (RF11 do PRD)

1. Só o admin DEVE poder criar, editar, despublicar e excluir notícias
   (constituição, princípio IV).
2. Uma notícia DEVE ter: **título**, **resumo**, **conteúdo** (com formatação:
   títulos, negrito, listas, links e imagens), **imagem de capa** e **categoria**.
3. Uma notícia PODE ter: **link da fonte** e **versão do jogo relacionada**.
4. A notícia DEVE poder ser salva como **rascunho**, **agendada** ou **publicada**.
   Rascunhos e agendadas só são visíveis para admins.
5. QUANDO a notícia for publicada, a data de publicação DEVE ser registrada.
6. O admin DEVE poder despublicar uma notícia (ela volta a rascunho e some do portal).
6a. QUANDO uma versão nova do jogo for importada, o sistema DEVE criar e publicar
    automaticamente uma notícia sobre ela, na categoria **Patch** (spec 002,
    req. 19). O admin PODE editar ou despublicar essa notícia como qualquer outra.

### Agendamento

6b. O admin DEVE poder **agendar** uma notícia para uma data e hora futuras.
6c. QUANDO chegar a data e hora agendadas, a notícia DEVE ser publicada
    automaticamente, e a data de publicação DEVE ser a data agendada.
6d. Até a publicação, o admin DEVE poder mudar a data agendada ou cancelar o
    agendamento (a notícia volta a rascunho).

### Categorias

6e. Toda notícia DEVE ter **uma categoria**. As categorias iniciais são **Patch**,
    **Torneio** e **Comunidade**.
6f. O admin DEVE poder criar novas categorias no painel do admin.
6g. Uma categoria com notícias NÃO DEVE poder ser excluída.

### Imagens

6h. A imagem de capa e as imagens do conteúdo DEVEM ser **enviadas pelo admin** e
    guardadas pelo próprio portal; o conteúdo aponta para elas pelo endereço
    gerado no envio. Imagens de outros sites não são usadas diretamente.
6i. DEVEM ser aceitas imagens JPG, PNG, WebP e GIF de até **5 MB** cada (padrão
    inicial, a reavaliar).

### Autor

6j. A notícia DEVE mostrar o **nome do admin** que a escreveu. A notícia
    automática de versão nova (req. 6a) DEVE mostrar como autor **"Dota Guide"**.

### Exibição (RF12)

7. A página de notícias DEVE listar as notícias publicadas, da mais recente para a
   mais antiga, com destaque para a última.
7a. O visitante DEVE poder filtrar a página de notícias por **categoria**. O filtro
    DEVE fazer parte do endereço da página, como na listagem de builds (spec 004).
8. Cada notícia DEVE ter uma página própria com endereço compartilhável, título e
   descrição próprios.
9. A home DEVE mostrar as **3 notícias mais recentes**.
10. QUANDO a notícia tiver uma versão relacionada, a página dela DEVE ter um link
    para as builds daquela versão (spec 004).
11. O conteúdo da notícia DEVE ser exibido sem executar código (constituição,
    princípio VI), mesmo sendo escrito pelo admin.

## Critérios de aceite

```
Cenário: publicar notícia
  Dado um admin logado
  Quando ele cria uma notícia com título, resumo, conteúdo e capa e publica
  Então ela aparece no topo da página de notícias e na home

Cenário: rascunho
  Dado uma notícia em rascunho
  Quando um visitante abre a página de notícias
  Então ela não aparece

Cenário: usuário comum
  Quando um usuário sem perfil admin tenta criar uma notícia pela API
  Então a API recusa

Cenário: notícia de patch
  Dado uma notícia publicada relacionada à versão 7.38
  Quando o visitante abre a notícia
  Então vê um link para as builds de 7.38

Cenário: home
  Dado 5 notícias publicadas
  Quando o visitante abre a home
  Então vê as 3 mais recentes

Cenário: despublicar
  Dado uma notícia publicada
  Quando o admin a despublica
  Então ela some da página de notícias e da home

Cenário: agendar
  Dado uma notícia agendada para amanhã às 10h
  Quando um visitante abre a página de notícias hoje
  Então ela não aparece
  Quando chega amanhã às 10h
  Então ela aparece publicada, com data de publicação de amanhã às 10h

Cenário: cancelar agendamento
  Dado uma notícia agendada
  Quando o admin cancela o agendamento
  Então ela volta a rascunho e não é publicada na data

Cenário: filtrar por categoria
  Dado notícias publicadas nas categorias Patch e Torneio
  Quando o visitante filtra por "Torneio"
  Então vê só as notícias de Torneio

Cenário: nova categoria
  Quando o admin cria a categoria "Cosméticos"
  Então ela pode ser escolhida em notícias novas e aparece no filtro

Cenário: excluir categoria em uso
  Dado a categoria Torneio com notícias
  Quando o admin tenta excluí-la
  Então o sistema recusa

Cenário: imagem no conteúdo
  Quando o admin envia uma imagem PNG de 2 MB no conteúdo da notícia
  Então a imagem fica guardada no portal e aparece na notícia

Cenário: imagem inválida
  Quando o admin tenta enviar uma imagem de 8 MB
  Então o sistema recusa e informa o limite

Cenário: autor
  Dado uma notícia escrita pelo admin "carlos_admin"
  Quando o visitante abre a notícia
  Então vê "carlos_admin" como autor

Cenário: autor da notícia automática
  Quando a notícia da versão 7.38 é criada pela importação
  Então ela aparece na categoria Patch, com o autor "Dota Guide"
```

## Casos de borda

- Notícia automática de versão nova sem imagem de capa (usar uma capa padrão).
- Imagem enviada e depois removida do conteúdo (fica guardada sem uso).
- Agendamento para uma data que já passou.
- Admin autor da notícia exclui a conta ou perde o perfil de admin (mostrar
  "conta excluída", como nas builds).
- Duas notícias com o mesmo título (endereço da página deve ser único).
- Notícia relacionada a uma versão que ainda não é a atual.
- Notícia excluída cujo endereço já foi compartilhado.

## Decisões

- **Só o admin publica notícias:** conforme o PRD.
- **Home com as 3 notícias mais recentes:** conforme o PRD (RF12).
- **Agendamento:** permitido (req. 6b–6d).
- **Categorias:** Patch, Torneio e Comunidade, e o admin cria novas (req. 6e–6g).
- **Imagens:** enviadas pelo admin e guardadas pelo portal; até 5 MB (req. 6h–6i).
  Usa o mesmo armazenamento de arquivos dos GIFs dos comentários (spec 006).
- **Autor:** nome do admin; notícia automática assinada por "Dota Guide" (req. 6j).

## Perguntas em aberto

1. O admin pode agendar a publicação de uma notícia para uma data futura?
   **R:** Pode (req. 6b–6d).
2. Notícias têm categorias (patch, torneio, comunidade) e filtro por categoria?
   **R:** Têm. Essas 3 por enquanto, e o admin pode acrescentar novas (req. 6e–6g).
3. As imagens do conteúdo são enviadas pelo admin para o portal ou apenas linkadas
   de outros sites?
   **R:** Ficam num armazenamento de arquivos do portal, e o conteúdo usa o link
   delas (req. 6h).
4. A notícia mostra o nome do admin que a escreveu?
   **R:** Sim (req. 6j).
