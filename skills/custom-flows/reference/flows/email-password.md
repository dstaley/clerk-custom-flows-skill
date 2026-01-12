# Build a custom email/password authentication flow

This guide will walk you through how to build a custom email/password sign-up and sign-in flow.

## Enable email and password authentication

To use email and password authentication, you first need to ensure they are enabled for your application.

1. In the Clerk Dashboard, navigate to the [**User & authentication**](https://dashboard.clerk.com/~/user-authentication/user-and-authentication) page.
2. Enable **Sign-up with email** and **Sign-in with email**.
3. Select the **Password** tab and enable **Sign-up with password**. Leave **Require a password at sign-up** enabled.

> \[!NOTE]
> By default, **Email verification code** is enabled for both sign-up and sign-in. This means that when a user signs up using their email address, Clerk sends an OTP to their email address. The user must then enter this code to verify their email and complete the sign-up process. When the user uses the email address to sign in, they are emailed an OTP to sign in. If you'd like to use **Email verification link** instead, see the [custom flow for email links](https://clerk.com/docs/pr/core-3/guides/development/custom-flows/authentication/email-links).

## Sign-up flow

To sign up a user using their email, password, and email verification code, you must:

1. Initiate the sign-up process by collecting the user's email address and password with the [`signUp.password()`](../sign-up-future.md#password) method.
2. Send a one-time code to the provided email address for verification with the [`signUp.verifications.sendEmailCode()`](../sign-up-future.md#verifications-send-email-code) method.
3. Collect the one-time code and verify it with the [`signUp.verifications.verifyEmailCode()`](../sign-up-future.md#verifications-verify-email-code) method.
4. If the email address verification is successful, finalize the sign-up with the [`signUp.finalize()`](../sign-up-future.md#finalize) method to create the user and set the newly created session as the active session.

**Next.js**

This example is written for Next.js App Router but it can be adapted for any React-based framework.

```tsx {{ filename: 'app/sign-up/page.tsx', collapsible: true }}
'use client'

import * as React from 'react'
import { useAuth, useSignUp } from '@clerk/nextjs'
import { useRouter } from 'next/navigation'

export default function SignUpPage() {
  const { signUp, errors, fetchStatus } = useSignUp()
  const { isSignedIn } = useAuth()

  const handleSubmit = async (formData: FormData) => {
    const email = formData.get('email') as string
    const password = formData.get('password') as string

    await signUp.password({
      email,
      password,
    })

    await signUp.verifications.sendEmailCode()
  }

  const handleVerify = async (formData: FormData) => {
    const code = formData.get('code') as string

    await signUp.verifications.verifyEmailCode({
      code,
    })
    if (signUp.status === 'complete') {
      await signUp.finalize({
        navigate: () => {
          router.push('/dashboard')
        },
      })
    }
  }

  if (signUp.status === 'complete' || isSignedIn) {
    return null
  }

  if (
    signUp.status === 'missing_requirements' &&
    signUp.unverifiedFields.includes('email_address')
  ) {
    return (
      <form action={handleVerify}>
        <input type="text" name="code" placeholder="Enter your verification code" />
        <button type="submit">Verify</button>
      </form>
    )
  }

  return (
    <form action={handleSubmit}>
      <input type="email" name="email" placeholder="Email" />
      <input type="password" name="password" placeholder="Password" />
      <button type="submit">Sign up</button>
    </form>
  )
}
```

## Sign-in flow

To authenticate a user using their email and password, you must:

1. Initiate the sign-in process by collecting the user's email address and password with the [`signIn.password()`](../sign-in-future.md#password) method.
2. If the attempt is successful, finalize the sign-in with the [`signIn.finalize()`](../sign-in-future.md#finalize) method to set the newly created session as the active session.

**Next.js**

This example is written for Next.js App Router but it can be adapted for any React-based framework.

```tsx {{ filename: 'app/sign-in/page.tsx', collapsible: true }}
'use client'

import * as React from 'react'
import { useAuth, useSignIn } from '@clerk/nextjs'
import { useRouter } from 'next/navigation'

export default function SignInPage() {
  const { signIn, errors, fetchStatus } = useSignIn()
  const { isSignedIn } = useAuth()

  const handleSubmit = async (formData: FormData) => {
    const email = formData.get('email') as string
    const password = formData.get('password') as string

    await signIn.password({
      email,
      password,
    })
    if (signIn.status === 'complete') {
      await signIn.finalize({
        navigate: () => {
          router.push('/dashboard')
        },
      })
    }
  }

  if (signIn.status === 'complete' || isSignedIn) {
    return null
  }

  return (
    <form action={handleSubmit}>
      <input type="email" name="email" placeholder="Email" />
      <input type="password" name="password" placeholder="Password" />
      <button type="submit">Sign in</button>
    </form>
  )
}
```
