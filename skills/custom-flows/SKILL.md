---
name: custom-flows
description: Build custom sign-in and sign-up authentication flows with Clerk's useSignIn and useSignUp hooks. Use this skill when the user asks to build sign-in or sign-up flows using Clerk, or when JavaScript/TypeScript files reference the hooks useSignIn or useSignUp.
---

## Overview

Clerk, an authentication and user management product, allows users to build completely custom UI using Clerk's `useSignIn()` and `useSignUp()` hooks. This skill helps provide reference material including API documentation along with sample code.

Reference material is contained within the `reference` folder. The first file listed files below contain generally applicable reference, whereas the remaining files are specific to the flow the user wants to implement.

Keep in mind that while the examples are specifc to Next.js, they can be adjusted to the specific React framework being used by the user. Make sure that navigation actions are using the router that the user's application uses. Furthermore, while the examples use plain unstyled HTML, ensure the code you write uses the styling system in use by the user's application if applicable.

## Table of Contents

<!-- prettier-ignore-start -->
Title | Description
--- | ---
[`SignInFuture`](./reference/sign-in-future.md) | The current active `SignIn` instance, for use in custom flows.
[`SignUpFuture`](./reference/sign-up-future.md) | The current active `SignUp` instance, for use in custom flows.
[useSignIn()](./reference/use-sign-in.md) | Access and manage the current user's sign-in state in your React application with Clerk's useSignIn() hook.
[useSignUp()](./reference/use-sign-up.md) | Access and manage the current user's sign-up state in your React application with Clerk's useSignUp() hook.
[Build your own UI (custom flows)](./reference/flows/overview.md) | Learn the process behind building custom sign-up and sign-in flows with Clerk.
[Build a custom email/password authentication flow](./reference/flows/email-password.md) | Learn how to build a custom email/password sign-up and sign-in flow using the Clerk API.
[Build a custom email or SMS OTP authentication flow](./reference/flows/email-sms-otp.md) | Learn how build a custom email or SMS one time code (OTP) authentication flow using the Clerk API.
[Build a custom sign-in flow with multi-factor authentication](./reference/flows/email-password-mfa.md) | Learn how to build a custom email/password sign-in flow that requires multi-factor authentication (MFA).
[Build a custom flow for authenticating with OAuth connections](./reference/flows/oauth-connections.md) | Learn how to use the Clerk API to build a custom sign-up and sign-in flow that supports OAuth connections.
[Build a custom flow for authenticating with enterprise connections](./reference/flows/enterprise-connections.md) | Learn how to use the Clerk API to build a custom sign-up and sign-in flow that supports enterprise connections.
[Sign-up with application invitations](./reference/flows/application-invitations.md) | Learn how to use the Clerk API to build a custom flow for handling application invitations.
[`Errors`](./reference/types/errors.md) | An interface that represents the errors that occurred during the last API request, for use in custom flows.
[`ClerkAPIResponseError`](./reference/types/clerk-api-response-error.md) | An interface that represents an error returned by the Clerk API.
[`ClerkAPIError`](./reference/types/clerk-api-error.md) | An interface that represents an error returned by the Clerk API.
[`Verification`](./reference/types/verification.md) | An interface that represents the state of the verification process of a sign-in or sign-up attempt.
[`Session` object](./reference/session.md) | The Session object is an abstraction over an HTTP session. It models the period of information exchange between a user and the server.
[`SignInFirstFactor`](./reference/types/sign-in-first-factor.md) | The SignInFirstFactor type represents the first factor verification strategy that can be used in the sign-in process.
[`SignInSecondFactor`](./reference/types/sign-in-second-factor.md) | The SignInSecondFactor type represents the second factor verification strategy that can be used in the sign-in process.
[SSO Types](./reference/types/sso.md) | Types related to SSO providers and strategies.
<!-- prettier-ignore-end -->
