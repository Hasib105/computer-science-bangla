# GraphQL পরিচিতি

## 🎯 GraphQL কি?

**GraphQL** হলো Facebook-এর তৈরি একটি query language যা API থেকে precisely যা দরকার তাই আনতে পারে।

```
REST Problem:
GET /users/123        → সব user data
GET /users/123/posts  → সব posts
GET /posts/1/comments → সব comments
(৩টা request, অনেক extra data)

GraphQL Solution:
query {
  user(id: 123) {
    name
    posts(limit: 5) {
      title
      comments(limit: 3) {
        text
      }
    }
  }
}
(১টা request, শুধু যা দরকার)
```

## 📊 GraphQL vs REST

```
┌────────────────────┬────────────────────────────────────┐
│        REST        │            GraphQL                 │
├────────────────────┼────────────────────────────────────┤
│ Multiple endpoints │ Single endpoint                    │
│ Over-fetching      │ Get exactly what you need          │
│ Under-fetching     │ Nested queries                     │
│ Server decides     │ Client decides                     │
│ Versioning needed  │ Schema evolution                   │
└────────────────────┴────────────────────────────────────┘
```

## 🔧 GraphQL Schema

```graphql
# Type definitions
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
}

# Queries (Read)
type Query {
  user(id: ID!): User
  users: [User!]!
  post(id: ID!): Post
}

# Mutations (Write)
type Mutation {
  createUser(name: String!, email: String!): User!
  createPost(title: String!, content: String!, authorId: ID!): Post!
}
```

## 📨 Query Examples

### Basic Query
```graphql
query {
  user(id: "123") {
    name
    email
  }
}

Response:
{
  "data": {
    "user": {
      "name": "আহমেদ",
      "email": "ahmed@mail.com"
    }
  }
}
```

### Nested Query
```graphql
query {
  user(id: "123") {
    name
    posts {
      title
      comments {
        text
        author {
          name
        }
      }
    }
  }
}
```

### Mutation
```graphql
mutation {
  createUser(name: "করিম", email: "karim@mail.com") {
    id
    name
  }
}
```

## ⚡ কখন GraphQL?

```
✓ Mobile apps (bandwidth sensitive)
✓ Complex, nested data
✓ Multiple clients with different needs
✓ Rapid frontend development

✗ Simple CRUD APIs
✗ File uploads
✗ Caching-heavy applications
✗ Real-time (WebSocket better)
```

## 📚 পরবর্তী টপিক

[gRPC পরিচিতি →](./03-grpc.md)
