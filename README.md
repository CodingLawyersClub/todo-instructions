# Display The Create Page
Create ToDo
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
- Import the create page:
```
import CreatePage from '../components/Create';
```
Because of the `index.js`rule mentioned above, the page we just created will be pulled. Now we need to define when this page is displayed. This is done through `Routes`. *Above `<Route component={Home} />` put the following:
```
<Route path="/create" component={CreatePage} />
```
This says when we go to our website, `localhost:3000/create` /create, we're going to automatically go to our Create page. You should see the create page loaded
![alt text](https://s3.amazonaws.com/clc-images/CreatePage.png)

