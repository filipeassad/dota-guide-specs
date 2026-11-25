# 005 — Votos e ranking

## Problema

Qualquer usuário pode publicar uma build, então a qualidade varia muito. Sem um
sinal da comunidade, o jogador não sabe em qual build confiar, e as builds boas
ficam misturadas com as incompletas ou erradas.

## Objetivos

- A comunidade indica quais builds funcionam, e elas aparecem primeiro.
- O ranking não pode ser manipulado com facilidade (constituição, princípio V).
- O ranking reflete a versão atual: uma build ótima de um patch antigo não
  ocupa o topo para sempre.

## Fora do escopo

- Ranking de autores (reputação do usuário).
- Medalhas, conquistas ou recompensas por votos.
- Ranking baseado em estatísticas reais de partidas, conforme o PRD.

## Requisitos

### Votos

1. O usuário logado DEVE poder dar voto **positivo** ou **negativo** em uma build
   publicada de outro autor (PRD, RF09). Basta ter conta; vale a regra de e-mail
   confirmado da spec 001 (req. 3 e 11).
2. Cada usuário DEVE ter no máximo **um voto por build** (constituição, princípio V).
3. O usuário DEVE poder trocar o voto (positivo ↔ negativo) ou retirá-lo.
4. O autor NÃO DEVE poder votar na própria build.
5. Builds em rascunho NÃO DEVEM receber votos.
6. Para o usuário, a única regra de quantidade é **um voto por build** (req. 2).
   Contra votos em massa por script, a API DEVE ter um limite técnico de
   frequência, alto o bastante para nunca ser atingido por uma pessoa votando
   normalmente (constituição, princípio V; valor definido no plano).
7. O usuário DEVE ver na build qual voto ele deu.

### Pontuação e ranking

8. A **pontuação** de uma build DEVE ser o saldo de votos (positivos menos
   negativos), calculado pela API (PRD, RF09). Todos os votos têm o **mesmo peso**,
   sem diferença entre votos recentes e antigos.
8a. A pontuação DEVE ser exibida como **saldo** (um número só, que pode ser
    negativo), sem mostrar positivos e negativos separados.
9. O ranking DEVE ser por versão: uma build compete só com as builds da mesma
   versão (PRD, RF09).
10. A cópia de uma build para outra versão (spec 003) DEVE começar sem votos.
11. QUANDO um voto for dado, trocado ou retirado, a pontuação exibida DEVE ser
    atualizada.
12. QUANDO uma build for excluída ou ocultada pela moderação, ela DEVE sair do ranking.

## Critérios de aceite

```
Cenário: votar
  Dado uma build publicada de "ana" com pontuação 0
  Quando "bruno" vota positivo
  Então a pontuação passa a 1
  E "bruno" vê o voto positivo marcado

Cenário: trocar voto
  Dado que "bruno" votou positivo numa build com pontuação 1
  Quando ele troca para negativo
  Então a pontuação passa a -1

Cenário: voto repetido
  Dado que "bruno" já votou positivo numa build
  Quando a API recebe outro voto positivo de "bruno" na mesma build
  Então a pontuação não muda

Cenário: autovoto
  Quando "ana" tenta votar na própria build
  Então a API recusa

Cenário: ranking por versão
  Dado uma build de 7.37c com pontuação 50 e uma de 7.37d com pontuação 5
  Quando o visitante vê o ranking de 7.37d
  Então a build de 7.37c não aparece nele

Cenário: votos em massa por script
  Quando uma conta envia votos em dezenas de builds diferentes em poucos segundos
  Então os votos acima do limite técnico são recusados

Cenário: cópia começa do zero
  Dado uma build de 7.37c com pontuação 30
  Quando o autor copia para 7.37d
  Então a cópia começa com pontuação 0

Cenário: saldo negativo
  Dado uma build com 3 votos positivos e 5 negativos
  Quando alguém abre a build
  Então vê a pontuação -2, sem os positivos e negativos separados

Cenário: votos antigos e novos valem igual
  Dado uma build com 10 votos positivos dados há 3 meses
  E outra build com 10 votos positivos dados ontem, na mesma versão
  Então as duas têm pontuação 10

Cenário: e-mail não confirmado
  Dado um usuário que ainda não confirmou o e-mail
  Quando ele tenta votar
  Então o sistema recusa e avisa que falta confirmar o e-mail
```

## Casos de borda

- Usuário que votou é excluído, suspenso ou banido: o voto continua contando
  (spec 001, req. 17 e 23).
- Build com muitos votos negativos: fica oculta, vai para o fim ou é sinalizada?
- Várias contas criadas pela mesma pessoa para votar.
- Build antiga com muitos votos versus build nova com poucos votos na mesma versão.

## Decisões

- **Voto positivo e negativo, pontuação = saldo, exibida como saldo:** conforme o
  PRD (RF09) e as respostas 1 e 5.
- **Mesmo peso para todos os votos:** sem peso de tempo (resposta 2). "Mais
  votadas na semana" é só uma ordenação da listagem (spec 004), não muda a pontuação.
- **Quem vota:** qualquer usuário com conta e e-mail confirmado (resposta 3, spec 001).
- **Limite:** um voto por build para o usuário; limite técnico só contra script
  (req. 6).
- **Ranking por versão:** conforme o PRD (RF09).

## Perguntas em aberto

1. O voto é positivo e negativo, ou só "curtir" (positivo)?
   **R:** Positivo e negativo.
2. Votos recentes devem pesar mais que votos antigos no ranking?
   **R:** Não, todos os votos têm o mesmo peso.
3. O usuário precisa cumprir algum requisito para votar (ex.: e-mail confirmado,
   conta com alguns dias), para dificultar contas falsas?
   **R:** Apenas ter uma conta (com e-mail confirmado, como exige a spec 001).
4. Qual o limite de votos por usuário (quantos em quanto tempo)?
   **R:** Um voto por build (req. 2 e 6).
5. A pontuação é exibida como saldo, ou positivos e negativos separados?
   **R:** Por saldo (req. 8a).