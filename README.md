# Certisfy Claim Verification And Integration

This verifier reference implementation is a complement to the [signer](https://github.com/CipherTrustee/certisfy-signer).

The code required for verifying is in [certisfy-js](https://github.com/CipherTrustee/certisfy-js), the required module import and setup is below:

```javascript
   import {createSDK} from "./js/certisfy/src/loader.js" 
   const certisfySDK = await createSDK();
   const {verifier} = certisfySDK;
```

### The verifier test console

The verifier test console app can be accessed here: [https://certisfy.com/verifier/](https://certisfy.com/verifier/).

The point of the console app is primarily to demonstrate claim verification independent of the Certisfy app ([https://certisfy.com/app/](https://certisfy.com/app/)). 

The way to use it is to generate claims from the Certisfy app, then use the verifier to verify the claims.

*You must attach the trust chain when creating your claim in the Certisfy app in order to be able to verify via the verifier.*

Developers can use the console app for basic testing of remote claim verification, it uses [certisfy-js](https://github.com/CipherTrustee/certisfy-js).

Currently a verifier is available only for Javascript.

The verifier test console implements this [API](https://github.com/CipherTrustee/certisfy-js/tree/master/docs/verifier) to faciliate claim verification.
