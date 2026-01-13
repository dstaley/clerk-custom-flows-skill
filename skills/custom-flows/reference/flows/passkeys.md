# Build a custom authentication flow using passkeys

Clerk supports passwordless authentication via [passkeys](https://clerk.com/docs/pr/core-3/guides/configure/auth-strategies/sign-up-sign-in-options#passkeys), enabling users to sign in without having to remember a password. Instead, users select a passkey associated with their device, which they can use to authenticate themselves.

This guide demonstrates how to use the Clerk API to build a custom user interface for creating, signing users in with, and managing passkeys.

## Enable passkeys

To use passkeys, first enable the strategy in the [Clerk Dashboard](https://dashboard.clerk.com/~/user-authentication/user-and-authentication?user_auth_tab=passkeys).

### Domain restrictions for passkeys

Passkeys are tied to the domain they are created on and **cannot be used across different domains**. However, passkeys **do work on subdomains** if they are registered on the root domain. For example:

- Passkeys created on `your-domain.com` **cannot be used** on `your-domain-admin.com` (different domains).
- Passkeys created on `your-domain.com` **can be used** on `accounts.your-domain.com` (subdomain of the same root domain).
- Passkeys created on `staging1.your-domain.com` **cannot be used** on `staging2.your-domain.com` (sibling subdomains) unless the passkey was scoped to `your-domain.com` (i.e. the shared root domain).

**If you're using [satellite domains](https://clerk.com/docs/pr/core-3/guides/dashboard/dns-domains/satellite-domains)**, in both development and production, passkeys won't be portable between your primary domain and your satellite domains so you should avoid using them.

If you're **not** using satellite domains:

- **In development**, you can either:
  - **The recommended approach**. Use Clerk's components, [Elements](https://clerk.com/docs/pr/core-3/guides/customizing-clerk/elements/overview), or custom flows, instead of the [Account Portal](https://clerk.com/docs/pr/core-3/guides/account-portal/overview). This ensures the passkey is created and used entirely on your development domain, so passkeys created on `localhost` will only work on `localhost`.
  - Create a passkey directly through the Account Portal instead of your local application to keep it tied to the Account Portal's domain. Passkeys created on your Account Portal (e.g., `your-app.accounts.dev`) will only work on that domain, which can cause issues if you switch between `localhost` and the Account Portal during development. If you choose this approach, ensure all testing happens on the same domain where the passkey was created.

- **In production,** your Account Portal is usually hosted on a subdomain of your main domain (e.g. `accounts.your-domain.com`), enabling passkeys to work seamlessly across your app. However, as stated above, if you use **satellite domains**, passkeys will not work as intended.

## Create user passkeys

To create a passkey for a user, you must call User.createPasskey(), as shown in the following example:

```tsx {{ filename: 'app/components/CustomCreatePasskeysButton.tsx' }}
export function CreatePasskeyButton() {
  const { isSignedIn, user } = useUser()

  const createClerkPasskey = async () => {
    if (!isSignedIn) {
      // Handle signed out state
      return
    }

    try {
      await user?.createPasskey()
    } catch (err) {
      // See https://clerk.com/docs/guides/development/custom-flows/error-handling
      // for more info on error handling
      console.error('Error:', JSON.stringify(err, null, 2))
    }
  }

  return <button onClick={createClerkPasskey}>Create a passkey</button>
}
```

## Sign a user in with a passkey

To sign a user into your Clerk app with a passkey, you must call SignIn.authenticateWithPasskey(). This method allows users to choose from their discoverable passkeys, such as hardware keys or passkeys in password managers.

```tsx {{ filename: 'components/SignInWithPasskeyButton.tsx' }}
export function SignInWithPasskeyButton() {
  const { signIn } = useSignIn()
  const router = useRouter()

  const signInWithPasskey = async () => {
    // 'discoverable' lets the user choose a passkey
    // without auto-filling any of the options
    try {
      await signIn.passkey({ flow: 'discoverable' })

      if (signIn.status === 'complete') {
        await signIn.finalize({
          navigate: async ({ session }) => {
            if (session?.currentTask) {
              // Check for tasks and navigate to custom UI to help users resolve them
              // See https://clerk.com/docs/guides/development/custom-flows/overview#session-tasks
              console.log(session?.currentTask)
              return
            }

            router.push('/')
          },
        })
      } else {
        // If the status is not complete, check why. User may need to
        // complete further steps.
        console.error(JSON.stringify(signInAttempt, null, 2))
      }
    } catch (err) {
      // See https://clerk.com/docs/guides/development/custom-flows/error-handling
      // for more info on error handling
      console.error('Error:', JSON.stringify(err, null, 2))
    }
  }

  return <button onClick={signInWithPasskey}>Sign in with a passkey</button>
}
```

## Rename user passkeys

Clerk generates a name based on the device associated with the passkey when it's created. Sometimes users may want to rename a passkey to make it easier to identify.

To rename a user's passkey in your Clerk app, you must call the update() method of the passkey object, as shown in the following example:

```tsx {{ filename: 'components/RenamePasskeyUI.tsx' }}
export function RenamePasskeyUI() {
  const { user } = useUser()
  const { passkeys } = user

  const passkeyToUpdateId = useRef<HTMLInputElement>(null)
  const newPasskeyName = useRef<HTMLInputElement>(null)
  const [success, setSuccess] = useState(false)

  const renamePasskey = async () => {
    try {
      const passkeyToUpdate = passkeys?.find(
        (pk: PasskeyResource) => pk.id === passkeyToUpdateId.current?.value,
      )

      await passkeyToUpdate?.update({
        name: newPasskeyName.current?.value,
      })

      setSuccess(true)
    } catch (err) {
      // See https://clerk.com/docs/guides/development/custom-flows/error-handling
      // for more info on error handling
      console.error('Error:', JSON.stringify(err, null, 2))
      setSuccess(false)
    }
  }

  return (
    <>
      <p>Passkeys:</p>
      <ul>
        {passkeys?.map((pk: PasskeyResource) => {
          return (
            <li key={pk.id}>
              Name: {pk.name} | ID: {pk.id}
            </li>
          )
        })}
      </ul>
      <input ref={passkeyToUpdateId} type="text" placeholder="Enter the passkey ID" />
      <input type="text" placeholder="Enter the passkey's new name" ref={newPasskeyName} />
      <button onClick={renamePasskey}>Rename passkey</button>
      <p>Passkey updated: {success ? 'Yes' : 'No'}</p>
    </>
  )
}
```

## Delete user passkeys

To delete a user's passkey from your Clerk app, you must call the delete() method of the passkey object, as shown in the following example:

```tsx {{ filename: 'components/DeletePasskeyUI.tsx' }}
export function DeletePasskeyUI() {
  const { user } = useUser()
  const { passkeys } = user

  const passkeyToDeleteId = useRef<HTMLInputElement>(null)
  const [success, setSuccess] = useState(false)

  const deletePasskey = async () => {
    const passkeyToDelete = passkeys?.find((pk: any) => pk.id === passkeyToDeleteId.current?.value)
    try {
      await passkeyToDelete?.delete()

      setSuccess(true)
    } catch (err) {
      // See https://clerk.com/docs/guides/development/custom-flows/error-handling
      // for more info on error handling
      console.error('Error:', JSON.stringify(err, null, 2))
      setSuccess(false)
    }
  }

  return (
    <>
      <p>Passkeys:</p>
      <ul>
        {passkeys?.map((pk: any) => {
          return (
            <li key={pk.id}>
              Name: {pk.name} | ID: {pk.id}
            </li>
          )
        })}
      </ul>
      <input ref={passkeyToDeleteId} type="text" placeholder="Enter the passkey ID" />
      <button onClick={deletePasskey}>Delete passkey</button>
      <p>Passkey deleted: {success ? 'Yes' : 'No'}</p>
    </>
  )
}
```
