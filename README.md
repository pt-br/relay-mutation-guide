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

Don't worry about the code for now, just start the server by running:

```
npm start
```

Go to <a href="http://localhost:3000">http://localhost:3000</a> and check if you can see the application running. If not, get back to the first step.

Ok, if you are here you already have the environment running. Let's go ahead.

### Understanding what is the Database
On a real application, the database usually would be based into some database management system, REST APIs, etc. For this guide, our database is pure JavaScript.

We have three class files: **Database.js**, **User.js** and **Phone.js**. *Database.js* is our main entry class for database operations. GraphQL queries and mutations will comunicate with this file to read and change data. *User.js* is a class that contains its own methods and contains instances of *Phone.js* as well. 

I'm not going deeper inside of this files, but you can read them if you want - They are pretty well commented and I'm sure you'll easily understand how they work.

PS: GraphQL is **not** a database, don't misunderstand that.

### What is a Mutation
A mutation is an operation that creates, changes or erases something (for reading, we use queries and not mutations). A mutation will always do something, then fetch something.

It's important to know that a mutation happens in two sides: Server(GraphQL) and Client(Relay).

### Creating a Mutation to Add a Phone
Let's start by creating a mutation to add a new phone. 

Go through ```/data/schema.js``` and observe that we are importing some GraphQL and GraphQL-Relay items in there.

We are also importing our **Database.js** class and creating an instance of it inside of the schema. This is required in order to make possible doing manipulations into our database by using GraphQL, in other words: GraphQL will be able to access methods inside of **Database.js**.

On ```schema.js```, write the following mutation at line 96:

```javascript
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

- It takes a name, as we used `AddPhone`
- It takes inputFields;
- It takes outPutFields;
- It takes a mutation method (mutateAndGetPayload).

It's very important to have a **name** for our mutations. It will be used both on GraphQL and Relay.

On the **inputFields** we must declare the things we are going to send for the mutation. Once we are going to add a new Phone, we must send a `model` and an `image` fields, that are the required data to create a new Phone into our database.

On the **outputFields** we are declaring what will be output by the mutation, in this case, we are just returning our User (that contains Phones).

On **mutateAndGetPayload** we are using data provided by the **inputFields**, in this case, `model` and `image`. Then we call a function from our **Database.js** called `insertPhone`, we are sending both `model` and `image` to this function. Inside of `insertPhone`, there's a return for the new phone being added, and this is what we are returning on **mutateAndGetPayload**.

It got simple for you now? I hope so :p 

### Write a container for our Mutations
Ok, you have a mutation and now you need to tell GraphQL that you do. 

Like the Root query, you must specify a "root mutation" that will contain all of your mutations.

On ```schema.js``` at line 134, write your root mutation:

```javascript
const Mutation = new GraphQLObjectType({
  name: 'Mutation',
  fields: () => ({
    addPhone: AddPhoneMutation,
  }),
});
```

I think it's self-explanatory.

Then, at line 145, modify your Schema export to contain the root mutation. It should be like this:

```javascript
export const Schema = new GraphQLSchema({
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

Let's go to our GraphiQL instance that is running on <a href="http://localhost:8080">http://localhost:8080</a>, I strongly recommend you to run it on a separate tab.

Run the following mutation on the left pannel:

```javascript
mutation {
	addPhone(input: {clientMutationId: "1" model: "Nexus 5", image: "https://goo.gl/Fq46CZ"}) {
    viewer {
      phones {
        edges {
          node {
            model
	    image
          }
        }
      }
    }
  }
}
```

**Explanation**: We are calling the mutation by ```addPhone```, the same name we have exported on our Root Mutation. Then, we need to supply the inputs(remember when we declared it on the **inputFields**?). 

- As you can notice, we are sending a parameter that was not into our **inputFields** declaration - `clientMutationId`. You don't need to worry about this field, it will be automatically filled by Relay under the hood. We only need to spoof it on GraphiQL(and any string value should work).

- What comes after the ```addPhone``` is the **outputFields** stuff.

After adding the Nexus 5 using our mutation, you can refresh the application on <a href="http://localhost:8080">http://localhost:8080</a> and check that the phone is there. 

Yes, your mutation worked pretty good!

### Your mutation working with Relay
Probably this is the part of the story you have been always stuck in, but don't worry, this time it will work.

Let's create the Relay part of your mutation - Go to ```/js/mutations``` folder and create a file named ```AddPhoneMutation.js```. 

Inside of this file, write the following code:

```javascript
import Relay from 'react-relay';

export default class AddPhoneMutation extends Relay.Mutation {

  static fragments = {
    viewer: () => Relay.QL`
      fragment on User {
        id
      }
    `,
  };

  getMutation() {
    return Relay.QL`mutation{addPhone}`;
  }

  getVariables() {
    let { phoneModel, phoneImage } = this.props;

    /**
     * If the fields come empty, force them to be null.
     * This way, it won't be filled on GraphQL.
     */
    phoneModel.length === 0 ? phoneModel = null : false;
    phoneImage.length === 0 ? phoneImage = null : false;

    return {
      model: phoneModel,
      image: phoneImage,
    };
  }

  getFatQuery() {
    return Relay.QL`
    fragment on AddPhonePayload {
      viewer
    }
    `;
  }

  getConfigs() {
    const { viewer } = this.props;
    return [{
      type: "FIELDS_CHANGE",
      fieldIDs: {
        viewer: viewer.id
      }
    }];
  }
}
```
**Explanation**: At the first moment it could be strange for you, but it will make sense. 

First of all, there's a sequence of the methods execution - I'll be explainig them at the same order they are executed. Also, part of these functions will be responsible to *send* data and other part will be *handling* the application after that the modifications happen.

- Firstly, we declare a static object called `fragments`. We are sending a GraphQL fragment for this object, specificaly an `User` fragment, because our mutation needs to know what is the `id` of the `User` that will be mutating. (Ok, we only have one `User` on this guide, but imagine this mutation handling an application that contains more than 1 `User`, it would require the `id` to know which `User` would be receiving a new `Phone`.

- The `getMutation()` method uses the `Relay.QL`(you can read more about this <a href="https://facebook.github.io/relay/docs/api-reference-relay-ql.html#content">here</a>) method to specify which mutation we'll be operating. Remember when we exported our mutation into our Root Mutation? That's what we must use here.

- On `getVariables()` we will be collecting the necessary data to send to our **inputFields**. Remember when we create them into our mutation on GraphQL? This function must return an object that matches exactly the **inputFields** we've declared on the `schema`. You will be sending this data by `props`, inside of your application from your React components. - We'll be reviewing this later.

- `getFatQuery()` we will be returning a fragment containing everything that could change as a result of this mutation. In our case, what changes here is our `User`(viewer). You can notice that the fragment name of the Fat Query is `AddPhonePayload`, but this is not declared anywhere - Yes, it isn't. This name is auto generated internally and it just concats the name of your mutation (`AddPhone`) + Payload. Remember when we declared the name of your mutation on the `schema`?

- At `getConfigs()` we advise how Relay should handle the `AddPhonePayload` returned by the server. In our case, we are using a `FIELDS_CHANGE` type (because our User has changed, now it has more Phones than before) and we specify the `id` of what is being changed, our `viewer` id (that is the `User` id). Here, like we passed the **inputFields** data to `getVariables()`, we must pass the `viewer` as a `prop` too. - We'll be reviewing this later.

At this point, your mutation should be working fine.

Now, we must configure how we will be sending the necessary data to our `AddPhoneMutation.js` (as `props`) into our React application, and setting how to call this mutation internally.

### Calling our Mutation from React

You've learned a lot until here, and you are almost close to get you mutation working by Relay. Let's adapt out React components to receive and call our `AddPhoneMutation.js`.

Go through `views/PhoneView.js` and import our mutation file at line 5:

```javascript
import AddPhoneMutation from '../mutations/AddPhoneMutation';
```

Scroll down until you reach the `Relay Container` of this view at line 119. Modify the fragment, including the mutation fragment at line 123. It should look like this:

```javascript
export default Relay.createContainer(PhoneView, {
  fragments: {
    viewer: () => Relay.QL`
      fragment on User {
        ${AddPhoneMutation.getFragment('viewer')}
        phones(first: 908098879) {
          edges {
            node {
              phoneId
              model
              image
            },
          },
        },
      }
    `,
  },
});
```

Ok, now your view is ready to receive the modifications comming from the mutation - It's time to set up how we will trigger it and pass the data as `props`.

If you check the line 51 of `PhoneView.js` you will notice that we have a component called `AddModal`:

```javascript
<AddModal viewer={viewer} handleModal={this.handleModal}/>
```

This component receives two `props`: `viewer` and `handleModal`. The `viewer` comes from our `Relay Container` as a  `prop` that Relay send to our React components. We are sending this to `AddModal` because we're going to call our mutation inside of this component - So let's do it.

Go to `js/components/AddModal.js`  and import our mutation here too, at line 4:

```javascript
import AddPhoneMutation from '../mutations/AddPhoneMutation';
```

To keep this guide as simple as possible, the `AddModal` component already have all the logic implemented, so you just need to worry about the mutation process.

Go to line 22, you will notice there’s a function in there called `addPhone` -  This function is triggered when user clicks on the `Add` button, that is on the modal.

Remove the `alert` that is inside of this function and let’s start working into passing data to our mutation. 

The form inside of this modal has two <a href="https://facebook.github.io/react/docs/uncontrolled-components.html">uncontrolled</a> inputs, their refs are `phoneModelInput` and `phoneImageInput`, and we are going to get its values to send as data for our mutation.

We will start declarating some constants inside of our function, so, right bellow the `addPhone()` declaration, write the following code:

```javascript
const { viewer } = this.props;
const { phoneModelInput, phoneImageInput } = this.refs;
```

Now we have three variables: `viewer` (that we are receiving as `props`) and `phoneModelInput` / `phoneImageInput` that are our inputs.

Ok, now we need to call our mutation sending the required data to our mutation. For this, we'll be using the `commitUpdate()` method from <a href="https://facebook.github.io/relay/docs/api-reference-relay-store.html">Relay.Store API</a>. This is one of the methods we can use to trigger a mutation.

The `commitUpdate()` method accepts two callbacks to be triggered after the server response:

- `onSuccess` is called if the mutation succeeded.
- `onFailure` is called if the mutation failed.

We will be implementing both, so we can close the `AddModal` when we reach `onSuccess` or show an error when we got an `onFailure` callback. In order to do that, let's implement `commitUpdate()` into our `addPhone()` function, it should look like this:

```javascript
addPhone = () => {
    const { viewer } = this.props;
    const { phoneModelInput, phoneImageInput } = this.refs;

    Relay.Store.commitUpdate(
      new AddPhoneMutation({
        viewer,
        phoneModel: phoneModelInput.value,
        phoneImage: phoneImageInput.value,
      }),
      {
        onFailure: (transaction) => {
          this.setState({
            error: 'Something went wrong, please try again.',
          });
        },
        onSuccess: (response) => {
          this.close();
        },
      },
    );
  };
  ```
  **Explanation**: We are calling the `commitUpdate()` from Relay.Store, then we create a new instance of our mutation by using `new AddPhoneMutation` - At this point, we will be sending the `props` we need inside our mutation.
  
 - The `viewer` prop will be used inside of the mutation on the `getConfigs()` function (Remember that?)
 
 - The `phoneModel` and `phoneImage` are receiving the value from our inputs and will be used by `getVariables()` function inside of the mutation. Then that function will be sending this data to the **inputFields** we declared at the `schema` (Makes sense now, right? xD)
 
 - At the `onFailure` callback we are setting a `state` that will just render an error into our `AddModal` component (You can reach this callback by clicking `Add` without filling the inputs).
 
 - At the `onSuccess` callback we just call a function that closes our modal.
 
 Ok then, time to test it! Go to <a href="http://localhost:3000">http://localhost:3000</a> and try adding a new Phone.
 
 If you have followed this guide correctly, you should be able to add `Phones` to `Users`, what means that **YOUR MUTATION IS WORKING!** 
 
 ### Next Steps
 Now that you know exatly how does a mutation work, why don't you try to implement two more mutations to delete and edit `Phones`? It will be fun!
 
 You can also check the <a href="https://github.com/pt-br/relay-phones">full application</a> in case you need to know how I'
 ve implemented these two mutations.
 
 I hope this guide have helped you!
 
