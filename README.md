# Uso-do-Amazon-dymamoDB

O Amazon DynamoDB é um banco de dados NoSQL (chave-valor e documento), mas que permite simular comportamentos relacionais através de técnicas de modelagem avançadas, como o Single Table Design. 

Embora ele não suporte JOINs nativos como o SQL, ele entrega performance de milissegundos em qualquer escala. Para o nosso cenário de E-commerce, aqui estão as boas práticas e conceitos que o expert aplicaria: 

1. Single Table Design (Uma Tabela para Tudo)
Em vez de criar várias tabelas (Cliente, Pedido, Produto), colocamos todos os itens em uma única tabela. Usamos nomes genéricos para as chaves:
PK (Partition Key): USER#123, ORDER#456, PRODUCT#789.
SK (Sort Key): Permite criar hierarquias e buscas por "começa com" (ex: buscar todos os itens de um pedido específico).

3. Índices Secundários (GSI e LSI)
Para realizar consultas que não usam a PK principal (ex: buscar um pedido pelo CPF em vez do ID_PEDIDO), usamos:
GSI (Global Secondary Index): Cria uma nova "visão" dos dados com uma PK/SK diferente. É aqui que o DynamoDB "ganha" flexibilidade relacional.

4. TTL (Time to Live) 
Ideal para o E-commerce! Você pode definir um tempo de expiração para itens como Carrinhos de Compras abandonados. O DynamoDB deleta os dados automaticamente sem custo de escrita, mantendo o banco limpo.

5. Streams e Triggers (Reatividade) 
O DynamoDB Streams captura toda alteração (Insert/Update) e pode disparar um AWS Lambda. No nosso cenário, isso serviria para enviar um e-mail de confirmação assim que o status do pedido mudasse para "Pago". 
Código de exemplo (Definição de Tabela via CLI/CloudFormation):
json
{
  "TableName": "EcommerceTable",
  "AttributeDefinitions": [
    { "AttributeName": "PK", "AttributeType": "S" },
    { "AttributeName": "SK", "AttributeType": "S" }
  ],
  "KeySchema": [
    { "AttributeName": "PK", "AttributeKeyType": "HASH" },
    { "AttributeName": "SK", "AttributeKeyType": "RANGE" }
  ],
  "BillingMode": "PAY_PER_REQUEST"
