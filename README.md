# Certisfy Claim Verification And Integration

Certisfy ([https://certisfy.com](https://certisfy.com)) is a trust projection and information verification toolkit. 
It can be used to meet a vast class of trust related needs for online use...even offline use.

This verifier is for use by developers to verify Certisfy claims within apps & services that use Certisfy for information verification. 
The verifier has no other dependencies except crypto functionality via pkijs ([https://pkijs.org/](https://pkijs.org/)) and associated utilities, the required dependencies are already included.

The code required for verification is in [certisfy-js](https://github.com/CipherTrustee/certisfy-js), the required module import and setup is below:

```javascript
   import {createSDK} from "./js/certisfy/src/loader.js" 
   const certisfySDK = await createSDK();
```

### The verifier test console

The verifier test console app can be accessed here: [https://certisfy.com/verifier/](https://certisfy.com/verifier/).

The point of the console app is primarily to demonstrate claim verification independent of the Certisfy app ([https://certisfy.com/app/](https://certisfy.com/app/)). 

The way to use it is to generate claims from the Certisfy app, then use the verifier to verify the claims.

*You must attach the trust chain when creating your claim in the Certisfy app in order to be able to verify via the verifier.*

Developers can use the console app for basic testing of remote claim verification, it uses [certisfy-js](https://github.com/CipherTrustee/certisfy-js).

Currently a verifier is available only for Javascript.

The verifier test console implements this [API](https://github.com/CipherTrustee/certisfy-js/tree/master/docs/verifier) to faciliate claim verification.
