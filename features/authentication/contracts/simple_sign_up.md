# Simple sign up

## Sign up [/auth/sign-up]

This resource represents a basic login.

### Sign up [] [POST]

The password should follow the following regex:
```regexp
/^(?=.{8,}$)(?=.*[A-Z])(?=.*[a-z])(?=.*[0-9]).*/
```

* **Body**
  ```json
  {
    "name": "Foo",
    "surname": "Bar",
    "email": "foo@mrmilu.com",
    "password": "Secret123&"
  }
  ```

+ **Response 200** (application/json)
  Back-end sends validation email to user and response to client.
  ```json
  {
    "token": "foobartoken"
  }
  ```

### Sign up validation [/validate] [POST]

User opens link from email with token. Front-end opens link and retrieves
token from it for sign up validation.

* **Body**
  ```json
  {
    "name": "Foo",
    "surname": "Bar",
    "email": "foo@mrmilu.com",
    "password": "Secret123&"
  }
  ```

+ **Response 200** (application/json)
  Back-end sends validation email to user and response to client.
  ```json
  {
    "token": "foobartoken"
  }
  ```