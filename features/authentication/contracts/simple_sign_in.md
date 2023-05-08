# Simple sign in

## Sign in [/auth/sign-in]

This resource represents a basic sign in.

### Sign in [] [POST] 🔓

* **Body**
  ```json
  {
    "email": "foo@mrmilu.com",
    "password": "Secret123&"
  }
  ```

+ **Response 200** (application/json)
  Back-end responds with authentication token.
  ```json
  {
    "token": "foobartoken"
  }
  ```