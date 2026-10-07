# Exploração da API — JSONPlaceholder / Users

## Resumo rápido

**O que é:** uma exploração manual da API JSONPlaceholder, com foco no recurso `users` e algumas consultas ao recurso `posts`.

**Como foi feita:** as primeiras consultas `GET` foram feitas no navegador, e as operações de escrita (`POST`, `PUT`, `PATCH` e `DELETE`) no Postman.

**O que mais chamou atenção:**

* `POST` e `DELETE` retornaram sucesso (`201` e `200`), mas o efeito esperado não aconteceu. O usuário "criado" não foi encontrado depois, e o usuário "excluído" continuou existindo. Foi preciso fazer uma consulta de confirmação para perceber isso.
* `/users/1` e `/users?id=1` encontram o mesmo usuário, mas devolvem estruturas diferentes (objeto e array).
* "Não encontrei o recurso" (`404`) e "a busca não teve resultados" (`200` + `[]`) são situações diferentes.
* A escolha entre `PUT` e `PATCH` depende da intenção da operação, e minha hipótese inicial precisou ser revista.

**Para quem quiser aprofundar:** o restante do documento mostra o raciocínio, as hipóteses e as evidências de cada etapa.

---

## 1. Contexto

Esta exploração faz parte da etapa inicial do portfólio de QA e tem como objetivo consolidar os fundamentos de testes de API antes da utilização de ferramentas de teste e automação.

A proposta não é apenas executar requisições, mas observar o comportamento da API, entender o propósito de cada endpoint e identificar quais informações são relevantes para uma futura estratégia de testes.

> **Nota:** este documento é introdutório e conceitual. Ele serve de base para as próximas etapas, que reaproveitarão estes mesmos cenários com outras ferramentas.

---

## 2. Objetivo

Nesta exploração, buscamos:

* compreender a estrutura de uma API REST;
* realizar requisições HTTP manualmente;
* diferenciar recursos, endpoints e parâmetros;
* diferenciar path parameters de query parameters;
* analisar requests e responses;
* observar diferentes status codes;
* analisar a estrutura dos dados retornados;
* identificar cenários positivos e negativos;
* relacionar o comportamento observado com possíveis validações de QA.

---

## 3. API utilizada

**JSONPlaceholder**

A API foi escolhida para esta primeira exploração por disponibilizar recursos REST simples, permitindo estudar os fundamentos de API Testing sem introduzir inicialmente regras de negócio complexas.

Ela é o primeiro de alguns laboratórios planejados no portfólio. Cada API tem uma função diferente: aqui o foco é entender HTTP, requests, responses, JSON e CRUD. Outras APIs entrarão depois, como o **ServeRest**, para trabalhar persistência real, autenticação e regras de negócio.

O foco desta etapa foi o recurso `users`, com algumas consultas complementares ao recurso `posts`.

---

## 4. Exploração realizada

### 4.1 Consulta da coleção de usuários

```http
GET /users
```

**Objetivo:** consultar a coleção de usuários.

**Resultado observado:**

* requisição realizada com sucesso;
* retorno de uma lista de usuários;
* response representada em JSON.

A resposta representa uma coleção, portanto sua estrutura é um **array de objetos**.

---

### 4.2 Consulta de um usuário específico

```http
GET /users/1
```

**Objetivo:** consultar o usuário identificado pelo ID `1`.

**Resultado observado:**

* `200 OK`;
* retorno de um objeto JSON;
* o usuário retornado possui o ID solicitado.

Neste caso, o `1` é um **path parameter**.

A estrutura pode ser representada como:

```text
/users/{id}
       ↓
       1
```

A operação representa a consulta de um recurso específico.

---

### 4.3 Consulta de um usuário inexistente

```http
GET /users/99999
```

**Objetivo:** verificar o comportamento da API quando o recurso solicitado não existe.

**Resultado observado:**

* `404 Not Found`;
* body representado como um objeto JSON vazio:

```json
{}
```

A requisição é válida, porém o recurso identificado pelo ID `99999` não foi encontrado.

Vale separar duas coisas: o `{}` é o que **observamos**; "o usuário não existe" é a **interpretação** mais natural desse resultado. Em um cenário profissional, o comportamento esperado para um ID inexistente deve vir do contrato da API, e não da nossa intuição.

Este cenário demonstra a diferença entre:

* uma requisição inválida;
* uma requisição válida para um recurso inexistente.

---

### 4.4 Consulta utilizando query parameter

```http
GET /users?id=1
```

**Objetivo:** consultar usuários utilizando o ID como filtro.

Neste caso:

```text
Endpoint:
 /users

Query parameter:
 id

Valor:
 1
```

**Resultado observado:**

* `200 OK`;
* retorno de uma lista;
* a lista contém o usuário de ID `1`.

Apesar de retornar o mesmo usuário observado em `/users/1`, a estrutura da resposta é diferente.

```text
/users/1
→ objeto

/users?id=1
→ array contendo um objeto
```

---

### 4.5 Consulta utilizando outro filtro

```http
GET /users?username=Bret
```

**Objetivo:** consultar usuários utilizando `username` como critério de filtragem.

**Resultado observado:**

* retorno do usuário correspondente ao filtro;
* estrutura de lista, assim como ocorre em consultas filtradas sobre a coleção.

Esse comportamento reforça a diferença entre consultar um recurso específico e consultar uma coleção utilizando filtros.

---

### 4.6 Consulta de posts

Também foi explorado o recurso `posts`.

```http
GET /posts
```

Retorna a coleção de posts.

Foi utilizado o filtro:

```http
GET /posts?userId=1
```

Neste caso:

```text
Endpoint:
 /posts

Query parameter:
 userId

Valor:
 1
```

A utilização do query parameter restringe os resultados aos posts relacionados ao `userId` informado.

Também foi realizada a consulta:

```http
GET /posts?userId=99999
```

Nesse cenário, a API retornou uma lista vazia:

```json
[]
```

Isso demonstra que uma consulta válida pode não encontrar registros e ainda assim retornar uma resposta HTTP válida.

---

### 4.7 Criação de usuário com POST

Foi realizada uma tentativa de criação de usuário utilizando:

```http
POST /users
```

#### Request

**Headers relevantes:**

```http
Content-Type: application/json
```

**Request body:**

```json
{
  "name": "Martins Neto",
  "username": "martins.qa",
  "email": "martins@example.com"
}
```

#### Resultado observado

A API retornou:

```text
201 Created
```

O response body apresentou os dados enviados juntamente com um identificador:

```json
{
  "name": "Martins Neto",
  "username": "martins.qa",
  "email": "martins@example.com",
  "id": 11
}
```

Também foi observado o header:

```http
Location: https://jsonplaceholder.typicode.com/users/11
```

A presença do `Location` chamou atenção por indicar uma referência ao recurso associado ao identificador retornado.

#### Validação complementar

Com base no identificador retornado pelo `POST`, foi realizada uma nova requisição:

```http
GET /users/11
```

O resultado foi:

```text
404 Not Found
```

com um response body vazio:

```json
{}
```

Esse resultado levantou uma dúvida importante: se a criação havia retornado `201 Created`, por que o recurso não estava disponível para consulta posteriormente?

#### Investigação

A documentação oficial do JSONPlaceholder esclarece que os recursos enviados por operações de escrita, como `POST`, não são realmente criados no servidor. A resposta é simulada como se a operação tivesse sido realizada.

Isso explica o comportamento observado:

```text
POST /users
     ↓
201 Created
     ↓
response simulado
     ↓
GET /users/11
     ↓
404 Not Found
```

Portanto, o `201 Created` observado representa o comportamento simulado da API e não pode ser utilizado como evidência de persistência real do usuário.

#### Análise

Este cenário mostra por que uma resposta não deve ser interpretada isoladamente.

O `201 Created` inicialmente indicava uma criação bem-sucedida. Mas, ao fazer uma validação complementar utilizando o identificador retornado, encontramos um resultado diferente.

A análise da documentação permitiu diferenciar uma possível "falha" de persistência de uma característica conhecida da API utilizada.

Esse cenário também reforçou a importância de conhecer as limitações do ambiente de teste antes de utilizar seus resultados como evidência.

---

### 4.8 Atualização com PUT

Foi realizada uma tentativa de atualização do usuário `1` utilizando:

```http
PUT /users/1
```

Foi enviado somente o campo que seria alterado:

```json
{
  "name": "Martins Neto"
}
```

#### Hipótese inicial

A ideia inicial era utilizar `PUT` mesmo enviando apenas o campo que seria alterado.

Durante a análise, porém, foi percebido que esse raciocínio não representava bem a diferença entre `PUT` e `PATCH`. A necessidade de alterar somente parte dos dados está mais relacionada ao propósito do `PATCH`.

#### Resultado observado

A API retornou:

```text
200 OK
```

e retornou uma representação do recurso com o identificador `1`.

O comportamento observado foi específico do JSONPlaceholder e não deve ser utilizado como regra universal para definir como toda API deve implementar `PUT`.

#### Aprendizado

O principal aprendizado desse teste foi perceber que a escolha do método HTTP deve considerar a intenção da operação e o contrato da API, e não somente o resultado apresentado pelo Postman.

De forma geral:

```text
PUT
→ atualização/substituição da representação do recurso

PATCH
→ alteração parcial do recurso
```

A definição exata do comportamento, dos campos obrigatórios e da resposta deve ser determinada pelo contrato da API.

---

### 4.9 Atualização parcial com PATCH

Para comparar o comportamento, foi realizada a mesma alteração utilizando:

```http
PATCH /users/1
```

**Request body:**

```json
{
  "name": "Martins Neto"
}
```

#### Resultado observado

A API retornou:

```text
200 OK
```

e apresentou o recurso com o nome alterado, mantendo os demais dados apresentados pela resposta.

#### Análise

Neste cenário, o `PATCH` representa melhor a intenção da operação: atualizar somente a informação que precisa ser modificada.

Isso também corrigiu a hipótese inicial levantada durante a exploração do `PUT`.

A escolha do `PATCH` não deve ser justificada simplesmente como "não precisar enviar campos desnecessários", mas pela intenção de realizar uma modificação parcial do recurso, conforme definido pelo contrato da API.

---

### 4.10 Exclusão com DELETE

Foi realizada uma tentativa de exclusão do usuário `1`:

```http
DELETE /users/1
```

#### Hipótese

Antes da execução, foi levantada a hipótese de que a API poderia retornar:

* um status indicando sucesso;
* uma mensagem ou confirmação da exclusão;
* sem necessidade de enviar o ID no body, já que ele está identificado no path.

Também foi definido que a melhor forma de validar a exclusão seria consultar novamente o mesmo recurso.

#### Resultado observado

A API retornou:

```text
200 OK
```

com um body vazio:

```json
{}
```

Nesse momento, ainda não seria possível concluir que o usuário havia sido realmente excluído. O `{}` é apenas o body da resposta da operação, e não uma confirmação de que o recurso deixou de existir.

#### Validação complementar

Foi realizada uma nova requisição:

```http
GET /users/1
```

O resultado foi:

```text
200 OK
```

com o objeto do usuário retornado normalmente.

#### Investigação e conclusão

O resultado demonstrou que o recurso continuava disponível após o `DELETE`.

Esse comportamento está de acordo com a característica do JSONPlaceholder de simular operações de escrita, em vez de realizar alterações persistentes nos dados.

O fluxo observado foi:

```text
DELETE /users/1
     ↓
200 OK + {}
     ↓
GET /users/1
     ↓
200 OK + usuário
     ↓
recurso continua disponível
```

Esse cenário reforçou novamente que o status code da operação, sozinho, não é suficiente para comprovar que o efeito funcional esperado realmente aconteceu.

---

### 4.11 Conclusão da exploração

O JSONPlaceholder é adequado para explorar:

* métodos HTTP;
* estrutura de requests e responses;
* headers;
* body;
* status codes;
* parâmetros;
* formatos de dados;
* comportamento de operações de escrita simuladas.

Por não persistir efetivamente os recursos criados, alterados ou excluídos, ele não deve ser utilizado para validar cenários que dependam de persistência real ou do ciclo completo de vida de um recurso.

Para esses cenários, será utilizada uma API com comportamento persistente, como o **ServeRest**.

---

## 5. Cenários explorados

| Cenário                       | Requisição                 | O que estamos investigando                                         | Resultado observado                           |
| ----------------------------- | -------------------------- | ------------------------------------------------------------------ | --------------------------------------------- |
| Consultar usuários            | `GET /users`               | Comportamento da consulta da coleção de usuários                   | Coleção de usuários retornada                 |
| Consultar usuário existente   | `GET /users/1`             | Consulta de um recurso específico utilizando path parameter        | `200 OK` + objeto JSON                        |
| Consultar usuário inexistente | `GET /users/99999`         | Comportamento quando o recurso solicitado não existe               | `404 Not Found` + `{}`                        |
| Filtrar usuário por ID        | `GET /users?id=1`          | Consulta da coleção utilizando `id` como query parameter           | `200 OK` + array contendo o usuário           |
| Filtrar por username          | `GET /users?username=Bret` | Consulta da coleção utilizando `username` como filtro              | Usuário correspondente retornado em uma lista |
| Filtrar posts por usuário     | `GET /posts?userId=1`      | Filtragem de posts utilizando `userId`                             | Posts correspondentes ao usuário retornados   |
| Filtrar posts sem resultados  | `GET /posts?userId=99999`  | Comportamento de uma consulta válida sem registros correspondentes | `200 OK` + `[]`                               |
| Criar usuário                 | `POST /users`              | Comportamento de uma operação de criação                           | `201 Created` + ID retornado                  |
| Validar criação               | `GET /users/11`            | Verificar se o recurso criado está disponível posteriormente       | `404 Not Found` + `{}`                        |
| Atualizar usuário             | `PUT /users/1`             | Comportamento de atualização utilizando `PUT`                      | `200 OK` + representação do recurso           |
| Atualizar parcialmente        | `PATCH /users/1`           | Comportamento de alteração parcial                                 | `200 OK` + recurso retornado                  |
| Excluir usuário               | `DELETE /users/1`          | Comportamento da operação de exclusão                              | `200 OK` + `{}`                               |
| Validar exclusão              | `GET /users/1`             | Verificar se o recurso realmente deixou de existir                 | `200 OK` + usuário existente                  |

> **Observação:** esta tabela registra o comportamento observado durante a exploração. Expectativas e critérios de aprovação serão definidos separadamente nos casos de teste, com base no contrato ou requisito aplicável.

---

## 6. Análise de QA

### 6.1 Path parameter x Query parameter

Durante a exploração, foi possível observar duas formas diferentes de localizar um usuário:

```http
GET /users/1
```

e:

```http
GET /users?id=1
```

Embora ambas possam retornar o mesmo usuário, representam intenções diferentes.

```text
/users/1
→ consulta de um recurso específico

/users?id=1
→ consulta da coleção utilizando um filtro
```

Essa diferença também pode alterar a estrutura da response:

```text
/users/1
→ objeto

/users?id=1
→ array
```

Essa diferença é relevante para qualquer sistema consumidor que dependa de uma estrutura específica de dados.

---

### 6.2 Status code e comportamento

O status code não deve ser analisado isoladamente.

No cenário:

```http
GET /users/99999
```

a API retornou:

```text
404 Not Found
```

Esse comportamento é coerente com a situação observada de um recurso inexistente.

Já em:

```http
GET /posts?userId=99999
```

a API retornou:

```text
200 OK
```

com:

```json
[]
```

Nesse caso, a consulta foi válida, mas não encontrou registros.

Isso demonstra que:

> **"Não encontrei dados" e "a requisição apresentou erro" são situações diferentes.**

---

### 6.3 Estrutura da response

Também foi observado que respostas relacionadas ao mesmo recurso podem possuir estruturas diferentes dependendo da operação.

Uma consulta direta:

```http
GET /users/1
```

retorna um objeto.

Uma consulta filtrada:

```http
GET /users?id=1
```

retorna um array.

Isso importa para quem consome a resposta. Um sistema que espera um objeto acessaria `user.name`; se receber um array, precisaria acessar `user[0].name`. O mesmo vale para os casos "sem resultado": `{}` (objeto vazio) e `[]` (lista vazia) podem significar "nenhum dado", mas são estruturas diferentes e podem quebrar um consumidor que espera a outra.

Em testes automatizados, essa diferença precisa ser considerada, pois não basta validar apenas o conteúdo. A estrutura da resposta também pode fazer parte do contrato esperado.

---

### 6.4 Status de sucesso não garante efeito funcional

Os testes de `POST` e `DELETE` permitiram observar uma situação importante.

No `POST`:

```text
POST /users
→ 201 Created
→ ID 11 retornado
→ GET /users/11
→ 404 Not Found
```

No `DELETE`:

```text
DELETE /users/1
→ 200 OK
→ GET /users/1
→ 200 OK + usuário
```

Nos dois casos, a resposta inicial indicava sucesso, mas as validações complementares mostraram que o efeito esperado de persistência não estava ocorrendo.

Isso reforça que:

> **Uma resposta tecnicamente bem-sucedida não significa necessariamente que o comportamento funcional esteja correto.**

A validação deve considerar o contrato, o objetivo da operação e, quando aplicável, o estado do recurso antes e depois da ação.

---

### 6.5 PUT x PATCH

A exploração também ajudou a diferenciar a intenção dos métodos `PUT` e `PATCH`.

Inicialmente, foi levantada a hipótese de utilizar `PUT` para alterar somente o nome do usuário.

Após analisar a intenção da operação, foi percebido que:

```text
PUT
→ atualização/substituição da representação do recurso

PATCH
→ alteração parcial do recurso
```

Portanto, quando o objetivo é modificar apenas parte do recurso, `PATCH` tende a representar melhor a intenção, desde que o contrato da API siga essa semântica.

O teste foi importante justamente porque a hipótese inicial foi confrontada com a execução e posteriormente revisada.

---

## 7. Possíveis validações de teste

A exploração permitiu identificar algumas validações que poderiam fazer parte de uma estratégia de testes.

### Consulta de usuário

* validar status code;
* validar que o ID retornado corresponde ao ID solicitado;
* validar estrutura da resposta;
* validar presença dos campos esperados;
* validar formato dos campos relevantes;
* validar que o recurso inexistente não retorna outro usuário.

### Consulta filtrada

* validar se o filtro realmente altera o conjunto de resultados;
* validar que os registros retornados atendem ao filtro;
* validar comportamento quando nenhum registro atende aos critérios;
* validar estrutura da resposta.

### Criação

* validar status code;
* validar estrutura do response;
* validar dados retornados;
* validar identificador retornado;
* quando houver persistência real, consultar o recurso criado posteriormente;
* validar o header `Location`, quando fizer parte do contrato.

### Atualização

* validar status code;
* validar que o campo esperado foi alterado;
* validar que os demais dados foram preservados quando essa for a regra;
* diferenciar atualização completa de alteração parcial;
* validar o comportamento de campos obrigatórios.

### Exclusão

* validar status code;
* validar o comportamento da response;
* quando houver persistência real, consultar o recurso após a exclusão;
* validar se o recurso não está mais disponível;
* considerar o comportamento definido pelo contrato para consultas posteriores.

---

## 8. Decisões e justificativas

Para esta primeira exploração, foi utilizada uma abordagem manual antes da introdução de ferramentas como Postman ou automação.

A decisão foi intencional: primeiro compreender o comportamento da API e os elementos de uma requisição e resposta HTTP, para posteriormente utilizar ferramentas que facilitem a execução e a repetição dos testes.

As consultas `GET` iniciais foram feitas no navegador. Para as operações `POST`, `PUT`, `PATCH` e `DELETE`, o navegador deixa de ser suficiente, e o Postman foi utilizado porque permitiu visualizar de forma clara:

* método HTTP;
* URL;
* parâmetros;
* headers;
* body;
* status code;
* response.

Também não foi assumido que uma determinada forma de consulta ou método HTTP é universalmente "melhor".

A escolha depende da intenção da operação e do contrato definido pela API.

Um exemplo disso ocorreu durante a comparação entre `PUT` e `PATCH`: a hipótese inicial foi confrontada com a execução e posteriormente revisada.

---

## 9. Aprendizados

Esta exploração permitiu consolidar:

* diferença entre endpoint e recurso;
* diferença entre path parameter e query parameter;
* estrutura de request e response;
* utilização de métodos HTTP;
* interpretação de status codes;
* diferença entre objeto e array JSON;
* diferença entre recurso inexistente e consulta sem resultados;
* importância de validar a estrutura da resposta;
* importância de diferenciar comportamento esperado de comportamento observado;
* diferença de intenção entre `PUT` e `PATCH`;
* importância de validar o efeito de uma operação, e não somente sua resposta;
* importância de conhecer as características e limitações da API utilizada para testes.

Também foi reforçado um princípio importante para testes:

> **Uma resposta tecnicamente bem-sucedida não significa necessariamente que o comportamento funcional esteja correto.**

A validação deve considerar o contrato e o objetivo da operação.

Outro aprendizado importante foi perceber que uma hipótese inicial pode estar incorreta e ainda assim ser útil para o processo de teste. No caso de `PUT` e `PATCH`, a execução ajudou a revisar a ideia inicial e compreender melhor a intenção de cada método.

---

## 10. Próximos passos

Com os principais métodos HTTP explorados no JSONPlaceholder, a próxima etapa será aprofundar os fundamentos antes de partir para automação.

A partir disso, serão aprofundados:

* headers;
* body;
* autenticação;
* autorização;
* cenários positivos e negativos;
* validações de contrato;
* casos de teste;
* análise de requisitos;
* decisão sobre o que deve ou não ser automatizado.

Posteriormente, a exploração será levada para uma API com comportamento mais próximo de um sistema real, como o ServeRest, permitindo trabalhar também com persistência, autenticação e regras de negócio.