# GRAPHQL

## GraphQL Basics
- Query vs Mutation vs Subscription
- Schema definition language (SDL)
- Resolvers
- DataLoader pattern

## Apollo Client/Server
- Caching strategies
- Error handling
- Optimistic updates
- Real-time subscriptions

## Common Interview Questions
1. GraphQL vs REST API?
2. N+1 problem trong GraphQL?
3. Caching trong GraphQL?
4. Schema stitching và Federation?

## Code Examples
```graphql
type Query {
  users: [User!]!
  user(id: ID!): User
}

type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
}
```
