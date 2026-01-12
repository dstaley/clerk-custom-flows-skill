# Build a custom sign-in flow with multi-factor authentication

[Multi-factor verification (MFA)](https://clerk.com/docs/pr/core-3/guides/configure/auth-strategies/sign-up-sign-in-options) is an added layer of security that requires users to provide a second verification factor to access an account.

Clerk supports second factor verification through **SMS verification code**, **Authenticator application**, and **Backup codes**.

This guide will walk you through how to build a custom email/password sign-in flow that supports **Authenticator application** and **Backup codes** as the second factor.

## Enable email and password

This guide uses email and password to sign in, however, you can modify this approach according to the needs of your application.

To follow this guide, you first need to ensure email and password are enabled for your application.

1. In the Clerk Dashboard, navigate to the [**User & authentication**](https://dashboard.clerk.com/~/user-authentication/user-and-authentication) page.
2. Enable **Sign-in with email**.
3. Select the **Password** tab and enable **Sign-up with password**. Leave **Require a password at sign-up** enabled.

## Enable multi-factor authentication

For your users to be able to enable MFA for their account, you need to enable MFA for your application.

1. In the Clerk Dashboard, navigate to the [**Multi-factor**](https://dashboard.clerk.com/~/user-authentication/multi-factor) page.
2. For the purpose of this guide, toggle on both the **Authenticator application** and **Backup codes** strategies.
3. Select **Save**.

> \[!WARNING]
> If you're using Duo as an authenticator app, please note that Duo generates TOTP codes differently than other authenticator apps. Duo allows a code to be valid for 30 seconds from _the moment it is first displayed_, which may cause frequent `invalid_code` errors if the code is not entered promptly. More information can be found in [Duo's Help Center](https://help.duo.com/s/article/2107).

## Sign-in flow

Signing in to an MFA-enabled account is identical to the regular sign-in process. However, in the case of an MFA-enabled account, a sign-in won't convert until both first factors and second factors verifications are completed.

To authenticate a user using their email and password, you need to:

1. Initiate the sign-in process by collecting the user's email address and password with the [`signIn.password()`](../sign-in-future.md#password) method.
2. Collect the TOTP code and verify it with the [`signIn.mfa.verifyTOTP()`](../sign-in-future.md#mfa-verify-totp) method.
3. If the TOTP verification is successful, finalize the sign-in with the [`signIn.finalize()`](../sign-in-future.md#finalize) method to set the newly created session as the active session.

> \[!TIP]
> For this example to work, the user must have MFA enabled on their account. You need to add the ability for your users to manage their MFA settings. See the [manage SMS-based MFA](https://clerk.com/docs/pr/core-3/guides/development/custom-flows/account-updates/manage-sms-based-mfa) or the [manage TOTP-based MFA](https://clerk.com/docs/pr/core-3/guides/development/custom-flows/account-updates/manage-totp-based-mfa) guide, depending on your needs.

**Next.js**

```tsx {{ filename: 'app/sign-in/page.tsx', collapsible: true }}
'use client'

import * as React from 'react'
import { useSignIn } from '@clerk/nextjs'
import { useRouter } from 'next/navigation'

export default function SignInForm() {
  const { signIn, errors, fetchStatus } = useSignIn()
  const router = useRouter()

  const handleSubmit = async (formData: FormData) => {
    const emailAddress = formData.get('email') as string
    const password = formData.get('password') as string

    await signIn.password({
      emailAddress,
      password,
    })
  }

  const handleSubmitTOTP = async (formData: FormData) => {
    const code = formData.get('code') as string
    const useBackupCode = formData.get('useBackupCode') === 'on'

    if (useBackupCode) {
      await signIn.mfa.verifyBackupCode({ code })
    } else {
      await signIn.mfa.verifyTOTP({ code })
    }

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
      <div>
        <h1>Verify your account</h1>
        <form action={handleSubmitTOTP}>
          <div>
            <label htmlFor="code">Code</label>
            <input id="code" name="code" type="text" />
            {errors.fields.code && <p>{errors.fields.code.message}</p>}
          </div>
          <div>
            <label>
              Use backup code
              <input type="checkbox" name="useBackupCode" />
            </label>
          </div>
          <button type="submit" disabled={fetchStatus === 'fetching'}>
            Verify
          </button>
        </form>
      </div>
    )
  }

  return (
    <>
      <h1>Sign in</h1>
      <form action={handleSubmit}>
        <div>
          <label htmlFor="emailAddress">Email</label>
          <input id="emailAddress" name="emailAddress" type="emailAddress" />
          {errors.fields.emailAddress && <p>{errors.fields.emailAddress.message}</p>}
        </div>
        <div>
          <label htmlFor="password">Password</label>
          <input id="password" name="password" type="password" />
          {errors.fields.password && <p>{errors.fields.password.message}</p>}
        </div>
        <button type="submit" disabled={fetchStatus === 'fetching'}>
          Continue
        </button>
      </form>
    </>
  )
}
```

{/* TODO: Add logic for MFA for phone code */}

## Next steps

Now that users can sign in with MFA, you need to add the ability for your users to manage their MFA settings. Learn how to build a custom flow for [managing TOTP MFA](https://clerk.com/docs/pr/core-3/guides/development/custom-flows/account-updates/manage-totp-based-mfa) or for [managing SMS MFA](https://clerk.com/docs/pr/core-3/guides/development/custom-flows/account-updates/manage-sms-based-mfa).
