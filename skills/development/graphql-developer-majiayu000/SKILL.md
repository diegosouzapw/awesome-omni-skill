---
name: graphql-developer
description: "[Extends solution-architect] GraphQL API specialist. Use for GraphQL schemas, Apollo Server/Federation, DataLoader, resolvers, subscriptions. Invoke alongside solution-architect for GraphQL API design."
---

# GraphQL Developer

> **Extends:** solution-architect
> **Type:** Specialized Skill

## Trigger

Use this skill alongside `solution-architect` when:
- Designing GraphQL schemas
- Implementing resolvers
- Setting up Apollo Server
- Configuring Apollo Federation
- Preventing N+1 queries with DataLoader
- Building GraphQL clients
- Implementing subscriptions
- Schema stitching or federation

## Context

You are a Senior GraphQL Developer with 5+ years of experience building GraphQL APIs. You have designed federated schemas for microservices architectures and understand performance optimization patterns. You follow schema design best practices and implement type-safe GraphQL systems.

## Expertise

### Versions

| Technology | Version | Notes |
|------------|---------|-------|
| GraphQL Spec | October 2021 | Latest stable |
| Apollo Server | 4.x | Server implementation |
| Apollo Federation | 2.x | Microservices |
| Apollo Client | 3.x | React client |
| GraphQL Yoga | 5.x | Alternative server |
| Pothos | 4.x | Code-first schemas |

### Core Concepts

#### Schema Design (SDL)

```graphql
# schema.graphql
type Query {
  user(id: ID!): User
  users(
    first: Int
    after: String
    filter: UserFilter
  ): UserConnection!
  me: User
}

type Mutation {
  createUser(input: CreateUserInput!): CreateUserPayload!
  updateUser(id: ID!, input: UpdateUserInput!): UpdateUserPayload!
  deleteUser(id: ID!): DeleteUserPayload!
}

type Subscription {
  userCreated: User!
  userUpdated(id: ID!): User!
}

type User {
  id: ID!
  email: String!
  name: String!
  avatar: String
  role: UserRole!
  posts(first: Int, after: String): PostConnection!
  createdAt: DateTime!
  updatedAt: DateTime
}

enum UserRole {
  ADMIN
  USER
  GUEST
}

input UserFilter {
  role: UserRole
  search: String
  createdAfter: DateTime
}

input CreateUserInput {
  email: String!
  name: String!
  password: String!
  role: UserRole = USER
}

input UpdateUserInput {
  email: String
  name: String
  role: UserRole
}

# Relay-style connections
type UserConnection {
  edges: [UserEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type UserEdge {
  cursor: String!
  node: User!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}

# Mutation payloads
type CreateUserPayload {
  user: User
  errors: [Error!]!
}

type UpdateUserPayload {
  user: User
  errors: [Error!]!
}

type DeleteUserPayload {
  success: Boolean!
  errors: [Error!]!
}

type Error {
  field: String
  message: String!
  code: ErrorCode!
}

enum ErrorCode {
  VALIDATION_ERROR
  NOT_FOUND
  UNAUTHORIZED
  INTERNAL_ERROR
}

scalar DateTime
```

#### Apollo Server Setup

```typescript
// server.ts
import { ApolloServer } from '@apollo/server';
import { expressMiddleware } from '@apollo/server/express4';
import { ApolloServerPluginDrainHttpServer } from '@apollo/server/plugin/drainHttpServer';
import express from 'express';
import http from 'http';
import cors from 'cors';
import { typeDefs } from './schema';
import { resolvers } from './resolvers';
import { createContext, Context } from './context';

async function startServer() {
  const app = express();
  const httpServer = http.createServer(app);

  const server = new ApolloServer<Context>({
    typeDefs,
    resolvers,
    plugins: [
      ApolloServerPluginDrainHttpServer({ httpServer }),
    ],
  });

  await server.start();

  app.use(
    '/graphql',
    cors<cors.CorsRequest>(),
    express.json(),
    expressMiddleware(server, {
      context: createContext,
    }),
  );

  await new Promise<void>((resolve) =>
    httpServer.listen({ port: 4000 }, resolve)
  );

  console.log(`🚀 Server ready at http://localhost:4000/graphql`);
}

startServer();
```

#### Resolvers with DataLoader

```typescript
// resolvers/user.ts
import { Resolvers } from '../generated/graphql';
import { Context } from '../context';

export const userResolvers: Resolvers<Context> = {
  Query: {
    user: async (_, { id }, { dataSources }) => {
      return dataSources.userLoader.load(id);
    },

    users: async (_, { first = 10, after, filter }, { dataSources }) => {
      const { users, totalCount, hasNextPage, hasPreviousPage } =
        await dataSources.userService.getUsers({ first, after, filter });

      return {
        edges: users.map((user) => ({
          cursor: Buffer.from(user.id).toString('base64'),
          node: user,
        })),
        pageInfo: {
          hasNextPage,
          hasPreviousPage,
          startCursor: users[0]
            ? Buffer.from(users[0].id).toString('base64')
            : null,
          endCursor: users[users.length - 1]
            ? Buffer.from(users[users.length - 1].id).toString('base64')
            : null,
        },
        totalCount,
      };
    },

    me: async (_, __, { user }) => {
      return user;
    },
  },

  Mutation: {
    createUser: async (_, { input }, { dataSources }) => {
      try {
        const user = await dataSources.userService.createUser(input);
        return { user, errors: [] };
      } catch (error) {
        return {
          user: null,
          errors: [{ message: error.message, code: 'VALIDATION_ERROR' }],
        };
      }
    },

    updateUser: async (_, { id, input }, { dataSources }) => {
      try {
        const user = await dataSources.userService.updateUser(id, input);
        return { user, errors: [] };
      } catch (error) {
        return {
          user: null,
          errors: [{ message: error.message, code: 'NOT_FOUND' }],
        };
      }
    },
  },

  User: {
    posts: async (parent, { first, after }, { dataSources }) => {
      return dataSources.postService.getPostsByUserId(parent.id, { first, after });
    },
  },
};
```

#### DataLoader for N+1 Prevention

```typescript
// dataSources/userLoader.ts
import DataLoader from 'dataloader';
import { User } from '../models';

export function createUserLoader(db: Database) {
  return new DataLoader<string, User | null>(async (ids) => {
    const users = await db.user.findMany({
      where: { id: { in: ids as string[] } },
    });

    const userMap = new Map(users.map((user) => [user.id, user]));

    // Return in same order as requested ids
    return ids.map((id) => userMap.get(id) || null);
  });
}

// context.ts
import { createUserLoader } from './dataSources/userLoader';

export interface Context {
  user: User | null;
  dataSources: {
    userLoader: DataLoader<string, User | null>;
    userService: UserService;
    postService: PostService;
  };
}

export async function createContext({ req }): Promise<Context> {
  const token = req.headers.authorization?.replace('Bearer ', '');
  const user = token ? await verifyToken(token) : null;

  return {
    user,
    dataSources: {
      userLoader: createUserLoader(db),
      userService: new UserService(db),
      postService: new PostService(db),
    },
  };
}
```

#### Apollo Federation

```graphql
# users-subgraph/schema.graphql
extend schema
  @link(url: "https://specs.apollo.dev/federation/v2.0",
        import: ["@key", "@shareable", "@external", "@provides", "@requires"])

type Query {
  user(id: ID!): User
  users: [User!]!
}

type User @key(fields: "id") {
  id: ID!
  email: String!
  name: String!
  role: UserRole!
}

# posts-subgraph/schema.graphql
extend schema
  @link(url: "https://specs.apollo.dev/federation/v2.0",
        import: ["@key", "@external"])

type Query {
  post(id: ID!): Post
  posts: [Post!]!
}

type Post @key(fields: "id") {
  id: ID!
  title: String!
  content: String!
  author: User!
}

type User @key(fields: "id") {
  id: ID! @external
  posts: [Post!]!
}

# Router configuration
# supergraph.yaml
federation_version: =2.0.0
subgraphs:
  users:
    routing_url: http://localhost:4001/graphql
    schema:
      file: ./users-subgraph/schema.graphql
  posts:
    routing_url: http://localhost:4002/graphql
    schema:
      file: ./posts-subgraph/schema.graphql
```

#### Subscriptions

```typescript
// subscriptions.ts
import { PubSub } from 'graphql-subscriptions';

const pubsub = new PubSub();

export const subscriptionResolvers = {
  Subscription: {
    userCreated: {
      subscribe: () => pubsub.asyncIterator(['USER_CREATED']),
    },
    userUpdated: {
      subscribe: (_, { id }) => {
        return pubsub.asyncIterator([`USER_UPDATED_${id}`]);
      },
    },
  },
};

// In mutation resolver
export const mutationResolvers = {
  Mutation: {
    createUser: async (_, { input }, { dataSources }) => {
      const user = await dataSources.userService.createUser(input);
      pubsub.publish('USER_CREATED', { userCreated: user });
      return { user, errors: [] };
    },
  },
};
```

#### Apollo Client (React)

```typescript
// client.ts
import { ApolloClient, InMemoryCache, createHttpLink } from '@apollo/client';
import { setContext } from '@apollo/client/link/context';

const httpLink = createHttpLink({
  uri: 'http://localhost:4000/graphql',
});

const authLink = setContext((_, { headers }) => {
  const token = localStorage.getItem('token');
  return {
    headers: {
      ...headers,
      authorization: token ? `Bearer ${token}` : '',
    },
  };
});

export const client = new ApolloClient({
  link: authLink.concat(httpLink),
  cache: new InMemoryCache({
    typePolicies: {
      Query: {
        fields: {
          users: {
            keyArgs: ['filter'],
            merge(existing, incoming, { args }) {
              if (!args?.after) return incoming;
              return {
                ...incoming,
                edges: [...(existing?.edges || []), ...incoming.edges],
              };
            },
          },
        },
      },
    },
  }),
});

// hooks/useUsers.ts
import { useQuery, gql } from '@apollo/client';

const GET_USERS = gql`
  query GetUsers($first: Int, $after: String) {
    users(first: $first, after: $after) {
      edges {
        cursor
        node {
          id
          name
          email
        }
      }
      pageInfo {
        hasNextPage
        endCursor
      }
    }
  }
`;

export function useUsers() {
  const { data, loading, error, fetchMore } = useQuery(GET_USERS, {
    variables: { first: 10 },
  });

  const loadMore = () => {
    if (data?.users.pageInfo.hasNextPage) {
      fetchMore({
        variables: {
          after: data.users.pageInfo.endCursor,
        },
      });
    }
  };

  return { users: data?.users.edges.map((e) => e.node), loading, error, loadMore };
}
```

### Project Structure

```
src/
├── schema/
│   ├── typeDefs/
│   │   ├── user.graphql
│   │   ├── post.graphql
│   │   └── index.ts
│   └── index.ts
├── resolvers/
│   ├── user.ts
│   ├── post.ts
│   └── index.ts
├── dataSources/
│   ├── userLoader.ts
│   ├── userService.ts
│   └── postService.ts
├── models/
│   ├── user.ts
│   └── post.ts
├── generated/
│   └── graphql.ts       # Generated types
├── context.ts
├── server.ts
└── codegen.ts
```

## Parent & Related Skills

| Skill | Relationship |
|-------|--------------|
| **solution-architect** | Parent skill - invoke for API architecture patterns |
| **backend-developer** | For resolver implementation, service layer |
| **frontend-developer** | For Apollo Client integration |
| **e2e-tester** | For GraphQL API testing |

## Standards

- **Schema-first**: Define schema before resolvers
- **Relay connections**: Use for pagination
- **DataLoader**: Prevent N+1 queries
- **Mutation payloads**: Include errors array
- **Input types**: Use for mutations
- **Enums**: For fixed value sets
- **Nullable defaults**: Be explicit

## Checklist

### Before Designing Schema
- [ ] Domain model understood
- [ ] Query patterns identified
- [ ] Pagination requirements clear
- [ ] Error handling strategy

### Before Deploying
- [ ] DataLoaders implemented
- [ ] N+1 queries eliminated
- [ ] Query complexity limits set
- [ ] Authentication configured
- [ ] Schema documentation complete

## Anti-Patterns to Avoid

1. **N+1 queries**: Use DataLoader
2. **Overfetching**: Design specific types
3. **No pagination**: Always paginate lists
4. **Generic errors**: Use typed error codes
5. **Missing input validation**: Validate all inputs
6. **Nested mutations**: Keep mutations flat
7. **No rate limiting**: Implement query cost analysis
