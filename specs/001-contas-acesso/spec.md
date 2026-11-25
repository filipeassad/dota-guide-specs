# 001 — Contas e controle de acesso

## Problema

Para publicar builds, votar e comentar, o portal precisa saber quem é cada pessoa.
Sem contas, não há autor para editar a própria build, não há como impedir voto
repetido e não há como separar o que só o admin pode fazer.

## Objetivos

- Qualquer pessoa lê o portal sem conta; criar conta leva menos de um minuto.
- Cada ação (criar build, votar, comentar, moderar, publicar notícia) é liberada
  só para o perfil certo, sempre verificado no servidor.
- Existe pelo menos um admin desde o primeiro dia.
- O admin consegue tirar do portal quem abusa (suspensão e banimento).

## Fora do escopo

- Login com Steam, Google ou outras contas externas (PRD, fora do escopo do MVP).
- Perfil público do usuário com estatísticas.

## Requisitos

### Perfis

1. O sistema DEVE ter três perfis: **visitante**, **usuário** e **admin** (glossário).
2. O visitante DEVE poder ler builds publicadas, comentários e notícias publicadas
   sem login.
3. O usuário com **e-mail confirmado** DEVE poder fazer tudo o que o visitante faz
   e, além disso, criar builds, votar e comentar.
4. O admin DEVE poder fazer tudo o que o usuário faz e, além disso, cadastrar
   versões do jogo, publicar notícias e moderar conteúdo e usuários (req. 16–19).
5. O primeiro admin DEVE ser criado fora do portal (por quem opera o sistema). Um
   admin PODE promover outro usuário a admin.

### Cadastro e confirmação de e-mail

6. O visitante DEVE poder criar conta com **nome de usuário**, **e-mail** e **senha**.
7. O nome de usuário DEVE ter **de 8 a 30 caracteres** e PODE ter caracteres
   especiais e espaços. Ele DEVE ser único e é o nome exibido como autor de builds
   e comentários. A unicidade diferencia maiúsculas e minúsculas: "InvokerMain" e
   "invokermain" são nomes diferentes.
   - Espaços no início e no fim DEVEM ser removidos.
7a. O nome de usuário NÃO DEVE conter, ignorando maiúsculas e minúsculas, nenhum
    dos termos reservados: **admin**, **moderador**, **dotaguide**, **dota guide**,
    **suporte**, **oficial**, **conta excluída**. A lista é só para evitar que
    alguém se passe pelo portal; nomes ofensivos não são verificados.
8. O e-mail DEVE ser único e NÃO DEVE ser exibido publicamente.
9. A senha DEVE ter **no mínimo 8 caracteres**, com pelo menos **1 letra
   maiúscula** e pelo menos **1 caractere especial**.
10. QUANDO a conta for criada, o sistema DEVE enviar um link de confirmação ao
    e-mail. O link DEVE ser de uso único e ter validade, e o usuário DEVE poder
    pedir um novo link.
11. ENQUANTO o e-mail não estiver confirmado, o usuário PODE entrar no portal, mas
    NÃO DEVE poder criar builds, votar nem comentar. O sistema DEVE avisar que
    falta confirmar o e-mail.

### Login e sessão

12. O usuário DEVE poder entrar com e-mail e senha e continuar logado entre visitas.
    A sessão DEVE expirar após **3 semanas sem uso**.
13. O usuário DEVE poder sair (encerrar a sessão no dispositivo atual).
14. O usuário DEVE poder redefinir a senha por um link enviado ao e-mail. O link
    DEVE ser de uso único e ter validade.
15. SE alguém errar a senha várias vezes seguidas, ENTÃO o sistema DEVE limitar
    novas tentativas por um tempo (constituição, princípio VI).

### Moderação de usuários (admin)

16. O admin DEVE poder **suspender** um usuário por um período. ENQUANTO suspenso,
    o usuário NÃO DEVE poder criar ou editar builds, votar nem comentar.
17. O admin DEVE poder **banir** um usuário. O usuário banido NÃO DEVE conseguir
    entrar no portal, e as sessões abertas dele DEVEM ser encerradas.
    - As builds, comentários e votos do usuário suspenso ou banido DEVEM continuar
      aparecendo e contando. Para remover um conteúdo específico, o admin usa a
      moderação (req. 18).
    - O e-mail de um usuário banido NÃO DEVE poder ser usado para criar outra conta.
18. O admin DEVE poder remover comentários de qualquer usuário (detalhes na spec 006).
19. QUANDO um usuário for suspenso ou banido, ele DEVERIA ver o motivo e, se
    suspenso, até quando.

### Conta

20. QUANDO um visitante tentar uma ação que exige login (criar build, votar,
    comentar), o sistema DEVE pedir login e, depois dele, voltar para onde a
    pessoa estava.
21. Toda permissão DEVE ser verificada pela API (constituição, princípio IV).
22. O usuário DEVE poder alterar o próprio nome de usuário e senha.
23. O usuário DEVE poder **excluir a própria conta**. QUANDO a conta for excluída:
    - os dados pessoais (e-mail, senha) DEVEM ser apagados (constituição, princípio VI);
    - as builds publicadas e os comentários DEVEM continuar aparecendo, **sem
      dono**: o autor é exibido como "conta excluída" e ninguém mais pode editá-los;
    - os votos DEVEM continuar contando;
    - as builds em **rascunho** DEVEM ser apagadas;
    - o nome de usuário NÃO DEVE poder ser usado por outra pessoa.

## Critérios de aceite

```
Cenário: visitante lê sem conta
  Dado um visitante sem login
  Quando ele abre uma build publicada
  Então vê a build completa e os comentários
  E não vê os botões de votar e comentar habilitados

Cenário: cadastro
  Quando um visitante cria conta com nome "invoker_main", e-mail e senha válidos
  Então a conta é criada, ele fica logado
  E recebe um link de confirmação no e-mail

Cenário: e-mail não confirmado
  Dado um usuário logado que ainda não confirmou o e-mail
  Quando ele tenta votar numa build
  Então o sistema recusa e avisa que falta confirmar o e-mail

Cenário: confirmação de e-mail
  Dado um usuário com e-mail não confirmado
  Quando ele abre o link de confirmação
  Então o e-mail fica confirmado e ele já pode votar, comentar e criar builds
  E o mesmo link não funciona uma segunda vez

Cenário: nome de usuário curto
  Quando um visitante tenta criar conta com o nome "pudge"
  Então o cadastro é recusado e o sistema informa o mínimo de 8 caracteres

Cenário: nome de usuário longo
  Quando um visitante tenta criar conta com um nome de 31 caracteres
  Então o cadastro é recusado e o sistema informa o máximo de 30 caracteres

Cenário: nome de usuário repetido
  Dado um usuário "invoker_main"
  Quando outra pessoa tenta criar conta com "invoker_main"
  Então o cadastro é recusado e o sistema informa que o nome já está em uso

Cenário: senha fraca
  Quando um visitante tenta criar conta com a senha "dotadota"
  Então o cadastro é recusado e o sistema informa as regras de senha

Cenário: sessão expira sem uso
  Dado um usuário logado que não abre o portal há 3 semanas
  Quando ele volta ao portal
  Então precisa entrar de novo

Cenário: ação que exige login
  Dado um visitante na página de uma build
  Quando ele clica em votar
  Então o sistema pede login
  E, após o login, ele volta para a mesma build

Cenário: rota de admin chamada por usuário comum
  Dado um usuário sem perfil admin
  Quando ele chama direto a ação de publicar notícia na API
  Então a API recusa a ação

Cenário: redefinição de senha
  Dado um usuário que esqueceu a senha
  Quando ele pede a redefinição e usa o link recebido
  Então define uma senha nova
  E o mesmo link não funciona uma segunda vez

Cenário: muitas tentativas de login
  Quando alguém erra a senha de uma conta várias vezes seguidas
  Então novas tentativas ficam bloqueadas por um tempo

Cenário: usuário suspenso
  Dado um usuário suspenso pelo admin por 7 dias
  Quando ele tenta comentar numa build
  Então o sistema recusa e informa até quando vai a suspensão

Cenário: usuário banido
  Dado um usuário logado
  Quando o admin o bane
  Então a sessão dele é encerrada
  E ele não consegue entrar de novo

Cenário: excluir a própria conta
  Dado um usuário com 2 builds publicadas, 1 rascunho, comentários e votos
  Quando ele exclui a própria conta
  Então ele não consegue mais entrar
  E as 2 builds e os comentários continuam aparecendo, com o autor "conta excluída"
  E os votos dele continuam contando
  E o rascunho é apagado

Cenário: nome com termo reservado
  Quando um visitante tenta criar conta com o nome "DotaGuide_Oficial"
  Então o cadastro é recusado e o sistema informa que o nome não é permitido

Cenário: nomes que só diferem em maiúsculas
  Dado um usuário "InvokerMain"
  Quando outra pessoa cria conta com "invokermain"
  Então a conta é criada

Cenário: conteúdo de usuário banido
  Dado um usuário com builds publicadas e comentários
  Quando o admin o bane
  Então as builds e os comentários dele continuam aparecendo

Cenário: banido tenta nova conta
  Dado um usuário banido com o e-mail "ana@exemplo.com"
  Quando alguém tenta criar conta com "ana@exemplo.com"
  Então o cadastro é recusado
```

## Casos de borda

- E-mail com letras maiúsculas e minúsculas diferentes ("Ana@x.com" e "ana@x.com").
- Nomes que só diferem em maiúsculas e minúsculas são permitidos e podem ser usados
  para imitar outro usuário ("InvokerMain" e "invokermain").
- Nome com vários espaços seguidos ou só com espaços.
- Nome ofensivo: não é verificado no cadastro; o admin pode banir o usuário.
- Pedido de redefinição para um e-mail que não existe (não revelar se a conta existe).
- Usuário troca o nome de usuário: builds e comentários antigos passam a mostrar o novo nome.
- Último admin tenta remover o próprio perfil de admin ou excluir a própria conta.
- Admin tenta suspender ou banir outro admin.
- Usuário banido tenta criar outra conta com o mesmo e-mail.
- Link de confirmação expira antes de o usuário abrir.

## Decisões

- **Perfis:** visitante, usuário e admin, conforme o PRD (req. 1–4).
- **Login externo (Steam):** fora do MVP, conforme o PRD.
- **Confirmação de e-mail:** obrigatória para criar builds, votar e comentar (req. 10–11).
- **Nome de usuário:** de 8 a 30 caracteres, caracteres especiais e espaços
  permitidos, único diferenciando maiúsculas e minúsculas, sem termos reservados
  (req. 7 e 7a).
- **Senha:** mínimo de 8 caracteres, 1 maiúscula e 1 caractere especial (req. 9).
- **Sessão:** expira após 3 semanas sem uso (req. 12).
- **Moderação:** o admin pode suspender e banir usuários e remover comentários no MVP (req. 16–18).
- **Exclusão de conta:** permitida; builds publicadas, comentários e votos
  continuam, sem dono; rascunhos são apagados (req. 23).
- **Conteúdo de suspenso/banido:** continua aparecendo e contando; o e-mail do
  banido não pode criar outra conta (req. 17).

## Perguntas em aberto

1. O e-mail precisa ser confirmado (link de verificação) antes de o usuário
   publicar builds, votar ou comentar?
   **R:** Sim.
2. Quais as regras do nome de usuário (tamanho, caracteres permitidos, lista de
   nomes proibidos)?
   **R:** Mínimo de 8 caracteres, pode ter caracteres especiais, sem nomes repetidos.
3. Quais as regras de senha (tamanho mínimo, outras exigências)?
   **R:** Pelo menos 8 caracteres, pelo menos 1 letra maiúscula e pelo menos 1
   caractere especial.
4. Por quanto tempo a sessão fica ativa sem uso?
   **R:** 3 semanas.
5. O admin pode suspender ou banir um usuário no MVP, ou isso fica para depois?
   **R:** Sim, e também pode remover comentários.
6. O usuário pode excluir a própria conta? Se sim, o que acontece com as builds e
   comentários dele?
   **R:** Pode. As builds ficam sem dono, mas continuam aparecendo.

### Novas perguntas (surgiram das respostas)

7. Nome de usuário tem tamanho máximo? E "invokermain" e "InvokerMain" contam como
   o mesmo nome?
   **R:** Por enquanto não, para as duas: sem tamanho máximo, e "invokermain" e
   "InvokerMain" são nomes diferentes.
8. Pode ter espaço no nome de usuário? Existe uma lista de nomes proibidos (ex.:
   "admin", "dotaguide")?
   **R:** Pode ter espaço. Lista básica de nomes proibidos (req. 7a); não precisa
   se preocupar com nomes ofensivos.
9. Ao excluir a conta, o que acontece com os **comentários** e os **votos** do
   usuário?
   **R:** Mesma ideia da build: continuam lá, mas sem usuário.
10. Ao excluir a conta, os **rascunhos** dele são apagados?
    **R:** Sim.
11. Suspensão e banimento: o que acontece com as builds e comentários do usuário
    banido (continuam visíveis ou são ocultados)? E com os votos dele?
    **R:** Continuam aparecendo.
12. Um usuário banido pode criar outra conta com o mesmo e-mail?
    **R:** Com o mesmo e-mail, não.

13. A constituição (princípio VI) exige tamanho máximo para todo texto escrito por
    usuários, e o nome de usuário aparece em builds, comentários e listagens. A
    resposta 7 diz "sem máximo por enquanto". Podemos definir um limite (sugestão:
    **30 caracteres**) ou registrar uma exceção à constituição?
    **R:** Máximo de 30 caracteres (req. 7).