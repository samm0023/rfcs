- Start Date: 2021-11-21
- RFC PR: (leave this empty)

# Summary

As we launch more kanvas apps for B2C , we are facing the need to manage moderation for any entity with public content within the community , therefore we see ourself needed to implement a general solution for our apps

# Example

Our apps will need to now add a new API

report.api.domain.com

Where we will provide the following methods:
- GET /v1/reports : Get all reports for the current app
- GET /v1/reports_types : Get all reports for the current app
  - Allow you to filter it by entity types (diff reports types for a post vs a comment)
- POST /v1/reports : create a new report for the given entity
- POST /v1/block_user : Block a given user from my list
- GET /v1/block_user : List of blocked users
- GET /v1/users/{id} : add property is_blocked to the user entity

Global Logic:

Blocked User
  - I blocked the user, he cant see my content but I can see his
    - User cant find each other, wont see user content across he platform
  - User property for is_blocked so user wont see it on the profile
  - what will happened with blocked user on notable?
  - When user is blocked remove the following status
  
Content entity 
  - now has a is_reported
  - only the reported user wont see the content

# Detailed design

Here you will find the DB diagram for the subsystem
https://dbdiagram.io/d/619ab2a202cf5d186b60f73c

## Backend
The package will provide diff interface and traits to add this methods to the dif entity's in the system that will allow modification

### Traits:
- CanBeModerated
  - getModerationStatus
  - isReported : we will have to add property to the entity tables
  - isReportedBy(User)
  - hideEntity(ContentInterface $entity)
<br />
- CanBeBlocked
  - isBlocked
- hasContent
  - isPublic

### Interface:
- ContentInterface
   - isPublic

### Workflow Hide Content
- Notification
- HideContent : it depends on the report type priority

## JSON Structure

POST /v1/reports
```json
{
  "entity_namespace": "Comments",
  "entity_id": 1,
  "report_type_id": 1,
  "description": "text",
  "title" : "title"
}
```

GET /v1/reports_types
```json
{
  "id": 1,
  "name": "Spam"
  "priority" : {
    "id": 1,
    "name": "High",
    "weight": 100
  }
}

```
POST /v1/block_user
```json
{
  "users_id": 1,
}
```

POST /v1/block_user
```json
{
  "users_id": 1,
}
```

GET /v1/block_user
```json
{
  "created_at": "2020-01-07 16:08:25",
  "displayname": "Cian",
  "firstname": "Cian",
  "id": 59,
  "is_follow": 0,
  "is_following": 0,
  "lastname": "Traynor",
  "photo": {
    "id": 2279,
    "filesystem_id": 2533,
    "name": "download.png",
    "field_name": "photo"
  },
  "total_followers": 577,
  "total_memos": 47
}
```

# Tradeoffs

What potential tradeoffs are involved with this proposal.

- Complexity adding a new subsystem
- Queries to filter out the user data
- Adding new traits to more models
- How will we handle flagging content as reported for each user?
- How will this system handle hiding all content of a blocked user?
  - Notifications
  - This is bidirectional , I wont be able to view your content and the blocked user wont be able to see mine
  - If I blocked a content creator what will happen to notable and recommended content?

# Alternatives

What are the alternatives?