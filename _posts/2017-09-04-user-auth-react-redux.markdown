---
layout: post
title:  "User Authentication in React With Redux"
date:   2017-09-04 12:11:03 +0200
categories: react
---

User authentication. At Flatiron School they tell us that it is one of the more complicated things and that we are not required to implement it during our time at school. Unfortunately I let this discourage me from even trying it. So this is why I decided to take the dive into cold waters and add user authentication to my react project, using redux.

The guide contains these parts:
  * Server side
  * Client side
  * Summary of the flow

If it is more helpful to start with an overview rather than concrete code, the summary at the bottom of the blog explains the flow between frontend and backend on a higher level.

<br>

### SERVER SIDE


CONFIGURATION
  * Add "bcrypt" to your gemfile
  * Add "jwt" to you gemfile
  * Remember to run bundle install

----------------------------
<br>

SET UP THE MODEL
  * Add "has_secure_password"
  * Add authenticate class method that takes in email (or username) and password



``` ruby

class User < ApplicationRecord
  has_many :habits
  has_many :categories, through: :habits
  has_secure_password

  def self.authenticate(email, password)
    user = User.find_by(email: email)
    user && user.authenticate(password)
  end
end

```
----------------------------
<br>


CREATE THE MIGRATION

Add the column for password_digest
​
``` ruby
class CreateUsers < ActiveRecord::Migration[5.1]
  def change
    create_table :users do |t|
      t.string :first_name
      t.string :last_name
      t.string :email
      t.string :password_digest

      t.timestamps
    end
  end
end

```
----------------------------
<br>

TEST

Test if it works in the Rails console.
 * Create a user with password. 
 * User.authenticate([email], [password])
 * It should return the User record.

 ----------------------------
 <br>


ADD THE ROUTES

``` ruby
Rails.application.routes.draw do
  namespace :api do
    namespace :v1 do
      post '/login', to: 'auth#create'
      post '/signup', to: 'users#create'
      get '/me', to: 'auth#show'

      resources :users
    end
  end

  get '/', to: 'application#index'
end
```
----------------------------
<br>
 
ADD APPLICATION CONTROLLER HELPER METHODS

``` ruby
class ApplicationController < ActionController::API

  private

  # JWT method encode takes in three parameters and returns a hashed string
  def issue_token payload
    JWT.encode(payload, secret, algorithm)
  end

  # This method will be called before the auth#show method to intercept that #show renders
  # in case the user is not found in the database
  def authorize_user!
    if !current_user.present?
      render json: {error: 'No user id present'}
    end
  end

  # The method is called by auth#show which renders the json object with the user info
  def current_user
    @current_user ||= User.find_by(id: token_user_id)
  end

  # Helper method for current_user
  def token_user_id
    decoded_token.first['id']
  end

  # Helper method for token_user_id
  def decoded_token
    if token
      begin
        JWT.decode(token, secret, true, {algorithm: algorithm})
      rescue JWT::DecodeError
        return [{}]
      end
    else
      [{}]
    end
  end

  # The jwt token from the POST request that the user is sending with each fetch
  # Helper method for decoded_token
  def token
    request.headers['Authorization']
  end

  # Provide a string that will serve as the secret signing key which is used for hashing the header and payload
  def secret
    "learnlovecode"
  end

  # Algo for jwt encoding
  def algorithm
    "HS256"
  end

end
```
----------------------------
<br>

ADD AUTH CONTROLLER

``` ruby
class Api::V1::AuthController < ApplicationController
  before_action :authorize_user!, only: [:show]

  def show
    render json: {
      id: current_user.id,
      email: current_user.email
    }
  end


  def create
    # see if there is a user with this username
    user = User.find_by(email: params[:email])
    # if the is, make sure that they have the correct password
    if user.present? && user.authenticate(params[:password])
      # if they do, render back a json response of the user info
      # issue token
      created_jwt = issue_token({id: user.id})
      render json: {email: user.email,jwt: created_jwt}
    else
      # otherwise, render back some error response
      render json: {
        error: 'Email or password incorrect'
      }, status: 404
    end
  end

end
```

Use Postman to test if it works
  * Send POST request to your API endpoint
  * Add user params in the request's body
  * Should return json specified in AuthController
​​
<br><br>
![Test With Postman]({{ site.url }}/assets/170904_postman.png)



<br><br><br>

### CLIENT SIDE

ADD AUTH ADAPTER

  * Add login method that takes in login parameters (email and password) to send the request to the database
  * Add current user method that finds the current user via the jwt token

``` javascript
const baseUrl = 'http://localhost:3000/api/v1'

export default class AuthAdapter {
  static login (loginParams) {
    return fetch(`${baseUrl}/login`, {
      method: 'POST',
      headers: headers(),
      body: JSON.stringify(loginParams)
    }).then(res => res.json())
  }

  static currentUser () {
    return fetch(`${baseUrl}/me`, {
      headers: headers()
    })
    .then(res => {
      return res.json()
    })
  }
}

function headers () {
  return {
    'content-type': 'application/json',
    'accept': 'application/json',
    'Authorization': localStorage.getItem('jwt')
  }
}
```
----------------------------
<br>

ADD LOGIN LOGIC

The login method is added to the Login component, which will be triggered once the user submits the login form.

``` javascript
onLogin = (loginParams) => {
  AuthAdapter.login(loginParams)
    .then( res => {
      if( res.error ){
        this.setState({ error: true })
      } else {
        localStorage.setItem('jwt', res.jwt)
        this.props.setUser(res.user)
        this.setState({ redirect: true })
      }
    })
  }
```
----------------------------
<br>
​
SET THE REDUCER AND ACTION

Once the user is found the redux store is updated with the user object (which is being assembled by auth#create). The reducer looks like this:

``` javascript
export default function currentUser (state = {}, action) {
  console.log("Action type is:", action.type);
  console.log("State is:", state);
  switch(action.type) {
    case 'SET_USER': {
      return action.payload
    }
    case 'CLEAR_USER': {
      return action.payload
    }
    default: {
      return state
    }
  }
}
```
<br>

The corresponding action is pretty straightforward:
``` javascript
import fetch from 'isomorphic-fetch';

export default function setUser(user) {
  return {type: "SET_USER", payload: user}
}
```
<br>

Don't forget to connect the dispatch method with the component so that it can update the redux store.

``` javascript
import React, { Component } from 'react'
import { connect } from 'react-redux'
import { bindActionCreators } from 'redux'
import SetUser from '../../actions/setUser'

class LoginForm extends Component {
  //stuff
}

function mapDispatchToProps(dispatch) {
  return bindActionCreators({
    setUser: SetUser
  }, dispatch)
}

export default connect(null, mapDispatchToProps)(LoginForm)
```
----------------------------
<br>

ADD THE LOGIN LOGIC TO ROUTES

The routes uses a simple ternary to authenticate users with isLoggedIn. App.isLoggedIn checks if the user has been set in the redux store:

``` javascript
isLoggedIn = () => {
    return !!this.props.currentUser.id
  }
```

It is used as follows in the Route:

``` javascript
  <Route exact path='/' render={()=> this.isLoggedIn() ? <Redirect to="/habits"/> : <Login /> } />
  ```
​

### SUMMARY

THE LOGIN PROCESS - INITIAL LOGIN

This process is triggered once when the user types in his email and password and submits the form.

Frontend:
  1. User submit from the Login component invokes AuthAdapter.login with the parameters email and password
  2. AuthAdapter.login assembles the header and sends the request to the API endpoint 'login'

Backend:
  1. The API endpoint hits the method auth#create
  2. auth#create does multiple things:
looks for the user with the given arguments (email, password)
creates the jwt token (with the helper method application#issue_token
returns the user object and the jwt token

Frontend:
  1. onLogin receives the user object
  2. it sets the localStorage with the jwt
  3. it sets the redux store with the user object
  4. the routes check if the user object is present in the redux store; if no user has been set, the user is redirected to the login/main page



The request to retrieve the current user will hit the endpoint 'me' and the controller method auth#show (see above for ApplicationController).

----------------------------
<br>


THE CURRENT USER PROCESS - AFTER INITIAL LOGIN

This process is triggered every time a page is being refreshed:

``` javascript
componentWillMount() {
  if (localStorage.getItem('jwt')) {
   Auth.currentUser()
     .then(user => {
       if (!user.error) {
          this.props.setUser(user.user)
        }
      }) // end then
  } // end if
}
```

The user has already been authenticated on intial login and the jwt token is stored in the browser, so there is no need to create a new one. Instead of asking the user to re-submit his login credentials the jwt from localStorage will be sent to the backend. There it is being decoded and the user object is being returned so that the redux store can be refreshed. It is very similar to the login process, except that we are hitting auth#show instead of auth#create

Frontend:
componentWillMount -> AuthAdapter.currentUser sends the jwt token to the API


Backend:
1. The API endpoint hits the method auth#show
2. auth#show does multiple things:
  * the helper method Application#authorize! intercepts auth#show if no user is being found
  * auth#show calls Application #current_user
  * Application#currentUser calls various helper methods:
  * #token grabs the token from the HTTP request header
  * #decoded_token uses the JWT method to turn the hashed string into the original object
  * #token_user_id returns the user id
  * #current_user uses the user id to query the database and returns the user object
  * finally, auth#show renders the json object with the user info

Frontend:
componentWillMount -> AuthAdapter.currentUser uses the object to set the user in the redux store
  * sets the localStorage with the jwt
  * sets the redux store with the user object
  * the routes check if the user object is present in the redux store; if no user has been set, the user is redirected to the login/main page
