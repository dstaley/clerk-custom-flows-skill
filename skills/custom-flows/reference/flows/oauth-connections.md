# Build a custom flow for authenticating with OAuth connections

## Before you start

You must configure your application instance through the Clerk Dashboard for the social connection(s) that you want to use. Visit [the appropriate guide for your platform](https://clerk.com/docs/pr/core-3/guides/configure/auth-strategies/social-connections/overview) to learn how to configure your instance.

## Create the sign-up and sign-in flow

**Next.js**

First, in your `.env` file, set the `NEXT_PUBLIC_CLERK_SIGN_IN_URL` environment variable to tell Clerk where the sign-in page is being hosted. Otherwise, your app may default to using the [Account Portal sign-in page](https://clerk.com/docs/pr/core-3/guides/account-portal/overview#sign-in) instead. This guide uses the `/sign-in` route.

```env {{ filename: '.env' }}
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
```

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
import { OAuthStrategy } from '@clerk/types'
import { useSignIn } from '@clerk/nextjs'

export default function Page() {
  const { signIn } = useSignIn()

  const signInWith = async (strategy: OAuthStrategy) => {
    await signIn.sso({
      strategy,
      redirectCallbackUrl: '/sso-callback',
      redirectUrl: '/sign-in/tasks', // Learn more about session tasks at https://clerk.com/docs/guides/development/custom-flows/overview#session-tasks
    })
  }

  // Render a button for each supported OAuth provider
  // you want to add to your app. This example uses only Google.
  return (
    <div>
      <button onClick={() => signInWith('oauth_google')}>Sign in with Google</button>
    </div>
  )
}
```

```tsx {{ filename: 'app/sso-callback/page.tsx' }}
'use client'

import { useClerk, useSignIn, useSignUp } from '@clerk/nextjs'
import { useRouter } from 'next/navigation'
import { useEffect, useRef } from 'react'

export function SSOCallback() {
  const clerk = useClerk()
  const { signIn } = useSignIn()
  const { signUp } = useSignUp()
  const router = useRouter()
  const hasRun = useRef(false)

  const navigateToApp = () => {
    router.push('/protected')
  }

  const navigateToSignIn = () => {
    router.push('/sign-in')
  }

  const navigateToSignUp = () => {
    router.push('/sign-up')
  }

  useEffect(() => {
    ;(async () => {
      if (!clerk.loaded || hasRun.current) {
        return
      }
      // Prevent Next.js from re-running this effect when the page is re-rendered during session activation.
      hasRun.current = true

      // If this was a sign-in, and it's complete, there's nothing else to do.
      if (signIn.status === 'complete') {
        await signIn.finalize({
          navigate: async () => {
            navigateToApp()
          },
        })
        return
      }

      // If the sign-up used an existing account, transfer it to a sign-in.
      if (signUp.isTransferable) {
        await signIn.create({ transfer: true })
        if (signIn.status === 'complete') {
          await signIn.finalize({
            navigate: async () => {
              navigateToApp()
            },
          })
          return
        }
        // The sign-in requires additional verification, so we need to navigate to the sign-in page.
        return navigateToSignIn()
      }

      if (
        signIn.status === 'needs_first_factor' &&
        !signIn.supportedFirstFactors?.every((f) => f.strategy === 'enterprise_sso')
      ) {
        // The sign-in requires the use of a configured first factor, so navigate to the sign-in page.
        return navigateToSignIn()
      }

      // If the sign-in used an external account not associated with an existing user, create a sign-up.
      if (signIn.isTransferable) {
        await signUp.create({ transfer: true })
        if (signUp.status === 'complete') {
          await signUp.finalize({
            navigate: async () => {
              navigateToApp()
            },
          })
          return
        }
        return navigateToSignUp()
      }

      if (signUp.status === 'complete') {
        await signUp.finalize({
          navigate: async () => {
            navigateToApp()
          },
        })
        return
      }

      if (signIn.status === 'needs_second_factor' || signIn.status === 'needs_new_password') {
        // The sign-in requires a MFA token or a new password, so navigate to the sign-in page.
        return navigateToSignIn()
      }

      // The external account used to sign-in or sign-up was already associated with an existing user and active
      // session on this client, so activate the session and navigate to the application.
      if (signIn.existingSession || signUp.existingSession) {
        const sessionId = signIn.existingSession?.sessionId || signUp.existingSession?.sessionId
        if (sessionId) {
          // Because we're activating a session that's not the result of a sign-in or sign-up, we need to use the
          // Clerk `setActive` API instead of the `finalize` API.
          await clerk.setActive({
            session: sessionId,
            navigate: async () => {
              return navigateToApp()
            },
          })
          return
        }
      }
    })()
  }, [clerk, signIn, signUp])

  return (
    <div>
      {/* Because a sign-in transferred to a sign-up might require captcha verification, make sure to render the
  captcha element. */}
      <div id="clerk-captcha"></div>
    </div>
  )
}
```

## Handle missing requirements

Depending on your instance settings, users might need to provide extra information before their sign-up can be completed, such as when a username or accepting legal terms is required. In these cases, the `SignUp` object returns a status of `"missing_requirements"` along with a `missingFields` array. You can use your existing sign-up page to collect these missing fields and complete the sign-up flow. Handling the missing requirements will depend on your instance settings. For example, if your instance settings require a phone number, you will need to [handle verifying the phone number](./email-sms-otp.md#sign-up-flow).

With OAuth flows, it's common for users to try to _sign in_ with an OAuth provider, but they don't have a Clerk account for your app yet. Clerk automatically transfers the flow from the `SignIn` object to the `SignUp` object, which returns the `"missing_requirements"` status and `missingFields` array needed to handle the missing requirements flow. This is why the "Continue" page uses the useSignUp() hook and treats the missing requirements flow as a sign-up flow.

**Next.js**

```tsx {{ filename: 'app/sign-up/page.tsx' }}
'use client'

import { useState } from 'react'
import { useSignUp } from '@clerk/nextjs'
import { useRouter } from 'next/navigation'

function snakeToCamel(str: string | undefined): string {
  return str ? str.replace(/([-_][a-z])/g, (match) => match.toUpperCase().replace(/-|_/, '')) : ''
}

export default function Page() {
  const router = useRouter()
  // Use `useSignUp()` hook to access the `SignUp` object
  // `missing_requirements` and `missingFields` are only available on the `SignUp` object
  const { signUp } = useSignUp()

  const handleSubmit = async (formData: FormData) => {
    const params = Object.fromEntries(formData.entries()) as any
    // Update the `SignUp` object with the missing fields
    // The logic that goes here will depend on your instance settings
    // E.g. if your app requires a phone number, you will need to collect and verify it here
    await signUp.update(params)
    if (signUp.status === 'complete') {
      await signUp.finalize({
        navigate: async ({ session }) => {
          if (session?.currentTask) {
            // Check for tasks and navigate to custom UI to help users resolve them
            // See https://clerk.com/docs/guides/development/custom-flows/overview#session-tasks
            console.log(session?.currentTask)
            router.push('/sign-in/tasks')
            return
          }

          router.push('/')
        },
      })
    }
  }

  if (signUp.status === 'missing_requirements') {
    // For simplicity, all missing fields in this example are text inputs.
    // In a real app, you might want to handle them differently:
    // - legal_accepted: checkbox
    // - username: text with validation
    // - phone_number: phone input, etc.
    return (
      <div>
        <h1>Continue sign-up</h1>
        <form action={handleSubmit}>
          {signUp.missingFields.map((field) => (
            <div key={field}>
              <label>
                {field}:
                <input type="text" name={snakeToCamel(field)} />
              </label>
            </div>
          ))}

          {/* Required for sign-up flows
          Clerk's bot sign-up protection is enabled by default */}
          <div id="clerk-captcha" />

          <button type="submit">Submit</button>
        </form>
      </div>
    )
  }

  // Handle other statuses if needed
  return (
    <>
      {/* Required for sign-up flows
      Clerk's bot sign-up protection is enabled by default */}
      <div id="clerk-captcha" />
    </>
  )
}
```
