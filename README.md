# QA Portfolio — Martins Neto

Portfólio de Quality Assurance em construção, com foco em **testes de API** e **automação**. Cada estudo é documentado com o raciocínio por trás (o que eu esperava, o que observei, o que concluí), não só com o resultado.

## Quem sou eu

Analista de QA com foco em automação de testes para aplicações Web, Mobile (Android) e APIs REST, atuando em projetos de e-commerce. No dia a dia trabalho com Robot Framework, Selenium, Appium, Postman/Newman, BDD (Gherkin) e pipelines de CI/CD (Azure DevOps e GitHub Actions). Certificação CTFL (ISTQB Foundation Level).

- LinkedIn: [martinsneto8080](https://www.linkedin.com/in/martinsneto8080/)
- GitHub: [MartinsNeto80](https://github.com/MartinsNeto80)

> Os projetos em que atuo profissionalmente são de clientes e não estão neste repositório. Este espaço reúne estudos e laboratórios próprios, feitos em APIs públicas.

## Para quem é este repositório

O objetivo é **demonstrar com evidência** as ferramentas que uso no trabalho e as que estou aprendendo: cada uma ganha seu laboratório, com resultado e conclusão.

Foi pensado para dois tipos de leitura:

- **Se você é recrutador ou líder técnico:** leia só o resumo no topo de cada documento (cerca de 1 minuto). Ele diz o que foi testado, como e o que ficou concluído.
- **Se você é QA e quer aprender:** siga o documento completo. Cada exploração mostra a hipótese, a execução, as observações e as conclusões, incluindo o que não funcionou como eu esperava.

## O que já existe

| Laboratório | O que foi feito | Onde ler |
|---|---|---|
| JSONPlaceholder (Users) | Exploração manual da API com navegador e Postman: GET, POST, PUT, PATCH e DELETE, status codes, headers, e a observação de que a escrita é simulada e não persiste. | [Laboratório](./01-fundamentos/api/jsonplaceholder/README.md) · [Exploração completa](./01-fundamentos/api/jsonplaceholder/exploracao-users.md) |
| PokéAPI (Pokémon e Tipos) | Exploração manual de consulta e paginação (`limit`, `offset`, `count`), erros, e relação entre recursos conferida nos dois sentidos. Inclui um teste de `POST` inconclusivo, bloqueado pela Cloudflare. | [Laboratório](./01-fundamentos/api/pokeapi/README.md) · [Exploração completa](./01-fundamentos/api/pokeapi/exploracao-pokeapi.md) |

## O que está planejado

Ainda **não implementado**; entra no repositório à medida que for feito e documentado.

- Novos laboratórios de API: SWAPI e ServeRest
- Ferramentas: Postman/Newman, cURL, Bruno, Karate, Robot Framework
- Automação Web e E2E: Playwright e Cypress
- Performance: k6 e JMeter
- Segurança básica (OWASP) e CI/CD com GitHub Actions

## Como eu documento

Contexto → objetivo → estratégia → hipótese → execução → observação → investigação → conclusão. Só afirmo aqui o que o repositório demonstra.

## Estrutura

```text
qa-portfolio/
└── 01-fundamentos/
    └── api/
        ├── jsonplaceholder/
        └── pokeapi/
```

---

Sugestões e feedback são bem-vindos, por issue ou pelo LinkedIn.