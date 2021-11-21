- Start Date: 2021-11-21
- RFC PR: (leave this empty)

# Summary

As we launch more kanvas apps for B2C , we are facing the need to manage moderation for any entity with public content within the community , therefore we see ourself needed to implement a general solution for our apps

# Example

Our apps will need to now add a new API

report.api.domain.com

Where we will provide the following methods:
- GET /v1/reports : Get all reports for the current app
- POST /v1/reports : create a new report for the given entity
- POST /v1/block_user : Block a given user from my list

Also the system will provide diff traits to add this methods to the dif entity's in the system that will allow modification

- CanBeModerated
  - getModerationStatus
  - isReported
  - isReportedBy(User)
<br />
- CanBeBlocked
  - isBlocked


# Detailed design

Here you will find the DB diagram for the subsystem
https://dbdiagram.io/d/619ab2a202cf5d186b60f73c

# Tradeoffs

What potential tradeoffs are involved with this proposal.

- Complexity adding a new subsystem
- Queries to filter out the user data
- Adding new traits to more models

# Alternatives

What are the alternatives?