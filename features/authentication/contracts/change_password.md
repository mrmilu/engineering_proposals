# Change password

## Sign up [/auth/password]

This resource represents a basic recovering password process.

### Request password change [/request] [POST] 🔓

Client requests password change identified by email.

* **Body**
  ```json
  {
    "email": "foo@mrmilu.com"
  }
  ```

+ **Response 204** (application/json)
  Back-end sends recover password email to user. Front-end opens link and retrieves
  token from it to be used in change password resource.

### Change password [/change] [POST] 🔓

Resource to update users password to new one.

* **Body**
  ```json
  {
    "token": "changepasswordtoken",
    "password": "Secret123&",
    "repeatPassword": "Secret123&",
  }
  ```

+ **Response 204**