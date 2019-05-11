This demo show you how to handle the scopes:

1. Add to your server.js the follow code

```
app.get("/course", checkJwt, checkScope(["read:courses"]), function(req, res) {
 res.json({
 courses: [
 { id: 1, title: "Building Apps with React and Redux" },
 { id: 2, title: "Creating Reusable React Components" }
 ]
 });
 });

function checkRole(role) {
return function(req, res, next) {
const assignedRoles = req.user["http://localhost:3000/roles"];
if (Array.isArray(assignedRoles) && assignedRoles.includes(role)) {
return next();
} else {
return res.status(401).send("Insufficient role");
}
};
}

app.get("/admin", checkJwt, checkRole("admin"), function(req, res) {
res.json({
message: "Hello from an admin API!"
});
});
```

2. Rewrite your Auth.js file:
   First your constructor should look like this:

```
...
 constructor(history) {
    this.history = history;
    this.userProfile = null;
    this.requestedScopes = "openid profile email read:courses";
    this.auth0 = new auth0.WebAuth({
      domain: process.env.REACT_APP_AUTH0_DOMAIN,
      clientID: process.env.REACT_APP_AUTH0_CLIENT_ID,
      redirectUri: process.env.REACT_APP_AUTH0_CALLBACK_URL,
      audience: process.env.REACT_APP_AUTH0_AUDIENCE,
      responseType: "token id_token",
      scope: this.requestedScopes
    });
  }
...
```

On your setSession method add the scopes handlers as follow>

```
 setSession = authResult => {
    console.log(authResult);
    // set the time that the access token will expire
    const expiresAt = JSON.stringify(
      authResult.expiresIn * 1000 + new Date().getTime()
    );

    // If there is a value on the `scope` param from the authResult,
    // use it to set scopes in the session for the user. Otherwise
    // use the scopes as requested. If no scopes were requested,
    // set it to nothing
    const scopes = authResult.scope || this.requestedScopes || "";

    localStorage.setItem("access_token", authResult.accessToken);
    localStorage.setItem("id_token", authResult.idToken);
    localStorage.setItem("expires_at", expiresAt);
    localStorage.setItem("scopes", JSON.stringify(scopes));
  };

```

On your logout method remove the scopes like this:

```
 logout = () => {
    localStorage.removeItem("access_token");
    localStorage.removeItem("id_token");
    localStorage.removeItem("expires_at");
    localStorage.removeItem("scopes");
    this.userProfile = null;
    this.auth0.logout({
      clientID: process.env.REACT_APP_AUTH0_CLIENT_ID,
      returnTo: "http://localhost:3000"
    });
  };
```

Last but not least add this method:

```
  userHasScopes(scopes) {
    const grantedScopes = (
      JSON.parse(localStorage.getItem("scopes")) || ""
    ).split(" ");
    return scopes.every(scope => grantedScopes.includes(scope));
  }
}
```

3. Add on your App.js file the following route:

```
        <Route
            path="/courses"
            render={props =>
              this.auth.isAuthenticated() &&
              this.auth.userHasScopes(["read:courses"]) ? (
                <Courses auth={this.auth} {...props} />
              ) : (
                this.auth.login()
              )
            }
        />
```
4. Add a file with a Courses component Called Courses.js:
```
import React, { Component } from "react";

class Courses extends Component {
  state = {
    courses: []
  };

  componentDidMount() {
    fetch("/course", {
      headers: { Authorization: `Bearer ${this.props.auth.getAccessToken()}` }
    })
      .then(response => {
        if (response.ok) return response.json();
        throw new Error("Network response was not ok.");
      })
      .then(response => this.setState({ courses: response.courses }))
      .catch(error => this.setState({ message: error.message }));

    fetch("/admin", {
      headers: { Authorization: `Bearer ${this.props.auth.getAccessToken()}` }
    })
      .then(response => {
        if (response.ok) return response.json();
        throw new Error("Network response was not ok.");
      })
      .then(response => console.log(response))
      .catch(error => this.setState({ message: error.message }));
  }

  render() {
    return (
      <ul>
        {this.state.courses.map(course => {
          return <li key={course.id}>{course.title}</li>;
        })}
      </ul>
    );
  }
}

export default Courses;

```
Rewrite your Nav.js to add userHasScopes prop, and to handle the Link to Courses, your code should look like this:

```
const { isAuthenticated, login, logout, userHasScopes } = this.props.auth;
    return (
      <nav>
        <ul>
          <li>
            <Link to="/">Home</Link>
          </li>
          <li>
            <Link to="/profile">Profile</Link>
          </li>
          <li>
            <Link to="/public">Public</Link>
          </li>
          {isAuthenticated() && (
            <li>
              <Link to="/private">Private</Link>
            </li>
          )}
          {isAuthenticated() &&
            userHasScopes(["read:courses"]) && (
              <li>
                <Link to="/courses">Courses</Link>
              </li>
            )}
          <li>
            <button onClick={isAuthenticated() ? logout : login}>
              {isAuthenticated() ? "Log Out" : "Log In"}
            </button>
          </li>
        </ul>
      </nav>
    );
```
