# Project Marketplace Ecommerce

Projeto de criação de um martketplace ecommerce com a Stack MEAN.


## Equipe  

Esse projeto está sendo iniciado e mantido pelos seguintes alunos do curso Be MEAN:  

- [Ednilson Amaral](https://github.com/ednilsonamaral)  
- [Eliel das Virgens](https://github.com/hc3)  
- [Jean Nascimento](https://github.com/suissa) (Professor da Webschool)  
- [Leonardo Ribeiro](https://github.com/leoribeirowebmaster)  
- [Lucas Baquião](https://github.com/lucastafarelbs)  
- [Rodrigo Galhardo](https://github.com/rodrigogalharo)


## Documentação

A documentação completa desse projeto é possível ser visualizada na [Wiki desse repositório]().

## Banco de Dados

Iremos utilizar uma arquitetura híbrida com **alguns** bancos NoSQL:

- MongoDB: Model
- Redis: Cache / Session / Events
- Neo4J: Relacionamentos
- Elasticsearch: Busca textual

### Model

O MongoDb será utilizado pois será nele que modelaremos nossa entidade/módulo de uma forma atômica e idêntica à programação.

Os dados serão sempre escritos no MongoDB e a partir dele será enviado para seus bancos relacionados.

Por exemplo:

> Definimos um modelo para uma *View atualizável* que será nosso Cache, onde esses dados sempre serão buscados antes no Cache e caso eles não existam será buscado no MongoDB para atualizar essa *View*. Mas os dados precisam sair do Cache para a resposta e não diretamente do MongoDB.

Como o MongoDB não foi feito para ter relacionamentos e cada *join* feito nele é muito custoso iremos mover toda essa funcionalidade para um banco de dados que foi feito especialmente para dados relacionados: de **GRAFOS**.

> Cada relacionamento definido no *Model* será convertido para grafos no Neo4J, por isso as buscas que não estiverem no Cache serão diretamente buscadas no banco de relacionamentos, que é o Neo4J. 

Outra funcionalidade que o MongoDB também não foi feito para é buscas textuais, logo iremos utilizar um banco específico para isso: o Elasticsearch.

> E toda busca que envolva textos enviados pelo cliente será buscada no Elasticsearch, o qual já terá pré-definido um *Schema* para facilitar sua busca.

#### Modelagem Atômica

Eu tenho um certo problema com esse negócio de modularizar as coisas, tanto que minha metodologia chamada [Atomic Design]() separa não apenas as entidade/módulos mas sim **cada campo/propriedade** que é utilizado.

Quando eu criei essa metodologia ainda não tinha percebido o quão perfeita ela caiu em ideias de arquitetura que já tinha há anos.

Na modelagem ralacional precisamos **normalizar** a base para fazer *"do jeito certo* e *normalizar* a base nada mais é que separar seus dados para que não haja duplicidade entre eles, pois bem eu quero levar esse conceito um pouco mais a fundo e **trazer isso para os grafos**. Tendo em vista que cada campo meu já é um módulos atômico que independe em qual sistema está sendo usado nada mais claro que colocar cada um como um como um vértice e que qualqer uso dele seja direto independendo do contexto em que está sendo utilizado.


#### Exemplo com Product

Vamos fazer um exemplo **BEM SIMPLES** com produto:

```js
{
  name: String,
  sku: String,
  description: String,
  price: {
    value: Number,
    currency: String
  },
  characteristics: [
    {
      name: String, // width
      value: Number, // 200
      unit: String // g
    }
  ],
  tags: [
    String
  ],
  created_at: Date,
  updated_at: Date,
}
```

Lembrando que cada campo é um módulo independente o módulo de `price` na verdade é uma composição de 2 outros:

- valueNumber: Number
- currency: String

Logo **sempre** que a propriedade `price` for utilizada ela **sempre** terá essas 2 propriedades assim definidas:

```js

const value = require('/atomic-modules/fields/valueNumber') // value: Number
const value = require('/atomic-modules/fields/valueNumber') // currency: String

price: {
  value,
  currency
}
```

Primeiramente definimos um modelo para nosso Cache o qual deve conter o resultado das buscas mais comuns, então vamos cachear a primeira busca que acontecerá **sempre** que o usuário entrar no Ecommerce:

- listar os últimos 50 produtos

> Tudo bem mas com quais dados?

Para responder essa questão precisamos saber como o Ecommerce mostrará as informações, mas vamos definir as mais básicas:

- name
- description
- price

Agora toda vez que 1 produto for **adicionado** ou **modificado** esses dados novos terão que ir diretamente para o Cache, logo após o sucesso da alterção no MongoDb.

Então para o Cache:

```js
{
  name: String,
  description: String,
  price: {
    value: Number,
    currency: String
  },
  tags: [
    String
  ],
  created_at: Date,
  updated_at: Date
}
```

*ps: Iremos deixar os campos `created_at` e `updated_at` para facilitar na ordenação e busca.*


#### Relacionamentos

Todo relacionamento entre entidades no Model será convertido em relações entre vértices no banco de dados específico para isso, o Neo4J.

## Licença
