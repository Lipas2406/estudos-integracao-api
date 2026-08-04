# Passo 1 — A requisição crua

**Tempo:** um bloco de 25 min. **Precisa instalar:** nada.

Abre o `index.html` com dois cliques e clica no botão.

---

## O que este passo ensina

Uma resposta HTTP tem **três partes separadas**, e confundir elas é a origem de metade dos bugs de
integração:

| Parte | O que é | Exemplo |
|---|---|---|
| **Status** | Um número dizendo como foi a conversa | `200`, `404`, `500` |
| **Cabeçalhos** | Informação **sobre** a resposta | `Content-Type: application/json` |
| **Corpo** | O dado em si | a lista de estados |

E uma coisa que quase todo tutorial esconde, porque usa `resposta.json()` direto:

> **O corpo chega como texto.** Sempre. Ele vira objeto só quando alguém chama `JSON.parse`.
> No código eu fiz isso em duas etapas de propósito, para você ver acontecer.

---

## O que fazer, na ordem

### 1. Rode e leia as três caixas
Antes de mexer em qualquer coisa. Olha o que apareceu em cada uma.

### 2. Abra o DevTools (F12), aba **Network**, e clique no botão de novo
Você vai ver a requisição aparecer na lista. Clica nela. Ali está a mesma informação das três
caixas, só que na ferramenta que você vai usar pelo resto da vida.

**Isso importa mais do que parece:** quando a integração real não funcionar, é nesta aba que
você vai descobrir por quê, não no `console.log`.

### 3. Compare os cabeçalhos das duas telas — eles são diferentes, e isso é a lição

Na caixa 2 da página aparecem **três** cabeçalhos: `cache-control`, `content-type` e `expires`.
No DevTools, na mesma requisição, aparecem **muitos mais** — inclusive um chamado
`Access-Control-Allow-Origin`, com valor `*`.

Por que o JavaScript vê menos que o DevTools?

> **Porque o navegador esconde a maioria dos cabeçalhos do código.** Por segurança, `resposta.headers`
> só expõe uma lista curta e fixa. Todo o resto existe, chegou, e o seu código simplesmente não
> alcança.

Guarde as duas coisas:

- **`Access-Control-Allow-Origin: *`** é o que permite este arquivo, aberto do seu disco, chamar o
  servidor do IBGE. Quando uma API **não** manda esse cabeçalho, o navegador bloqueia a chamada, e o
  erro que aparece é confuso e não diz o nome real do problema. Você vai encontrar isso numa API interna,
  quase garantido.
- **Se um dado importante vier em cabeçalho não padrão** (é comum: total de registros, link da
  próxima página, limite de uso), o servidor precisa autorizar o acesso a ele explicitamente. Se não
  autorizar, seu código não lê, mesmo o dado estando ali. **Isso é pergunta para quem mantém a API.**

*Nota honesta: a primeira versão deste arquivo dizia que o `Access-Control-Allow-Origin` apareceria
na caixa 2. Estava errado. Só descobri porque rodei a página antes de te entregar, e o que apareceu
foram três cabeçalhos, não ele. O erro virou este item, que ensina mais do que o texto original.*

### 4. Quebre de propósito (obrigatório antes de avançar)

Faça um de cada vez, veja o que acontece, e desfaça:

| Quebra | Como | O que observar |
|---|---|---|
| URL errada | troque `estados` por `estadoss` | qual status voltou? |
| Sem internet | desligue o Wi-Fi e clique | a página trava, mostra erro, ou fica em "Pedindo..." para sempre? |
| Sem converter | comente o `JSON.parse` e use `textoCru` direto | por que `.length` dá um número gigante? |

**A terceira é a mais reveladora.** `textoCru.length` conta caracteres; `objeto.length` conta itens
da lista. São números completamente diferentes para o mesmo dado. Se você confundir isso num sistema
de estoque, conta peça errada.

---

## Critério de "terminei"

Você consegue responder, sem consultar:

1. Qual a diferença entre status, cabeçalho e corpo?
2. Por que `JSON.parse` existe, se o dado já "parece" um objeto?
3. O que aconteceu quando você desligou o Wi-Fi, e por que isso é um problema de verdade?

A pergunta 3 é a ponte para o Passo 2, que é o passo mais importante da trilha.

---

## Uma coisa que você vai notar e é de propósito

Este código **não trata nenhum erro**. Se a requisição falhar, a página simplesmente para, sem
avisar nada a quem está olhando.

Isso é exatamente o comportamento que o Passo 2 vai consertar, e é o mesmo defeito que você já
reconhece de outro contexto: **a tela afirma um estado que os dados não sustentam.** Aqui ela fica
eternamente em "Pedindo...", como se ainda houvesse esperança.

Repare que você já sabe diagnosticar isso. O que falta é o vocabulário para consertar, e é ele que
vem no próximo passo.
