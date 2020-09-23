### Initializing the SDK

To use the SDK, it must be initialized by passing the license key and the base URL.

Android:

```java
Authoriti authoritiSDK = new Authoriti("authoriti-xxxxxxxxxx", "https://api.authoriti.net")
```

iOS:

```Swift
    sdk = try? AuthoritiSDK(licenseKey: "authoriti-xxxxxxxxxx", endpoint: "https://api.authoriti.net")
```

All SDK methods should be invoked on an instance of the Authoriti class.
