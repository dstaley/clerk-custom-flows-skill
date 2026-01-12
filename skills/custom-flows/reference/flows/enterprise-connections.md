# Build a custom flow for authenticating with enterprise connections

## Before you start

You must configure your application instance through the Clerk Dashboard for the enterprise connection(s) that you want to use. Visit [the appropriate guide for your platform](https://clerk.com/docs/pr/core-3/guides/configure/auth-strategies/enterprise-connections/overview) to learn how to configure your instance.

## Create the sign-up and sign-in flow

**Next.js**

The following example **will both sign up _and_ sign in users**, eliminating the need for a separate sign-up page. However, if you want to have separate sign-up and sign-in pages, the sign-up and sign-in flows are equivalent, meaning that all you have to do is swap out the `SignIn` object for the `SignUp` object using the useSignUp() hook.

The following example:

1. Accesses the SignIn object using the useSignIn() hook.
2. Starts the authentication process by calling [`SignIn.sso(params)`](../sign-in-future.md#sso). This method requires the following params:
   - `redirectUrl`: The URL that the browser will be redirected to once the user authenticates with the identity provider if no additional requirements are needed, and a session has been created
   - `redirectCallbackUrl`: The URL that the browser will be redirected to once the user authenticates with the identity provider if additional requirements are needed
3. Creates a route at the URL that the `redirectCallbackUrl` param points to. The following example re-uses the `/sign-in` route, which should be written to handle when a sign-in attempt is in a non-complete status such as `needs_second_factor`.

```tsx {{ filename: 'app/sign-in/page.tsx' }}
'use client'

import * as React from 'react'
import { useSignIn } from '@clerk/nextjs'

export default function Page() {
  const { signIn, fetchStatus } = useSignIn()

  const signInWithEnterpriseSSO = async (formData: FormData) => {
    const email = formData.get('email') as string

    await signIn.sso({
      identifier: email,
      strategy: 'enterprise_sso',
      // The URL that the user will be redirected to if additional requirements are needed
      redirectCallbackUrl: '/sign-in',
      redirectUrl: '/',
    })
  }

  return (
    <form action={signInWithEnterpriseSSO}>
      <input id="email" type="email" name="email" placeholder="Enter email address" />
      <button type="submit" disabled={fetchStatus === 'fetching'}>
        Sign in with Enterprise SSO
      </button>
    </form>
  )
}
```
