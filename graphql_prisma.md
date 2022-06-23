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

# Database Storage With Prisma v1

## <span style="color:lightgreen">Prisma Overview:</span>

---

Prisma is a GraphQL ORM
Primsa wraps our database up and exposes it as a GraphQL API that can be used to read and write from the actual database

### <span style="color:turquoise">Prisma Files:</span>

datamodel.graphql

- A set of type definitions for GraphQL, similar to what we put in schema.graphql
- Prisma uses this when it determines your database structure, creating database tables for all of our custom object types (similar to models in Django)

prisma.yml

- Yaml is similar to JSON, just a set of key,value pairs and is a great language when it comes to configuration

docker-compose.yml

- File that is actually going to start up the Docker container
  - Version: Specifies the version currently being used
  - Services: Holds all of the connection information for the database
