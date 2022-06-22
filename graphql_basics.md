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

# GraphQL Basics: Schemas and Queries

## <span style="color:lightgreen">Creating a GraphQL API</span>

---

In order to start out using graphql-yoga, the first steps are to set up and define

1. Type Definitions
2. Resolvers

### <span style="color:turquoise">Type Definitions (schema):</span>

Also known as the Application Schema, this is set up and used to define all operations that can be performed on the API and what our custom data types look like.

<br>

![alt text](v_notes_files/schema.jpg 'Title')
<br>

Can then use the 'query' operation thats made available to pull the JSON data that we want from any of the queries

### <span style="color:turquoise">Resolvers:</span>

Application Resolvers are a set of functions that are defined for each of the operations that can be performed on our API

- For example: In the screenshot of the application schema above, a resolver was defined for the 'hello' query, the 'course' query, the 'courseInstructor' query and the 'me' query.
  - Those functions know what to do when that query runs, they know how to get and return the correct data

```javascript
// GraphQL
query {
  hello
  course
  courseInstructor
  me {
    id
    name
  }
}
```

### <span style="color:turquoise">First Query Example:</span>

First query overview example using graphql-yoga on localhost:4000

Type Definitions (schema):

- To define the query: start with the query name, then we add a : and define the expected type

Resolvers:

- In general, the structure of the resolvers object is going to mirror the schema object

Start Server:

- `GraphQLServer()` expects an object as the argument
- type definitions & resolvers

```javascript
import { GraphQLServer } from 'graphql-yoga';

// Type Definitions (schema):
const typeDefs = gql`
  type Query {
    hello: String
  }
`;
// Resolvers:
const resolvers = {
  Query: {
    hello() {
      return 'This is my first query!';
    },
  },
};
// Start Server:
const server = new GraphQLServer({
  typeDefs,
  resolvers,
});

server.start(() => {
  console.log('The server is up!');
});
```

---

<br>

## <span style="color:lightgreen">Scalar Types:</span>

---

Using each of the 5 Scalar types, set up a new set of resources:

- String, Boolean, Int, Float, ID

NOTE: Not using ! after a type definition means the returned value can be null

```javascript
// Type Definitions (schema):
const typeDefs = gql`
  type Query {
    id: ID!
    name: String!
    age: Int!
    employed: Boolean!
    gpa: Float
  }
`;
// Resolvers:
const resolvers = {
  Query: {
    id() {
      return 'abc123';
    },
    name() {
      return 'James Applegate';
    },
    age() {
      return 25;
    },
    employed() {
      return true;
    },
    gpa() {
      return null;
    },
  },
};
```

---

<br>

## <span style="color:lightgreen">Creating Custom Types:</span>

---

For our example use in creating a blogging app, we're going to have custom types for all of the different types of data we have.

For example, custom types to represent each of these fields, which would have their own specified fields:

- User
- Post
- Comment

### <span style="color:turquoise">Building out the User type schema:</span>

Type Definitions (schema):

- Note that we set up an additional type definition in the application schema: one for 'Query' and one for 'User'

Resolvers:

- Note that we created a resolver function for me() since that is what we reference for the Query definition, but we return an object since an object will be returned when User is queried from 'me: User'

```javascript
// Type Definitions (schema):
const typeDefs = gql`
  type Query {
    me: User
  }
  type User {
    id: ID!
    name: String!
    email: String!
    age: Int
  }
`;
// Resolvers:
const resolvers = {
  Query: {
    me() {
      return {
        id: '123098',
        name: 'Mike',
        email: 'Mike@example.com',
        age: 28,
      };
    },
  },
};
```

Then we query our new dataset in GraphQL:

```javascript
// GraphQL
query {
  me {
    id
    name
    email
    age
  }
}
```

---

<br>

## <span style="color:lightgreen">Operation Arguments:</span>

---

This will allow us to pass data from the client to the server

There are 4 arguments that get passed to all resolver functions:

**parent:**

- Very common and useful when working with relational data
- Ex: if a user has many posts, then the parent argument is used when figuring out how to get the users posts

**args:**

- Contains the operation arguments supplied to the resolver

**context:**

- Typically shortened to 'ctx' - is used for contextual data
- Ex: If a user is logged in, the context might contain the ID of that user so that it can be accessed throughout the application

**info:**

- Contains information about the actual operations that were sent along to the server

### <span style="color:turquoise">Using Resolver args:</span>

Using the 'args' argument in the `greeting()` resolver, we pass a name as an argument into the query call with GraphQL:

```javascript
// GraphQL
query {
  greeting(name: "James")
  me {
    id
    name
    email
    age
  }
  post {
    id
    title
    body
    published
  }
}
```

And extract that name in the resolver function from the 'args' argument in order to conditionally return a response based on if a name was provided:

```javascript
// Type Definitions (schema):
const typeDefs = gql`
  type Query {
    greeting(name: String, position: String): String!
    me: User!
    post: Post!
  }
  type User {
    id: ID!
    name: String!
    email: String!
    age: Int
  }
`;
// Resolvers:
const resolvers = {
  Query: {
    greeting(parent, args, ctx, info) {
      console.log(args);
      if (args.name) {
        return `Hello ${args.name}! You are the worst ${args.position}`;
      } else {
        return 'Hello';
      }
    },
    me() {
      return {
        id: '123098',
        name: 'Mike',
        email: 'Mike@example.com',
        age: 28,
      };
    },
  },
};
```

---

<br>

## <span style="color:lightgreen">Working With Arrays:</span>

---

Setting your schema to accept arrays is done by setting the type as an array [], then specifying the expected types that should be in the array

```javascript
// Type Definitions (schema):
const typeDefs = gql`
  type Query {
    add(numbers: [Float!]!): Float!
    grades: [Int!]!
  }
`;
// Resolvers
const resolvers = {
  Query: {
    add(parent, args, ctx, info) {
      if (args.numbers.length === 0) {
        return 0;
      }
      return args.numbers.reduce((accumulator, currentValue) => {
        return accumulator + currentValue;
      });
    },
    grades(parent, args, ctx, info) {
      return [99, 80, 93];
    },
  },
};
```

### <span style="color:turquoise">Arrays With Custom Types:</span>

Using our custom User and Post type definitions, we created new queries in our schema for 'users' and 'posts'. In our resolvers, we created filter methods that allow us to filter on the User or Post data based off parameters set by the client

```javascript
// Type Definitions (schema):
const typeDefs = gql`
  type Query {
    users(query: String): [User!]!
    posts(query: String): [Post!]!
    me: User!
    post: Post!
  }
  type User {
    id: ID!
    name: String!
    email: String!
    age: Int
  }
  type Post {
    id: ID!
    title: String!
    body: String!
    published: Boolean!
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

    posts(parent, args, ctx, info) {
      if (!args.query) {
        return posts;
      }
      return posts.filter(post => {
        const isTitleMatch = post.title.toLowerCase().includes(args.query.toLowerCase());
        const isBodyMatch = post.body.toLowerCase().includes(args.query.toLowerCase());
        return isTitleMatch || isBodyMatch;
      });
    },
  },
};
```

---

<br>

## <span style="color:lightgreen">Relational Data:</span>

---

After updating the posts array to include 'author' ID's which will be used to relate to a user object, we can then pull user data based off the author for a post

```javascript
// Demo Data
const users = [
  {
    id: '1',
    name: 'James',
    email: 'james@example.com',
    age: 25,
  },
];
const posts = [
  {
    id: '10',
    title: 'Course 101',
    message: '',
    published: true,
    author: '1',
  },
  {
    id: '11',
    title: 'Programming Music',
    message: '',
    published: true,
    author: '1',
  },
];
```

When the query below is run, its first going to run the resolver function for the posts query

```javascript
// GraphQL
query {
  posts {
    id
    title
    author {
      name
    }
  }
}
```

Then GraphQL is going to see what data was requested in the query call, since we asked for author in this case, GraphQL is going to see that author does not live inside of the Post type, so its going to call the author() function for each individual post with the Post object as the parent argument

- parent.author is going to be the string pk given in the posts array
- we can use this pk to iterate over the User objects to find the correct User object to return

```javascript
// Type Definitions (schema):
const typeDefs = gql`
  type Query {
    users(query: String): [User!]!
    posts(query: String): [Post!]!
    me: User!
    post: Post!
  }

  type User {
    id: ID!
    name: String!
    email: String!
    age: Int
  }

  type Post {
    id: ID!
    title: String!
    body: String!
    published: Boolean!
    author: User!
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
    posts(parent, args, ctx, info) {
      if (!args.query) {
        return posts;
      }
      return posts.filter(post => {
        const isTitleMatch = post.title
          .toLowerCase()
          .includes(args.query.toLowerCase());
        const isBodyMatch = post.body
          .toLowerCase()
          .includes(args.query.toLowerCase());
        return isTitleMatch || isBodyMatch;
      });
    },

  Post: {
    author(parent, args, ctx, info) {
      return users.find(user => {
        return user.id === parent.author;
      });
    },
  },
};
```

### <span style="color:turquoise">Setting The Reverse Relationship:</span>

Now that we can assocate and grab the user data for each post object, we can also set it up so that we can grab all the posts for each user object

- We add `posts: [Post!]!` in the User type definition
- Set up a separate User resolver that we can then use to compare the parent.id to the post.author

```javascript
// Type Definitions (schema):
const typeDefs = gql`
  type Query {
    users(query: String): [User!]!
    posts(query: String): [Post!]!
    me: User!
    post: Post!
  }
  type User {
    id: ID!
    name: String!
    email: String!
    age: Int
    posts: [Post!]!
  }
  type Post {
    id: ID!
    title: String!
    body: String!
    published: Boolean!
    author: User!
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
    posts(parent, args, ctx, info) {
      if (!args.query) {
        return posts;
      }
      return posts.filter(post => {
        const isTitleMatch = post.title.toLowerCase().includes(args.query.toLowerCase());
        const isBodyMatch = post.body.toLowerCase().includes(args.query.toLowerCase());
        return isTitleMatch || isBodyMatch;
      });
    },
  },
  Post: {
    author(parent, args, ctx, info) {
      return users.find(user => {
        return user.id === parent.author;
      });
    },
  },
  User: {
    posts(parent, args, ctx, info) {
      return posts.filter(post => {
        return post.author === parent.id;
      });
    },
  },
};
```
