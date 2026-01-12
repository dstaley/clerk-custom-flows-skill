# useSignIn()

The `useSignIn()` hook provides access to the [`SignInFuture`](./sign-in-future.md) object, which allows you to check the current state of a sign-in attempt and manage the sign-in flow. You can use this to create a [custom sign-in flow](./flows/overview.md#sign-in-flow).

## Returns

***

`signIn` [`SignInFuture`](./sign-in-future.md)

The current active `SignInFuture` instance, for use in custom flows.

***

`errors` [`Errors`](./types/errors.md)

The errors that occurred during the last API request.

***

`fetchStatus` `'idle' | 'fetching'`

The fetch status of the underlying `SignInFuture` resource.

***

## Examples

For example usage of the `useSignIn()` hook, see the [Build your own UI](./flows/overview.md) guide.
