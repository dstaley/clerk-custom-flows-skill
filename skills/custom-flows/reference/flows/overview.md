# Build your own UI (custom flows)

A **custom flow** refers to a user interface built entirely from scratch using the Clerk API.

Custom flows are considered **advanced** and are generally not recommended for most use cases. They require more development effort and are not as easy to maintain as the prebuilt components. However, if prebuilt components don't meet your specific needs or if you require more control over the logic, you can rebuild the existing Clerk flows using the Clerk API.

**If you choose this approach, the Clerk support team will do their best to assist you, but they cannot guarantee a resolution due to the highly customized nature of custom flows.**

## How authentication flows work in Clerk

Before building custom authentication flows, read the following sections to get a general understanding of how authentication flows work in Clerk.

### Sign-up flow

The SignUp object is the pivotal concept in the sign-up process. It is used to gather the user's information, verify their email address or phone number, add OAuth accounts, and finally, convert them into a User.

Every `SignUp` must meet specific requirements before being converted into a `User`. These requirements are defined by the instance settings you selected in the [Clerk Dashboard](https://dashboard.clerk.com/). For example, on the [**User & authentication**](https://dashboard.clerk.com/~/user-authentication/user-and-authentication) page, you can [configure email and password, email links, or SMS codes as authentication strategies](https://clerk.com/docs/pr/core-3/guides/configure/auth-strategies/sign-up-sign-in-options).

Once all requirements are met, the `SignUp` will turn into a new `User`, and an active session for that `User` will be created on the current Client.

Don't worry about collecting all the required fields at once and passing them to a single request. The API is designed to accommodate progressive multi-step sign-up forms.

The following steps outline the sign-up process:

1. Initiate the sign-up process by collecting the user's authentication information and passing the appropriate parameters to the create() method.
2. Prepare the verification.
3. Attempt to complete the verification.
4. If the verification is successful, set the newly created session as the active session by passing the `SignIn.createdSessionId` to the setActive() method on the `Clerk` object.
5. Use the `navigate` parameter in setActive() to access Session.currentTask and check for pending session tasks.

#### The state of a `SignUp`

The `SignUp` object will show **the state of the current sign-up** in the `status` property.

If you need further help on where things are and what you need to do next, you can also consult the `required_fields`, `optional_fields`, and `missingFields` properties.

***

`requiredFields`

All fields that must be collected before the `SignUp` converts into a `User`.

***

`optionalFields`

All fields that can be collected, but are not necessary to convert the `SignUp` into a `User`.

***

`missingFields`

A subset of `requiredFields`. It contains all fields that still need to be collected before a `SignUp` can be converted into a `User`. Note that this property will be updated dynamically. As you add more fields to the `SignUp`, they will be removed. Once this property is empty, your `SignUp` will automatically convert into a `User`.

***

#### Verified fields

Some properties of the `SignUp`, such as `emailAddress` and `phoneNumber`, must be **verified** before they are **fully** added to the `SignUp` object.

The `SignUp` object will show **the state of verification** in the following properties:

***

`unverifiedFields`

A list of all User attributes that need to be verified and are pending verification. This is a list that gets updated dynamically. When verification for all required fields has been successfully completed, this value will become an empty array.

***

`verifications`

An object that describes the current state of verification for the SignUp. There are currently three different keys: `email_address`, `phone_number`, and `external_account`.

***

### Sign-in flow

The SignIn object is the pivotal concept in the sign-in process.

Sign-ins are initiated by creating a `SignIn` object on the current `Client`. If the sign-in is successfully authenticated, it will transform into an active session for that User on the current `Client`.

The following steps outline the sign-in process:

1. Initiate the sign-in process by collecting the user's authentication information and passing the appropriate parameters to the create() method.
2. Prepare the first factor verification. Users must complete a first factor verification to prove their identity. This can be something like providing a password, an email link, a one-time password (OTP), a Web3 wallet address, or providing proof of their identity through an external social account (SSO/OAuth).
3. Attempt to complete the first factor verification.
4. Optionally, if you have enabled [multi-factor (MFA)](https://clerk.com/docs/pr/core-3/guides/configure/auth-strategies/sign-up-sign-in-options) for your application, you will need to prepare the second factor verification for users who have set up MFA for their account.
5. Attempt to complete the second factor verification.
6. If verification is successful, set the newly created session as the active session by passing the `SignIn.createdSessionId` to the setActive() method on the `Clerk` object.
7. Use the `navigate` parameter in setActive() to access Session.currentTask and check for pending session tasks.

### Session tasks

Session tasks require users to complete specific actions after authentication, such as selecting an Organization. These tasks ensure that users meet all requirements before gaining full access to your application.

For detailed information about configuring and implementing session tasks, see the [dedicated guide](https://clerk.com/docs/pr/core-3/guides/configure/session-tasks).

#### Steps to handle session tasks

After completing the sign-up or sign-in process, you should check if the user has any pending tasks:

1. **Check for pending tasks**: Use the `navigate` parameter in setActive() to access Session.currentTask and check for pending session tasks.
2. **Handle the task**: If a task exists, build UI to guide the user through completing the required action based on the task type. See the [available task types](https://clerk.com/docs/pr/core-3/guides/configure/session-tasks#available-tasks) to understand what actions may be required.
3. **Complete the flow**: Once all required tasks are completed, the session becomes fully `active` and the user can access protected content.

You can build your own UI to handle specific task types. For example, if the task is `choose-organization`, you can create a [custom Organization creation component](https://clerk.com/docs/pr/core-3/guides/development/custom-flows/organizations/create-organizations) to allow users to create or select an Organization before proceeding.

## Next steps

Now that you have a general understanding of how authentication flows work in Clerk, you can start building your custom flows. To get started, choose the guide that best fits your needs from the list of guides in the navigation on the left.
