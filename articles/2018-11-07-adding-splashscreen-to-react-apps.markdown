---
layout: post
title: "Adding splash screens to React single page application"
description: "Learn how to add splash screen to your react application."
date: 
category: React
banner:
  text: "Splash screens make your app look more appealing."
author:
  name: "Abdulazeez Abdulazeez Adeshina"
  url: "https://twitter.com/kvng_zeez"
  mail: "laisibizness@gmail.com"
  avatar: "https://cdn.auth0.com/blog/guest-authors/abdulazeez.png"
design:
  image: 
  bg_color: "#012C6C"
tags:
- javascript
- react
- node
- firebase
- express
related:


---

**TL;DR:** In this article, you will learn how to develop a splash screen for Sing-Page Apps (SPA). You can find the finalized code in [this GitHub repo]()

---

## What Is A Splash Screen 

A splash screen is a page displayed to notify a user when an application is currently in a loading state. In most cases, splash screens are employed when data is fetched from within or outside the application e.g database . The splash screen, serves as a replacement for blank pages which is usually shown when loading data externally.

### Why Your App Should Have A Splash Screen

In your application, a splash screen should be added to indicate when your app is loading and to tell the user that the app isn't dormant. In a situation whereby your loading app shows a blank page for about a minute or two, the user will get bored and loose interest thinking it is a dummy app whereas the splash screen does otherwise. Therefore, adding a splash screen to your application keeps the user engaged while your app loads fully.

### Brief Introduction to React's Higher-Order Component

A Higher-Order Component ( HOC ), is a function that takes in a component as an argument and returns a new component. A higher-order component is usually used to share common functionality between components such as checking the authentication status in a web app.

An example of a higher-order component is the splash screen component which you'll be using later.

### What you'll be building

The application which you'll be adding a splash screen, is a Single-Page Application (SPA) that uses an Auth0 protected serverless backend server which displays a list of Node.js frameworks to an authenticated user. 

### Cloning The Sample App repo

To begin, you'll have to clone [this repo](https://github.com/Youngestdev/react-splashscreen) :

```bash
git clone https://github.com/Youngestdev/react-splashscreen
```

Next, navigate into the folder directory :

```bash
cd react-splashscreen
```

After that, you install the application dependencies :

```bash
npm install
```


After the installation, start the application by running the command :

```bash
npm run start
```

A browser window will be automatically opened displaying the live app at `http://localhost:3000`.

## Building the Splash Screen Component


With that covered, open the project directory in your code editor of choice and create a new file `withSplashScreen.js` in the `src/components` folder.

Next, open the `withSplashScreen.js` file and add the following into it :

```javascript
import axios from 'axios';
import React, {Component} from 'react';
import { CircleSpinner } from 'react-spinners-kit';
import auth0Client from '../Auth';
import firebaseClient from '../services/firebase';


const fbTokenFactory = 'https://wt-21d474de14a60e389a0fa7c8e77c224f-0.sandbox.auth0-extend.com/webtasks/firebase';

function LoadingMessage() {
  return (
    <div className="row justify-content-center">
      <CircleSpinner color="#686769" />
    </div>
  );
}

function withSplashScreen(WrappedComponent) {
  return class extends Component {
    constructor(props) {
      super(props);
      this.state = {
        loading: true,
      };
    }

    async componentDidMount() {
      try {
        await auth0Client.loadSession();
        const fbCustomToken = await axios.get(fbTokenFactory, {
          headers: { authorization: `Bearer ${auth0Client.getIdToken()}` }
        });
        firebaseClient.setToken(fbCustomToken.data.firebaseToken)
      } catch (err) {
        console.log(err);
      }
      this.setState({
        loading: false,
      });
    }

    render() {
      // while checking user session, show "loading" message
      if (this.state.loading) return LoadingMessage();

      // otherwise, show the desired route
      return <WrappedComponent {...this.props} />;
    }
  };
}

export default withSplashScreen;

```

In the code above, you define a variable `fbTokenFactory` and a function `LoadingMessage()` before the `withSplashScreen` component. 

The `fbTokenFactory` variable, holds the url to the backend server deployed on a [serverless sandbox](https://webtask.io) which is later called in the axios handler :

```javascript

axios.get(fbTokenFactory, {
  headers: { authorization: `Bearer ${auth0Client.getIdToken()}` }
});
        
```

The `loadingMessage()` on the other hand, displays a circular spinner when invoked. It is invoked in the `withSplashScreen` render method.

The `withSplashScreen` component, which is a higher-order component, returns a new component passed in as an argument prefixed with the `componentDidMount` lifecycle method that deals with session checking to detect the authentication state of a user and handles the firebase realtime database custom token generation. The authentication management is handled with this line of code :

```javascript
await auth0Client.loadSession();
```

## Wrapping The Application With The Splash Screen Component

With the splash screen component built, it's not yet functional. Therefore, you'll wrap the `App` component in `App.js` in the `withSplashScreen` function since it's a higher-order component.

Open `App.js` and replace the code with :

```javascript
import React, {Component, Fragment} from 'react';
import {Route} from 'react-router-dom'
import NavBar from './components/NavBar';
import Welcome from './components/Welcome';
import withSplashScreen from './components/withSplashScreen';

class App extends Component {
  render() {
    return (
      <Fragment>
        <NavBar />
        <div className="container-fluid">
          <Route path="/" exact component={Welcome} />
        </div>
      </Fragment>
    );
  }
}

export default withSplashScreen(App);

```

In the code above, you imported the `withSplashScreen` component from `./components/withSplashScreen` and then deleted the callback route because you won't be needing it anymore as the `App` component will handle the redirection itself due as a result of being wrapped in the `withSplashScreen` component.

> The callback route, which houses the `Callback` component was responsible for handling the user authentication process and redirecting users after login. You can read more about it [here](https://auth0.com/docs/users/redirecting-users).

Next, open the `Frameworks.js` file in `src/components` and replace the code with this :

```javascript
import React, {Component} from 'react';
import firebaseClient from '../services/firebase';
import {CircleSpinner} from 'react-spinners-kit';

function Framework(framework, id) {
  return (
    <div className="mb-2 row" key={id}>
      <div className="card">
        <div className="card-body">
          <h5 className="card-title">{framework.name}</h5>
          <hr className="my-4" />
          <p className="card-text">{framework.about}</p>
        </div>            
      </div>
    </div>
  )
}

class Frameworks extends Component {
  constructor(props) {
    super(props);

    this.state = {
      frameworks: [],
      loading: true
    };
  }

  componentDidMount() {
    firebaseClient.setFrameworksListener(querySnapshot => {
      const frameworks = [];
      querySnapshot.forEach(framework => {
        frameworks.push(framework.data());
      });
      this.setState({
        frameworks,
        loading: false
      });
    });
  }

  render() {
    const {loading, frameworks} = this.state;
    return (
      <div>
        {loading && <CircleSpinner color="" />}
        {!loading && frameworks.map((framework, id) => (Framework(framework, id)))}
      </div>
    )
  }
}

export default Frameworks;
```

In the code above, you updated the `Frameworks` component by adding a circle spinner component `CircleSpinner` imported from `react-spinners-kit` to indicate when the app is loading by showing a spinning icon instead of having a blank dummy page.

## Refactoring The Authentication Handler

The authentication logic is managed and controlled in the `Auth.js` file, it stores the credentials needed to connect, login and out of Auth0. With the introduction of the higher-order component `withSplashScreen`, the need for a callback route cease to exist. Therefore, open `Auth.js` file and edit the callback route on the eight line from `'http://localhost:3000/callback'` to `'http://localhost:3000'.`


## Running Application

With all that covered, you have successfully added a splash screen to the app. Refresh the App window in your browser and feel the splash screen effect.

## Conclusion

In this article, you learned, through a practical guide, how to add a splash screen to a Single-Page App (SPA).

Implementing splash screens is relatively easy. Therefore, try to implement splash screens in your web apps to improve user experience. Happy hacking!
