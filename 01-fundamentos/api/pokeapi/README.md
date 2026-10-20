# PokéAPI

Exploração manual de uma API REST de consulta, com dados relacionados e listas paginadas, usada como laboratório para praticar a leitura crítica de respostas e a validação de paginação e de relações entre recursos.

## O que foi explorado

- Consulta de um recurso por ID e por nome
- Recurso inexistente e formato da resposta de erro
- Listas paginadas: `limit`, `offset`, `count`, `next` e `previous`
- Comportamento nas bordas da paginação (última página e página além do fim)
- Estrutura de objetos aninhados
- Relação entre recursos (`pokemon` e `type`), conferida nos dois sentidos
- Uma tentativa de `POST`, que se mostrou inconclusiva (veja abaixo)

## Principais aprendizados

- a raiz de uma resposta de lista pode ser um objeto que contém a lista, e não um array;
- "não encontrei o recurso" (`404`) e "a página não tem itens" (`200` com lista vazia) são situações diferentes;
- `offset` é uma posição na lista, e não um ID;
- uma relação entre recursos pode ser validada nos dois sentidos;
- `Content-Type` e corpo da resposta evitam conclusões erradas: um `403` veio de uma camada de segurança (Cloudflare), e não da API;
- uma hipótese que não se confirma é informação útil, desde que seja investigada.

## Documentação da exploração

A análise detalhada, com hipóteses, resultados observados, investigações, limitações e decisões, está em:

[Exploração da API — PokéAPI / Pokémon e Tipos](./exploracao-pokeapi.md)

Os conceitos básicos de HTTP, métodos e status codes estão no laboratório anterior e não são repetidos aqui: [JSONPlaceholder](../jsonplaceholder/README.md).

## Limitações

- A API é de leitura. O teste de `POST` foi bloqueado pela Cloudflare e **não permite concluir nada sobre escrita**.
- Os dados refletem a consulta feita em 13/10/2026. A API é pública e o total de itens pode mudar.
- A API pede uso justo (poucas requisições e cache local). Isso será considerado quando as consultas forem automatizadas.

## Próximos passos

As consultas e as observações deste laboratório servem de base para a exploração da SWAPI, outra API de leitura com recursos relacionados, e depois para a automação com Postman/Newman e outras ferramentas.