# Authoriti Android SDK v0.1

**September 2020**

## Introduction

This document provides detailed information about the Authoriti Android SDK.

The Authoriti registration flow is described below.

![](https://authoriti-sdk.s3.amazonaws.com/images/registration-workflow.jpg)

The Authoriti Permission Code validation flow is described below.

![](https://authoriti-sdk.s3.amazonaws.com/images/validation-workflow.jpg)

The Authoriti SDK work flows are described below

![](https://authoriti-sdk.s3.amazonaws.com/images/flowchart.jpg)

testing 
<iframe width="816" height="1000" src="https://miro.com/app/embed/o9J_klx5LPY=/?" frameborder="0" scrolling="no" allowfullscreen></iframe>
## Prerequisites

- Supports Android SDK version 21 and above

### Setup

1. Specify the permissions in the App manifest file: [TODO]

2. Add the Authoriti SDK dependency in **build.gradle** [TODO]

### Initializing the SDK

To use the SDK, it must be initialized by passing the license key and the base URL.

```java
Authoriti authoritiSDK = new Authoriti("authoriti-xxxxxxxxxx", "https://api.authoriti.net")
```

All SDK methods should be invoked on an instance of the Authoriti class.

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

### Wallet Management

1. Exporting a Wallet: A Qr code for the users wallet protected by the master password can be generated by using the following method.

```java
String qrCode = WalletManagement.exportWallet(WebView webview, OnWalletExportedCallback callback);
```

This method requires an instance of `WebView` - the `WebView` should be attached to the current `Activity` - to be passed in as the first argument.

The `exportWallet` method must not be run in the UI Thread, and the application is responsible for displaying the Qr code using an appropriate Qr code viewer.

- Get the result by implementing the following callback

```java
WalletManagement.OnWalletExportCallback {
	void exportComplete(ErrorResponse error, String qrCode);
}
```

2. Importing a Wallet: A Qr code can be used to reconstruct a users wallet by using the following method

- The importWallet method requires an instance of `WebView` - the `WebView` should be attached to the current `Activity` - to be passed in as the first argument. The second argument should be the Base 64 encoded Qr code, and the third argument should be a user provided password that checks the integrity of the Qr code.
-

```java
	WalletManagement.importWallet(WebView webview, String qrCode, String password, WalletManagement.OnWalletImportedCallback callback);
```

- Get the result by implementing the following interface

```java
WalletManagement.OnWalletImportedCallback {
	void importComplete(ErrorResponse error, String message)
}
```

**Note: This action if successful will delete the current Wallet of the user**

3. Deleting a Wallet: The users Wallet can be wiped by invoking the following method

```java
WalletManagement.deleteWallet(String password, (error, message) -> {
	if (error) {
		// Show error.getErrorMessage();
	} else {
		// Show message to confirm wallet has been successfully imported
	}
});
```

4. Listing accounts: The users `Account`'s can be fetched by calling the following method

```java
List<Account> accounts = WalletManagement.listAccounts();
```

5. Deleting an individual account: An Account can be deleted by calling the following method

```java
WalletManagement.deleteAccount(Account account);
```

6. Synchronizing the users Wallet with server: The `syncWallet` method call checks the server for updates to the users Wallet and updates the Wallet as necessary

```java
WalletManagement.sync(WalletManagement.OnSyncCompletedCallback callback);
```

7. Changing the master password: The `changeMasterPassword` takes the user's current password and updates it a new password

```java
boolean success = WalletManagement.changeMasterPassword(String oldPassword, String newPassword);
```

### Purposes and Permission Code Generation

![](https://authoriti-sdk.s3.amazonaws.com/images/permissioncode.png)

A Permission Code is a 10 digit code that encrypts certain authorizations (Purpose), and also digitally signs the authorizations - meaning if the Permission Code is validated by Authoriti servers, the Permission Code

- Must have been generated by the user whose public key validated the Permission Code, and
- The user must have given the authorizations encrypted in the Permission Code, and
- No one else could have generated this Permission Code with the set of Authorizations for which it was validated

1. Purpose: Purposes are customer specific and the invitation code(s) a user uses determines the purposes they can generate Permission Codes for. The SDK provides methods for getting purposes and the _Schema_ of each purpose. A customer may add or remove purposes tied to an invitation code. Thus the list of purposes stored in the system must be refreshed. The SDK provides a method for refreshing the list of purposes, and it is the responsibility of the developer to call the refresh method periodically.

- Syncing Purposes: The following method downloads the list of purposes the user has access to

```java
sdk.syncPurposes(Purpose.OnPurposesFetchedCallback callback)

// To get the results implement the following interface
// interface OnPurposesFetched {
//	void onPurposesFetched(ErrorResponse error, List<Purpose> purposes);
// }
```

- Getting Purposes: The following method returns the list of purposes stored on the device

```java
List<PurposeData> purposes = sdk.getPurposes();

// purposeData.getGroup() returns the group to which the purpose belongs to
// purposeData.getPresetPicker() is null if no picker was set by the server. If it's not null, then the picker returned by getPresetPicker() should default to purposeData.getPresetPickerValue()
```

The group to which a purpose belongs may be fetched by calling the method `getGroup()` on an instance of Purpose Data.

- Getting schema from a purpose: The schema defines the relevant UI and the relevant data of a purpose and is used to generate Permission Codes for the purpose. The SDK provides methods to fetch schema from instances of `PurposeData`. It is up to the implementation to decide the UI which an app user would use to enter the required data.

```java
List<Schema> schema = purposeData.getSchema();
```

Each schema object represents a UI element which must be populated with data to generate a Permission Code. Each schema element provides the following methods to help with rendering the input form.

```java
Picker picker = schema.getInputPicker();
// possible Picker types are

// Dropdown pickers. if picker type is anyone of the following, these UI displayed should be a dropdown
// Picker.ACCOUNT_ID: A dropdown with pre-populated list of user accounts. User accounts should be fetched with sdk.listAccounts()
// Picker.INDUSTRY, Picker.LOCATION_COUNTRY, Picker.LOCATION_STATE, Picker.GEO, Picker.REQUESTOR: Dropdown values should be populated with picker.getValues()
// Picker.DATA_TYPE: Dropdown values should be populated with picker.getDataTypes(requestorId) where requestorId is the value of the Picker.REQUESTOR selected item.

// Time picker: if picker type is Picker.TIME, the UI element should be an appropriate time picker

// Input picker: if picker type is Picker.INPUT, the UI element should be an input field

// A picker should only be rendered if picker.renderUI() is true.
// A picker's label should be picker.getlabel()
// A picker's input hint should be picker.getTitle()

```

- Check Permission Code Request: The following method will check the server for pending Permission Code requests. If a request is available, the callback will return a `List<Schema>` object, with instances of the Schema object populated with a `getRequestValue()` which signifies the value that the input field must contain.

```java
sdk.checkPermissionCodeRequest(PermissionCode.OnPermissionCodeRequestChecked callback);
```

To get the Permission Code request, the following interface should be implemented

```java
interface OnPermissionCodeRequestChecked {
	void onPermissionCodeRequestRecieved(ErrorResponse error, List<Schema> schema)
}

// schema will be null if there's no pending Permission Code request for the user.
// The implementation details of polling such as frequency of the check requests is left upto the developer
```

- Generate Permission Code: To generate a Permission Code, the following method should be called with a `PurposeData ` object as the first argument - signifying the purpose for which the Permission Code is being generated, a `List<Schema>` object with each `Schema` element populated with user input data by calling `setUserInput(String)` on its instance.

```java
List<Schema> schema = purposeData.getSchema();
for (Schema s: schema)) {
	s.setUserInput(userInput); // userInput is the user entered data for this scheme
}
String permissionCode = sdk.generatePermissionCode(purposeData, schema);
```

- Pushing permission code to the server: Permission Code is sent to the Customer by the User by a method of their choosing. However, a Permission Code generated in response to a Permission Code request or over a phone call, can be sent to the server using the following method

```java
sdk.sendPermissionCode(Account account, String permissionCode, PermissionCode.OnPermissionCodeSentCallback listener);
```
