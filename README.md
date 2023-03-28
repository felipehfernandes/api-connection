# Integração entre as Bases RD Station e Exact Sales

> Aqui vamos apontar e resolver os problemas de integração entre as bases [RD Station](https://plugcrm.net/) e [Exact Sales](https://app.exactspotter.com/Account/NormalLogin?email=fernando%40absolutasaude.com.br), duas bases de CRM utilizadas pela empresa [Absoluta Saúde](https://absolutasaude.com.br/)

## O Problema

O problema a ser resolvido está relacionado à importação da base de _leads_ da base RD para a base Exact, e são eles:

- **Importação incompleta do número de _leads_**: não foram importados todos os leads para a base Exact, sendo que temos 18197 _leads_ na base Exact e 32346 _leads_ na base RD. Assim, _temos 14149 leads que não foram importados para a base Exact_;
- **Importação incompleta das informações dos _leads_**: alguns _leads_ não foram importados com todas as informações para a base Exact, mesmo tendo essas informações na base RD.

## A Solução

Iremos solucionar esse problema realizando as conexões via API de ambas as bases, e assim, importar todos os _leads_ da base RD para a base Exact, e também, importar todas as informações dos _leads_ para a base Exact. Alguns cuidados serão tomados para que não haja duplicidade de _leads_ na base Exact, e também, para que não haja sobreposição de informações de _leads_ na base Exact.

## [Exact Sales](https://app.exactspotter.com/Account/NormalLogin?email=fernando%40absolutasaude.com.br)

> Temos **18197 leads** nessa base

### API

- Manual: [Exact Spotter V2 - API v3](https://exactsal.es/apiv3-spotterv2)
- Chave: https://api.exactspotter.com/v3/Leads

#### Limitações
- Limite de 500 leads por requisição
- Limite de 30 requisições a cada 20 segundos

#### Exemplo de Requisição

```
# GET Request para leads da Exact

import requests

headers = {
    'Content-Type': 'application/json',
    'token_exact': 'token'
}

response = requests.get(
    'https://api.exactspotter.com/v3/Leads', headers=headers)
```

### Tratamento dos Dados
> Para a construção da base de leads da Exact, foram necessários alguns ajustes nos dados. Esses ajustes foram realizados com o intuito de diminuir o tamanho do arquivo final e tornar possível a comparação das duas bases, e foram eles:

- Remover alguns itens não utilizados na comparação dos leads com a base RD
- Remover caracteres especiais do campo de CNPJ, deixando apenas os números e certificando que esse número tenha 14 caracteres.

### Criação da Base
> A base de leads da Exact foi criada com o intuito de ser utilizada para a comparação com a base RD, e para isso, dada a capacidade e limitações da API, foi realizada a requisição utilizando o método `GET` com o parâmetro `skip` para pular de 500 em 500 leads, e assim, foi possível obter todos os leads da base Exact.

## [RD Station](https://plugcrm.net/)

> Temos **32346 leads** nessa base

### API

- Manual: [Developers RD Station](https://developers.rdstation.com/reference/instruções-e-requisitos)
- Chave: https://crm.rdstation.com/api/v1/organizations?token=token

#### Limitações

- Limite de 200 leads por requisição
- Limite de 120 requisições por minuto

### Exemplo de Requisição

```
# GET Request para leads da RD

import requests

params = {
    'token': 'token'
}

response = requests.get(
    'https://crm.rdstation.com/api/v1/organizations', params=params)
```

### Criação da Base
> A base de leads da RD foi criada com o intuito de ser utilizada para a comparação com a base Exact, e para isso, dada a capacidade e limitações da API, foi realizada a requisição utilizando o método `GET` com os seguintes parâmetros:

- `organization_segment`: a base RD foi organizada diretamente na aplicação web com os segmentos divididos por data de criação do lead, dessa forma, foram criados segmentos com menos de 10000 leads, e assim, foi possível realizar a requisição para obter todos os leads da base RD;
- `page`: para cada segmento, podemos navegar pelas páginas de leads;
- `limit`: para cada página, podemos ter até 500 leads por requisição, então foi realizada a requisição de página em página até que não existisse mais leads para serem obtidos para aquele segmento.

# Atualizando Leads no Exact

> Atualizando leads na base do exact que foram criados, mas estão com informações incompletas. Para isso, iremos utilizar a API do Exact para atualizar os leads que estão com informações incompletas. Então, vamos:

1. Para cada lead da base Exact, vamos verificar se ele está na base RD;
2. Se ele estiver na base RD, vamos verificar se ele está com informações incompletas;
3. Se o lead estiver sem celular 1, celular 2 e endereço, vamos atualizar o lead com as seguintes informações da base RD:
    - `phone`
    - `phone2`
    - `address`
    - `city`
    - `state`
    - `country`
    - `cpfcnpj`
4. Se o lead estiver sem celular 1 e 2, vamos atualizar o lead com as seguintes informações da base RD:
    - `phone`
    - `phone2`
    - `cpfcnpj`

Esses procedimentos foram realizados para evitar que informações sejam sobrepostas na base Exact, e também, para evitar que informações sejam perdidas na base Exact.

# Criando Leads Exact

> Criando leads que existem na base RD e não existem na base Exact

- Será feita uma requisição para a criação do lead (`request_API`) e outra requisição para adicionar os contatos com o `id` do lead criado (`request_API_Contacts`)
- Para a página do *Lead/Oportunidade* principal, serão adicionados os números de celular vindos da base do RD:
    - Celular 1
    - Celular 2
- Para o perfil de *Contato*, serão adicionados os números de celular vindos da base do RD:
    - Cel e Cel 2 dos Sócios 1 e 2 (se existente)
