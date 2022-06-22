<style>
th, thead {
    border-top:1pt solid;
    border-bottom: 2px solid;
    border-left: none;
    border-right: none;
}
td {
    border-top: 1px solid;
    border-bottom: 1px solid;
    border-left: 1px solid;
    border-right: 1px solid;
}
</style>

# GraphQL Basics: Mutations

## <span style="color:lightgreen">Creating Data With Mutations (Part 1):</span>

---

Similar to with regular queries, with Mutations there are a client side operation and a server definition.

So if we want to support a mutation then we have to actually define it on the server before it can be used

### <span style="color:turquoise">Define a new Type Definition:</span>

Listing out all of the mutations that we want to support

- Give the mutations a name, set up the arguments and set up the return value

```javascript
// Type Definitions (schema):
const typeDefs = gql`
  type Query {
    users(query: String): [User!]!
    posts(query: String): [Post!]!
    comments: [Comment!]!
    me: User!
    post: Post!
  }
  type Mutation {
    createUser(name: String!, email: String!, age: Int): User!
  }
  type User {
    id: ID!
    name: String!
    email: String!
    age: Int
    posts: [Post!]!
    comments: [Comment!]!
  }
`;
```

### <span style="color:turquoise">Create a Resolver Method:</span>

Similar to the ones we've created for our other queries

```javascript
// Resolvers
const resolvers = {
  Query: {
    users(parent, args, ctx, info) {
      if (!args.query) {
        return users;
      }
      return users.filter(user => {
        return user.name.toLowerCase().includes(args.query.toLowerCase());
      });
    },
  },
  Mutation: {
    createUser(parent, args, ctx, info) {
      const emailTaken = users.some(user => user.email === args.email);
      if (emailTaken) {
        throw new Error('Email is already taken!');
      }
      const user = {
        id: uuidv4(),
        name: args.name,
        email: args.email,
        age: args.age,
      };
      users.push(user);
      return user;
    },
  },
};
```

---

<br>

## <span style="color:lightgreen">Creating Data With Mutations (Part 2):</span>

---

We can add an additional mutation for the Post object like below, similar to the actions taken above with the createUser mutation

```javascript
// Type Definitions (schema):
const typeDefs = gql`
  type Query {
    users(query: String): [User!]!
    posts(query: String): [Post!]!
    comments: [Comment!]!
    me: User!
    post: Post!
  }
  type Mutation {
    createUser(name: String!, email: String!, age: Int): User!
    createPost(title: String!, body: String!, published: Boolean!, author: ID!): Post!
  }
  type User {
    id: ID!
    name: String!
    email: String!
    age: Int
    posts: [Post!]!
    comments: [Comment!]!
  }
  type Post {
    id: ID!
    title: String!
    body: String!
    published: Boolean!
    author: User!
    comments: [Comment!]!
  }
`;
// Resolvers
const resolvers = {
  Query: {
    users(parent, args, ctx, info) {
      if (!args.query) {
        return users;
      }
      return users.filter(user => {
        return user.name.toLowerCase().includes(args.query.toLowerCase());
      });
    },
  },
  Mutation: {
    createUser(parent, args, ctx, info) {
      const emailTaken = users.some(user => user.email === args.email);
      if (emailTaken) {
        throw new Error('Email is already taken!');
      }
      const user = {
        id: uuidv4(),
        name: args.name,
        email: args.email,
        age: args.age,
      };
      users.push(user);
      return user;
    },
    createPost(parent, args, ctx, info) {
      const userExists = users.some(user => user.id === args.author);
      if (!userExists) {
        throw new Error('User not found');
      }
      const post = {
        id: uuidv4(),
        title: args.title,
        body: args.body,
        published: args.published,
        author: args.author,
      };
      posts.push(post);
      return post;
    },
  },
};
```

Now if we run our createPost mutation, we can immediately access to the related data as well in the query, such as the author (User) object information or the comments information

```javascript
// GraphQL
mutation {
  createPost(
    title: "My New Post"
    body: ""
    published: false
    author: "271fc6fe-8738-46aa-8c8a-eee2316b2d9c"
  ) {
    id
    title
    body
    published
    author {
      name
    }
    comments {
      id
    }
  }
}
```

---

<br>

## <span style="color:lightgreen">Using Input Type on Mutations:</span>

---

Example below, the mutations were updated so that they reference the input type that we want to use (instead of listing out each expected type in the mutation type itself)

```javascript
  type Mutation {
    createUser(data: CreateUserInput): User!
    createPost(data: CreatePostInput): Post!
    createComment(data: CreateCommentInput): Comment!
  }

  input CreateUserInput {
    name: String!
    email: String!
    age: Int
  }
  input CreatePostInput {
    title: String!
    body: String!
    published: Boolean!
    author: ID!
  }
  input CreateCommentInput {
    text: String!
    author: ID!
    post: ID!
  }
```

The mutation resolver also needs to be updated so that it references args.data instead of just args

```javascript
// Resolvers
const resolvers = {
  Mutation: {
    createUser(parent, args, ctx, info) {
      const emailTaken = users.some(user => user.email === args.data.email);

      if (emailTaken) {
        throw new Error('Email is already taken!');
      }

      const user = { id: uuidv4(), ...args.data };
      users.push(user);
      return user;
    },
};
```

Then we just change our queries slightly so that we are populating the 'data' attribute with the new user information

```javascript
// GraphQL
mutation {
  createUser(data: { name: "John", email: "john@example.com", age: 32 }) {
    id
    name
    email
    age
  }
}
```

---

<br>

## <span style="color:lightgreen">Updating & Deleting Data With Mutations:</span>

---

Updating data using mutations:

```javascript
// GraphQL
type Mutation {
  createUser(data: CreateUserInput): User!
  updateUser(id: ID!, data: UpdateUserInput!): User!
  deleteUser(id: ID!): User!
}

input CreateUserInput {
  name: String!
  email: String!
  age: Int
}
input UpdateUserInput {
  name: String
  email: String
  age: Int
}
```

Mutations.js file:

```javascript
const Mutation = {
  updatePost(parent, args, { db }, info) {
    const { id, data } = args;
    const post = db.posts.find(post => post.id === id);

    if (!post) {
      throw new Error('Post not found');
    }

    if (typeof data.title === 'string') {
      post.title = data.title;
    }

    if (typeof data.body === 'string') {
      post.body = data.body;
    }

    if (typeof data.published === 'boolean') {
      post.published = data.published;
    }

    return post;
  },
  deletePost(parent, args, { db }, info) {
    const postIndex = db.posts.findIndex(post => post.id === args.id);

    if (postIndex === -1) {
      throw new Error('Post not found');
    }
    const deletedPosts = db.posts.splice(postIndex, 1);
    db.comments = db.comments.filter(comment => comment.post !== args.id);

    return deletedPosts[0];
  },
};
```
