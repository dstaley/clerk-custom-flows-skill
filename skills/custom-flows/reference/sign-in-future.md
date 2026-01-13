# `SignInFuture`

> \[!IMPORTANT]
> The APIs described here are stable, and will become the default in the next major version of `clerk-js`.

The `SignInFuture` class holds the state of the current sign-in and provides helper methods to navigate and complete the sign-in process. It is used to manage the sign-in lifecycle, including the first and second factor verification, and the creation of a new session.

## Properties

***

`createdSessionId` `null | string`

The identifier of the session that was created upon completion of the current sign-in. The value of this property is `null` if the sign-in status is not `'complete'`.

***

`existingSession?` `{ sessionId: string }`

If present, reflects that the sign-in was not able to create a new session because the identifier already exists in an existing session. Contains the ID of the existing session.

***

`firstFactorVerification` [`VerificationResource`](./types/verification.md)

The state of the verification process for the selected first factor. Initially, this property contains an empty verification object, since there is no first factor selected.

***

`id?` `string`

The unique identifier for the current sign-in attempt.

***

`identifier` `null | string`

The authentication identifier value for the current sign-in. `null` if the `strategy` is `'oauth_<provider>'` or `'enterprise_sso'`.

***

`isTransferable` `boolean`

Indicates that there is not a matching user for the first-factor verification used, and that the sign-in can be transferred to a sign-up.

***

`secondFactorVerification` [`VerificationResource`](./types/verification.md)

The state of the verification process for the selected second factor. Initially, this property contains an empty verification object, since there is no second factor selected.

***

`status` `SignInStatus`

The current status of the sign-in.

***

`supportedFirstFactors` <code><a href="./types/sign-in-first-factor.md">SignInFirstFactor</a>[]</code>

Array of the first factors that are supported in the current sign-in. Each factor contains information about the verification strategy that can be used.

***

`supportedSecondFactors` <code><a href="./types/sign-in-second-factor.md">SignInSecondFactor</a>[]</code>

Array of the second factors that are supported in the current sign-in. Each factor contains information about the verification strategy that can be used. his property is populated only when the first factor is verified.

***

`userData` `UserData`

An object containing information about the user of the current sign-in. This property is populated only once an identifier is given to the `SignIn` object through `signIn.create()` or another method that populates the `identifier` property.

***

## Methods

### `create()`

Creates a new `SignIn` instance initialized with the provided parameters. The instance maintains the sign-in lifecycle state through its `status` property, which updates as the authentication flow progresses.

What you must pass to `params` depends on which [sign-in options](https://clerk.com/docs/pr/core-3/guides/configure/auth-strategies/sign-up-sign-in-options) you have enabled in your app's settings in the Clerk Dashboard.

You can complete the sign-in process in one step if you supply the required fields to `create()`. Otherwise, Clerk's sign-in process provides great flexibility and allows users to easily create multi-step sign-in flows.

> \[!WARNING]
> Once the sign-in process is complete, call the `signIn.finalize()` method to set the newly created session as the active session.

```ts
function create(params: SignInFutureCreateParams): Promise<{ error: unknown }>
```

#### `SignInFutureCreateParams`

***

`actionCompleteRedirectUrl?` `string`

The URL that the user will be redirected to, after successful authorization from the OAuth provider and Clerk sign-in.

***

`identifier?` `string`

The authentication identifier for the sign-in. This can be the value of the user's email address, phone number, username, or Web3 wallet address.

***

`redirectUrl?` `string`

The full URL or path that the OAuth provider should redirect to after successful authorization on their part.

***

`strategy?` <code><a href="./types/sso.md#o-auth-strategy">OAuthStrategy</a> | "saml" | "enterprise_sso"</code>

The first factor verification strategy to use in the sign-in flow. Depends on the `identifier` value. Each authentication identifier supports different verification strategies.

***

`ticket?` `string`

The [ticket _or token_](./flows/application-invitations.md) generated from the Backend API. **Required** if `strategy` is set to `'ticket'`.

***

`transfer?` `boolean`

When set to `true`, the `SignIn` will attempt to retrieve information from the active `SignUp` instance and use it to complete the sign-in process. This is useful when you want to seamlessly transition a user from a sign-up attempt to a sign-in attempt.

***

### `emailCode.sendCode()`

Used to send an email code to sign-in

```ts
function sendCode(params: SignInFutureEmailCodeSendParams): Promise<{ error: unknown }>
```

#### `SignInFutureEmailCodeSendParams`

***

`emailAddress?` `string`

The user's email address. Only supported if [Email address](https://clerk.com/docs/pr/core-3/guides/configure/auth-strategies/sign-up-sign-in-options#email) is enabled.

***

`emailAddressId?` `string`

The ID for the user's email address that will receive an email with the one-time authentication code.

***

### `emailCode.verifyCode()`

Used to verify a code sent via email to sign-in

```ts
function verifyCode(params: SignInFutureEmailCodeVerifyParams): Promise<{ error: unknown }>
```

#### `SignInFutureEmailCodeVerifyParams`

***

`code` `string`

The one-time code that was sent to the user.

***

### `emailLink.sendLink()`

Used to send an email link to sign-in

```ts
function sendLink(params: SignInFutureEmailLinkSendParams): Promise<{ error: unknown }>
```

#### `SignInFutureEmailLinkSendParams`

***

`verificationUrl` `string`

The full URL that the user will be redirected to when they visit the email link.

***

### `emailLink.waitForVerification()`

Will wait for verification to complete or expire

```ts
function waitForVerification(): Promise<{ error: unknown }>
```

### `finalize()`

Used to convert a sign-in with `status === 'complete'` into an active session. Will cause anything observing the session state (such as the `useUser()` hook) to update automatically.

```ts
function finalize(params?: SignInFutureFinalizeParams): Promise<{ error: unknown }>
```

#### `SignInFutureFinalizeParams`

***

`navigate?` <code>(opts: { session: <a href="./session.md">SessionResource</a> }) => Promise<unknown></code>

A custom navigation function to be called just before the session and/or organization is set.

***

### `mfa.sendPhoneCode()`

Used to send a phone code as a second factor to sign-in

```ts
function sendPhoneCode(): Promise<{ error: unknown }>
```

### `mfa.verifyBackupCode()`

Used to verify a backup code as a second factor to sign-in

```ts
function verifyBackupCode(params: SignInFutureBackupCodeVerifyParams): Promise<{ error: unknown }>
```

#### `SignInFutureBackupCodeVerifyParams`

***

`code` `string`

The backup code that was provided to the user when they set up two-step authentication.

***

### `mfa.verifyPhoneCode()`

Used to verify a phone code sent as a second factor to sign-in

```ts
function verifyPhoneCode(params: SignInFutureMFAPhoneCodeVerifyParams): Promise<{ error: unknown }>
```

#### `SignInFutureMFAPhoneCodeVerifyParams`

***

`code` `string`

The one-time code that was sent to the user as part of the `signIn.mfa.sendPhoneCode()` method.

***

### `mfa.verifyTOTP()`

Used to verify a TOTP code as a second factor to sign-in

```ts
function verifyTOTP(params: SignInFutureTOTPVerifyParams): Promise<{ error: unknown }>
```

#### `SignInFutureTOTPVerifyParams`

***

`code` `string`

The TOTP generated by the user's authenticator app.

***

### `passkey()`

Initiates a passkey-based authentication flow, enabling users to authenticate using a previously registered passkey. When called without parameters, this method requires a prior call to `signIn.create({ strategy: 'passkey' })` to initialize the sign-in context. This pattern is particularly useful in scenarios where the authentication strategy needs to be determined dynamically at runtime.

```ts
function passkey(params?: SignInFuturePasskeyParams): Promise<{ error: null | ClerkError }>
```

#### `SignInFuturePasskeyParams`

***

`flow?` `"autofill" | "discoverable"`

The flow to use for the passkey sign-in.

`'autofill'`: The client prompts your users to select a passkey before they interact with your app. `'discoverable'`: The client requires the user to interact with the client.

***

### `password()`

Used to submit a password to sign-in.

```ts
function password(params: SignInFuturePasswordParams): Promise<{ error: unknown }>
```

#### `SignInFuturePasswordParams`

***

`password` `string`

The user's password. Only supported if [password](https://clerk.com/docs/pr/core-3/guides/configure/auth-strategies/sign-up-sign-in-options#password) is enabled.

***

`emailAddress` `string`

The user's email address. Only supported if [Email address](https://clerk.com/docs/pr/core-3/guides/configure/auth-strategies/sign-up-sign-in-options#email) is enabled.

***

`identifier` `string`

The authentication identifier for the sign-in. This can be the value of the user's email address, phone number, username, or Web3 wallet address.

***

`phoneNumber` `string`

The user's phone number in [E.164 format](https://en.wikipedia.org/wiki/E.164). Only supported if [phone number](https://clerk.com/docs/pr/core-3/guides/configure/auth-strategies/sign-up-sign-in-options#phone) is enabled.

***

### `phoneCode.sendCode()`

Used to send a phone code to sign-in

```ts
function sendCode(params: SignInFuturePhoneCodeSendParams): Promise<{ error: unknown }>
```

#### `SignInFuturePhoneCodeSendParams`

***

`channel?` `'sms' | 'whatsapp'`

The mechanism to use to send the code to the provided phone number. Defaults to `'sms'`.

***

### `phoneCode.verifyCode()`

Used to verify a code sent via phone to sign-in

```ts
function verifyCode(params: SignInFuturePhoneCodeVerifyParams): Promise<{ error: unknown }>
```

#### `SignInFuturePhoneCodeVerifyParams`

***

`code` `string`

The one-time code that was sent to the user.

***

### `resetPasswordEmailCode.sendCode()`

Used to send a password reset code to the first email address on the account

```ts
function sendCode(): Promise<{ error: unknown }>
```

### `resetPasswordEmailCode.submitPassword()`

Used to submit a new password, and move the `signIn.status` to `'complete'`.

```ts
function submitPassword(params: SignInFutureResetPasswordSubmitParams): Promise<{ error: unknown }>
```

#### `SignInFutureResetPasswordSubmitParams`

***

`password` `string`

The new password for the user.

***

`signOutOfOtherSessions?` `boolean`

If `true`, signs the user out of all other authenticated sessions.

***

### `resetPasswordEmailCode.verifyCode()`

Used to verify a password reset code sent via email. Will cause `signIn.status` to become `'needs_new_password'`.

```ts
function verifyCode(params: SignInFutureEmailCodeVerifyParams): Promise<{ error: unknown }>
```

#### `SignInFutureEmailCodeVerifyParams`

***

`code` `string`

The one-time code that was sent to the user.

***

### `sso()`

Used to perform OAuth authentication.

```ts
function sso(params: SignInFutureSSOParams): Promise<{ error: unknown }>
```

#### `SignInFutureSSOParams`

***

`redirectCallbackUrl` `string`

The URL to redirect to if a session was not created, and needs additional information.

***

`redirectUrl` `string`

The URL to redirect to after the user has completed the SSO flow.

***

`strategy` <code><a href="./types/sso.md#o-auth-strategy">OAuthStrategy</a> | 'enterprise_sso'</code>

The strategy to use for authentication.

***

`popup?` `Window`

If provided a `Window` opened via `window.open()`, the sign-in flow will be performed in a popup window.

***

`oidcPrompt?` `string`

The value to pass to the [OIDC `prompt` parameter](https://openid.net/specs/openid-connect-core-1_0.html#:~:text=prompt,reauthentication%20and%20consent.) in the generated OAuth redirect URL.

***

`enterpriseConnectionId?` `string`

***

### `ticket()`

Used to perform a ticket-based sign-in.

```ts
function ticket(params?: SignInFutureTicketParams): Promise<{ error: unknown }>
```

#### `SignInFutureTicketParams`

***

`ticket` `string`

The [ticket _or token_](./flows/application-invitations.md) generated from the Backend API.

***

### `web3()`

Used to perform a Web3-based sign-in.

```ts
function web3(params: SignInFutureWeb3Params): Promise<{ error: unknown }>
```

#### `SignInFutureWeb3Params`

***

`strategy` `"web3_base_signature" | "web3_metamask_signature" | "web3_coinbase_wallet_signature" | "web3_okx_wallet_signature"`

The verification strategy to validate the user's sign-in request.

***
