- Start Date: 2022-01-01
- RFC PR: -

# Summary

KanvasAPI has a CRUD behavior library that we have developed throught the years in order to make it easier for Frontend to consume the API. It allows anyone without an specific knowledge to query data from any endpoint. They can filter, specify columns, and other cool magic (🎩). However, development cycles are slowing down implementations of new endpoint, and this in turn is affecting Frontend development speed.

We have to evolve this package to the next level. After looking and poking around we have found the solution we were looking for in the form of GraphQL.

Graphql provides the same features of our CRUD Behavior, and then some more, so we don't have to reinvent the wheel. It also provides a new challenge of having to migrate over our API's to this new standard.

Due to current limited resources this new integration will have to be made in a step by step process. In order to help us we will be using Hasura's services to provide the GraphQL interfaces, thus allowing Frontend developers to move at pace without having to wait for Backend to deliver new endpoints.

Once the denormalized structure has been created Backend will proceed to fill in the necessary data.

- Hasura will be our GraphQL interface.
- Kanvas API will use GraphQL as a replacement for ElasticSearch.
- Search will be move to AppSearch.
- Hasura will use remote schema and action transformers to manage CRUD and convert REST to GraphQL.
- Hasura will be used to query data.
- Kanvas will slowly start to implement full GraphQL CRUD support.

# Example

```php
<?php

class Leads extends BaseModel
{
  use GraphQLIndexable;

  public function afterSave()
  {
    $this->saveToGraphQL(\Hasura\Leads::class);
  }
}

trait GraphQLIndexable
{
  public function saveToGraphQL(string $postgresModel)
  {
    return HasuraIndex::add($postgresModel, $this);
  }
}

Class LeadsHasura extends PostgresBaseModel
{
  public string $title;
  public int $user_id;

  public function add(ModelInterface $entity)
  {
    $this->title = $entity->title;
    $this->users_id = $entity->user_id;
    return $this->saveOrFail();
  }
}

```

# Motivation

Follow [Pipe's use case](https://hasura.io/case-studies/pipe/), GraphQL has our query endpoints for fast iteration, and Hasura with Postgres as our no-SQL database to replace ElasticSearch.

# Detailed design

 - Use Hasura instead of developing our full GraphQL CRUD.
 - Use Postgres as ElasticSearch replacement for extendable queryable endpoints. 
 - Have native Hasura support integrated into Baka.

# Tradeoffs

What potential tradeoffs are involved with this proposal.

- Add another db to our connection layer
- Third party service for our GraphQL (this can be a win)
