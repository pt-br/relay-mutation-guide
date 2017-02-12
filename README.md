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

Then, you can start the server by:

```
npm start
```

Go to <a href="http://localhost:3000">http://localhost:3000</a> and check if you can see the application running. If not, get back to the first step.

Ok, if you are here you already have the environment running. Let's go ahead.

### Understanding what is the Database
On a real application, the database usually would be based into some database management system, Rest APIs and etc. For this guide, our Database is pure JavaScript.

We have three class files: **Database.js**, **User.js** and **Phone.js**. *Database.js* is our main entry class for database operations. GraphQL queries and mutations will comunicate with this file to change data. *User.js* is a class that contains its own methods and contains instances of *Phone.js* as well. 

I'm not going deeper inside of this files, but you can read them if you want - they are pretty well commented and I'm sure you'll easily understand how they work.

PS: GraphQL is **not** a database, don't misunderstand that.

