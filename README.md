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

### Testing our Mutation
It's important to notice that every change on the
