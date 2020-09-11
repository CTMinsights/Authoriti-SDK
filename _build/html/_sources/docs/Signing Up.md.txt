### Signing Up

A valid Authoriti user must have an invitation code along with a valid username and password combination. The SDK provides methods for checking the validity of an invitation code, and onboarding a new user using the invitation code, username and password combination.

1. Check invitation code

- Call `checkInvitationCode` method and pass the invitation code as the first argument

```java
authoritiSDK.checkInvitationCode("abcdefg", InvitationCode.OnResultReceivedCallback callback)
```

- Get the result by implementing the following listener

```java
interface OnResultReceivedCallback {
	void onResultReceived(ErrorResponse error, InvitationCodeResult result)
}

// result.isValid() is true if the provided invitation code was a valid invitation code.
// result.getCustomerName() returns the name of the customer who created the invitation code.
```

**Note: Calling the checkInvitationMethod is optional since _signup_ requires the invitation code to be passed. The _checkInvitationCode_ method is useful to build interfaces that use a two step signup process with the first step being checking if the user has a valid invitation code and then asking the user to enter their username and password. The string from _getCustomerName()_ can identify the customer the user is registering to.**

2. Signup user

- Call`signupUser` method with invitation code, username and password

```java
authoritiSDK.signupUser("abcdefg", "tom", "123456", User.OnUserSignedUpCallback callback)
```

- Get the result by implementing the following listener

```java
User.OnUserSignedUpCallback {
	void onUserSignedUp(ErrorResponse error, User user);
}

// user.getAccounts() can be used to a *List<Account>* object which holds the list of accounts that the signup call imported.
```

After a successful signup call, the SDK initializes a Wallet object which holds the users account information. The contents of the wallet is securely stored on the device encrypted using the _Master Password_ - the _Master Password_ is the password used by the user during signup.
