# Social auth

Because we are using firebase as source of truth for our social authentication
flow, all the resources will be focused with this in mind.

## Social sign in & sign up [/auth/social]

This resource represents a basic social authentication flow.

### Social sign in & sign up [] [POST] 🔓

The client should first use firebase sdk to sign in with the corresponding
social authentication provider (web or mobile based). Once logged through
firebase the client app should retrieve the **id_token** and send it to the back-end.
The backend will be able with the firebase sdk, to authenticate the user and
get its data.

It's **important** that the back-end has a system implemented to identify if a
user exists with the given email and if it does associate it to this social
auth instance; and if not create a new one. This will cover a sign in and sign up
process through social auth.

* **Body**
  ```json
  {
    "firebase_token": "id_token"
  }
  ```

+ **Response 200** (application/json)
  Back-end validates user (no email validation needed to Social Auth users) and 
sends response to client. 
  ```json
  {
    "token": "foobartoken"
  }
  ```