# Exploração da API — PokéAPI / Pokémon e Tipos

## Resumo rápido

**O que é:** uma exploração manual da PokéAPI, com foco nos recursos `pokemon` e `type`, feita no Postman em 13/10/2026.

**Como foi feita:** antes de cada consulta, registrei o que esperava encontrar (a hipótese). Depois comparei com o que a API respondeu e investiguei as diferenças.

**O que mais chamou atenção:**

* A lista de Pokémon não vem como um array solto: vem dentro de um objeto, com o total (`count`) e links para a página anterior e a seguinte. Quem consome a API precisa ler `results`, e não a raiz.
* "Pokémon inexistente" e "página inexistente da lista" são tratados de forma diferente: o primeiro retorna `404`, o segundo retorna `200` com lista vazia.
* O `offset` é uma **posição na lista**, e não um ID. Os últimos itens da lista têm IDs acima de 10000.
* O tipo de um Pokémon vem como referência a outro recurso (nome + URL), e a relação se confirma nos dois sentidos: `pokemon/25` aponta para `type/13`, e `type/13` lista o Pikachu.
* O `POST` retornou `403`, mas o corpo era uma página de bloqueio da Cloudflare, e não uma resposta da API. O teste, sozinho, **não permite concluir nada sobre escrita**. A documentação diz que a API só aceita `GET`, mas não explica esse bloqueio.

**Para quem quiser aprofundar:** o restante do documento mostra as hipóteses, as evidências e o que não foi possível concluir.

---

## 1. Contexto

Esta exploração é o segundo laboratório da etapa de fundamentos do portfólio. O primeiro, com o JSONPlaceholder, tratou de HTTP, métodos, status codes e operações de escrita simuladas. Os conceitos básicos estão em [Exploração da API — JSONPlaceholder / Users](../jsonplaceholder/exploracao-users.md) e não são repetidos aqui.

> **Nota:** este documento é introdutório. Ele registra comportamentos observados; os critérios de aprovação serão definidos depois, nos casos de teste.

---

## 2. Objetivo

Nesta exploração, buscamos:

* ler com atenção a estrutura de uma resposta de API;
* entender como a paginação funciona (`limit`, `offset`, `count`, `next`, `previous`);
* observar como a API trata consultas por ID e por nome;
* observar como a API reage a recursos inexistentes e a páginas inexistentes;
* seguir a relação entre dois recursos e conferir se ela é coerente nos dois sentidos;
* separar o que foi **esperado**, o que foi **observado** e o que pode ser **concluído**.

---

## 3. API utilizada

**PokéAPI** — `https://pokeapi.co/api/v2/`

Foi escolhida por oferecer dados mais ricos e relacionados que os do JSONPlaceholder (objetos aninhados, referências entre recursos e listas paginadas), sem exigir autenticação. Como o foco aqui é **consulta**, o laboratório não repete o estudo de `POST`, `PUT`, `PATCH` e `DELETE` feito antes.

Recursos explorados: `pokemon` (item e lista paginada) e `type`.

---

## 4. Exploração realizada

Cada etapa registra a hipótese (quando houve), o que foi observado e a interpretação.

### 4.1 Consulta por ID e por nome

```http
GET /pokemon/25
GET /pokemon/pikachu
GET /pokemon/Pikachu
```

**Hipótese:** retorno `200` com um objeto contendo o Pokémon e seus detalhes. Não havia hipótese sobre o nome em minúsculas ou com inicial maiúscula.

**Observado:** as três consultas retornaram `200`, `Content-Type: application/json` e o mesmo objeto (o Pikachu).

**Interpretação:** nos casos testados, a API aceita ID e nome, e não diferenciou maiúscula de minúscula. Isso foi observado com um único nome. A documentação confirma que os recursos aceitam ID ou nome, mas não trata de maiúsculas e minúsculas, então esse comportamento ficou como observação, não como regra documentada.

---

### 4.2 Pokémon inexistente

```http
GET /pokemon/0
GET /pokemon/naoexiste
```

**Hipótese:** `404` com um objeto vazio no body.

**Observado:** as duas consultas retornaram `404`, com `Content-Type: application/json; charset=utf-8` e o body:

```json
{
    "status": 404,
    "message": "Not Found"
}
```

**Interpretação:** o status bateu com a hipótese, mas o body não estava vazio. A API devolve uma resposta estruturada para o erro, o que é relevante para quem consome: dá para tratar o erro lendo o corpo, e não só o status.

---

### 4.3 Lista de Pokémon

```http
GET /pokemon?limit=5&offset=0
```

**Hipótese:** uma lista com 5 objetos.

**Observado:** a raiz da resposta é um **objeto**, não um array. Dentro dele, `results` traz os 5 Pokémon, cada um apenas com `name` e `url`. Na raiz também aparecem `count` (1351 na data da consulta), `next` (link para a próxima página) e `previous` (`null`).

**Interpretação:** os 5 itens estavam lá, mas dentro de um objeto, então a hipótese bateu em parte. Cada item da lista é uma versão enxuta, bem diferente do objeto completo de `pokemon/25`, e a URL de cada item leva ao detalhe.

---

### 4.4 Paginação com `limit` e `offset`

Sem conhecimento prévio sobre `offset`, a hipótese ficou em aberto. Antes das consultas, o conceito foi estudado: `limit` define quantos itens vêm por página, e `offset` define quantos itens são pulados antes de começar.

| Consulta | Observado |
|---|---|
| `?limit=5&offset=5` | Itens de `charizard` (ID 6) a `caterpie` (ID 10); `previous` aponta para `offset=0` e `next` para `offset=10`. |
| `?limit=5&offset=1349` | Apenas 2 itens; `next: null`; `previous` aponta para `offset=1344`. |
| `?limit=5&offset=1351` | `200` com `results: []`; `next: null`; `previous` aponta para `offset=1346`. |
| `?limit=100000` | `200`, com os 1351 itens de uma só vez; `next` e `previous` nulos. |

**Interpretação:**

* O `count` bateu com o comportamento: 1351 − 1349 = 2 itens na última página, e 0 além do fim.
* A API aceitou `limit=100000` sem aplicar um teto, pelo menos para esse valor. O consumidor pode pedir tudo de uma vez, o que pode pesar. A documentação informa que o tamanho padrão da página é de 20 itens e que `limit` e `offset` servem para ajustar a paginação, mas não define um teto.
* O `offset` conta **posições na lista**, não IDs: os dois últimos itens (`baxcalibur-mega` e `meowstic-female-mega`) têm IDs 10325 e 10326. Pelos nomes, parecem formas alternativas de Pokémon, mas isso é uma suposição a confirmar na documentação.

---

### 4.5 Relação entre recursos: `pokemon` e `type`

```http
GET /pokemon/25
GET /type/13
GET /type/electric
```

**Observado em `pokemon/25`:** o objeto tem 21 campos na raiz (entre eles `abilities`, `moves`, `sprites`, `stats` e `types`). O campo `types` é uma lista com um item:

```json
{ "slot": 1, "type": { "name": "electric", "url": ".../type/13/" } }
```

O tipo vem como **referência** a outro recurso (nome e URL), e não como texto simples.

**Observado em `type/13`:** `200`, objeto `electric`, com o campo `pokemon` listando 114 Pokémon, entre eles `pikachu` (`slot: 1`). Pokémon com dois tipos aparecem com `slot: 2` neste recurso (14 casos).

**Hipótese sobre `type/electric`:** retornar o mesmo conteúdo que `type/13`. **Observado:** as duas respostas são idênticas.

**Interpretação:** a relação é consistente nos dois sentidos (`pokemon/25` → `type/13` → `pokemon/25`), e ID e nome funcionam também no recurso `type`.

---

### 4.6 Tentativa de `POST`

```http
POST /pokemon
```

Sem hipótese registrada, sem body.

**Observado:** status `403`, com `Content-Type: text/html; charset=UTF-8`. O corpo é uma página HTML da **Cloudflare** ("Sorry, you have been blocked"), não uma resposta JSON da API.

**Interpretação:** a requisição foi bloqueada por uma camada de segurança antes de qualquer resposta de API. O `403` não diz nada sobre se o endpoint aceita ou rejeita `POST`. O teste é **inconclusivo** para escrita. O que ele mostrou foi o valor de olhar o `Content-Type` e o corpo, e não só o status code.

**Conferência na documentação:** a documentação oficial descreve a PokéAPI como uma API "consumption-only", em que apenas o método `GET` está disponível nos recursos. Isso é coerente com o fato de o `POST` não ter funcionado, mas **não explica o `403` da Cloudflare**: a causa do bloqueio continua sem confirmação. O contrato (a documentação) diz que não há escrita; o comportamento observado foi um bloqueio de outra camada.

> O corpo da página contém o endereço de IP de quem fez a requisição e um identificador de requisição. Por isso, a evidência não foi anexada ao repositório.

---

## 5. Cenários explorados

| Cenário | Requisição | O que estamos investigando | Resultado observado |
|---|---|---|---|
| Consultar por ID | `GET /pokemon/25` | Consulta de um recurso por ID | `200` + objeto do Pikachu |
| Consultar por nome | `GET /pokemon/pikachu` | Consulta por nome | Mesma resposta do ID 25 |
| Nome com maiúscula | `GET /pokemon/Pikachu` | Sensibilidade a maiúsculas | Mesma resposta (caso testado) |
| ID inexistente | `GET /pokemon/0` | Recurso inexistente | `404` + `{"status": 404, "message": "Not Found"}` |
| Nome inexistente | `GET /pokemon/naoexiste` | Recurso inexistente | `404` + `{"status": 404, "message": "Not Found"}` |
| Primeira página | `GET /pokemon?limit=5&offset=0` | Estrutura da lista paginada | `200` + objeto com `count`, `next`, `previous`, `results` |
| Segunda página | `GET /pokemon?limit=5&offset=5` | Efeito do `offset` | Itens 6 a 10; links coerentes |
| Fim da lista | `GET /pokemon?limit=5&offset=1349` | Última página | 2 itens; `next: null` |
| Além do fim | `GET /pokemon?limit=5&offset=1351` | Página inexistente | `200` + `results: []` |
| `limit` alto | `GET /pokemon?limit=100000` | Limite de itens por página | `200` + todos os 1351 itens |
| Relação por ID | `GET /type/13` | Seguir a referência de `types` | `200` + tipo `electric` com o Pikachu na lista |
| Relação por nome | `GET /type/electric` | Consulta por nome do tipo | Resposta idêntica à de `type/13` |
| Escrita | `POST /pokemon` | Reação a uma operação de escrita | `403` com página da Cloudflare (inconclusivo) |

> **Observação:** esta tabela registra o comportamento observado em 13/10/2026. A API é pública e os dados podem mudar (o `count`, por exemplo). Expectativas e critérios de aprovação serão definidos nos casos de teste.

---

## 6. Análise de QA

### 6.1 Duas formas de "não encontrado"

Um Pokémon inexistente retorna `404`. Uma página inexistente da lista retorna `200` com `results: []`. Quem decide apenas pelo status code trata o segundo caso como sucesso e pode seguir adiante sem dados. É o mesmo princípio visto no JSONPlaceholder, em outra forma: "não encontrei" pode ser um erro ou uma resposta válida sem itens, conforme o recurso.

### 6.2 Estrutura da lista

A raiz da lista é um objeto, com `results` dentro dele. Um consumidor que espere um array na raiz quebra. Em testes automatizados, vale validar a presença e o tipo de `count`, `next`, `previous` e `results`, além do conteúdo.

### 6.3 Paginação e bordas

As bordas são onde a paginação costuma falhar, e foram as consultas que mais ensinaram: última página parcial, página além do fim, `next` nulo no fim e `previous` nulo no começo. Também vale registrar que `limit` sem teto é um ponto de atenção para desempenho.

### 6.4 Relações entre recursos

A resposta de `pokemon` entrega o tipo como referência, e o recurso `type` devolve a lista de Pokémon daquele tipo. Conferir os dois lados é um teste de consistência que não depende de nenhum dado "correto" externo: se o Pikachu aponta para `electric`, `electric` precisa listar o Pikachu.

### 6.5 Status code não conta a história inteira

O `403` do `POST` parecia dizer "a API recusa escrita", mas o `Content-Type` e o corpo mostraram que a resposta vinha da Cloudflare. Sem olhar esses dois itens, a conclusão teria sido errada.

---

## 7. Possíveis validações de teste

### Consulta de um Pokémon

* validar status code e `Content-Type`;
* validar que o ID retornado corresponde ao solicitado;
* validar que consulta por ID e por nome retornam o mesmo recurso;
* validar a presença e o tipo dos campos relevantes (`id`, `name`, `types`, `stats`);
* validar o formato de erro para ID e nome inexistentes.

### Lista e paginação

* validar a estrutura (`count`, `next`, `previous`, `results`);
* validar que a quantidade de itens respeita o `limit`;
* validar que `offset` avança a lista e que não há repetição entre páginas;
* validar a última página e a página além do fim;
* validar a coerência entre `next`, `previous` e os parâmetros enviados.

### Relações

* validar que as URLs de referência respondem com sucesso;
* validar a consistência nos dois sentidos (`pokemon` ↔ `type`).

---

## 8. Decisões e justificativas

* **Hipótese antes da consulta.** Cada consulta foi precedida de uma expectativa registrada, para comparar com o resultado e investigar divergências. Algumas hipóteses não bateram, e isso fez parte do aprendizado.
* **Foco em consulta.** Como a API é voltada à leitura, este laboratório não repete o estudo de métodos de escrita.
* **Sem insistir no bloqueio do `POST`.** O teste foi tratado como inconclusivo. Não foram feitas variações para contornar o bloqueio, por se tratar de uma API pública gratuita. A documentação reforça a cautela: pede para limitar a frequência das requisições, guardar os recursos em cache local e informa que quem descumpre a política de uso justo pode ter o IP banido de forma permanente.
* **Sem evidência com dados sensíveis.** O corpo do bloqueio da Cloudflare contém o IP de quem fez a requisição e não foi anexado.
* **Postman** foi usado para ver status, headers e body de forma clara.

---

## 9. Aprendizados

* a diferença entre `limit` e `offset`, e como `count`, `next` e `previous` descrevem a paginação;
* a raiz de uma resposta pode ser um objeto que contém a lista;
* `404` para um recurso e `200` com lista vazia são formas diferentes de "não encontrado";
* `offset` é posição, e não ID;
* a relação entre recursos pode ser conferida nos dois sentidos;
* olhar `Content-Type` e corpo evita conclusões erradas a partir de um status code;
* uma hipótese que não bate é informação útil, desde que seja investigada.

---

## 10. Próximos passos

A próxima exploração será a **SWAPI**, outra API de leitura com recursos relacionados, o que permite comparar o comportamento de paginação e de relações. Depois, as consultas desta exploração servirão de base para testes com ferramentas como Postman/Newman e Bruno.

Pendente neste laboratório: a causa exata do `403` da Cloudflare no `POST` continua sem explicação. A documentação diz que só `GET` está disponível, mas não descreve esse bloqueio. Em testes futuros, vale lembrar do pedido de uso justo da API (poucas requisições e cache local), principalmente em testes automatizados.