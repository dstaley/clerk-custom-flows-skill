# Build a custom email or SMS OTP authentication flow

Clerk supports passwordless authentication, which lets users sign in and sign up without having to remember a password. Instead, users receive a one-time password (OTP) via email or SMS, which they can use to authenticate themselves.

This guide will walk you through how to build a custom SMS OTP sign-up and sign-in flow. The process for using email OTP is similar, and the differences will be highlighted throughout.

> \[!WARNING]
> Phone numbers must be in [E.164 format](https://en.wikipedia.org/wiki/E.164).

## Enable SMS OTP

To use SMS OTP, you first need to enable it for your application.

1. In the Clerk Dashboard, navigate to the [**User & authentication**](https://dashboard.clerk.com/~/user-authentication/user-and-authentication) page.
2. Select the **Phone** tab and enable **Sign-up with phone** and **Sign-in with phone** and keep the default settings.

## Sign-up flow

To sign up a user using an OTP, you must:

1. Initiate the sign-up process by collecting the user's phone number with the [`signUp.create()`](../sign-up-future.md#create) method.
2. Send a one-time code to the provided phone number for verification with the [`signUp.verifications.sendPhoneCode()`](../sign-up-future.md#verifications-send-phone-code) method.
3. Collect the one-time code and verify it with the [`signUp.verifications.verifyPhoneCode()`](../sign-up-future.md#verifications-verify-phone-code) method.
4. If the phone number verification is successful, finalize the sign-up with the [`signUp.finalize()`](../sign-up-future.md#finalize) method to create the user and set the newly created session as the active session.

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
    const phoneNumber = formData.get('phoneNumber') as string

    await signUp.create({ phoneNumber })

    await signUp.verifications.sendPhoneCode()
  }

  const handleVerify = async (formData: FormData) => {
    const code = formData.get('code') as string

    await signUp.verifications.verifyPhoneCode({ code })
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
    signUp.unverifiedFields.includes('phone_number')
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
      <input type="tel" name="phoneNumber" placeholder="Phone number" />
      <button type="submit" disabled={fetchStatus === 'fetching'}>
        Sign up
      </button>
    </form>
  )
}
```

To create a sign-up flow for email OTP, use the [`signUp.emailCode.sendCode()`](../sign-up-future.md) and [`signUp.emailCode.verifyCode()`](../sign-up-future.md) methods. These methods work the same way as their phone number counterparts do in the previous example. You can find all available methods in the [`SignUpFuture`](../sign-up-future.md) object documentation.

## Sign-in flow

To authenticate a user with an OTP, you must:

1. Initiate the sign-in process by calling the [`signIn.phoneCode.sendCode()`](../sign-in-future.md#phone-code-send-code) method using the identifier provided, which for this example is a phone number.
2. Verify the phone code provided by the user with the [`signIn.phoneCode.verifyCode()`](../sign-in-future.md#phone-code-verify-code) method.
3. If the attempt is successful, finalize the sign-in with the [`signIn.finalize()`](../sign-in-future.md#finalize) method to set the newly created session as the active session.

**Next.js**

This example is written for Next.js App Router but it can be adapted to any React-based framework.

```tsx {{ filename: 'app/sign-in/page.tsx', collapsible: true }}
'use client'

import * as React from 'react'
import { useSignIn } from '@clerk/nextjs'
import { useRouter } from 'next/navigation'

export default function Page() {
  const { signIn, errors, fetchStatus } = useSignIn()
  const router = useRouter()

  async function handleSubmit(formData: FormData) {
    const phoneNumber = formData.get('phoneNumber') as string

    await signIn.phoneCode.sendCode({ phoneNumber })
  }

  async function handleVerification(formData: FormData) {
    const code = formData.get('code') as string

    await signIn.phoneCode.verifyCode({ code })
    if (signIn.status === 'complete') {
      await signIn.finalize({
        navigate: () => {
          router.push('/')
        },
      })
    }
  }

  if (signIn.status === 'needs_second_factor') {
    return (
      <>
        <h1>Verify your phone number</h1>
        <form action={handleVerification}>
          <label htmlFor="code">Enter your verification code</label>
          <input id="code" name="code" type="text" />
          {errors.fields.code && <p>{errors.fields.code.message}</p>}
          <button type="submit" disabled={fetchStatus === 'fetching'}>
            Verify
          </button>
        </form>
      </>
    )
  }

  return (
    <>
      <h1>Sign in</h1>
      <form action={handleSubmit}>
        <label htmlFor="phoneNumber">Enter phone number</label>
        <input id="phoneNumber" name="phoneNumber" type="tel" />
        {errors.fields.phoneNumber && <p>{errors.fields.phoneNumber.message}</p>}
        <button type="submit" disabled={fetchStatus === 'fetching'}>
          Continue
        </button>
      </form>
    </>
  )
}
```

To create a sign-in flow for email OTP, use the [`signIn.emailCode.sendCode()`](../sign-in-future.md#email-code-send-code) and [`signIn.emailCode.verifyCode()`](../sign-in-future.md#email-code-verify-code) methods. These methods work the same way as their phone number counterparts do in the previous example. You can find all available methods in the [`SignInFuture`](../sign-in-future.md) object documentation.
