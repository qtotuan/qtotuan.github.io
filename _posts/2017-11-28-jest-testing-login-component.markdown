---
layout: post
title:  "Jest: How to mock a module that my connected component is using?"
date:   2017-11-28 12:11:03 +0200
categories: jekyll update
---
These days have been marked with desperate hair-pulling, teeth-clenching googling, and mumbled swearing. The reason? Learning Jest.

During my bootcamp time a few months ago, we've been continuously doing test-driven development. But we've never actually written any tests. They've just magically always been provided thanks to smart instructor-fairies. So now I have a job. And testing will be a huge part of my daily developer life. It was time that I get my hands dirty and learn how to write my own tests. Getting closer to feeling like a developer - fo' reelz!

A few things I already know about my future tasks:
1. I will develop with React - yay!
2. I will need to write tests with Jest/Enzyme - whaaat!

After days of set up, writing tests for dumb React components, and using Jest's awesome snapshot feature, and questions like:
"Why won't it set up the Enzyme adapter before each test?"
"What's mocking?"
"How do I test redux reducers and actions?"
I came across a component that was more challenging to test: my login component.

The main problem:
The component uses a module with a fetch to the API. How do I replace this method with a dummy function?

Consider my app.

The login component renders a form and on submit, sends the login params to the backend to verify it, and sets the user in the redux store:

``` javascript
// Login.js

import React from 'react'
import { connect } from 'react-redux'
import { bindActionCreators } from 'redux'
import { setUser } from '../../actions/setUser'
import AuthAdapter from '../../authAdapter/authAdapter'

export class Login extends React.Component {

  constructor(props) {
    super()

    this.state  = {
      email: '',
      password: ''
    }
  }

  handleChange(event) {
    let key = event.target.name
    let value = event.target.value
    this.setState({ [key]: value })
  }

  handleSubmit(event) {
    event.preventDefault()
    this.handleLogin(this.state)
  }

  handleLogin(loginParams) {
    AuthAdapter.login(loginParams)
      .then(res => {
          this.props.setUser(res)
        }
      })
  }

  render() {
    return(
      <div>
        <form onSubmit={this.handleSubmit.bind(this)}>
          <label>Email</label>
          <input type="email" name='email' onChange={this.handleChange.bind(this)}/><br/>
          <label>Password</label>
          <input type="password" name='password' onChange={this.handleChange.bind(this)} />
          <button type="submit">LOG IN</button>
        </form>  
      </div>
    )
  }
}

mapDispatchToProps(dispatch) {
  return {
    setUser: bindActionCreators(setUser, dispatch)
  }
}

export default connect(null, mapDispatchToProps)(Login)

```

<br />

The AuthAdapter module:

``` javascript
// AuthAdapter.js

export default class AuthAdapter {
  static login (loginParams) {

    return fetch("http://localhost:3000/api/v1/authentication", {
      method: 'POST',
      headers: {
        'content-type': 'application/json',
        'accept': 'application/json'
      },
      body: JSON.stringify(loginParams)
    }).then(res => res.json())
  }
}

```
<br />
The test file:

``` javascript
//Login.test.js

import React from 'react'
import { shallow } from 'enzyme'
import { Login } from '../../src/components/user/login'

describe('Login component', () => {
  let wrapper
  const mockSetUserfn = jest.fn()

  beforeEach(() => {
    jest.clearAllMocks()
  })

  // ... tests ...

  it('should call mock setUser function on submit', () => {
    const wrapper = shallow(<Login setUser={mockSetUserfn} />)
    const form = wrapper.find("form")
    form.simulate('submit', { preventDefault() {} })
    expect(mockSetUserfn).toHaveBeenCalledTimes(1)
  })
})


```
The test verifies if the redux function setUser is being called on form submit. Note that the test is using the unconnected login component. It replaces the redux setUser function with jest.fn() and passes the mock function as a prop. The redux actions and reducers are all being tested in their own test suites, so all the login tests need to verify is that mockSetUserfn is being called.

<br />

Running the test I get this:

```
ReferenceError: fetch is not defined
```

Aha - 'fetch' is used in my AuthAdapter module, that is imported in the login component. This error makes me realize two things:
1. Jest is running the tests in node, which does not know about 'fetch'. I'd need to use something like isomorphic-fetch to make fetch available in node as a global.
2. But, do I need to test AuthAdapter.login? In general tests should be kept as contained as possible, so I feel that AuthAdapter deserves its own test. If I returned dummy data I can still test that the login behavior works, and the tests will be faster not having to wait for a response from the API.

So it looks like I need to mock the AuthAdapter, so the tests do not actually make a call to the API. I want the test to work with some replacement function. So my question was: how do I replace a module method called by the component? The answer: manually mock the module.

This is the file I want the tests to use instead:

``` javascript
// ../__mocks__/authAdapter

export default class AuthAdapter {
  static login() {
    return Promise.resolve({ name: "John Doe", email: "john@doe.com" })
  }
}

```

The [official Jest documentation](https://facebook.github.io/jest/docs/en/manual-mocks.html) supplies an example, which uses 'jest.genMockFromModule()'. However in my case this does not work, because I do not want to extend the functionality of an existing module, but override the existing function 'AuthAdapter.login'.

The goal is to import the mocked AuthAdapter instead of the original one. This is achieved by mocking the import in the test. On top of the test file, I add this:

``` javascript
// Login.test.js

jest.mock('../../src/authAdapter/authAdapter')

// ...tests...
```

jest.mock actually knows to look in the adjacent folder called '\__mocks\__' and use the mock file instead of the real one.

Great! The tests are now using the mock function instead of the fetch.

But somehow my tests do not pass:

```
Expected mock function to have been called one time, but it was called zero times.
```

After some playing around I realize that the problem is this: the handleLogin function of my component is async.

``` javascript
// Login.js

// ...

handleLogin(loginParams) {
  AuthAdapter.login(loginParams)
    .then(res => {
        this.props.setUser(res)
      }
    })
}

// ...
```

The 'expect' in test test is called before the callback from '.then' in the component. The Jest test pushes these functions on the call stack, in this order: form.simulate, expect. The crux is that 'setUser' is still in the event queue, when expect is already being processed. So 'expect' is correctly registering zero function calls, and in the NEXT tick the function is actually triggered. Too late...

In order to solve this I need to promisify form.simluate so that I can chain the expect to it in a '.then':

``` javascript
// Login.test.js

// ...

it('should call mock setUser function on submit', () => {
  const wrapper = shallow(<Login setUser={mockSetUserfn} />)
  const form = wrapper.find("form")
  Promise.resolve(form.simulate('submit', { preventDefault() {} }))
    .then(() => {
      expect(mockSetUserfn).toHaveBeenCalledTimes(1)
    })
})

// ...
```

<br />
Huzzah!

Testing is fun. After all, if there is one thing I've learned, it is: "Green lights are good."

<br />
<br />
Sources:
* [Jest documentation: Manual mocks](https://facebook.github.io/jest/docs/en/manual-mocks.html)

* [Unit Testing Redux Connected Components](https://hackernoon.com/unit-testing-redux-connected-components-692fa3c4441c)

* [Jest — Mock a function called inside a React Component](https://stackoverflow.com/questions/43500235/jest-mock-a-function-called-inside-a-react-component)

* [Philip Roberts: What the heck is the event loop anyway?](https://www.youtube.com/watch?v=8aGhZQkoFbQ)
