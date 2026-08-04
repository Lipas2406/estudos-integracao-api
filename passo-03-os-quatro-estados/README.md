# Passo 3 — Os quatro estados

**Tempo:** um bloco de 25 a 50 min. **Precisa instalar:** nada — o React vem por CDN.

Abre o `index.html` com dois cliques.

> Primeiro passo com React. Ele entra agora, e não no começo, porque só aqui o problema que ele
> resolve fica visível.

---

## A instrução, e é para seguir na ordem

1. Clique em **1. Tudo certo**. Os dois lados mostram 27 resultados. Iguais.
2. Clique em **3. Endereço errado**.
3. **Olhe o lado esquerdo.**

O que aparece lá:

```
Erro: ERRO_HTTP

27 resultados
 • Rondônia
 • Acre
 • Amazonas
```

**A tela está dizendo duas coisas que não podem ser verdade ao mesmo tempo.** Deu erro, e mesmo
assim tem uma lista de 27 estados na cara do usuário. Ele não tem como saber que aquela lista é
lixo da consulta anterior.

O lado direito, na mesma situação, mostra só: *"Não consegui buscar (ERRO_HTTP)."*

---

## Por que o lado esquerdo faz isso

Ele guarda o estado em **três variáveis separadas**:

```js
const [carregando, setCarregando] = useState(false);
const [erro, setErro]             = useState(null);
const [dados, setDados]           = useState(null);
```

Isso sai naturalmente, escrevendo conforme a necessidade aparece: "preciso saber se está
carregando", "preciso guardar o erro", "preciso guardar os dados". Cada uma faz sentido sozinha.

O problema é o que elas permitem **juntas**. Três variáveis independentes têm **oito combinações**,
e só quatro delas descrevem uma situação real. As outras quatro são estados que não existem no
mundo — como "carregando e com erro ao mesmo tempo" — e nada no código impede que aconteçam.

O bug que você acabou de ver é uma delas: quando a segunda busca falhou, ninguém limpou `dados`. A
variável continuou com a lista velha, e o JSX que diz `{dados && <lista/>}` fez o trabalho dele.

**Ninguém escreveu esse comportamento. Ele emergiu.** É a diferença entre um bug que alguém digitou
e um bug que a modelagem permitiu.

## Por que o lado direito não consegue fazer isso

Ele guarda **um estado só**, com nome:

```js
const [estado, setEstado] = useState({ nome: 'OCIOSO' });
```

E a cada momento ele é substituído inteiro, nunca acumulado:

| `nome` | Significa |
|---|---|
| `OCIOSO` | ainda não pediu nada |
| `CARREGANDO` | pediu, está esperando |
| `ERRO` | não deu para buscar |
| `VAZIO` | buscou, respondeu, não há resultado |
| `COM_DADOS` | buscou e tem conteúdo |

São cinco situações e **exatamente uma** vale por vez. "Erro com dados na tela" deixa de ser um bug
a evitar: vira algo que o código **não consegue representar**.

> **A regra:** estado impossível que o modelo permite vai acontecer. Não porque alguém errou, mas
> porque nada impediu. Modelar como uma situação por vez elimina a categoria inteira de bug, em vez
> de corrigir um caso de cada vez.

---

## O quarto estado, que quase todo mundo esquece

Repare que `VAZIO` é diferente de `ERRO`, e diferente de `COM_DADOS`.

Buscar municípios de uma UF que não existe **não é erro**: o servidor entendeu, respondeu, e a
resposta é "nada". Quem trata isso como erro assusta o usuário à toa. Quem trata como sucesso mostra
uma lista vazia sem explicação, e a pessoa fica achando que a tela quebrou.

Clique no cenário **2** e compare os dois lados de novo. O da direita diz *"Nenhum resultado.
Confira se o que você pediu existe."* — que é a única frase honesta ali.

---

## Você já sabe disso, em outro contexto

No sistema que você está construindo, o ciclo de ressuprimento tem **19 status possíveis**, num enum,
e um ciclo está em exatamente um deles. Ninguém modelaria aquilo como
`estaSeparado`, `estaExpedido`, `estaRecebido` em booleanos separados — seria óbvio que isso
permitiria "expedido mas não separado".

**É o mesmo raciocínio.** A diferença é que no domínio ele parece natural, e na tela quase todo
mundo escorrega para os booleanos. O estado da interface merece o mesmo cuidado que o estado do
negócio.

---

## O que fazer, na ordem

### 1. Reproduza o bug
Cenário 1, depois cenário 3. Olhe a caixa **estado interno** dos dois lados no momento do erro:
à esquerda `erro: ERRO_HTTP` e `dados: lista de 27` convivendo; à direita, só `ERRO`.

### 2. Teste os quatro cenários nos dois lados
Preste atenção especial no cenário 2, que é o estado `VAZIO`.

### 3. Leia o `useEffect` dos dois painéis
Tem uma variável `vivo` nos dois, com um `return () => { vivo = false; }` no fim. Ela não faz nada
de visível hoje. **Ela é o assunto do Passo 4** — deixa passar por enquanto, mas repare que existe.

### 4. Quebre de propósito (obrigatório antes do Passo 4)

| Quebra | Como | O que observar |
|---|---|---|
| Conserte o jeito A | acrescente `setErro(null); setDados(null);` antes do `buscar()` | o bug some? some **esse** bug, ou a categoria inteira? |
| Estrague o jeito B | troque o `setEstado({nome:'ERRO'...})` por um que faça _spread_ do estado anterior (`{...estado, nome:'ERRO'}`) | o lado bom passa a mentir também? |
| Tire um estado | remova o ramo `VAZIO` do jeito B | o que a tela mostra no cenário 2, e isso é honesto? |

**A primeira é a mais importante, e a resposta é incômoda:** dá para consertar o jeito A. Só que
você conserta **aquele caminho**. Amanhã alguém acrescenta um quinto cenário e esquece de limpar de
novo. O jeito B não precisa de disciplina, precisa de estrutura.

---

## Critério de "terminei"

Você consegue responder, sem consultar:

1. Por que três booleanos permitem oito combinações e só quatro fazem sentido?
2. Qual a diferença entre `VAZIO` e `ERRO`, e por que a tela precisa distingui-los?
3. O bug do lado esquerdo foi digitado por alguém, ou emergiu?
4. Se você consertar o lado esquerdo com um `setDados(null)`, o problema acabou?

Se a 4 te fizer hesitar, o passo funcionou.

---

## Uma observação sobre este código não ser "React de verdade"

Aqui o React vem por CDN e o JSX é traduzido no próprio navegador. É ótimo para aprender e
**inadequado para produção**: cada carregamento traduz o código de novo, e o navegador avisa isso no
console.

Foi de propósito, para o passo caber em dois cliques e a atenção ficar no conceito. O ferramental de
verdade (Node, npm, um empacotador) entra mais para a frente, quando ele resolver um problema que
você já sentiu — do mesmo jeito que o React entrou agora, e não no Passo 1.
