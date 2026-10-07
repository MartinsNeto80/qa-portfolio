# JSONPlaceholder

Exploração de uma API REST utilizada como laboratório para consolidar os fundamentos de testes de API.

## O que foi explorado

- Métodos HTTP: GET, POST, PUT, PATCH e DELETE
- Path parameters
- Query parameters
- Requests e responses
- Headers e body
- Status codes
- Estruturas JSON
- Validação do comportamento após uma operação
- Análise de cenários positivos e negativos

## Principais aprendizados

Durante a exploração, alguns pontos se destacaram:

- uma mesma informação pode ser consultada de maneiras diferentes, com impactos na estrutura da resposta;
- o status code precisa ser analisado dentro do contexto da operação;
- uma resposta HTTP bem-sucedida não necessariamente comprova o efeito funcional esperado;
- o comportamento observado deve ser confrontado com o contrato e a documentação da API;
- operações simuladas precisam ser diferenciadas de operações com persistência real.

## Documentação da exploração

A análise detalhada, incluindo requisições realizadas, resultados observados, investigações, decisões e aprendizados, está disponível em:

[Exploração da API — JSONPlaceholder / Users](./exploracao-users.md)

## Próximos passos

Os fundamentos observados aqui serão utilizados em outras APIs e cenários, permitindo comparar comportamentos, identificar diferenças de contrato e avançar gradualmente para a construção dos testes.