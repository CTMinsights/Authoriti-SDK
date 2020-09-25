## Signing Up

### Android

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

**Note: Calling the checkInvitationMethod is optional since _signup_ requires the invitation code to be passed. The _checkInvitationCode_ method is useful to build interfaces that use a two step signup process with the first step being checking if the user has a valid invitation code and depending on the type of invitation code checking if Driver's License Validation is required; then asking the user to enter their username and password. The string from _getCustomerName()_ can identify the customer the user is registering to.**

If Driver's License Validation is required, then after the signup call, DLV can be validated using the following API call

```java
	DLV.initialize(sdk); // sdk is an instance of Authoriti class
	startActivityForResult(new Intent(Activity.this, DocumentUploadActivity.class), DLV_REQUEST_CODE);
```

The callback `onActivityResult` of `Activity` will recieve the driver's license validation result. Here's an example

```java
	@Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == DLV_REQUEST_CODE && resultCode == Activity.RESULT_OK) {
            if (data != null) {
                ProcessedData processedData = data.getParcelableExtra(DLV.EXTRA_PROCESSED_DATA);
                if (processedData != null) {
                    new AlertDialog.Builder(this)
                            .setMessage(processedData.formattedString)
                            .setPositiveButton(android.R.string.ok, null)
                            .show();
                }
            }
        }
    }
```

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


### iOS

A valid Authoriti user must have an invitation code along with a valid username and password combination. The SDK provides methods for checking the validity of an invitation code, and onboarding a new user using the invitation code, username and password combination.

1. Check invitation code

- Call `check` method in `invitationUtility` and pass the invitation code as the first argument. The second argument is a callback that returns the result

```swift
sdk.invitationUtility.check(invite_code: "abcdefg", ((_ success: Bool) -> Void))
```

**Note: Calling the checkInvitationMethod is optional since _signup_ requires the invitation code to be passed. The _checkInvitationCode_ method is useful to build interfaces that use a two step signup process with the first step being checking if the user has a valid invitation code and depending on the type of invitation code checking if Driver's License Validation is required; then asking the user to enter their username and password. The string from _getCustomerName()_ can identify the customer the user is registering to.**

2. Signup user

- Call`signUp` method of `userUtility` with a `UserRequestModel` object to signup a new user

```swift
sdk.userUtility.signUp(mode: UserRequestModel(code: "abcdefg", username: "username", password: "pwd"), result: (model, error) -> void)

// result: callback to get the result of the signup API
```

- Call `listAccounts` method `userUtility` to get a list of accounts

```swift
var accounts = sdk.userUtility.listAccount()
```

After a successful signup call, the SDK initializes a Wallet object which holds the users account information. The contents of the wallet is securely stored on the device encrypted using the _Master Password_ - the _Master Password_ is the password used by the user during signup.
