# redline

**Um agente que explica por que o CI ficou vermelho.**

Quando um pipeline falha, `redline` pega os logs, o diff e o repositório, reproduz a
falha quando consegue, e responde **por que quebrou** — com evidência e um próximo
passo. Não um resumo do log: uma explicação.

---

## Por que isto existe

Uma execução vermelha é o **único** momento em que a explicação existe. E ela é
sistematicamente destruída, sempre pelo mesmo motivo: o filtro é escrito para o caso
que se espera, e o caso que se espera é o verde.

Três episódios medidos em dezessete horas, no ecossistema que originou este projeto:

| o que foi rodado | o que sobrou |
|---|---|
| `pnpm test \| grep` | `11 failed`. Nenhum nome de teste. As duas execuções seguintes passaram — a explicação deixou de existir. |
| `pnpm gates >/dev/null 2>&1 && echo OK` | A saída inteira descartada. O `echo` como único oráculo. |
| vinte commits contra um gate local verde | O CI esteve vermelho o tempo todo. Quem percebeu foi alguém de fora. |

Nenhum dos três foi descuido. **Todos foram o comando escrito antes de o resultado
importar.** É esse buraco que `redline` ocupa.

---

## Regra inviolável deste repositório

> Este projeto consome `@theokit/sdk` **exclusivamente do registry npm**.
>
> Sem `file:`, sem `pnpm link`, sem `workspace:`, sem alcançar um checkout local.
> **Se algo só funciona com o código-fonte à mão, isso é um issue — não um contorno.**

O motivo não é purismo. Este repositório existe para medir a experiência de quem
instala o SDK sem conhecê-lo por dentro. No instante em que ele consome código local,
deixa de medir o que é publicado e passa a medir o que existe numa máquina — que é
exatamente a classe de defeito que ele foi criado para encontrar.

A mesma disciplina vale para o conhecimento: **quem constrói aqui não deve consultar o
fonte do SDK.** Documentação, tipos publicados e mensagens de erro são a superfície
inteira. Contornar um problema porque se conhece o interno é perder a medição.

---

## O que ele exercita

`redline` foi escolhido por atravessar a coluna do SDK, não uma fatia dela.

| capacidade | por que este produto precisa dela |
|---|---|
| `buildRepoMap` | entender um repositório que ele não conhece |
| `compactTranscript` | **um log de CI não cabe na janela** — e a informação útil são três linhas |
| `sandbox` | reproduzir sem confiar no que o log afirma |
| `subagents` | um por job que falhou, em paralelo |
| `persistence` | *"já vimos esta falha?"* — reincidência é o dado mais valioso |
| `isTransientError` | espelho de rede caindo e teste quebrado pedem respostas **opostas** |
| `subscription` | transmitir a análise enquanto ela acontece |
| `server/auth` | webhook autenticado — é um serviço, não um script |
| `Eval` + `Scorers` | a explicação estava certa? Sem isso, é chute com confiança |

**Dois decidem se o produto existe.** `compactTranscript`, porque se a compactação
descartar as três linhas que importam não há explicação a dar. E `isTransientError`,
porque chamar de "falha de CI" tanto um `apt` indisponível quanto um teste quebrado é
errar precisamente naquilo que o produto vende — hoje, um passo de rede pendurou um PR
por 1h51 enquanto um teste realmente quebrado ficou vermelho por quatro horas.

---

## Estado

**Dia 0.** Nada implementado. Decisões em aberto:

- [ ] Primeiro repositório observado — candidato: `usetheoai-lab/TheoCode` (público, CI real)
- [ ] Comenta sozinho no PR ou espera aprovação humana
- [ ] Superfície de entrada: webhook, GitHub App ou polling

---

## Como reportamos

Toda fricção vira issue, **inclusive a documental**. *"O README não disse"* é um issue
legítimo e é o mais sub-reportado de todos, porque quem tropeça nele resolve sozinho e
segue adiante. Para adoção, é o que mais custa.

Issues sobre o SDK vão para o repositório do SDK, com: o que se tentou, o que se
esperava, o que aconteceu, e a versão exata instalada.
