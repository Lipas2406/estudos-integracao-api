# Passo 2 — Tudo que dá errado

**Tempo:** um bloco de 25 a 50 min. **Precisa instalar:** nada.

Abre o `index.html` com dois cliques e clica nos cinco cenários, na ordem.

> Este é o passo mais importante da trilha. Integração é 20% caminho feliz e 80% lidar com o que
> falha. Quem só aprende o caminho feliz escreve código que funciona na demonstração e mente em
> produção.

---

## A ideia da página

A **mesma requisição**, feita de dois jeitos, lado a lado:

- **Jeito ingênuo** — três linhas, o que a maioria dos tutoriais ensina.
- **Jeito cuidadoso** — não devolve "os dados", devolve **qual dos resultados possíveis aconteceu**.

Você clica no cenário e compara não o código, mas **o que cada um mostra para quem está usando**.

---

## O que realmente acontece em cada cenário

Rodei os cinco antes de publicar. Estes são os resultados medidos, não previstos:

| # | Cenário | Jeito ingênuo | Jeito cuidadoso |
|:-:|---|---|---|
| 1 | Tudo certo | 27 resultados, Rondônia | `OK` |
| 2 | UF que não existe | **"0 resultados. Primeiro: undefined"** | `SEM_RESULTADO`, status 200 |
| 3 | Endereço errado | **explode:** `SyntaxError: Unexpected end of JSON input` | `ERRO_HTTP`, status 404 |
| 4 | Servidor inalcançável | **explode:** `TypeError: Failed to fetch` | `FALHA_DE_REDE` |
| 5 | Prazo de 1ms | devolveu a lista de 27 | `DEMOROU_DEMAIS` |

---

## As quatro lições, uma por cenário

### Cenário 2 — `200` não quer dizer que deu certo

O servidor respondeu **200**. Do ponto de vista dele, tudo certo: a pergunta chegou, foi entendida, e
a resposta é "não tenho nada para essa UF".

O jeito ingênuo mostra **"0 resultados. Primeiro: undefined"**. Repare no que aconteceu ali: o
`undefined` vazou para a tela do usuário. Ninguém escreveu isso — é o JavaScript dizendo "você pediu
o primeiro item de uma lista vazia".

> **A regra:** status conta como foi a **conversa**, não se o **pedido** foi atendido. São duas
> perguntas diferentes, e você precisa fazer as duas.

Num sistema de estoque, isso é a diferença entre "esse técnico não tem nenhuma peça pendente" e
"você digitou o TC errado". A tela que confunde as duas faz alguém tomar decisão sobre dado que não
existe.

### Cenário 3 — o `fetch` **não** rejeita quando o status é de erro

Este é o que pega todo mundo, inclusive gente experiente.

O servidor devolveu **404**. O jeito ingênuo tem `try/catch`, então deveria pegar, certo? Pegou —
mas repare **na mensagem**:

```
SyntaxError: Failed to execute 'json' on 'Response': Unexpected end of JSON input
```

**A mensagem fala de JSON. O problema real foi 404.** O erro só apareceu porque o corpo veio vazio e
o `.json()` engasgou. Se o servidor tivesse respondido 404 com uma página de erro em HTML, o
`.json()` também quebraria, e a mensagem continuaria apontando para o lugar errado.

> **A regra:** o `fetch` só rejeita quando a requisição **não aconteceu**. Se o servidor respondeu
> qualquer coisa — 404, 500, 403 — para o `fetch` isso é sucesso. Você tem que checar `resposta.ok`
> na mão, sempre.

Isso é a mesma família de problema que você já reconhece: **a verificação passou porque estava
olhando para o lugar errado.** Aqui, a mensagem de erro aponta para o sintoma e esconde a causa.

### Cenário 4 — este é o único que o `fetch` rejeita de verdade

`TypeError: Failed to fetch`. A requisição não saiu: o endereço não existe, o DNS não resolveu.

Compare com o cenário 3: os dois "deram erro", mas por motivos completamente diferentes, e a ação do
usuário é diferente em cada um. No 3 ele digitou algo errado; no 4 a conexão dele caiu. Uma tela que
diz "erro ao carregar" nos dois casos não ajuda ninguém.

**Detalhe cruel:** `Failed to fetch` é também a mensagem que aparece quando o navegador bloqueia por
CORS — aquele cabeçalho do Passo 1. Mesma mensagem, causa completamente diferente. É por isso que
você vai precisar da aba Network para diagnosticar, e não do `console.log`.

### Cenário 5 — leia com atenção, porque aqui o ingênuo "ganhou"

O jeito ingênuo devolveu os 27 estados. O cuidadoso desistiu. Parece que o cuidadoso é pior.

**Não é. Ele é o único dos dois que tem prazo.**

O prazo de 1ms é artificialmente curto, para você ver o mecanismo disparar. Na vida real seriam 5 ou
10 segundos. A pergunta certa não é "quem trouxe o dado", é: **o que o ingênuo faria se o servidor
demorasse cinco minutos?** Esperaria os cinco minutos, com a tela parada em "carregando", sem nada
que o usuário pudesse fazer.

> **A regra:** o `fetch` não tem prazo. Sem `AbortController`, ele espera para sempre. "Para sempre"
> não é exagero de texto, é literal.

Não ter prazo não é o mesmo que ser rápido. É não ter a capacidade de desistir.

---

## O que fazer, na ordem

### 1. Clique nos cinco e leia as duas colunas
Antes de olhar o código. Foque na caixa **"o que o usuário vê"**, que é a única coisa que importa
para quem não é você.

### 2. Leia a função `buscarCuidadoso` no `index.html`
Ela tem cinco pontos de saída, numerados nos comentários. Cada um é um dos casos acima. Isso não é
excesso de zelo: é o mínimo para não mentir.

### 3. Repare no que ela devolve
Ela **não** devolve os dados. Devolve `{ situacao: '...' }`, e quem chamou decide o que exibir para
cada situação. Essa separação é o que permite a tela ser honesta — a função relata, a tela traduz.

### 4. Quebre de propósito (obrigatório antes do Passo 3)

| Quebra | Como | O que observar |
|---|---|---|
| Tire a checagem de `ok` | comente o bloco `if (!resposta.ok)` | qual cenário passa a mentir, e o que ele mostra? |
| Tire a checagem de lista vazia | comente o bloco `SEM_RESULTADO` | o cuidadoso vira o ingênuo em qual cenário? |
| Tire o `AbortController` | remova o `signal` do fetch | o cenário 5 ainda desiste? |
| Aumente o prazo | troque `prazo: 1` por `prazo: 8000` no cenário 5 | por que agora os dois concordam? |

**A segunda é a mais importante.** Ela mostra que a diferença entre um código honesto e um mentiroso
pode ser três linhas.

---

## Critério de "terminei"

Você consegue responder, sem consultar:

1. Em qual dos cinco cenários o `fetch` rejeita, e por que só nesse?
2. Por que a mensagem de erro do cenário 3 fala de JSON se o problema foi 404?
3. Um servidor respondeu 200 com corpo `[]`. Deu certo?
4. O que acontece com um `fetch` sem `AbortController` se o servidor nunca responder?

Se a 3 te fizer hesitar, o passo funcionou.

---

## Onde isso te encontra no trabalho

Toda API interna vai ter esses cinco casos, mais alguns próprios. Quando você for conversar sobre
integrar, as perguntas que nascem daqui são exatamente as que mostram que você já pensou no assunto:

- Quando não há resultado, vem lista vazia com 200, ou vem um status de erro?
- Erro de negócio (o pedido não faz sentido) vem com status 4xx, ou vem 200 com um campo de erro no
  corpo?
- Qual o tempo máximo esperado de resposta? Existe algum endpoint que costuma demorar?
- Quando dá erro, o corpo vem em JSON ou em HTML?

Essas quatro perguntas cabem numa mensagem curta e economizam semanas de tentativa e erro.
