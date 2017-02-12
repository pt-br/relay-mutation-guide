# You gonna learn mutations today.

[![You Gonna Learn Today!](http://i.imgur.com/6mE3XvY.png)](http://www.youtube.com/watch?v=ExPzkvMutmI "You Gonna Learn Today")

#### This guide will fit if you find yourself into one(or more) of the following situations:
* I'm freaking out about how to handle mutations on GraphQL and Relay;
* I tried reading someone else's code, but it's not too much explained and it's complex;
* The examples I found on internet have less explanation and a lots of code that will output something only 1h after going through the tutorial;
* Star Wars' example of facebook sucks.

### Intro
In this guide we will be creating a simple application that uses a real world example to get things easier for you.
It's important to say that this is not a complete guide for GraphQL and Relay, the focus of this guide is only on **Mutations**.

So, before you get started, be sure to know at least the basics of:

* ES6
* React
* GraphQL
* GraphiQL

### Getting the Playground
As I said, we will be working on an application for this guide. You can check it running accessing a <a targe="_blank" href="https://relay-phones.herokuapp.com">demo build on heroku</a>. I've prepared a special boilerplate of this application, removing everything related to mutations, so we can do it together on this guide.

Let's put the hand on the code, start by clonning the boilerplate:
```
git clone https://github.com/pt-br/relay-phones-guide.git
```

Go inside the folder you've clonned the repository, and install the node modules:

```
npm install
```

Don't worry about the code for now, just start the server by running. 

```
npm start
```

Go to <a href="http://localhost:3000">http://localhost:3000</a> and check if you can see the application running. If not, get back to the first step.

Ok, if you are here you already have the environment running. Let's go ahead.

### Understanding what is the Database
On a real application, the database usually would be based into some database management system, REST APIs, etc. For this guide, our Database is pure JavaScript.

We have three class files: **Database.js**, **User.js** and **Phone.js**. *Database.js* is our main entry class for database operations. GraphQL queries and mutations will comunicate with this file to change data. *User.js* is a class that contains its own methods and contains instances of *Phone.js* as well. 

I'm not going deeper inside of this files, but you can read them if you want - they are pretty well commented and I'm sure you'll easily understand how they work.

PS: GraphQL is **not** a database, don't misunderstand that.

### What is a Mutation
A mutation is an operation that creates, changes or erases something (for reading, we use queries and not mutations). A mutation will always do something, then fetch something.

It's important to know that a mutation happens in two sides: Server(GraphQL) and Client(Relay).

### Creating a Mutation to Add a Phone
Let's start by creating a mutation to add a new phone. 

Go through ```/data/schema.js``` and observe that we are importing some GraphQL and GraphQL-Relay items in there.

We are also importing our **Database.js** class and creating an instance of it inside of the schema. This is required in order to make possible doing manipulations into our database by using GraphQL, in other words: GraphQL will be able to access methods inside of **Database.js**.

On ```schema.js```, write the following mutation at line 96:

```
const AddPhoneMutation = mutationWithClientMutationId({
  name: 'AddPhone',
  inputFields: {
    model: {
      type: new GraphQLNonNull(GraphQLString),
    },
    image: {
      type: new GraphQLNonNull(GraphQLString),
    },
  },
  outputFields: {
    viewer: {
      type: UserType,
      resolve: () => database.getUser(),
    },
  },
  mutateAndGetPayload: ({ model, image }) => {
    const newPhone = database.insertPhone(model, image);
    return newPhone;
  },
});
```

**Explanation**: `mutationWithClientMutationId` is a helper to build mutations from the package `graphql-relay`. You don't need to know much more about this, you only need to know a few things:

- It takes a name, as we used 'AddPhone'
- It takes inputFields;
- It takes outPutFields;
- It takes a mutation method (mutateAndGetPayload).

It's very important to have a **name** for our mutations. It will be used both on GraphQL and Relay.

On the **inputFields** we must declare the things we are going to send for the mutation. Once we are going to add a new Phone, we must send a `model` and an `image` fields, once it's the required data to create a new Phone into our database.

On the **outputFields** we are declaring what will be output by the mutation, in this case, we are just returning our User (that contains Phones).

On **mutateAndGetPayload** we are using data provided by the **inputFields**, in this case, `model` and `image`. Then we call a function from our **Database.js** called `insertPhone`, we are sending both `model` and `image` to this function. Inside of `insertPhone`, there's a return for the new phone being added, and this is what we are returning on **mutateAndGetPayload**.

It got simple for you now? I hope so :p 

### Write a container for our Mutations
Ok, you have a mutation and now you need to tell GraphQL that you do. 

Like the Root query, you must specify a "root mutation" that will contain all of your mutations.

On ```schema.js``` at line 134, write your root mutation:

```
const Mutation = new GraphQLObjectType({
  name: 'Mutation',
  fields: () => ({
    addPhone: AddPhoneMutation,
  }),
});
```

I think it's self-explanatory.

Then, at line 145, modify your Schema export to contain the root mutation. It should be like this:

```
export var Schema = new GraphQLSchema({
  query: Root,
  mutation: Mutation,
});
```

### Testing our Mutation
Ok, the GraphQL side of the mutation is done!

It's important to notice that every change on the ```schema.js``` will require you to stop the server and run a command to update the schema. In order to update our schema, let's do it now. 

Stop the server, then run:

```
npm run update-schema
```

And then start the server again:

```
npm start
```

Alright, your schema is now updated and you can test your mutation.

Let's go to our GraphiQL instance that is running on <a href="http://localhost:8080">http://localhost:8080</a>, I stronggly recommend you to run it on a separated tab.

Run the following mutation on the left pannel:

```
mutation {
	addPhone(input: {clientMutationId: "1" model: "Nexus 5", image: "https://goo.gl/Fq46CZ"}) {
    viewer {
      phones {
        edges {
          node {
            id
            model
          }
        }
      }
    }
  }
}
```

**Explanation**: We are calling the mutation by ```addPhone```, the same name we have exported on our Root Mutation. Then, we need to supply the inputs(remember when we declared it on the **inputFields**?). 

As you can notice, we are sending a parameter that was not into our **inputFields** declaration - `clientMutationId`. You don  need to worry about this field, it will be automatically filled by Relay under the hood. We only need to spoof it on GraphiQL(and any string value should work).

After adding the Nexus 5 using our mutation, you can refresh the application on <a href="http://localhost:8080">http://localhost:8080</a> and check that the phone is there. 

Yes, your mutation worked pretty good!
