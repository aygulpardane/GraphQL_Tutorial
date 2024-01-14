# GraphQL_Tutorial
 Part 1
To build our homepage grid feature, we used a schema-first approach, meaning we considered data needs from the client's perspective before even starting to code.

We defined our schema and used Apollo Server to build a basic GraphQL endpoint that provides mocked responses.

We then used the Apollo Sandbox Explorer to interactively build and test queries against our local GraphQL server.

Finally, we developed the client side of our Catstronauts app. We used React, Apollo Client, and the useQuery hook to perform a query on our GraphQL server and to display our tracks in a nice grid card layout.

Part 2
GraphQL Query:

1) Where is data stored and how is it structured? Tha will determine the shape of our data source and resolvers
2) Make a data source file.
    - With GraphQL, one query is often composed of a mix of different fields and types, coming from different endpoints, with different cache policies
    - RESTDataSource class takes care of that
    - declare a class that extends the RESTDataSource class and assign baseURL
    - then declare new methods for the HTTP requests, GET specifically. The methods use API endpoints
    - Can also insert arguments that'll be used to make calls to endpoints like /:id
3) Now make resolvers. A resolver's mission is to populate the data for a field in your schema. It has the same name as the field that it populates data for. It can fetch data from any data source, then transforms that data into the shape your client requires.
    - to create a resolver for a specific field (not a type), add a Query key to the resolvers object. Inside it wil be the object with the key name of the field and the value (of the key value pair) is the resolver function for that field
    - resolver has 4 parameters:
        - parent:
        parent is the returned value of the resolver for this field's parent. This will be useful when dealing with resolver chains.
        - args:
        args is an object that contains all GraphQL arguments that were provided for the field by the GraphQL operation. When querying for a specific item (such as a specific track instead of all tracks), in client-land we'll make a query with an id argument that will be accessible via this args parameter in server-land. We'll cover this further in Lift-off III.
        - contextValue:
        contextValue is an object shared across all resolvers that are executing for a particular operation. The resolver needs this argument to share state, like authentication information, a database connection, or in our case the RESTDataSource.
        - info:
        info contains information about the operation's execution state, including the field name, the path to the field from the root, and more. It's not used as frequently as the others, but it can be useful for more advanced actions like setting cache policies at the resolver level.
    - the function will return the results from the specific method defined in the data source file
    4) in the ApolloServer options, put in typeDefs and resolvers, require in resolvers and the data source file. Then in the startStandaloneServer function, put a second argument configuring the server's options
        - This is where we'll define a context function that returns an object that all our resolvers will share: contextValue
        - context is set to an async function which returns an object
        -  we want to access the dataSources.trackAPI (and its methods) from the contextValue parameter of our resolvers
        - set a dataSources property inside the obj which is set to another obj with the name of the RESTDataSource class we extended, and it returns an instance of the data source class we imported earlier
        - pass the server's cache to the data source class like const { cache } = server;


Part 3
To update schema and add more modules and update the resolvers:

1) Go to schema and update the schema file
2) Then go to the data source (this is the API, database, service graphql interacts with to handle client requests) and add methods to GET specific data from the specific endpoint and return the results (in this case, from REST API). The method will be used in resolvers.
3) Then go to resolvers.
    - If you added a new field in a type (in this case the track field in the Query type in the schema file), then inside the resolvers object, inside the Query key, add a resolver function with the same name of the field, in this case 'track'
    - If you added a new type, Track, then add a new key to the resolvers object, reflecting the type Track. The key is also named Track. Inside it, add the specific resolver functions you need to populate the specific field.

    type --> new object with the same key name inside the resolvers object
    field --> resolver function inside the object inside the resolvers object

Part 4
To mutate data:

1) Go to schema file and add type Mutation. Then add the return type for the mutation - that'll be a new type defined below the Mutation type.
2) To figure out which endpoints we need to use to update the data, go to the data source file, which is track-api.js. Add a new method and add the parameters. Inside it, make a request for mutation like patch, put, delete. The data source makes a patch call to the REST api
3) Time to write resolvers to put datasource to work. Need to add another entry to resolvers object for the mutation.
    - add the resolver function as usual but this time we need to wait for the response to return so make the function async. Can't immediately return the results because it builds part of the response from the REST operation status (status code, success, and message)
    - then return the code, success, and message AND the modified object at the end
4) What happens when there's a 404 not found response:
    - wrap the resolver function's return in a try block and catch the error
    - catch takes error as parameter and returns code values graphql provides and is more specific.
    - success is also false.
    - message is another object and is dynamic.
    - The objects that were supposed to be returned will be set to null
5) Client side:
    - add the useMutation hook which returns an array of objects.
    - the first is the mutation function we'll use to call later on an onClick for example
