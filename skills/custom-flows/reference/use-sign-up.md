# useSignUp()

The `useSignUp()` hook provides access to the [`SignUpFuture`](./sign-up-future.md) object, which allows you to check the current state of a sign-up attempt and manage the sign-up flow. You can use this to create a [custom sign-up flow](./flows/overview.md#sign-up-flow).

## Returns

***

`signUp` [`SignUpFuture`](./sign-up-future.md)

The current active `SignUpFuture` instance, for use in custom flows.

***

`errors` [`Errors`](./types/errors.md)

The errors that occurred during the last API request.

***

`fetchStatus` `'idle' | 'fetching'`

The fetch status of the underlying `SignUpFuture` resource.

***

## Examples

For example usage of the `useSignUp()` hook, see the [Build your own UI](./flows/overview.md) guide.
