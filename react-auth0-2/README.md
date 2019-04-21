
# Quick Start

1. This demo contains log in button and login logic implementation.
```
import React, { Component } from "react";
import { Link } from "react-router-dom";

class Home extends Component {
  render() {
    const { isAuthenticated, login } = this.props.auth;
    return (
      <div>
        <h1>Home</h1>
        {isAuthenticated() ? (
          <Link to="/profile">View profile</Link>
        ) : (
          <button onClick={login}>Log In</button>
        )}
      </div>
    );
  }
}

export default Home;
```


2. Verify in your Auth0 dashboard if everything is properly set. 

3. Run the following:

```
npm install
npm start
```
4. Do the login and return to your dashboard and you can see the user logged information. 
