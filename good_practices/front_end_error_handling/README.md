# Good practices: Front end error handling

## Types of errors
First of all we must identify the types of errors that can occur in a front end application.
We can divide them into two groups:
 - **External errors:** These are errors that occur outside the application, such as a server error or a network error.
 - **Internal errors:** These are errors that occur within the application, such as a validation error or a logic error.
Errors produced inside the application as side effect of an external error are considered external errors. For example:
a handled server error like a unsuccessful login attempt.

In general internal errors are easier to handle because the frameworks we use give us
tools to capture them and show them to the user in a friendly way.

In React for example we can use the `ErrorBoundary` component to capture errors that occur in components
and display whatever we want to the user.

The main problem it's the lack of a structured error object that allows us to know what happened and where it happened,
so we can give the user as much information as possible.

This also applies to external errors, but in this case we have to deal with the fact that we don't have control over
every error that can occur, so we have to be prepared to handle them all.

For this the main task will be to define a structured error object that allows us to work with both scenarios.

## The error object

The ideal error object should have the following structure[^1]:

[^1]: Typescript will just as a prototyping language in this proposal.

```typescript
type ErrorType = 'internal' | 'external';

interface AppError {
  code: ErrorCodes;
  message: string;
  type: ErrorType;
  data?: any;
  stack: string;
}
```

There are three key properties in this object:

 - **code:** We should define an internal error code convention and avoid magic strings at all costs.
 - **message:** It should be a string that describes the error, but **it should not be used as message to display to the user**
 - **stack:** It should be a string that contains the stack trace of the error.

The `data` property is optional and it's used to pass additional information about the error
that it could help different use cases for handling it.

### Error codes

The error codes should be defined in a single place, so we can have a single source of truth for them.

```typescript
// These are a bunch of codes that we should make as standard as possible.
// Each application then will have its own codes.

enum ErrorCodes {
  UNSUCCESSFUL_LOGIN = 'UNSUCCESSFUL_LOGIN',
  UNAUTHORIZED = 'UNAUTHORIZED',
  SIGN_UP_ERROR = 'SIGN_UP_ERROR'
}
```

## Error handling

### Internal errors

We will start with the internal errors, since they are the easiest to handle.
We should capture errors consciously as we develop our application and use the
structured error object to pass the information along the application.

```typescript
class SomeDomainObject {
  name: string
  items?: Array<{ value: string, label: string }>

  constructor (params: SomeDomainObject) {
    this.name = params.name
    this.items = params.items
  }
  
  changeItemValue(prev: string, next: string) {
    const item = this.item.findIndex(item => item.value === prev)
    if (item === -1) {
      throw new AppError({
        code: ErrorCodes.ITEM_NOT_FOUND,
        message: 'Item not found',
        type: 'internal',
        stack: new Error().stack
      })
    }
    this.items[item].value = next
  }
}
```

The issue comes with **uncontrolled errors**, for this we will rely on the framework implementation
to handle errors globally or per section: in React it will be the `ErrorBoundary` component and in Flutter
it will be the `ErrorWidgetBuilder`, `Flutter.onError` and `PlatformDispatcher.instance.onError`.

It's fundamental that errors are always re thrown if for some reason needs catching them, this way
we can guarantee that the errors bubbles up to the framework, and it can handle them.

Errors with no type will be automatically catalogued as internal errors be the framework error handling
implementation.