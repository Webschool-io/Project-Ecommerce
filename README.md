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

#### Relacionamentos

Todo relacionamento entre entidades no Model será convertido em relações entre vértices no banco de dados específico para isso, o Neo4J.

## Licença
