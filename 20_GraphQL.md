# GRAPHQL - INTERVIEW PREPARATION

## 1. GraphQL Core Concepts

### What is GraphQL?
- **Query language** cho APIs và runtime để thực thi queries
- **Type system** để mô tả dữ liệu
- **Single endpoint** thay vì multiple endpoints như REST
- **Client-driven** - client quyết định data cần thiết

### Core Operations
```graphql
# Query - Read data
type Query {
  users: [User!]!
  user(id: ID!): User
}

# Mutation - Modify data  
type Mutation {
  createUser(input: UserInput!): User!
  updateUser(id: ID!, input: UserInput!): User
  deleteUser(id: ID!): Boolean!
}

# Subscription - Real-time data
type Subscription {
  userCreated: User!
  messageAdded(chatId: ID!): Message!
}
```

### Schema Definition Language (SDL)
```graphql
# Scalar types
scalar DateTime

# Object types
type User {
  id: ID!
  name: String!
  email: String!
  createdAt: DateTime!
  posts: [Post!]!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
  comments: [Comment!]!
}

# Input types
input UserInput {
  name: String!
  email: String!
}

# Enums
enum PostStatus {
  DRAFT
  PUBLISHED
  ARCHIVED
}

# Interfaces
interface Node {
  id: ID!
}

# Unions
union SearchResult = User | Post | Comment
```

## 2. Resolvers và Execution

### Resolver Functions
```javascript
// GraphQL resolvers
const resolvers = {
  Query: {
    users: () => User.findAll(),
    user: (parent, { id }) => User.findById(id),
  },
  
  Mutation: {
    createUser: async (parent, { input }) => {
      const user = await User.create(input);
      pubsub.publish('USER_CREATED', { userCreated: user });
      return user;
    }
  },
  
  Subscription: {
    userCreated: {
      subscribe: () => pubsub.asyncIterator(['USER_CREATED'])
    }
  },
  
  // Field resolvers
  User: {
    posts: (parent) => Post.findByUserId(parent.id),
    fullName: (parent) => `${parent.firstName} ${parent.lastName}`
  }
};
```

### Resolver Chain
```javascript
// Parent resolver truyền data xuống child resolver
const resolvers = {
  Query: {
    user: () => ({ id: 1, name: 'John' }) // Parent
  },
  User: {
    // parent = { id: 1, name: 'John' }
    posts: (parent) => Post.findByUserId(parent.id) // Child
  }
};
```

## 3. Common Problems & Solutions

### N+1 Problem
```javascript
// ❌ N+1 Problem - Multiple DB queries
const resolvers = {
  Query: {
    users: () => User.findAll() // 1 query
  },
  User: {
    // N queries (1 for each user)
    posts: (user) => Post.findByUserId(user.id)
  }
};

// ✅ Solution 1: DataLoader
const DataLoader = require('dataloader');

const postLoader = new DataLoader(async (userIds) => {
  const posts = await Post.findByUserIds(userIds);
  return userIds.map(id => posts.filter(post => post.userId === id));
});

const resolvers = {
  User: {
    posts: (user) => postLoader.load(user.id)
  }
};

// ✅ Solution 2: Include in original query
const resolvers = {
  Query: {
    users: () => User.findAll({ include: ['posts'] })
  }
};
```

### Error Handling
```javascript
import { GraphQLError } from 'graphql';

const resolvers = {
  Query: {
    user: async (parent, { id }) => {
      const user = await User.findById(id);
      
      if (!user) {
        throw new GraphQLError('User not found', {
          extensions: {
            code: 'USER_NOT_FOUND',
            argumentName: 'id'
          }
        });
      }
      
      return user;
    }
  }
};

// Custom error handling
class ValidationError extends GraphQLError {
  constructor(message, field) {
    super(message, {
      extensions: {
        code: 'VALIDATION_ERROR',
        field
      }
    });
  }
}
```

## 4. Apollo Server Implementation

### Basic Setup
```javascript
import { ApolloServer } from '@apollo/server';
import { startStandaloneServer } from '@apollo/server/standalone';

const typeDefs = `
  type Query {
    hello: String
  }
`;

const resolvers = {
  Query: {
    hello: () => 'Hello World!'
  }
};

const server = new ApolloServer({
  typeDefs,
  resolvers,
  // Context cho authentication
  context: ({ req }) => ({
    user: getUser(req.headers.authorization)
  })
});

const { url } = await startStandaloneServer(server, {
  listen: { port: 4000 }
});
```

### Middleware & Plugins
```javascript
const server = new ApolloServer({
  typeDefs,
  resolvers,
  plugins: [
    // Caching plugin
    responseCachePlugin(),
    
    // Custom plugin
    {
      requestDidStart() {
        return {
          willSendResponse(requestContext) {
            console.log('Response:', requestContext.response);
          }
        };
      }
    }
  ],
  
  // Error formatting
  formatError: (error) => {
    console.log('GraphQL Error:', error);
    return new Error('Internal server error');
  }
});
```

## 5. Apollo Client

### Basic Setup
```javascript
import { ApolloClient, InMemoryCache, gql } from '@apollo/client';

const client = new ApolloClient({
  uri: 'http://localhost:4000',
  cache: new InMemoryCache()
});

// Query
const GET_USERS = gql`
  query GetUsers {
    users {
      id
      name
      email
    }
  }
`;

// Mutation
const CREATE_USER = gql`
  mutation CreateUser($input: UserInput!) {
    createUser(input: $input) {
      id
      name
      email
    }
  }
`;

// Subscription
const USER_SUBSCRIPTION = gql`
  subscription OnUserCreated {
    userCreated {
      id
      name
      email
    }
  }
`;
```

### React Integration
```jsx
import { useQuery, useMutation, useSubscription } from '@apollo/client';

function Users() {
  const { loading, error, data, refetch } = useQuery(GET_USERS, {
    pollInterval: 5000, // Poll every 5s
    fetchPolicy: 'cache-and-network'
  });
  
  const [createUser, { loading: creating }] = useMutation(CREATE_USER, {
    // Update cache after mutation
    update: (cache, { data: { createUser } }) => {
      const { users } = cache.readQuery({ query: GET_USERS });
      cache.writeQuery({
        query: GET_USERS,
        data: { users: [...users, createUser] }
      });
    },
    // Optimistic update
    optimisticResponse: {
      createUser: {
        __typename: 'User',
        id: 'temp-id',
        name: 'Loading...',
        email: 'loading@example.com'
      }
    }
  });
  
  // Subscription
  useSubscription(USER_SUBSCRIPTION, {
    onSubscriptionData: ({ subscriptionData }) => {
      // Handle new user
    }
  });
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return (
    <div>
      {data.users.map(user => (
        <div key={user.id}>{user.name}</div>
      ))}
    </div>
  );
}
```

## 6. Advanced Topics

### Caching Strategies
```javascript
// Apollo Client Cache
const cache = new InMemoryCache({
  typePolicies: {
    User: {
      fields: {
        posts: {
          // Merge strategy for pagination
          merge(existing = [], incoming) {
            return [...existing, ...incoming];
          }
        }
      }
    }
  }
});

// Server-side caching
import responseCachePlugin from 'apollo-server-plugin-response-cache';

const server = new ApolloServer({
  plugins: [
    responseCachePlugin({
      sessionId: (requestContext) => 
        requestContext.request.http.headers.get('session-id'),
      
      // Cache based on user
      sessionId: (requestContext) => {
        return requestContext.contextValue.user?.id || null;
      }
    })
  ]
});
```

### Schema Stitching & Federation
```javascript
// Schema Federation
// User service
const userTypeDefs = gql`
  extend type Query {
    user(id: ID!): User
  }
  
  type User @key(fields: "id") {
    id: ID!
    name: String!
    email: String!
  }
`;

// Post service  
const postTypeDefs = gql`
  extend type Query {
    posts: [Post!]!
  }
  
  type Post {
    id: ID!
    title: String!
    author: User!
  }
  
  extend type User @key(fields: "id") {
    id: ID! @external
    posts: [Post!]!
  }
`;
```

### File Upload
```javascript
// Server
import { GraphQLUpload, graphqlUploadExpress } from 'graphql-upload';

const typeDefs = gql`
  scalar Upload
  
  type Mutation {
    uploadFile(file: Upload!): String!
  }
`;

const resolvers = {
  Upload: GraphQLUpload,
  Mutation: {
    uploadFile: async (parent, { file }) => {
      const { filename, createReadStream } = await file;
      const stream = createReadStream();
      
      // Save file logic
      return filename;
    }
  }
};

// Client
const UPLOAD_FILE = gql`
  mutation UploadFile($file: Upload!) {
    uploadFile(file: $file)
  }
`;

function FileUpload() {
  const [uploadFile] = useMutation(UPLOAD_FILE);
  
  const handleUpload = (file) => {
    uploadFile({
      variables: { file }
    });
  };
}
```

## 7. Common Interview Questions

### 1. GraphQL vs REST?
**Trả lời:**
- **GraphQL**: Single endpoint, client quyết định data, strong type system, real-time subscriptions
- **REST**: Multiple endpoints, server quyết định data structure, caching dễ hơn
- **Khi dùng GraphQL**: Complex data relationships, mobile apps cần optimize bandwidth
- **Khi dùng REST**: Simple CRUD, caching requirements cao, team không familiar với GraphQL

### 2. N+1 Problem là gì và cách giải quyết?
**Trả lời:**
- **Problem**: 1 query chính + N queries phụ cho từng item
- **Solutions**:
  - DataLoader pattern (batching + caching)
  - Include related data trong query gốc
  - Query optimization ở database level

### 3. Caching strategies trong GraphQL?
**Trả lời:**
- **Client-side**: Apollo Client InMemoryCache, normalized cache
- **Server-side**: Query-level caching, field-level caching
- **CDN**: Caching GET requests for persisted queries
- **Database**: Query optimization, connection pooling

### 4. Authentication trong GraphQL?
**Trả lời:**
```javascript
// Context-based auth
const server = new ApolloServer({
  context: ({ req }) => {
    const token = req.headers.authorization;
    const user = verifyToken(token);
    return { user };
  }
});

// Resolver-level auth
const resolvers = {
  Query: {
    secretData: (parent, args, { user }) => {
      if (!user) throw new GraphQLError('Unauthorized');
      return getSecretData();
    }
  }
};

// Schema directives
directive @auth(role: String) on FIELD_DEFINITION

type Query {
  adminData: String @auth(role: "ADMIN")
}
```

### 5. Schema evolution và versioning?
**Trả lời:**
- **Additive changes**: Thêm fields, types (safe)
- **Non-breaking**: Optional arguments, deprecate fields
- **Breaking changes**: Remove fields (avoid), change field types
- **Best practices**: Use deprecation, maintain backwards compatibility

## 8. Performance & Best Practices

### Query Complexity Analysis
```javascript
import depthLimit from 'graphql-depth-limit';
import costAnalysis from 'graphql-cost-analysis';

const server = new ApolloServer({
  plugins: [
    depthLimit(10), // Max query depth
    costAnalysis({
      maximumCost: 1000,
      createError: (max, actual) => {
        return new Error(`Query cost ${actual} exceeds maximum ${max}`);
      }
    })
  ]
});
```

### Security Considerations
```javascript
// Query whitelisting
const server = new ApolloServer({
  plugins: [
    require('graphql-query-whitelist')({
      whitelist: queryWhitelist
    })
  ]
});

// Rate limiting
import { shield, rateLimit } from 'graphql-shield';

const permissions = shield({
  Query: {
    users: rateLimit({ window: '1m', max: 5 })
  }
});
```

### Production Setup
```javascript
const server = new ApolloServer({
  typeDefs,
  resolvers,
  
  // Disable introspection in production
  introspection: process.env.NODE_ENV !== 'production',
  
  // Disable playground in production  
  plugins: [
    process.env.NODE_ENV === 'production'
      ? ApolloServerPluginLandingPageDisabled()
      : ApolloServerPluginLandingPageLocalDefault()
  ],
  
  // Error handling
  formatError: (error) => {
    // Log error
    console.error('GraphQL Error:', error);
    
    // Don't expose internal errors in production
    if (process.env.NODE_ENV === 'production') {
      return new Error('Internal server error');
    }
    
    return error;
  }
});
```

## 9. Testing GraphQL

### Unit Testing Resolvers
```javascript
import { createTestClient } from 'apollo-server-testing';

describe('User resolvers', () => {
  it('should fetch user by id', async () => {
    const { query } = createTestClient(server);
    
    const GET_USER = gql`
      query GetUser($id: ID!) {
        user(id: $id) {
          id
          name
          email
        }
      }
    `;
    
    const result = await query({
      query: GET_USER,
      variables: { id: '1' }
    });
    
    expect(result.data.user).toEqual({
      id: '1',
      name: 'John Doe',
      email: 'john@example.com'
    });
  });
});
```

### Integration Testing
```javascript
import request from 'supertest';
import { createApp } from '../app';

describe('GraphQL API', () => {
  const app = createApp();
  
  it('should create user', async () => {
    const mutation = `
      mutation {
        createUser(input: { name: "Test", email: "test@example.com" }) {
          id
          name
          email
        }
      }
    `;
    
    const response = await request(app)
      .post('/graphql')
      .send({ query: mutation })
      .expect(200);
      
    expect(response.body.data.createUser.name).toBe('Test');
  });
});
```

## 10. Monitoring & Debugging

### Apollo Studio Integration
```javascript
const server = new ApolloServer({
  typeDefs,
  resolvers,
  plugins: [
    ApolloServerPluginUsageReporting({
      sendVariableValues: { all: true },
      sendHeaders: { all: true }
    })
  ]
});
```

### Custom Metrics
```javascript
const server = new ApolloServer({
  plugins: [
    {
      requestDidStart() {
        return {
          willSendResponse(requestContext) {
            // Log query performance
            const duration = Date.now() - requestContext.request.startTime;
            console.log(`Query took ${duration}ms`);
          }
        };
      }
    }
  ]
});
```
