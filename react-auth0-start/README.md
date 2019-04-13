# Quick Start

1. Configure your React app in the Auth0 Dashboard:

 a) Create an Auth0 account
 
 b) In your dashboard, click on the Applications section on the vertical menu and then click on Create Application.
 + If you want to know more visit: https://auth0.com/blog/react-tutorial-building-and-securing-your-first-app/#Securing-your-React-App

2. Create a .env file in the project root (same directory as package.json) that contains the following:

```
REACT_APP_AUTH0_DOMAIN=YOUR AUTH0 DOMAIN HERE
REACT_APP_AUTH0_CLIENT_ID=YOUR AUTH0 CLIENT ID HERE
REACT_APP_AUTH0_CALLBACK_URL=http://localhost:3000/callback
```

3. Run the following:

```
npm install
npm start
```
