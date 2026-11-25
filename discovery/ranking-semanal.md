# Discovery: cálculo de "mais votadas na semana"

> **Status:** decidida | **Relacionado a:** spec 004 (req. 8),
> spec 005 (votos e pontuação) | **Princípios afetados:** V (ranking justo,
> calculado no servidor), VII (teste automatizado), VIII (operação enxuta)
> **Última atualização:** 2026-11-24

## 1. Pergunta que este discovery responde

A listagem pode ser ordenada por **mais votadas na semana**: o maior **saldo de
votos** (positivos menos negativos) dos últimos 7 dias primeiro (spec 004,
pergunta 2). Como calcular esse número de forma correta, barata e estável para o
carregamento contínuo (ver [paginacao-continua.md](paginacao-continua.md))?

## 2. O que é "a semana"

| Opção | Como funciona | Problema |
| --- | --- | --- |
| Semana do calendário (segunda a domingo) | Zera toda segunda | Na segunda de manhã todas as builds têm saldo 0 e a ordenação perde o sentido. **Descartada** |
| **Janela móvel de 7 dias** | Conta os votos das últimas 168 horas, a partir de agora | Nenhum: a lista sempre reflete a última semana. **Recomendada** |

## 3. Como um voto entra na conta

Cada usuário tem no máximo um voto por build (spec 005), que pode ser trocado ou
retirado. Para a janela de 7 dias, vale **o estado atual do voto e a data da
última vez que ele mudou**:

| Situação | Conta na semana? |
| --- | --- |
| Voto positivo dado há 2 dias | +1 |
| Voto positivo dado há 10 dias, nunca mudou | Não conta (fora da janela) |
| Voto positivo dado há 10 dias, trocado para negativo hoje | −1 (a mudança é de hoje) |
| Voto dado há 2 dias e retirado hoje | Não conta (o voto não existe mais) |
| Voto de usuário que excluiu a conta ou foi banido | Conta normalmente (spec 001, req. 17 e 23) |
| Voto em build excluída ou ocultada | A build não aparece na listagem |

**Alternativa considerada:** guardar cada mudança de voto como um evento e somar a
diferença (trocar +1 por −1 hoje somaria −2 na semana). É mais "exato" sobre o
movimento, mas exige uma tabela de eventos e é mais difícil de explicar ao
usuário. **Descartada para o MVP.**

Fórmula:

```
saldo da semana da build = soma do valor (+1 ou −1) dos votos atuais da build
                           cuja última mudança foi nas últimas 168 horas
```

## 4. Opções de cálculo

| Opção | Como funciona | Prós | Contras |
| --- | --- | --- | --- |
| **A. Consulta na hora** | Cada listagem soma os votos da janela, agrupando por build | Sempre exato; nada a manter | O valor muda entre uma página e outra do carregamento contínuo (repete ou pula builds); custo cresce com o volume de votos |
| **B. Contador por dia** | Uma linha por build e por dia com o saldo daquele dia, atualizada junto com o voto; a semana soma os últimos 7 dias | Leitura rápida | Janela fica "por dia", não por hora; trocar voto exige corrigir o dia antigo e o novo; mais regra para testar |
| **C. Recalcular de hora em hora** | Uma tarefa agendada recalcula o saldo da semana de todas as builds com votos na janela e guarda o resultado junto da build | Leitura tão rápida quanto ordenar por pontuação; o valor **não muda durante a rolagem** (estável por 1 hora); usa o mesmo agendador da importação de versões (spec 002, req. 15), sem peça nova | Até 1 hora de atraso |

## 5. Recomendação

**Opção C**, recalculando a cada hora:

1. "Mais votadas na semana" não precisa ser em tempo real: 1 hora de atraso não
   muda a experiência.
2. É a única opção em que a ordem fica parada enquanto a pessoa rola a lista, o
   que evita builds repetidas ou puladas no carregamento contínuo.
3. Não adiciona infraestrutura: o agendador já existe para a importação de
   versões (princípio VIII).
4. A **pontuação total** (spec 005) continua sendo atualizada na hora, a cada voto.
   Só o saldo da semana é recalculado de hora em hora.

Detalhes para o plano da API:

- O recálculo considera só builds com algum voto na janela e zera o saldo das
  builds que saíram dela.
- Guardar a hora do último recálculo, para a interface poder mostrar "atualizado
  às 14h".
- Índice na data da última mudança do voto, para o recálculo não ler a tabela
  de votos inteira.
- Testes (princípio VII): voto dentro e fora da janela, voto trocado, voto
  retirado, build que sai da janela, empate desfeito pela última atualização
  (spec 004, req. 8a).

## 6. Perguntas em aberto

- 1 hora de atraso é aceitável? (Alternativa: 15 minutos, com custo um pouco maior.)
  **R:** Sim. A pontuação da build muda na hora a cada voto; só a posição em
  "mais votadas na semana" é atualizada de hora em hora.
- A listagem deve mostrar o saldo da semana no card quando essa ordenação estiver
  ativa, além da pontuação total?
  **R:** Não. O card sempre mostra a pontuação de todos os votos da build (saldo:
  positivos menos negativos, spec 005). O saldo da semana só serve para ordenar.

## 7. Decisão

- Janela móvel de 7 dias; conta o estado atual de cada voto com a data da última
  mudança (seções 2 e 3).
- Recálculo de hora em hora (opção C).
- O card mostra só a pontuação total (saldo de todos os votos).

## 8. Próximos passos

- [x] Revisar e aprovar a recomendação (opção C)
- [x] Registrar a decisão na spec 004 (req. 8)
- [ ] No `plan.md` da 005 na `dota-guide-api`: coluna do saldo da semana, tarefa de
      recálculo, índice e testes
