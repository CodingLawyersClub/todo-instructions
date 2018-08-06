# Display The Create Page

- First Sign Up. It doesn't matter what username or password you use. You'll need to be logged in.
- Create a folder under components named “Create”
- Create a file inside of “Create” folder named “index.js” This file becomes the default when ‘components/Create’ is imported
- First, we import React. This pulls in the react library and is essential for creating our page:

```
React, { Component } from 'react';
```

- Next we define a class named `CreatePage`:
  - `export default` says that when this page is imported, we're going to give whoever is calling it this class
  - `class CreatePage extends Component` says that this page is a type of a React Component. This gives us a ton of stuff for free out of the box, and is how we get the awesomeness of React in our app.
  
```
export default class CreatePage extends Component {

}
```

- Next, we need to add the `render()` method. This is where we define what HTML this page will display. For now, just mark something that let's us know we're on this page. A `<div>` is a generic HTML component to display info.

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

# Link Up the Route

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

# Create a Form With A Submit

- Import a Button, a Form component, a FormInput, Yup, Formik, and Box. You'll see what all these do in practice soon.

```
import { Button } from 'semantic-ui-react'
import Form from '../Form';
import FormInput from '../FormInput'
import * as Yup from 'yup';
import { Formik } from 'formik';
import { Box } from 'rebass';
```

- First, let's set up a validator. A validator is used to make sure the user doesn't submit data they shouldn't. When you've submitted forms and you've gotten a "password isn't long enough" message, a validator is what is checking the logic. We're goign to use a validation library called Yup because it's super simple to use. Paste the following code, and we'll walk through it:

```
const FormSchema = Yup.object().shape({
    text: Yup.string()
        .required('Text for your To-Do is required')
});
```

- `const FormSchema` defines a variable named FormSchema
- `Yup.object().shape()...` defines the validation object. This says what our form is going to be expecting. We're only going to have one field for the text of our To-Do:

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

# Link Up With MobX and Hit the Internet
- Now that we have our form good to go, we need something to happen when you click Create. We use MobX to initiate our network requests and load data. So let's get started creating what we call a MobX store.
- Go to `src/stores/` and create a new file called `toDoStore.js`
- At the top of `toDoStore.js` import the following:

```
import { observable, action, flow } from 'mobx';
import agent from '../agent';
import error from '../utils/error';
```

You'll see these imports in action later.

- Define a class named `ToDoStore`:

```
class ToDoStore {

}
```

- Finally export the store:

```
export default new ToDoStore();
```

- Altogether `toDoStore.js` should look like this:

```
import { observable, action, reaction } from 'mobx';

class ToDoStore {

}

export default new ToDoStore();
```

- Go to `src/index.js`. We're now going to add this store we created to our app.

- First, import it on line 13:

```
import toDoStore from './stores/toDoStore';
```

- Then add it to the list of stores on line 23:

```
const stores = {
  authStore,
  commonStore,
  userStore,
  toDoStore <-- Add this one!
};
```

Great, now this new store is in your app! It doesn't do anything yet though. Let's change that!

- First, we need to define our network request to make. All of our network requests are made in the `agent.js` file. Navigate to it (TIP: don't forget you can always search for a file in VSCode by searching CMD + P).

- Below `Account` add a new variable named `ToDo`. This is where all the network requests around the `ToDo` model will go. This just keeps things organized and much easier to read.

```
const ToDo = {

};
```

- Let's add a method called `create`. This method is going to create a new To-Do in our database by calling an endpoint /toods. We use the `.post` method when we want to create something. We'll be passing in the data, the `toDo` from the form directly. We'll see how that works in a bit.

```
const ToDo = {
  create: (toDo) =>
    requests.post(`/todos`, toDo),
};
```

- Export the ToDo so other files can know about its existence:

```
export default {
  Account,
  Auth,
  ToDo <-- Add it here!
};
```

- Go back to `toDoStore.js`. Let's link everything together.
- Create a new action called `createToDo`. It should look like this:

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

- Inside of `createToDo` let's pass in the toDo. This is the text from our input before. Let's also call that `create` method we put in `agent` on the `ToDo` object. The `yield` says that we're making a network request, and do not move on to running the next line until the result comes back, which we're defining as `response`. We're going to `console.log` the result, which will print it out for us to see in the browser. It will look like this:

```
@action
createToDo = flow(function* (toDo) {
  const response = yield agent.ToDo.create(toDo);
  console.log("OUR CREATED TODO", response.toDo);
});
```

# Link Up Our createToDo() method with our Create page
- Go back to `Create/index.js`. Add the following to get our MobX store into our create page:

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

- First, we need to go to our `agent.js` file to write the method to call our `/todos` endpoint. Let's do that now. On `agent.js` go to `ToDo` and copy and paste what we have under `create`. We're going to modify it slightly to be our `find` function. Check out the arrows for how I did this.

```
create: (toDo) => <-- rename to `find` and get rid of the to `toDo` parameter. The endpoint we built doesn't need any parameters besides the user, which is automatically sent up for us. That means we don't need to pass antyhing.
  requests.post('/todos', toDo), <-- Because we are GETting data data, not POSTing data, we need to make this a `.get`. We also need to get rid of the parameter for the same reason as before.
```

When it's all said and done our `find` method should look like this:

```
find: () =>
  requests.get('/todos')
```

Great! Now we have all we need in our `agent` file. Let's go to our `mobX` store to call it!

- Go to toDoStore.js. Let's add a new method to fetch our To-Dos called `findToDos`. Because it's a simple network request, we're going to copy `createToDo` and adjust it. Check out the arrows for how I did this:

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

Now, we need to call this `findToDos` from somewhere. Let's set that up!

- Go to `Home/index.js`. Above the `render()` method, add a new method named `componentDidMount`.

```
componentDidMount() {

}
```

- `componentDidMount` is a super important React lifecycle method. Basically, whenever you do the `extends Copmonent` the file automatically gets React lifecycle methods like `componentDidMount`. Whenever the page loads, `componentDidMount` is called and the code inside of it is run. This means that it's an excellent place to make network requests because when we navigate to home (localhost:3005), this method will be called. Let's fetch our To-Dos in here!

- Like before, we need to import our `toDoStore` into our page. We do that the same way as the `Create` page.

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
- `Flex` is a layout component. It helps with aligning things on the page. `justifyContent` means which way to align items. `flex-end` means to push everything to the right. So all this is saying is, everything inside of `<Flex />` I want to align right.
- `Box` is what goes inside of `Flex`. Think of `Flex` and `Box` as peanut-butter and jelly. They always go together. `Flex` says how to align the `Box`. Because our `Flex` is `flex-end`, it is our `Box` that is going to be pushed all the way to the right.
- `Button` is pretty self explanatory. This `Button` lives inside of my `Box` which is being pushed all the way to the right. `content` is what the button will say. `icon` is if we want it to have an icon. A list of icons can be found online, but don't worry about that for now. `labelPosition` says where you want the button to go, which I want it on the right of my "New" text.
 - `onClick` is one of the most important methods. It does what you'd think it does. When this button is clicked, whatever is inside of that arrow is going to be called
 - `this.props.history.push` is how we navigate to pages. the `/create` page is where we want to go. Don't worry too much about what's going on here. Just remember, if you want to go to a different page we created, you use `this.props.history.push("/PAGE_ROUTE_HERE")`. We defined our page route in `src/routes/index.js` if you forget the route name.

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


