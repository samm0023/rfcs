- Start Date: 2021-09-06
- RFC PR: TBD

# Summary

Frontend implementation for Artifact (creating and defining dynamic, custom entities).

The purpose of Artifact is to allow the definition, creation, and modification of custom entities with support from the Kanvas API. This solution is proposed on the basic need to quickly define new entities for data storage and querying without having to place unnecessary development burden on the backend team for simple things. This allows to clear bottlenecks and speed overall development of apps.

## Entities
* Interfaces: Defines the overall structure of the entity.
* Migrations: Historic schema changes of the entity.
* Lambdas: Defines custom API endpoints for GraphQL integrated functions.

# Example

## Implementation in Client JS

```ts
import Artifact from '@kanvas/artifact';
import TokenProvider from 'core/token-provider';
import HttpClient from 'core/http-client';
import Auth from 'modules/auth';
import ClientOptions from 'types/client-options';

export default class KanvasSDK {
  public readonly http: HttpClient;
  public readonly tokenProvider: TokenProvider;
  public readonly auth: Auth;
  public readonly artifact: Artifact;

  constructor(options: ClientOptions) {
    this.tokenProvider = new TokenProvider(options);
    this.http = new HttpClient(options, this.tokenProvider);
    this.auth = new Auth(this.http, this.tokenProvider);
    this.artifact = new Artifact(options.baseUrl);
  }
}
```

## Implementation in App

```ts
import KanvasSDK from '@kanvas/client-js';

const kanvas = new KanvasSDK({
  appKey: '<app key>',
  baseUrl: '<baseURL>'
});

kanvas.artifact('Vehicle').get();
```

# Motivation

We need to accelerate and streamline new entity implementations without having to wait for backend to deliver. This will also ensure that backend can focus on bigger and more important features and implementations.

By being able to both define our own custom entities and define our own API endpoints, frontend can focus on delivering at a much faster pace.

# Detailed design

Artifact will be its own standalone library, allowing it to be imported to any project and be used directly.

* Create entities (define the interfaces / types)
  * Create base properties (Text, Numbers, Relationships, etc...)
    * Limited to Elasticsearch types (maybe?)
  * Functions
    * Lambda
* Update entities
  * Properties:
    * Add
    * Remove
    * Update
  * Functions
    * Add
    * Remove
    * Update
  * Consume entities
    * In tandem with backend API
    * Support RESTful requests
    * Support GraphQL requests

# Trade Offs

* Learning GraphQL from both the frontend and backend aspects.
* Mid-term to Long-term implementation.
* Defining and implementing migrations.
* Learning Postgres.

# Alternatives

### Custom/Dynamic Entities

* Supabase
* Strapi
* Firebase

### Lambda

* Supabase
* Firebase
* Amazon
* Vercel

## Trade Offs

* Learning to use the tool.
* Integration with existing Kanvas structure/services.
* Cost.
* Uptime/response reliance.

# Unresolved questions

* Migration implementation.
* Lambda implementation and context (how/where to consume structure for data querying).
