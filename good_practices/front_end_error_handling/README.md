# Good practices: Front end error handling

## Types of errors
First of all we must identify the types of errors that can occur in a front end application.
We can divide them into two groups:
 - **External errors:** These are errors that occur outside the application, such as a server error or a network error.
 - **Internal errors:** These are errors that occur within the application, such as a validation error or a logic error.
Errors produced inside the application as side effect of an external error are considered external errors. For example:
a handled server error like an unsuccessful login attempt.

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

[^1]: Typescript will just be used as a prototyping language in this proposal.

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

The `data` property is optional, and it's used to pass additional information about the error
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

Then we could create a centralized error handler at UI level to capture errors and
display some sort of snackbar or error message for the user.

Example of centralized UI error handler:

```typescript
interface TryActionInput<T> {
  action: Promise<T>;
  rethrowError: boolean;
}

// We could use this function to execute all promises in our program
// and catch all type of errors and show a snackbar error.
async function tryAction<T>({ action, rethrowError }: TryActionInput<T>) {
  showGlobalLoader();
  try {
    await action();
  } catch (e) {
    if(e instanceof AppError) {
      // this error code should have a corresponding translation to show a user frienldy message
      uiProvider.showSnackBar(e.code); 
    } else {
      uiProvider.showSnackBar(ErrorCodes.GENERIC_ERROR);
    }

    if (rethrowError) throw e;
  }
}
```

Also, we could benefit from UI frameworks implementations to capture and display these errors
but this methods will help us better to handle **uncontrolled errors**. With these implementations 
we can handle errors globally or per section: in React it will be the `ErrorBoundary` component and in Flutter
it will be the `ErrorWidgetBuilder`, `Flutter.onError` and `PlatformDispatcher.instance.onError`.

It's fundamental that errors are always re thrown (there might be a special scenario where this is not needed),
so way we can guarantee that errors bubble up to be caught by the framework error handler implementation.

Finally, we should make sure that errors with no type are **automatically catalogued as internal errors** when implementing
the frameworks error handler or our custom centralized implementation.

### External errors

This type of errors are more difficult to handle because they come from sources
that we might not know, partially ignore or that are not correctly documented. So the main
idea is to try to capture every possible known errors and transform them to our app error
object. For those that fall outside our knowledge a generic error should be created to display
a friendly message to the user and log all details to our error monitoring system to track what it's
really happening in those specific scenarios.