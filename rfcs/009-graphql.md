- Start Date: (fill in today's date, YYYY-MM-DD)
- RFC PR: (leave this empty)

# Summary

To make it easier for frontend to consume our Kanvas API we have develop over the years the CRUD Behavior library, it allows anybody with knowledge of how it works to query the data from endpoints. They can filter, specify the columns and cool magic. But its getting old and showing it age making development slower for the frontend. 

Seeing this we now how to evolve and take this package to the next level. After looking around we can see the industry as found the solution we are looking for, in the form of GraphQL.

Graphql provides the same feature of our CRUD Behavior and much more, so we don't have to reinvent the wheel. But it provides a new challenge of having to migrate over our API's to the new standard.

Knowing we have to provide all the feature we currently have for fast development and understanding our limited resource we have decided to take a step by step approach in order to allow use for fast iteration.

So we are going to start using Hasura to provide use with our graphql interface allowing frontend devs to move without the need of backend to create the query endpoints. Once both teams have found the denormalize structure they can implement and backend will fill up the info

	- Hasura.io will be our graphql interface
	- Kanvas backend will use graphql as a replacement for elasticsearch
	- For search we will move to app search
	- Hasura will use remote schema and action transformers to manage CRUD and convert REST to graphql
	- Hasura will main use be to start to query data
	- Kanvas will start to implement slowing full graphql CRUD support

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

Follow pipe use case https://hasura.io/case-studies/pipe/  , use graphql has our query endpoints for fast iteration and hasura with postgres as our noSQL database to replace elastic.

# Detailed design

 - Use Hasura instead of developing our full graphql CRUD
 - Use postgresSQL as our elasticsearch replacements for extendable queryable endpoints 
 - Had native Hasura support to baka

# Tradeoffs

What potential tradeoffs are involved with this proposal.

- Add another db to our connection layer
- Third party service for our graphql (this can be a win)
