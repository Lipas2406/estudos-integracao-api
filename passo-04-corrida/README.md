# Passo 4 — Condição de corrida

**Tempo:** um bloco de 25 a 50 min. **Precisa instalar:** nada.

Abre o `index.html` com dois cliques e aperta o botão azul.

---

## O que acontece

O botão seleciona **Minas Gerais** e, 300 ms depois, troca para **Roraima**. Duas buscas ficam no ar
ao mesmo tempo. Roraima é rápida e responde primeiro; Minas demora e responde depois.

Resultado medido, com o painel da esquerda:

```
Roraima (RR)
853 municípios
 • Abadia dos Dourados
 • Abaeté
 • Abre Campo
 • Acaiaca
```

**O título diz Roraima e a lista é de Minas Gerais.** Roraima tem 15 municípios; ali estão 853.

O registro de chegada conta a história:

```
1. chegou RR (15)  -> escreveu na tela
2. chegou MG (853) -> escreveu na tela      <- painel sem defesa
3. chegou MG (853) -> DESCARTADO            <- painel com cleanup
```

A resposta certa chegou primeiro e foi **sobrescrita pela errada**, que chegou atrasada. Nada no
código percebeu, porque nada no código perguntou.

---

## Por que isso acontece

Cada busca é uma promessa que resolve quando quiser. Quem escreveu isto:

```js
buscarMunicipios(estado).then(lista => setDados(lista));
```

está dizendo: *"quando esta resposta chegar, coloque na tela"* — e nunca: *"...se ela ainda for a
resposta que interessa"*.

Enquanto o usuário clica em uma coisa por vez e espera, funciona. Basta ele trocar de ideia antes de
a primeira chegar para a ordem de chegada deixar de ser a ordem dos cliques.

> **A regra:** a última resposta a chegar não é necessariamente a última que foi pedida. Se o seu
> código assume que é, ele está apostando na rede.

## Por que o cleanup resolve

O `useEffect` pode devolver uma função. **O React executa essa função antes de rodar o efeito
seguinte.** É o gancho onde a busca anterior é marcada como abandonada:

```js
useEffect(() => {
  let atual = true;                       // esta busca ainda interessa?
  buscar(...).then(lista => {
    if (!atual) return;                   // chegou tarde: ignora
    setDados(lista);
  });
  return () => { atual = false; };        // <- roda quando `estado` muda
}, [estado]);
```

A resposta de Minas **continua chegando** — ninguém cancelou nada. Ela só encontra um `atual` que já
virou `false` e é descartada em silêncio, como mostra a linha 3 do registro.

### O `AbortController` faz o mesmo, e mais um pouco

No Passo 2 você usou `AbortController` para timeout. Ele também serve aqui, e é melhor num aspecto:
além de ignorar a resposta, ele **cancela a requisição**, liberando a conexão.

A flag `atual` é mais simples de entender e resolve o problema da tela; o `AbortController` resolve
também o desperdício de rede. Em código de produção, normalmente se usa o segundo. Entender o
primeiro é o que faz o segundo parecer óbvio.

---

## Uma honestidade sobre o atraso

Os tempos reais desta API, medidos antes de montar o passo:

| UF | Municípios | Tempo | Tamanho |
|---|---:|---:|---:|
| MG | 853 | 0,16 s | 382 KB |
| SP | 645 | 0,18 s | 284 KB |
| RR | 15 | 0,13 s | 6 KB |

A diferença entre a mais lenta e a mais rápida é de **cerca de 30 ms**. Com isso, a corrida
aconteceria de vez em quando, dependendo do humor da rede — e um bug que aparece de vez em quando
não serve para aprender.

Por isso os estados marcados em laranja têm um atraso declarado no código. **O atraso é artificial;
o mecanismo do bug é inteiramente real.**

E isso já é a lição seguinte: desmarque a caixa "simular servidor lento" e tente reproduzir clicando
rápido. Você provavelmente não vai conseguir. **A dificuldade de reproduzir não é evidência de que
não existe** — é a característica que torna esse tipo de bug caro. Ele aparece no cliente, no celular
ruim, na rede da empresa, e nunca na sua máquina.

---

## Isto não é assunto de front-end

O mesmo raciocínio aparece do outro lado do sistema. Se dois usuários pedirem um número de registro
ao mesmo tempo, e o código for *ler o último, somar um, gravar*, os dois podem receber o mesmo
número. Ninguém errou de digitação: a modelagem assumiu que as coisas acontecem uma de cada vez.

Aqui a consequência é uma lista errada na tela. Lá é um registro duplicado no banco. **Concorrência
não é um tópico de banco de dados nem de front-end — é o que acontece sempre que algo espera.**

---

## O que fazer, na ordem

### 1. Aperte o botão azul e leia o registro
Confira que a linha "DESCARTADO" existe. Ela é a prova de que a resposta velha chegou e foi
recusada, e não de que ela nunca chegou.

### 2. Faça na mão
Clique em Minas Gerais e, antes de 2,5 s, clique em Roraima. Mesmo resultado.

### 3. Desmarque "simular servidor lento" e tente de novo
Clique o mais rápido que conseguir entre dois estados. Se conseguir reproduzir uma vez em vinte, já
provou o ponto.

### 4. Quebre de propósito (obrigatório antes do Passo 5)

| Quebra | Como | O que observar |
|---|---|---|
| Tire o cleanup | apague o `return () => { atual = false; }` do painel B | ele vira o painel A? |
| Troque a defesa | em vez da flag, use `AbortController` com `signal` | a linha "DESCARTADO" ainda aparece? Por quê? |
| Inverta os atrasos | dê 2500 ao RR e 0 ao MG | quem mente agora, e a mentira é a mesma? |

**A segunda é a mais instrutiva.** Com `AbortController`, a promessa é **rejeitada** em vez de
resolvida, então o `.then` nem roda — e o registro muda de comportamento. Duas soluções corretas com
efeitos observáveis diferentes.

---

## Critério de "terminei"

Você consegue responder, sem consultar:

1. Por que a resposta que chega por último não é necessariamente a que foi pedida por último?
2. O cleanup cancela a requisição antiga?
3. Por que um bug de corrida costuma aparecer no usuário e não em quem programou?
4. Que problema do lado do servidor tem a mesma causa?

A resposta da 2 é **não**, e se você hesitou, vale reler a seção do cleanup.

---

## Nota: um erro que cometi montando este passo

A primeira versão do botão de demonstração fazia `setEstado(MG)` direto. Quando Minas Gerais já
estava selecionada, **nada acontecia**: o React compara o valor novo com o atual e, sendo o mesmo
objeto, não re-renderiza nem dispara o efeito. A corrida nem começava, e eu levei duas tentativas
para entender por que a demonstração não demonstrava.

A correção foi zerar a seleção antes (`setEstado(null)`), e o comentário disso está no código. Fica
como lembrete de que "pedir de novo a mesma coisa" e "pedir uma coisa diferente" são eventos
diferentes para o React.
