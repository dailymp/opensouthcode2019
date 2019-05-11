# Quick Start

## This demo contains some enhacements

1.  Redirect to previous page after login

### To do so you need to rewrite your Auth.js to the following code:

- Add a const

```
 const REDIRECT_ON_LOGIN = "redirect_on_login";
```

- Rewrite your login and handleAuthentication method as:

```
login = () => {
    localStorage.setItem(
      REDIRECT_ON_LOGIN,
      JSON.stringify(this.history.location)
    );
    this.auth0.authorize();
  };

  handleAuthentication = () => {
  this.auth0.parseHash((err, authResult) => {
    if (authResult && authResult.accessToken && authResult.idToken) {
      this.setSession(authResult);
      const redirectLocation =
        localStorage.getItem(REDIRECT_ON_LOGIN) === "undefined"
          ? "/"
          : JSON.parse(localStorage.getItem(REDIRECT_ON_LOGIN));
      this.history.push(redirectLocation);
    } else if (err) {
      this.history.push("/");
      alert(`Error: ${err.error}. Check the console for further details.`);
      console.log(err);
    }
    localStorage.removeItem(REDIRECT_ON_LOGIN);
  });
};
```

2.  Create PrivateRoute component:

- Create a file called PrivateRoute.js

```
import React from "react";
import { Route } from "react-router-dom";
import PropTypes from "prop-types";
import AuthContext from "./AuthContext";

function PrivateRoute({ component: Component, scopes, ...rest }) {
  return (
    <AuthContext.Consumer>
      {auth => (
        <Route
          {...rest}
          render={props => {
            // 1. Redirect to login if not logged in.
            if (!auth.isAuthenticated()) return auth.login();

            // 2. Display message if user lacks required scope(s).
            if (scopes.length > 0 && !auth.userHasScopes(scopes)) {
              return (
                <h1>
                  Unauthorized - You need the following scope(s) to view this
                  page: {scopes.join(",")}.
                </h1>
              );
            }

            // 3. Render component
            return <Component auth={auth} {...props} />;
          }}
        />
      )}
    </AuthContext.Consumer>
  );
}

PrivateRoute.propTypes = {
  component: PropTypes.func.isRequired,
  scopes: PropTypes.array
};

PrivateRoute.defaultProps = {
  scopes: []
};

export default PrivateRoute;

```

- Use your PrivateRoute component on your App.js like this:

```
          <PrivateRoute path="/profile" component={Profile} />
          <PrivateRoute path="/private" component={Private} />
          <PrivateRoute
            path="/courses"
            component={Courses}
            scopes={["read:courses"]}
          />
```

3.  Share auth object via React’s context

- Create a file AuthContext.js

```
import React from "react";
const AuthContext = new React.createContext();
export default AuthContext;

```

- On your App.js file import the context:

```
import AuthContext from "./AuthContext";
```

- On the constructor of the App.js

```
 this.state = {
      auth: new Auth(this.props.history),
    };
```

- On the render method destructure the state:

```
  const { auth } = this.state;
```

- Also update your return inside render like this:

```
return (
      <AuthContext.Provider value={auth}>
        <Nav auth={auth} />
        <div className="body">
          <Route
            path="/"
            exact
            render={props => <Home auth={auth} {...props} />}
          />
          <Route
            path="/callback"
            render={props => <Callback auth={auth} {...props} />}
          />
          <Route path="/public" component={Public} />
          <PrivateRoute path="/profile" component={Profile} />
          <PrivateRoute path="/private" component={Private} />
          <PrivateRoute
            path="/courses"
            component={Courses}
            scopes={["read:courses"]}
          />
        </div>
      </AuthContext.Provider>
    );
```

NOTE: You already have an <AuthContext.Consumer> on your PrivateRoute component needed to comunicate the auth prop from the  
</AuthContext.Provider>

-

4.  Store tokens in memory

- Declare some variable on Auth.js file

```
let _idToken = null;
let _accessToken = null;
let _scopes = null;
let _expiresAt = null;
```

- Your setSession should be like this:

```
 setSession = authResult => {
    console.log(authResult);
    // set the time that the access token will expire
    _expiresAt = authResult.expiresIn * 1000 + new Date().getTime();

    // If there is a value on the `scope` param from the authResult,
    // use it to set scopes in the session for the user. Otherwise
    // use the scopes as requested. If no scopes were requested,
    // set it to nothing
    _scopes = authResult.scope || this.requestedScopes || "";

    _accessToken = authResult.accessToken;
    _idToken = authResult.idToken;
  };
```

- Your isAuthenticated must be like this:

```
isAuthenticated() {
    return new Date().getTime() < _expiresAt;
  }
```

- Your logout is simplified to this:

```
 logout = () => {
    this.auth0.logout({
      clientID: process.env.REACT_APP_AUTH0_CLIENT_ID,
      returnTo: "http://localhost:3000"
    });
  };
```

- Your getAccessToken to this:

```
  getAccessToken = () => {
    if (!_accessToken) {
      throw new Error("No access token found.");
    }
    return _accessToken;
  };
```

-Your userHasScopes to this:

```
  userHasScopes(scopes) {
    const grantedScopes = (_scopes || "").split(" ");
    return scopes.every(scope => grantedScopes.includes(scope));
  }
```

5.  Silent auth and token renewal:
    You must create a new function on Auth.js

```
renewToken(cb) {
    this.auth0.checkSession({}, (err, result) => {
      if (err) {
        console.log(`Error: ${err.error} - ${err.error_description}.`);
      } else {
        this.setSession(result);
      }
      if (cb) cb(err, result);
    });
  }
```

- On your App.js you should add to your state inside constructor one more piece to control tokenRenewalComplete

```
class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
      auth: new Auth(this.props.history),
      tokenRenewalComplete: false
    };
  }

```
- You must add a new lifecycle method:
```
componentDidMount() {
  this.state.auth.renewToken(() =>
    this.setState({ tokenRenewalComplete: true })
  );
}

```
- On your App render method you should add a check if (!this.state.tokenRenewalComplete) return "Loading..."; like this:
```

render() {
const { auth } = this.state;
// Show loading message until the token renewal check is completed.
if (!this.state.tokenRenewalComplete) return "Loading...";
return (

```
- Also to silent token renewal you can add a function on your Auth.js
```

scheduleTokenRenewal() {
  const delay = _expiresAt - Date.now();
    if (delay > 0) setTimeout(() => this.renewToken(), delay);
}

```

- And in your setSession you must add   this.scheduleTokenRenewal(); like this: 

```
setSession = authResult => {
    console.log(authResult);
    // set the time that the access token will expire
    _expiresAt = authResult.expiresIn * 1000 + new Date().getTime();

    // If there is a value on the `scope` param from the authResult,
    // use it to set scopes in the session for the user. Otherwise
    // use the scopes as requested. If no scopes were requested,
    // set it to nothing
    _scopes = authResult.scope || this.requestedScopes || "";

    _accessToken = authResult.accessToken;
    _idToken = authResult.idToken;
    this.scheduleTokenRenewal();
  };
```


- Run the following:

```
npm install
npm start
```


