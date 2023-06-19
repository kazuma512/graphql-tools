# GraphQL-tools: generate and mock GraphQL.js schemas

This package allows you to use the GraphQL schema language to build your [GraphQL.js](https://github.com/graphql/graphql-js) schema, and also includes useful schema tools like per-type mocking.


## Documentation

[Read the docs.](https://the-guild.dev/graphql/tools/docs/introduction)

## Example

When using `graphql-tools`, you describe the schema as a GraphQL type language string:

```js

const schema = `
type Author {
  id: ID! # the ! means that every author object _must_ have an id
  firstName: String
  lastName: String
  posts: [Post] # the list of Posts by this author
}

type Post {
  id: ID!
  title: String
  author: Author
  votes: Int
}

# the schema allows the following query:
type Query {
  posts: [Post]
}

# this schema allows the following mutation:
type Mutation {
  upvotePost (
    postId: ID!
  ): Post
}

# we need to tell the server which types represent the root query
# and root mutation types. We call them RootQuery and RootMutation by convention.
schema {
  query: Query
  mutation: Mutation
}
`;

export default schema;
```

Then you define resolvers as a nested object that maps type and field names to resolver functions:

```js
const resolverMap = {
  Query: {
    posts() {
      return posts;
    },
  },
  Mutation: {
    upvotePost(_, { postId }) {
      const post = find(posts, { id: postId });
      if (!post) {
        throw new Error(`Couldn't find post with id ${postId}`);
      }
      post.votes += 1;
      return post;
    },
  },
  Author: {
    posts(author) {
      return filter(posts, { authorId: author.id });
    },
  },
  Post: {
    author(post) {
      return find(authors, { id: post.authorId });
    },
  },
};

export default resolverMap;
```

At the end, the schema and resolvers are combined using `makeExecutableSchema`:

```js
import schema from './data/schema.js';
import resolverMap from './data/resolvers';
import { makeExecutableSchema } from 'graphql-tools';

const executableSchema = makeExecutableSchema({
  typeDefs: schema,
  resolvers: resolverMap,
});
```

## Contributions

Contributions, issues and feature requests are very welcome. If you are using this package and fixed a bug for yourself, please consider submitting a PR!
