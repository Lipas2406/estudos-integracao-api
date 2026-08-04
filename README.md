# Estudos de integração de APIs

Trilha prática para aprender **integração entre sistemas**, do zero, um conceito por vez.

Não é um tutorial para ler: cada passo é uma coisa que roda, com um exercício obrigatório de
**quebrar de propósito** antes de avançar. A ideia é simples — só entendi de verdade quando consigo
explicar por que quebrou.

Os dados vêm das **APIs públicas do IBGE**, que são abertas, estáveis e não pedem cadastro.

---

## Por que este repositório existe

Estou construindo um sistema de orquestração de ressuprimento de estoque que, para entregar o que
promete, vai precisar conversar com outro sistema interno da empresa. A API existe, mas não é
formalmente documentada: o acesso é conversado caso a caso com quem a mantém.

Ou seja, o gargalo não é escrever o código da chamada. É **saber o que perguntar**. Este repositório
é onde eu construo esse vocabulário, com uma API pública, antes de gastar o tempo de quem mantém a
outra.

---

## A trilha

| Passo | Conceito | Framework |
|:---:|---|---|
| 1 | A requisição crua: status, cabeçalhos e corpo são coisas separadas | nenhum |
| 2 | Tudo que dá errado — e por que `200` não significa que deu certo | nenhum |
| 3 | Os quatro estados de qualquer integração: carregando, erro, sucesso e **sucesso vazio** | React |
| 4 | Chamadas dependentes e condição de corrida | React |
| 5 | Volume: paginação, filtro e busca | React |
| 6 | Autenticação, e por que a chave nunca fica no frontend | React |
| 7 | Traduzir o aprendizado em perguntas técnicas objetivas | — |

Os dois primeiros passos são de propósito **sem framework**. `fetch`, status, cabeçalho e erro
funcionam igual em React, Vue ou HTML puro — misturar as duas aprendizagens faz confundir o que é
conceito de integração com o que é comportamento de framework. O React entra no passo 3, quando o
problema que ele resolve já é palpável.

---

## Como rodar

**Passos 1 e 2:** abrir o `index.html` da pasta. Não precisa instalar nada, nem servidor.

**Passos 3 em diante:** instruções na pasta de cada um.

Cada passo tem um `README.md` próprio com o que ele ensina, o que fazer, o que quebrar e o critério
objetivo de "terminei". Abrindo a pasta do passo aqui no GitHub, ele aparece renderizado logo abaixo
da lista de arquivos.

---

## Coisas que já aprendi, e que motivaram o formato

**`200` não quer dizer que deu certo.** Pedir os municípios de uma UF que não existe devolve status
`200` com uma lista vazia. Quem trata `200` como sucesso mostra "nenhum município encontrado" quando
o erro real foi digitar a UF errada — a tela afirma uma coisa que os dados não sustentam.

**O `fetch` não rejeita quando o status é de erro.** Ele só rejeita quando a requisição não
aconteceu. Um `404` chega como resposta normal, e o `try/catch` sozinho não pega. Pior: se o corpo do
erro vier vazio, quem chama `.json()` recebe um `SyntaxError` falando de JSON — uma mensagem que
aponta para o sintoma e esconde a causa.

**O `fetch` não tem prazo.** Sem `AbortController`, ele espera indefinidamente. Não ter prazo não é
ser rápido: é não ter a capacidade de desistir.

**O JavaScript não enxerga todos os cabeçalhos da resposta.** O navegador expõe ao código apenas uma
lista curta; o resto chega, existe, e é invisível para você. Descobri escrevendo errado no material
do passo 1 e corrigindo depois de rodar. A nota do erro ficou lá, à vista, porque ensina mais do que
o texto certo teria ensinado.

---

## Estado

Em construção, um passo por vez. Passos 1 e 2 prontos.
