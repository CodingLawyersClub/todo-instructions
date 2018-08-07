# What We're Building

We're going to build a ToDo app! I know it's not the most exciting thing in the world, but it's basically a rite of passage to build at least one ToDo app. The cool thing about the app we're going to build is that even such a small app covers 90% of what all apps contain.

It's going to be built on something called a MERN stack. MERN stands for Mongo + Express + React + Node.

The app is going to be stupid simple. It's only going to do two things:
1. Allow a logged in user to create ToDos
2. Allow a logged in user to fetch his/her ToDos

That's it! I have already built it, so if you want to see what the final product looks like, you can check it out here: https://barabander-todo-frontend.herokuapp.com. Feel free to create an account and create some ToDos to try it out!

# Back End

## Define the `ToDo` Model

We're using MongoDB which is a really popular no-SQL database. To make things even simpler, we're using a library called `mongoose` that acts as a wrapper for MongoDB. One of the things we do we `mongoose` is define our models. Our models are how we want to represent the stuff we want to save to our database. Let's dive straight in.

Inside of `todo-server` create a new file under `models` named `ToDo.js`. Once that's done, let's import monogoose:

```
const mongoose = require('mongoose');
```

Great, now let's define our `schema`. A `schema` is the blueprint for what properties our model will contain. Define a schema as follows

```
let ToDoSchema = new mongoose.Schema({

}, {timestamps: true});
```

We're creating a new `mongoose` schema named `ToDoSchema`. The `timestamps` but just gives us a lot of stuff around when it was created / updated for free. Just copy and paste it. We're almost always going to have that piece.

Cool, that was easy. So our schema right now is pretty boring as we haven't defined anything yet. What kind of things would you say should go on a ToDo? Well, you'd want text that actually says what we need to do. So let's add that property:

```
let ToDoSchema = new mongoose.Schema({
    text: { type: String, required: true, trim: true },
}, {timestamps: true});
```

Woah, woah, there's a lot of stuff in there. Let's go through it.

1. We defined a property on our schema called `text`. 
2. We gave it a type of `String`. A `String` is a fancy way of saying text.
3. We want it `required`. We don't want the ability to create ToDo's without any text. This will make it impossible to do that.
4. We want to `trim` it. This simply removes any whitespace the user may add.

Cool, you've defined your first property! What else would our ToDo need? Maybe who's ToDo it is? Our app is going to have a lot of users and I don't want to be seeing your ToDos!. Let's add another property to our schema.

```
let ToDoSchema = new mongoose.Schema({
    text: { type: String, required: true, trim: true },
    user: { type: mongoose.Schema.Types.ObjectId, required: true } <-- Add this
}, {timestamps: true});
```

Great, let's walk through this property now.
1. The type is an `ObjectId`. I already have a model named `User`. This may seem a little weird – why isn't the type User? Well, you don't put full models on other models. You put a reference to them, which is it's `ObjectId`. The `ObjectId` is a unique value for every `User`.
2. We also want to `require` the reference to the user. We don't want you to be able to create a ToDo without referencing who created it.

Great, that's looking good for now! Let's do one more thing to this file.

Below the schema definition, add the following:

```
mongoose.model('ToDo', ToDoSchema);
```

This is necessary to register the schema we just created with `mongoose`. When we're all said and done, `ToDo.js` should look like this:

```
const mongoose = require('mongoose');

let ToDoSchema = new mongoose.Schema({
    text: { type: String, required: true, trim: true },
    user: { type: mongoose.Schema.Types.ObjectId, required: true }
}, {timestamps: true});

mongoose.model('ToDo', ToDoSchema);
```

OK, the only thing left to do is to let our app know about our `ToDo` model.

Go to `app.js`. Search for `require('./models/User');`

Below that line, let's put in our new model:

```
require('./models/User');
require('./models/ToDo'); <-- Add this
```

Great now our app knows about our ToDo!

## ToDo Routes

Now we need to define an endpoint to work with our `todos`. Basically, an endpoint is a specific url we call to interact with our database, whether it be to pull or push data. Behind most websites you use there are endpoints that are fueling the data you see on your screen. As an example, you can check out the data feeding a website like CryptoKitties [here](https://api.cryptokitties.co/v2/kitties).

To set up our routes, first let's create a new file named `todos.js` under `routes/api/`. Let's import what we're going to need, and walk through each one.

```
var mongoose = require('mongoose');
var router = require('express').Router();
var ToDo = mongoose.model('ToDo');
var User = mongoose.model('User');
var auth = require('../auth');
```

1. `mongoose`, you know what this is now. A wrapper for MongoDB
2. `router` this is how we create our routes
3. `ToDo`, the model we just created
4. `User`, a model I created in advance
5. `auth`, a library that deals with state of a user, logged in or logged out.

At the bottom of the file, add the following:

```
module.exports = router;
```

This exports the `router` to the rest of our app. The `todos.js` should look like this:

```
var mongoose = require('mongoose');
var router = require('express').Router();
var ToDo = mongoose.model('ToDo');
var User = mongoose.model('User');
var auth = require('../auth');
  
module.exports = router;
```

Great! Now, let's add it to the rest of our app.

Go to `routes/api/index.js`. Let's import our `/todos` route:

```
router.use('/', require('./users'));
router.use('/todos', require('./todos')); <-- Add this!
```

Great! Our app now knows our `todos` route. There's nothing there though! Let's change that.

## Build the Create ToDo Route

OK, let's create our first route. In `api/todos` Below the imports and above the export, put the following:

```
router.post('/', auth.required, async (req, res, next) => {

});
```

OK, let's walk through this.
1. We're defining a POST method by writing `router.post`. A POST method means that this method will be writing to the database, creating our ToDo
2. `/` defines the route. Because we wrote `router.use('/todos', require('./todos'));` in our `api/index.js`, the route is actually `websitename/api/todos`.
3. `auth.required`. This is some magic that requires our users to be logged in when calling this endpoint. Do not worry about how this is working behind the scenes
4. `async (req, res, next) =>` The `async` gives us the ability to use special syntax called `await`. We are almost always going to have it on our endpoints, so just default to including it. the `(req, res, next` and automatically passed. You'll see how we use them in a bit

Cool! So we have the skeleton of our endpoint. Let's start filling it in :).

First let's put in something called a `try...catch`. These are really useful for error handling. Add the following code inside of the method:

```
try { 

} catch (e) {

}

```

Cool, now what the heck is this. Basically, we're going to "try" to execute code in the first `{ }`. If that fails at any point, the code inside of the `catch` is going to automatically be run.

Let's put stuff in our `try`

First, this endpoint is creating ToDos from our app. The user is giong to pass up text. The user passes text through the request `body`. So, let's get what the user passed up:

```
const todoFromRequest = req.body;  
const { text } = todoFromRequest;

```

Cool, now we have the `text` off the `toDoFromRequest`. The `const { text }` structure is getting a property off of `toDoFromRequest` and defining it as `text`.

OK, now let's find our user who is making the request. We're going to need this user to save to our `ToDo`. The user's id is automatically sent up in the `payload`. We're then going to query our database by calling `findById` to find the exact user making the request. The `await` is some magic syntax that tells us to wait for that query to complete before moving on.

```
const user = await User.findById(req.payload.id);

```

Great! Because we now have the `text` and the `user` we have everything we need to create our ToDo!. Let's make a new `ToDo` and pass in the `user` and `text`:

```
const toDo = new ToDo({user, text});
```

Let's then save it to our database by calling `save()`:

```
const savedToDo = await toDo.save();
```

Wow, you're now saving ToDo's to our database! Now, I don't want to return all of the data of our `ToDo`, just what the frontend actually needs. Lets go back to our model at `ToDo.js` and define a method to structure our `ToDo` to return.

In `ToDo.js` write the following method:

```
ToDoSchema.methods.toJSON = function(){
    return {
        id: this.id,
        text: this.text
    }
}
```

This looks scary but it's actually really simple. Let's walk through it.

1. We're defining a method on the ToDoSchema named `toJSON`
2. The `function()` is just saying this is a method
3. We are returning the `id` of the todo `this.id` and the text `this.text`

Now, if we call this method, we're only going to return the `id` and the `text` of the `ToDo`. We try and minimize what we share with the frontend whenever possible.

Cool, back in `todos`, let's return our `savedToDo` and call our new `toJSON` method:

```
return res.json({toDo: savedToDo.toJSON()})
```

Amazing, now let's add some error handling.

Inside of the `catch {..}` let's add the following:

```
console.error(e);
next(e);
```

`console.error(e)` is going to log errors for us that occur so we can see what went wrong
Do not worry about what `next(e)` is. Just include it.

Fantastic! We are completely done with this create endpoint! It should look like this:

```
router.post('/', auth.required, async (req, res, next) => {
    try {
        const todoFromRequest = req.body;        
        const { text } = todoFromRequest;
        const user = await User.findById(req.payload.id);
        const toDo = new ToDo({user, text});
        const savedToDo = await toDo.save();
        return res.json({toDo: savedToDo.toJSON()})
    } catch (e) {
        console.error(e);
        next(e);
    }
});
```

## Build the Find ToDos Route

So we can create ToDos. But as a logged in user, I want to be able to the ToDos I've created! Let's make another route:

```
router.get('/', auth.required, async (req, res, next) => {

});
```

This route should look largely the same except for one huge difference. It's a `.get` instead of a `.post`. See, we are now GETting data, not POSTing, so we need to update that endpoint to reflect that. Now when we call `GET` `/todos`, we're going to hit this endpoint.

So let's start filling it in. Add a `try...catch` like before. Fill in the `catch` block in exactly the same way

```
router.get('/', auth.required, async (req, res, next) => {
    try {

    } catch (e) {
        console.error(e);
        next(e);
    }
});
```

Great, now let's actually set up our query in the `try` block. Add the following:

```
const user = await User.findById(req.payload.id);
const toDos = await ToDo.find({user}).sort({ createdAt: 'descending' });;
return res.json({toDos: toDos.map((toDo) => toDo.toJSON(toDo))});
```

Let's walk through it.

1. We find the user from the request again. This is the exact same thing we did in the creation endpoint. We'll need the user because we only want to return ToDos for the user making the request. I wouldn't want you to be able to see my ToDos!.
2. We make a new query with `ToDo.find`. The first thing we do is pass in parameters. The `find({user})` says, listen, I don't want all ToDos, I just want the ToDos where the `user` property is the exact `user` that is making this request.
3. We sort them by when they were created with `.sort`. It `descending` so the newest will be first in the list, oldest will be last
4. We return our ToDos with `return res.json`. You can see we `map` over them first. This basically says, take the list of `toDos` you found, go over every single one, and call our `toJSON` method. This makes it so we're only returning what we want to return.

Great! You now have the ENTIRE backend set up. Give yourself a pat on the back. Let's go and call these endpoints from the front end!

# Front End

## Create Page Structure

First Sign Up. It doesn't matter what username or password you use. You'll need to be logged in.
Create a folder under components named “Create”
Create a file inside of “Create” folder named “index.js” This file becomes the default when ‘components/Create’ is imported
First, we import React. This pulls in the react library and is essential for creating our page:

```
React, { Component } from 'react';
```

Next we define a class named `CreatePage`:
  
```
export default class CreatePage extends Component {

}
```

`export default` says that when this page is imported, we're going to give whoever is calling it this class
`class CreatePage extends Component` says that this page is a type of a React Component. This gives us a ton of stuff for free out of the box, and is how we get the awesomeness of React in our app.


Next, we need to add the `render()` method. This is where we define what HTML this page will display. For now, just mark something that let's us know we're on this page. A `<div>` is a generic HTML component to display info.

```
render() {
  return (
    <div> Welcome to the create page!</div>
  )
}
```

Altogether now, it will look like this:

```
import React, { Component } from 'react';
export default class extends Component {
    render () {
        return (
            <div> Welcome to the create page!</div>
        )
    }
}
```

## Link Up the Route

Go to the `routes.js` at `src/routes.js`
Import the create page:

```
import CreatePage from '../components/Create';
```

Because of the `index.js`rule mentioned above, the page we just created will be pulled. Now we need to define when this page is displayed. This is done through `Routes`. 

Above `<Route component={Home} />` put the following:

```
<Route path="/create" component={CreatePage} />
```

This says when we go to our website, `localhost:3005/create` /create, we're going to automatically go to our Create page. You should see the create page loaded
![alt text](https://s3.amazonaws.com/clc-images/CreatePage.png)

## Create a Form With A Submit

Import a Button, a Form component, a FormInput, Yup, Formik, and Box. You'll see what all these do in practice soon.

```
import { Button } from 'semantic-ui-react'
import Form from '../Form';
import FormInput from '../FormInput'
import * as Yup from 'yup';
import { Formik } from 'formik';
import { Box } from 'rebass';
```

First, let's set up a validator. A validator is used to make sure the user doesn't submit data they shouldn't. When you've submitted forms and you've gotten a "password isn't long enough" message, a validator is what is checking the logic. We're goign to use a validation library called Yup because it's super simple to use. Paste the following code, and we'll walk through it:

```
const FormSchema = Yup.object().shape({
    text: Yup.string()
        .required('Text for your To-Do is required')
});
```

`const FormSchema` defines a variable named FormSchema
`Yup.object().shape()...` defines the validation object. This says what our form is going to be expecting. We're only going to have one field for the text of our To-Do:

```
text: Yup.string()
    .required('Text for your To-Do is required')
```

Now, it's time to set up our form to put our validation in. Copy the following code for the `render()` method. I have put comments to explain what each does.

```
render () {
    return (
        <Formik // A form library that takes care of a lot of magic for us
        initialValues={{text: ''}} // Defines what the initial text will be, a blank string ('')
        validationSchema={FormSchema} // Uses our schema we defined above, so 'text' can't be blank
        render={({ errors, touched, onSubmit }) => ( // Tells it what to draw
        <Form>
            <Box>
                <Box p={10}>
                    <FormInput name="text" placeholder="Clean the basement" type="text" />
                </Box>
                <Box p={10}>
                    <Button type="submit">Create</Button>
                </Box>
           </Box>
        </Form>
        )}
    />)    
}
```

Save the file and watch the page reload. You just set up your first form! Try clicking the "Create" button without putting in any text. That's the FormSchema in action :).

## Link Up With MobX and Hit the Internet

Now that we have our form good to go, we need something to happen when you click Create. We use MobX to initiate our network requests and load data. So let's get started creating what we call a MobX store.
Go to `src/stores/` and create a new file called `toDoStore.js`
At the top of `toDoStore.js` import the following:

```
import { observable, action, flow } from 'mobx';
import agent from '../agent';
import error from '../utils/error';
```

You'll see these imports in action later.

Define a class named `ToDoStore`:

```
class ToDoStore {

}
```

Finally export the store:

```
export default new ToDoStore();
```

Altogether `toDoStore.js` should look like this:

```
import { observable, action, reaction } from 'mobx';

class ToDoStore {

}

export default new ToDoStore();
```

## Add the `toDoStore` to our App

Go to `src/index.js`. We're now going to add this store we created to our app.

First, import it on line 13:

```
import toDoStore from './stores/toDoStore';
```

Then add it to the list of stores on line 23:

```
const stores = {
  authStore,
  commonStore,
  userStore,
  toDoStore <-- Add this one!
};
```

Great, now this new store is in your app! It doesn't do anything yet though. Let's change that!

## Building Our Requests

First, we need to define our network request to make. All of our network requests are made in the `agent.js` file. Navigate to it (TIP: don't forget you can always search for a file in VSCode by searching CMD + P).

Below `Account` add a new variable named `ToDo`. This is where all the network requests around the `ToDo` model will go. This just keeps things organized and much easier to read.

```
const ToDo = {

};
```

Let's add a method called `create`. This method is going to create a new To-Do in our database by calling an endpoint /toods. We use the `.post` method when we want to create something. We'll be passing in the data, the `toDo` from the form directly. We'll see how that works in a bit.

```
const ToDo = {
  create: (toDo) =>
    requests.post(`/todos`, toDo),
};
```

Export the ToDo so other files can know about its existence:

```
export default {
  Account,
  Auth,
  ToDo <-- Add it here!
};
```

## Write the `createToDo` Function

Go back to `toDoStore.js`. Let's link everything together.
Create a new action called `createToDo`. It should look like this:

```
@action
createToDo = flow(function* () {

});
```

There's a lot of fancy code going on there. There's only two things I want you to understand.
1. This is an action. The `@action` gives this function special properties. When you're taking an action (hitting the internet, for example) be on the safe side and define it as an `action`
2. The `flow` we use because we're making a network request. Always include this when hitting the internet.
That's all you need to know for now.

Let's fill this function!

Inside of `createToDo` let's pass in the toDo. This is the text from our input before. Let's also call that `create` method we put in `agent` on the `ToDo` object. The `yield` says that we're making a network request, and do not move on to running the next line until the result comes back, which we're defining as `response`. We're going to `console.log` the result, which will print it out for us to see in the browser. It will look like this:

```
@action
createToDo = flow(function* (toDo) {
  const response = yield agent.ToDo.create(toDo);
  console.log("OUR CREATED TODO", response.toDo);
});
```

## Link Up Our `createToDo()` method with our Create page

Go back to `Create/index.js`. Add the following to get our MobX store into our create page:

```
import { inject, observer } from 'mobx-react' <-- Import this on top
...
@inject("toDoStore")
@observer
export default class Create extends Component {
```

MAKE SURE THE `@inject`, `@observer` and `export` statement ARE TOUCHING EXACTLY HOW YOU SEE ABOVE.

Now, let's call our create method.

Under `<Formik` add the `onSubmit` exactly how you see here:

```
<Formik
onSubmit={formValues => {
    this.create(formValues) 
}}
```

This means that when the Create button is clicked, our form is automatically going to call this `onSubmit`. It is passing the values from the form (the text of our toDo) and then calling a method that does not exist yet called `createToDo`. Let's define that method now.

On line 41, add a `createToDo` method:

```
create(formValues) {

}
```

On every React component there is a magical property called, well, `props`. We're going to call our `createToDo` method by accessing the `props` on this Create Page. Pass in the formValues because this is what contains the `text` of our To-Do:

```
create(formValues) {
  this.props.toDoStore.createToDo(formValues)
}
```

Believe it or not, you're all synced up on the front end to create To-Dos. Let the page reload, add some text to the input, and click Create. Open up your console by right clicking --> Inspect --> Console. You should see the log you wrote before with some data. That is your To-Do returned from the server :)

# Displaying the To-Dos for a User

Now that we've created our To-Do, let's display them for a particular user!

## Write the `find()` Method

First, we need to go to our `agent.js` file to write the method to call our `/todos` endpoint. Let's do that now. On `agent.js` go to `ToDo` and copy and paste what we have under `create`. We're going to modify it slightly to be our `find` function. Check out the arrows for how I did this.

```
create: (toDo) => <-- rename to `find` and get rid of the to `toDo` parameter. The endpoint we built doesn't need any parameters besides the user, which is automatically sent up for us. That means we don't need to pass antyhing.
  requests.post('/todos', toDo), <-- Because we are GETting data data, not POSTing data, we need to make this a `.get`. We also need to get rid of the parameter for the same reason as before.
```

When it's all said and done our `find` method should look like this:

```
find: () =>
  requests.get('/todos')
```

Great! Now we have all we need in our `agent` file. Let's go to our MobX store to call it!

## Write the `findToDos()` Method

Go to toDoStore.js. Let's add a new method to fetch our To-Dos called `findToDos`. Because it's a simple network request, we're going to copy `createToDo` and adjust it. Check out the arrows for how I did this:

```
@action
createToDo = flow(function* (toDo) { <-- rename this function to 'findToDos' to reflect what this function will be doing. We're no longer passing a parameter, so get rid of 'toDo'
    const response = yield agent.ToDo.create(toDo); <-- call the new function on ToDo we created named 'find' and get rid of the passed parameter, 'toDo' as, again, we're no longer passing a parameter
    console.log("OUR CREATED TODO", response.toDo); <-- rename the log to 'OUR FOUND TODOS' to reflect what this method is returning, and change 'response.toDo' to 'response.toDos' because we're getting back an array of toDos rather than one
});
```

When it's all said and done, it should look like this:

```
@action
findToDos = flow(function* () {
    const response = yield agent.ToDo.find();
    console.log("OUR FOUND TODOS", response.toDos);
});
```

## Call `findToDos()` from `componentDidMount`

Now, we need to call this `findToDos` from somewhere. Let's set that up!

Go to `Home/index.js`. Above the `render()` method, add a new method named `componentDidMount`.

```
componentDidMount() {

}
```

`componentDidMount` is a super important React lifecycle method. Basically, whenever you do the `extends Copmonent` the file automatically gets React lifecycle methods like `componentDidMount`. Whenever the page loads, `componentDidMount` is called and the code inside of it is run. This means that it's an excellent place to make network requests because when we navigate to home (localhost:3005), this method will be called. Let's fetch our To-Dos in here!

Like before, we need to import our `toDoStore` into our page. We do that the same way as the `Create` page.

```
import { inject, observer } from 'mobx-react' <-- Import this on top
...
@inject("toDoStore")
@observer
export default class Home extends Component {
```

Great! Now we have access to our `findToDos` via `props`. Let's call it like before:

```
componentDidMount() {
  this.props.toDoStore.findToDos()
}
```

That's it. Now save, and reload the page. Open up the chrome inspector and go to console. You should see your `OUR FOUND TODOS` logged. Everything is working. Now we need to display our To-Dos!

# Displaying our To-Dos

Logging To-Dos is fun but I think it's way more fun to see them in a Table! Let's do that now.
We haven't really used what MobX is awesome for which is maintaining state, but that's about to change.

In toDoStore.js let's add a new property called `toDos` right below the class definition. We're going to mark it as `@observable` and set it to be a blank array `[]`

```
...

class ToDoStore {
    @observable toDos = [];
...
````

`@observable` is some MobX magic that says: "Look at this variable, `toDos` and when it changes, I want you to automatically redraw my page with the new values inside." I like to think of it as a flag that I know my components are watching to see if they change. The blank array we set because we currently don't have any ToDos, so our list is currently blank!

Now, go to the `findToDos` method, remove the `console.log` and and add the following:

```
@action
findToDos = flow(function* () {
    const response = yield agent.ToDo.find();
    this.toDos = response.toDos; <-- Add this!
});

```

This simply sets our `@observable` array `toDos` to be what we get back from our server. Great, now let's use this value in our component.

First, we need to create a component that displays our To-Do. One simple way to do this is to put it in a table. Every table has table rows, so we're going to make a `ToDoRow` component.

Create a new file under components named `ToDoRow.js`. Like all React components the bare structure should look like this:

```
import React, { Component } from 'react';

export default class TableCell extends Component {
      
  render() {
  
  }
}
```

Let's fill this in!

First we need to import a couple of things in addition to React. Let's do that.

```
import { Table } from 'semantic-ui-react';
import dates from '../utils/dates';
```

`semantic-ui-react` is an amazing library that gives us a ton of UI for free. It is going to do most of the heavy lifting to make our table row look nice

`dates` is a library I made that is a wrapper for an amazing library named `moment.js`. Don't worry too much about this one.

Great, now let's create our table row.

Each row is going to be passed a specific `toDo` to display via the `props`. I wasn't kidding when I told you props are magic! Let's get them off of props.

```
 const { toDo } = this.props;
```

This is a useful short hand you'll see a lot. It is identical to doing the following:

```
const toDo = this.props.toDo
```

the `const { toDo }` is pulling and defining at the same time. Do whichever you feel more comfortable with.

Now we need to put this `toDo` we pulled off our props to use. We're going to use the `Table.Row` and `Table.Cell` off of `semantic-ui`. It's going to look something like this:

```
return (
    <Table.Row>
      <Table.Cell>{toDo.text}</Table.Cell>
      <Table.Cell>{dates.formatDate(toDo.createdAt)}</Table.Cell>
    </Table.Row>
)
```

There's a lot going on here, so let's walk through it. First, were wrapping everything in a `Table.Row` because that is what this component is supposed to return. Inside of our `Row` we have two cells. One cell we want to display the ToDo's text, and the other we want to display the date it was created. In React, we display text to the page with those `{ }` brackets. The first cell is displaying the `text` off the ToDo, the second is displaying the `createdAt` date with some formatting from my date library. The whole component should look like this:

```
import React, { Component } from 'react';
import { Table } from 'semantic-ui-react';
import dates from '../utils/dates';

export default class TableCell extends Component {
      
  render() {
    const { toDo } = this.props;
    return (
        <Table.Row>
          <Table.Cell>{toDo.text}</Table.Cell>
          <Table.Cell>{dates.formatDate(toDo.createdAt)}</Table.Cell>
        </Table.Row>
    )
  }
}
```

Boom, you've created a component! Let's import it and put it to use!


Go to `Home/index.js` and import two things:

```
import ToDoRow from '../../components/ToDoRow';
import Table from '../../components/Table';

```

This simply imports a `Table` component I've created along with the `ToDoRow` we just made.

First, lets get our ToDos from the `toDoStore`. This is incredibly simple because they live on our props now. The `@observer` on this component (above the class) tells this component to listen to any `@observables` we defined in our `toDoStore`. Do you remember what we defined as an `@observable`? Our `toDos`! Everytime the `toDos` array changes (like when a network request finishes and fetches data) this component, `Home/index.js` will redraw and pass in the `toDos` in the props. Which means we can do this now:

```
...
render() {
    const { toDos } = this.props.toDoStore;
...
```

This simply gets the `toDoStore` and pulls off the `toDos` array we defined earlier. MobX will take care of automatically redrawing everything for us :)

Now

Delete the `<div>` that says `Welcome to your new website!` and replace it with the following:

```
<Table
striped
headings={["Text", "Created At"]}
rows={toDos.map((toDo) => <ToDoRow key={toDo.id} toDo={toDo} />)}
/>
```

`striped` is some magic that gives us auto striping. Don't worry about this too much.
`headings` is the headings on our table. They need to be in the same order as our cells we defined on `ToDoRow`. So, the first one is `Text` and the second is `Created At`.
`rows` is the star of the show. It takes in an array of our `ToDoRow`. We `map` over each `toDo` in our `toDos` array, and pass it to the `ToDoRow` to draw. The `key` is just a unique property that react requires whenever we do a `map`. It's ok if this doesn't make complete sense yet, you're going to be seeing `map` a ton and I promise it will become clear.

The whole `Home/index.js` should look like this:

```
import React, { Component } from 'react';
import ToDoRow from '../../components/ToDoRow';
import Table from '../../components/Table';
import Layout from '../../components/Layout'
import { Helmet } from "react-helmet";
import { inject, observer } from 'mobx-react'

@inject("toDoStore")
@observer
export default class Splash extends Component {

    componentDidMount() {
      this.props.toDoStore.findToDos()
    }

    render() {
        const { toDos } = this.props.toDoStore;        
        return (
            <Layout showLogo={true}>
              <Helmet>
                <title>New Website</title>
              </Helmet>
              <Table
              striped
              headings={["Text", "Created At"]}
              rows={toDos.map((toDo) => <ToDoRow key={toDo.id} toDo={toDo} />)}
              />
            </Layout>
        )
    }
}

```

Save and run. You should see your table of ToDos for the logged in user! Woohoo! Go to localhost:3005/create and make a bunch of ToDos. Go back to the homepage. You should see them in the table, nice and striped!

You should feel super proud of yourself. I know there's a ton of moving pieces right now and you certainly will not understand everything that goes on. Just know that it really doesn't get any more complicated than this. You've built an app that can create and fetch data with a logged in user. That is 90% of the work out there. Let's clean things up a little bit and push to production to show our friends.

# Clean Up

Right now we have everything working but our app kind of sucks. Let's clean it up to be much more production friendly to share with our friends.

## Create a Logged Out State On ToDo list page

We have made it so users can only see their own ToDos. The problem is, if you're not logged in, you won't see anything and our request via `findToDos()` will fail. Let's have a logged out state.

On `Home/index.js` add the `userStore`. This came with the template so don't worry too much about it, but basically it keeps track of the current user for us.

```
@inject("toDoStore", "userStore")
```

The `userStore` has an `@observable` called `currentUser`. The `currentUser` is the logged in user. If there is no logged in user, it is undefined. We can use this variable to check if our user is logged in or not to check whether or not we should call `findToDos()`

Change `componentDidMount()` to look like the following:

```
componentDidMount() {
  if (this.props.userStore.currentUser) {
    this.props.toDoStore.findToDos()
  }
}
```

See what's going on here? We find the property called `currentUser` on the `userStore`. If there is a value there, we know the user is logged in, and we make the request. If not, well, the block of code `this.props.toDoStore.findToDos()` is never run! Perfect!

Let's change the UI to conditional display. We're going to user a neat little trick. Look at the following block of code:

```
{true && <div>I will display!</div>}
```

Because the first part of this statement is `true` the second part is going to be displayed. That is in contrast to:

```
{false && <div>I will NOT display</div>}
```

OR

```
{undefined && <div>I will NOT display</div>}
```

Because the first part of this statement is `false` or `undefined` the second part is NOT going to display. The compiler stops looking because the first part is `false` or `undefined`. Why look at the rest if the first part is `false` or `undefined`?

We can use this neat feature to display info based on a logged in user. Instead of false/undefined vs true, we're going to use `currentUser` vs an `undefined currentUser`

Replace the relevant parts as follows:

```
{currentUser && <div>
  <Flex justifyContent="flex-end">
    <Box>
      <Button onClick={() => this.props.history.push("/create") } content='New' icon='plus' labelPosition='right' />
    </Box>
  </Flex>
  <Table
  isLoading={isFetchingToDos}
  striped
  selectable                
  headings={["Text", "Created At"]}
  rows={toDos.map((toDo) => <ToDoRow key={toDo.id} toDo={toDo} />)}
  />
</div>}
{!currentUser && <div>Login / Sign-Up to to see your To-Dos!</div>}
```

We wrapped the `<Flex>` and `<Table>` elements in a `<div>` because we only want them to show if there is a `currentUser`. If there is no `currentUser` (which is `!currentUser`) we want to tell them to login or sign-up.

## Turning `Create` into a Private Route

Our `Create` page requires a logged in user. Lets change our route to require that.

Go to `routes/index.js`

We have a special component called `<PrivateRoute` which requires a logged in user. All you need to do is turn this:

```
<Route path="/create" component={CreatePage} />
```

Into a `<PrivateRoute />`

```
<PrivateRoute path="/create" component={CreatePage} />
```

Now when you navigate to Create page, it will kick the user home if they're not logged in.

## Linking to the Create Page

Expecting our users to navigate to `localhost:3005/create` is kind of ridiculous. Let's create a button that links to it.

Navigate back to `Home.index.js`.

Lets import some stuff to clean this page up. Add the following:
```
import Heading from '../../components/Heading';
import { Box, Flex } from 'rebass';
import { Button } from 'semantic-ui-react';
import Container from '../../components/Container';
```

`Heading` we use for the title of the page. It's just big text.
`Box` and `Flex` we use for layouting purposes. You'll see this in action soon
`Button` is from our amazing `semantic-ui-react` library and gives us a really nice button for free
`Container` is a wrapper component that gives some padding

First, everything feels very cramped. We have a component that adds padding named `Container`. Let's wrap our `<Table />` inside of a `Container`. Let's also put inside of this container a `<Header />` above our `<Table />` to clearly display what this page is. Finally, let's rename the `<title />`, which is what you see on the tab, to "ToDo List". It should look like this:

```
render() {
    const { toDos } = this.props.toDoStore;        
    return (
        <Layout showLogo={true}>
          <Helmet>
            <title>New Website</title>
          </Helmet>
          <Container>
            <Heading>ToDo List</Heading>
            <Table
            striped
            selectable                
            headings={["Text", "Created At"]}
            rows={toDos.map((toDo) => <ToDoRow key={toDo.id} toDo={toDo} />)}
            />
          </Container>
        </Layout>
    )
}
```

Let the page reload. It should start looking better already. Let's link to the Create page now.

Between `<Heading />` and `<Table />` add the following:

```
<Flex justifyContent="flex-end">
  <Box>
    <Button onClick={() => this.props.history.push("/create") } content='New' icon='plus' labelPosition='right' />
  </Box>
</Flex>
```

Let's walk through each piece:
`Flex` is a layout component. It helps with aligning things on the page. `justifyContent` means which way to align items. `flex-end` means to push everything to the right. So all this is saying is, everything inside of `<Flex />` I want to align right.
`Box` is what goes inside of `Flex`. Think of `Flex` and `Box` as peanut-butter and jelly. They always go together. `Flex` says how to align the `Box`. Because our `Flex` is `flex-end`, it is our `Box` that is going to be pushed all the way to the right.
`Button` is pretty self explanatory. This `Button` lives inside of my `Box` which is being pushed all the way to the right. `content` is what the button will say. `icon` is if we want it to have an icon. A list of icons can be found online, but don't worry about that for now. `labelPosition` says where you want the button to go, which I want it on the right of my "New" text.
 `onClick` is one of the most important methods. It does what you'd think it does. When this button is clicked, whatever is inside of that arrow is going to be called
 `this.props.history.push` is how we navigate to pages. the `/create` page is where we want to go. Don't worry too much about what's going on here. Just remember, if you want to go to a different page we created, you use `this.props.history.push("/PAGE_ROUTE_HERE")`. We defined our page route in `src/routes/index.js` if you forget the route name.

Your page should look like this:

```
import React, { Component } from 'react';
import ToDoRow from '../../components/ToDoRow';
import Table from '../../components/Table';
import Layout from '../../components/Layout'
import { Helmet } from "react-helmet";
import { inject, observer } from 'mobx-react';
import Heading from '../../components/Heading';
import { Box, Flex } from 'rebass';
import { Button } from 'semantic-ui-react';
import Container from '../../components/Container';

@inject("toDoStore")
@observer
export default class Splash extends Component {

    componentDidMount() {
      this.props.toDoStore.findToDos()
    }

    render() {
        const { toDos } = this.props.toDoStore;        
        return (
            <Layout showLogo={true}>
              <Helmet>
                <title>ToDo List</title>
              </Helmet>
              <Container>
                <Heading>ToDo List</Heading>
                <Flex justifyContent="flex-end">
                  <Box>
                    <Button onClick={() => this.props.history.push("/create") } content='New' icon='plus' labelPosition='right' />
                  </Box>
                </Flex>
                <Table
                striped
                selectable                
                headings={["Text", "Created At"]}
                rows={toDos.map((toDo) => <ToDoRow key={toDo.id} toDo={toDo} />)}
                />
              </Container>
            </Layout>
        )
    }
}
```

Let it reload. Try clicking "New". You should be brought to the `/create` page. Everything should be looking much better now. Let's add some loading indicators.

## Loading Indicators

Internet requests take time and users freak out when they take action but appears nothing is happening. This is why almost every modern website you use has a spinner to show feedback that it's working on the request. Let's add some spinners to show when things are loading.

First, go to `toDoStore.js`. Add two `@observable` properties

```
@observable isCreatingToDo;
@observable isFetchingToDos;
```

These variables are going to be how we track loading for creating and fetching To-Dos. Notice the `is` in front of each variable. That is convention for variables that are true or false (known as Boolean variables). Now let's use them:

```
@action
createToDo = flow(function* (toDo) {
    this.isCreatingToDo = true;
    const response = yield agent.ToDo.create(toDo);
    this.isCreatingToDo = false;
});

@action
findToDos = flow(function* () {
    this.isFetchingToDos = true;
    const response = yield agent.ToDo.find();
    this.toDos = response.toDos;
    this.isFetchingToDos = false;
});
```
As you can see, we simply set the variable to `true` right before fetching. After the fetch as completed (after the `yield`, we set them back to `false`. Think of it like turning a light switch on to look for something in a room, and when you've found it, turning the light switch back off.

Now, let's use them on our component. Go back to `Home/index.js`

Where we pull off the `toDos` now we're also going to pull off one of our new variables, isFetchingToDos:

```
const { toDos, isFetchingToDos } = this.props.toDoStore;                
```

Great, now we have our loading variable. Like with our `toDos` `@observable`, our page will automatically watch the `isFetchingToDos` variable.

`<Table />` has some magic that will show a spinner when `isLoading` is set to `true` and hide the spinner when `isLoading` is set to `false`. `<Table />` should now look like this:

```
<Table
isLoading={isFetchingToDos}
striped
selectable                
headings={["Text", "Created At"]}
rows={toDos.map((toDo) => <ToDoRow key={toDo.id} toDo={toDo} />)}
/>
```

Let the page reload. You should see a flash of a spinner now. It's extremely fast because the network request is straight to the server running on our computer. In production, it will prove very useful.

Go to the Create page. Let's do the same dance.

Add the `isCreatingToDo` in your `render()` method before the `return`

```
const { isCreatingToDo } = this.props.toDoStore;
```

Our `<Form />` has a special component called `loading`. Let's pass it `isCreatingToDo`.

```
<Form loading={isCreatingToDo}>
```

Your `render()` should look like this now:

```
render () {
    const { isCreatingToDo } = this.props.toDoStore;

    return (
        <Formik // A form library that takes care of a lot of magic for us
        onSubmit={formValues => {
            this.create(formValues)
        }}
        initialValues={{text: ''}} // Defines what the initial text will be, a blank string ('')
        validationSchema={FormSchema} // Uses our schema we defined above, so 'text' can't be blank
        render={({ errors, touched, onSubmit }) => ( // Tells it what to draw
        <Form loading={isCreatingToDo}>
            <Box>
                <Box p={10}>
                    <FormInput name="text" placeholder="Clean the basement" type="text" />
                </Box>
                <Box p={10}>
                    <Button type="submit">Create</Button>
                </Box>
           </Box>
        </Form>
        )}
    />)    
}
```

Let the Create page reload. Click the create button. You should see a spinner appear over the entire form. Look at that loading UI in action!

## Finalize the Create Page

The Create page looks a little lonely. Also, when you successfully create a ToDo nothing happens. Lets fix that.

First, let's import `Layout`. `Layout` gives us the navigation bar on top. Let's also wrap the whole thing in a `Container`, so import that as well. Finally, I want to give the page a `<Heading>` explaining what this page is for. Your imports should look like this:

```
import Layout from '../../components/Layout';
import Container from '../../components/Container';
import Heading from '../../components/Heading';
```

And your new components should be put in like this:

```
return (
    <Layout>
        <Container>
            <Heading>Create a To-Do!</Heading>
            <Formik // A form library that takes care of a lot of magic for us
            onSubmit={formValues => {
                this.create(formValues)
            }}
            initialValues={{text: ''}} // Defines what the initial text will be, a blank string ('')
            validationSchema={FormSchema} // Uses our schema we defined above, so 'text' can't be blank
            render={({ errors, touched, onSubmit }) => ( // Tells it what to draw
            <Form loading={isCreatingToDo}>
                <Box>
                    <Box p={10}>
                        <FormInput name="text" placeholder="Clean the basement" type="text" />
                    </Box>
                    <Box p={10}>
                        <Button type="submit">Create</Button>
                    </Box>
            </Box>
            </Form>
            )}
            />
        </Container>
    </Layout>
)
```

Cool, on reload the Create page should have the navigation bar now and look much nicer.

Ok, so now we just need to do something when the ToDo is successfully created. I like bringing back to the ToDo list. Let's do that.

Go to the `create` method. Add `async` before it. This means that the method is `asynchronous` (99% of the time, it means it's making a network request, which this is doing). Let's add that

```
async create(formValues) {
```

When you add the `async`, you can now use something called `await`. Don't get caught up on what this is. Just know you use it on the line you do your network requests. In this case, that's calling our `createToDo` from our `toDoStore`

```
await this.props.toDoStore.createToDo(formValues);
```

Once this is complete, the line after it will run. We want to switch pages, to go back to the home page. How do we switch pages? Remember `this.props.history.push`? Home is `/` (because it's the entry point), so it's going to look like this.

```
 this.props.history.push("/");
```

So, this is what your final `create` method should look like

```
async create(formValues) {
    await this.props.toDoStore.createToDo(formValues);
    this.props.history.push("/");
} 
```

Let the page reload and create a ToDo. It should automatically bring you to the Home page on successful creation :)

# Deploying to Production

## Create an mLab Instance

Right now we're running our database on our computer. This is great for local development, but our users can't access our computer! Luckily, there's a very easy service called mLab that you can access through Heroku. Let's set it up.

1. Login to your Heroku account.
2. Go to billing: https://dashboard.heroku.com/account/billing. Although we're not doing anything that costs money, you're going to need to add a credit card. To repeat, everything we do here will be free but a credit card is required to configure mLab. Add your credit card. Click the Heroku logo in the top left to go back to the dashboard (or go here https://dashboard.heroku.com/apps)
2. Go to your backend To-Do app. It should look something like "Your_Last_Name-todo-backend".
3. Go to the "Resources" tab.
4. There should be a search field under "Add-ons." Search for "mLab MongoDB" and select it
5. A modal will pop up for the sandbox plan. Don't touch anything. Just click "Provision"

That's it. Your database is configured.

## Enter Config Vars

We need to set config variables for production. These exist on the server and should never be shared.

1. While on your backend app, click "Settings"
2. Under Config Vars, click "Reveal Config Vars"
3. You should see a `MONGO_URI` that was automatically created for you. Let's add some more.
4. Add  KEY: `SECRET` VALUE: Pick a random string here https://www.random.org/strings/?num=20&len=20&digits=on&upperalpha=on&loweralpha=on&unique=on&format=html&rnd=new
5. Add KEY: `DOMAIN` VALUE: Open a new tab. Go back to heroku.com but this time select the frontend-todo. Go to Settings and scroll down to Domain. Copy the domain. Mine, for example, is https://barabander-todo-frontend.herokuapp.com

 

## Set Up the Front End

First we have to set our buildpack in Heroku. It is not important what a buildpack is. Just know if you the CLC templates, you'll need it.

1. Go to heroku.com
2. Select the frontend todo app (mine is barabander-todo-frontend)
3. Go to the "Settings" tab
4. Scroll down to "Buildpacks"
5. Click "Add Buildpack"
6. Paste the following where it says "Enter Buildpack URL": https://github.com/mars/create-react-app-buildpack.git
7. Click "Save Changes"

We need to set one Config Var
1. While on your frontend app, click "Settings"
2. Under Config Vars, click "Reveal Config Vars"
3. Add KEY: `NODE_ENV` VALUE: 'production'

## Push to Github

Our app may not be a work of art but let's share it with our friends.

Go to Github Desktop. On the top left, select the current respository as `todo`. You can see all the changes we've made from when this file was last committed (or saved to a specific version). We want all these changes so you can leave everything. If you didn't want specific changes you would uncheck. We'll do this another time. For summary, put, "V1" or something else that  captures the current state of our app. Then press commit. The top right should now give you an option to "push" the branch. Pushing saves to Github and automatically deploys our app to Heroku. Push the app. Do the exact same process for the `todo-backend` by selecting it from the dropdown in the top left and following the same instructions.

Great! All your code is uploaded. Because auto deploy is on, it's automatically deploying to your endpoint. After a couple of minutes you can go to the public url you noted in `agent.js`. As a reminder, mine is https://barabander-todo-frontend.herokuapp.com/ so yours should look very similar.

