# Week 3 — Decentralized Authentication

### Task 1: Provision Amazon Cognito User Pool

Follow instructions and created Cognito user pool.

![Screenshot 2023-03-25 at 10 38 03 AM](https://user-images.githubusercontent.com/125124581/227697552-59bd70b8-89fe-4d51-a09c-7bc40b2a1cda.png)

Creating user in Cognito

![Screenshot 2023-03-18 at 12 18 10 PM](https://user-images.githubusercontent.com/125124581/227697618-79ae0301-5f96-4a9c-9f85-44f5237e30d8.png)

Manually verifying user

```sh

aws cognito-idp admin-set-user-password --user-pool-id "yourpoolid" --username learnkhaire --password REDACTED --permanent

```

Adding Optional User Attributes in Cognito

![Screenshot 2023-03-20 at 11 53 58 PM](https://user-images.githubusercontent.com/125124581/227697706-af186c20-6ea8-4a70-910a-c1dfa7864f79.png)

### Task 2: Configuring AWS Amplify

AWS Amplify is a development platform provided by Amazon Web Services (AWS) that allows developers to quickly build and deploy mobile and web applications. It provides a set of tools, libraries, and services that can be used to develop applications for iOS, Android, and the web.

Amplify provides various features that make it easier for developers to build and deploy applications, such as:

1) Authentication: Amplify provides authentication features that can be easily integrated into applications, such as user sign-up, sign-in, and social sign-in.
2) APIs: Amplify provides a set of tools that make it easy to create and integrate APIs into applications.
3) Data storage: Amplify provides data storage options that can be used to store and retrieve data in the cloud.
4) Analytics: Amplify provides analytics tools that can be used to gain insights into user behavior and application performance.
5) Hosting: Amplify provides a hosting platform that can be used to deploy and host web applications.


## Installing AWS Amplify Library

```sh
cd frontend-react-js
npm i aws-amplify --save
```

## Configuring Amplify

```py
import { Amplify } from 'aws-amplify';

Amplify.configure({
  "AWS_PROJECT_REGION": process.env.REACT_AWS_PROJECT_REGION,
  "aws_cognito_identity_pool_id": process.env.REACT_APP_AWS_COGNITO_IDENTITY_POOL_ID,
  "aws_cognito_region": process.env.REACT_APP_AWS_COGNITO_REGION,
  "aws_user_pools_id": process.env.REACT_APP_AWS_USER_POOLS_ID,
  "aws_user_pools_web_client_id": process.env.REACT_APP_CLIENT_ID,
  "oauth": {},
  Auth: {
    // We are not using an Identity Pool
    // identityPoolId: process.env.REACT_APP_IDENTITY_POOL_ID, // REQUIRED - Amazon Cognito Identity Pool ID
    region: process.env.REACT_AWS_PROJECT_REGION,           // REQUIRED - Amazon Cognito Region
    userPoolId: process.env.REACT_APP_AWS_USER_POOLS_ID,         // OPTIONAL - Amazon Cognito User Pool ID
    userPoolWebClientId: process.env.REACT_APP_AWS_USER_POOLS_WEB_CLIENT_ID,   // OPTIONAL - Amazon Cognito Web Client ID (26-char alphanumeric string)
  }
});
```

Adding Variable in docker-compose file.

```dockerfile
      REACT_AWS_PROJECT_REGION: "${AWS_DEFAULT_REGION}"
      REACT_APP_AWS_COGNITO_REGION: "us-east-1"
      REACT_APP_AWS_USER_POOLS_ID: "yourpoolid"
      REACT_APP_CLIENT_ID: "yourclientid"
```

### Task 3 Implement API calls to Amazon Coginto for custom login, signup, recovery and forgot password page

Updating configuration as below.

## Homefeedpage

In HomeFeedPage.js:

```js
//Add this to the import block
import { Auth } from 'aws-amplify';

//replace existing checkAuth function with the following function
const checkAuth = async () => {
  Auth.currentAuthenticatedUser({
    // Optional, By default is false. 
    // If set to true, this call will send a 
    // request to Cognito to get the latest user data
    bypassCache: false 
  })
  .then((user) => {
    console.log('user',user);
    return Auth.currentAuthenticatedUser()
  }).then((cognito_user) => {
      setUser({
        display_name: cognito_user.attributes.name,
        handle: cognito_user.attributes.preferred_username
      })
  })
  .catch((err) => console.log(err));
};
```

The Auth module offered by the AWS Amplify library is used by this method to determine whether the user has been authenticated. If the user is authorised or not, it may be determined using the currentAuthenticatedUser() method. 
The bypassCache argument has a false default value and is optional. When set to true, the function contacts Cognito to request the most recent user information.

## ProfileInfo

In ProfileInfo.js:

```js
//Add this to the import block
import { Auth } from 'aws-amplify';

//replace existing signOut function with the following function
  const signOut = async () => {
    try {
        await Auth.signOut({ global: true });
        window.location.href = "/"
    } catch (error) {
        console.log('error signing out: ', error);
    }
  }
```

## Sign-In Page

in SigninPage.js:

```js
import { Auth } from 'aws-amplify';

const onsubmit = async (event) => {
  setErrors('')
  event.preventDefault();
  Auth.signIn(email, password)
  .then(user => {
    localStorage.setItem("access_token", user.signInUserSession.accessToken.jwtToken)
    window.location.href = "/"
  })
  .catch(error => {
    if (error.code == 'UserNotConfirmedException') {
      window.location.href = "/confirm"
    }
    setErrors(error.message)
  });
  return false
}
```

By executing the setErrors() function, the function first clears any prior errors. Then, it employs the event.preventDefault() method to stop the default form submission behaviour. 

The user is signed in once the code calls the Auth.signIn() method. The function redirects the user to the home page and stores the user's accessToken.jwtToken in the local storage of the browser if the sign-in is successful. 

The function determines whether UserNotConfirmedException is the error code if there is a problem. The function then leads the user to the email confirmation page if such is the case. If not, the setErrors() function is called in order to set the error message in the state variable.


![Screenshot 2023-03-20 at 11 53 47 PM](https://user-images.githubusercontent.com/125124581/227698499-f7488ffc-78bc-4be2-8eae-b76ba37251c7.png)

## Signup Page

```js
const onsubmit = async (event) => {
  event.preventDefault();
  setErrors('');
  console.log('username', username);
  console.log('email', email);
  console.log('name', name);
  try {
    const { user } = await Auth.signUp({
      username: email,
      password: password,
      attributes: {
        name: name,
        email: email,
        preferred_username: username,
      },
      autoSignIn: {
        enabled: true,
      },
    });
    console.log(user);
    window.location.href = `/confirm?email=${email}`;
  } catch (error) {
    console.log(error);
    setErrors(error.message);
  }
  return false;
};
```

The Amazon Amplify library's Auth module is used to register users by collecting their login, password, email address, and other information. 

Using the event.preventDefault() method, the programme first disables the default behaviour of form submission. The setErrors() function is then called to erase any earlier errors. 

The code then logs the login, email address, and name data to the console. 

In order to register the user, the code then invokes the Auth.signUp() method. The user's email, password, name, and username are passed together with an autoSignIn object that enables automatic sign-in once the user has validated their identity. 

The function redirects the user to the email confirmation page with the email query if the sign-up is successful and logs the user object to the console.

## Confirmation Page

ConfirmationPage.js

```js
const resend_code = async (event) => {
  setErrors('');
  try {
    await Auth.resendSignUp(email);
    console.log('Code resent successfully');
    setCodeSent(true);
  } catch (err) {
    console.log(err);
    if (err.message === 'Username cannot be empty') {
      setErrors('You need to provide an email in order to send Resend Activation Code.');
    } else if (err.message === 'Username/client id combination not found.') {
      setErrors('Email is invalid or cannot be found.');
    }
  }
};
```

```js
const onsubmit = async (event) => {
  event.preventDefault();
  setErrors('');
  try {
    await Auth.confirmSignUp(email, code);
    window.location.href = '/';
  } catch (error) {
    setErrors(error.message);
  }
  return false;
};
```

To resend the activation code to the user's email, it makes use of the Auth module offered by the AWS Amplify library. 

The setErrors() function is first used by the function to remove any prior errors. The activation code is then sent again to the user's email by calling the Auth.resendSignUp() method. The function logs a message to the console and changes the codeSent state to true if the code was sent successfully. 

The function checks the error message to determine if it matches any certain instances if there is a problem. If the error message is 'Username cannot be empty', the function sets the error message to indicate that the user needs to provide an email to resend the activation code. 'Username/client id combination not allowed' is the error message

## Recovery Page

recoverypage.js

```js
const onsubmit_send_code = async (event) => {
  event.preventDefault();
  setErrors('');
  Auth.forgotPassword(username)
    .then((data) => setFormState('confirm_code'))
    .catch((err) => setErrors(err.message));
  return false;
};
```

```js
const onsubmit_confirm_code = async (event) => {
  event.preventDefault();
  setErrors('');
  if (password === passwordAgain) {
    Auth.forgotPasswordSubmit(username, code, password)
      .then((data) => setFormState('success'))
      .catch((err) => setErrors(err.message));
  } else {
    setErrors('Passwords do not match');
  }
  return false;
};
```

The event.preventDefault() method is used by the function to first prevent the default form submission action. It then invokes the setErrors() function to erase any earlier errors. 
The code then uses the Auth.forgotPassword() method to commence the lost password flow for the user. The username value is passed as a parameter by the function. 

The function uses the setFormState() function to set the form state to "confirm code" if the forgot password request is successful. 
When an error occurs, the function uses the setErrors() function to save the error message in the state variable.


### Task 4 Verify JWT Token server side to serve authenticated API endpoints in Flask Application

We have added following line in SigninPage.js for configuring authentication for storage.

```js
localStorage.setItem("access_token", user.signInUserSession.accessToken.jwtToken)
``` 

To pass the JWT to backend, in HomeFeedPage.js update the const with Authorization header:

```js
const res = await fetch(backend_url, {
        headers: {
          Authorization: `Bearer ${localStorage.getItem("access_token")}`
        },
```

Adding CORS confing in app.py and header read to the flask backend

```js
cors = CORS(
  app, 
  resources={r"/api/*": {"origins": origins}},
  headers=['Content-Type', 'Authorization'], 
  expose_headers='Authorization',
  methods="OPTIONS,GET,HEAD,POST"
)
```

```js

@app.route("/api/activities/home", methods=['GET'])
def data_home():
  app.logger.info("AUTH HEADER")
  app.logger.info(request.headers.get('Authorization'))
  data = HomeActivities.run(LOGGER)
  return data, 200
  
```

Installing Flask-AWSCognito

```sh
pip install Flask-AWSCognito
pip freeze >> requirements.txt
```




