# Certisfy Claim Verification And Integration

Certisfy ([https://certisfy.com](https://certisfy.com)) is a trust projection and information verification toolkit. 
It can be used to meet a vast class of trust related needs for online use...even offline use.

This verifier is for use by developers to verify Certisfy claims within apps & services that use Certisfy for information verification. 
The verifier has no other dependencies except crypto functionality via pkijs ([https://pkijs.org/](https://pkijs.org/)) and associated utilities, the required dependencies are already included.

The code required for verification is in [certisfy-js](https://github.com/CipherTrustee/certisfy-js), the required module import and setup is below:

```javascript
   import * as certisfy from "./js/certisfy/certisfy.js" 
   certisfy.SET_SDK_MODE(true); 
   certisfy.loadTrustRoots();
```

### The verifier test console

The verifier test console app can be accessed here: [https://certisfy.com/verifier/](https://certisfy.com/verifier/).

The point of the console app is primarily to demonstrate claim verification independent of the Certisfy app ([https://certisfy.com/app/](https://certisfy.com/app/)). 

The way to use it is to generate claims from the Certisfy app, then use the verifier to verify the claims.

*You must attach the trust chain when creating your claim in the Certisfy app in order to be able to verify via the verifier.*

Developers can use the console app for basic testing of remote claim verification, it uses [certisfy-js](https://github.com/CipherTrustee/certisfy-js).

Currently a verifier is available only for Javascript.


### Verification API

The verifier exposes the following API functions to faciliate claim verification.

1. `verifyClaim(claim,receiverId,useAttachedTrustChain)`

    This function performs verification of claims, it is the function called when you click the verify button
    in Certisfy and in the verifier console app.

    **Arguments**

    - `claim`\
      Claim from Certisfy, can take either text or json object 

    - `receiverId`\
      The receiver id (ie persona) for which the claim should have been generated. 
      Note, if you ommit or pass `null` for `receiverId`, it is assumed `receiverId` match is irrelevant and thus the corresponding verification flag (`pkiIdentityReceiverMatch`) for receiver match is set to true. 
      
      Not the most intuitive logic but it is, what it is. 
      Bottom line, this should not be an issue since you should always verify a claim against a particular receiver id. 
      `receiverId` can be text or it can be an array of receiver ids that you want the claim verified against.

    - `useAttachedTrustChain`\
      Set this to true so the verifier will accept trust chains attached to claims. 
      You can also just pass a trust chain that should be used. If not set or set to false, this verifier will fail because of a lack of access to the trust chain. 
      
      Within the Certisfy app, attached trust chains can be ignored since the Certisfy certificate registry can be consulted for trust chains, ie if the full trust chain is in fact in the registry.

    **Usage** 
    
    ```javascript
    const verification = await certisfy.verifyClaim(claim,receiverId,claim.trustChain || true);
    ```
	
    The `verification` object represents the result of the verification. Review the console to see what it looks
    like and also take a look at the markup of the console app to see how this verification object maps to 
    verification result presented in the UI.
    
    This object is rather opaque, see `getVerificationResult` below for a way to get a simplier result object.
    
2. `verifyDHExchangeClaim(userCode,alicePrivateKey,useDHExchange,receiverId,useAttachedTrustChain)`

    This function performs verification of claims in similar manner to `verifyClaim`, in fact it uses `verifyClaim`.
    In stead of receiving claim object directly, this function fetches an encrypted claim from the Certisfy API and decrypts
    it before verifying it.

    **Arguments**

    - `userCode`\
      This is the short code that will be used to fetch the encrypted claim. 
            
    - `alicePrivateKey`\
      This is the private key of Alice's part of a diffie-hellman key exchange.
      
    - `useDHExchange`\
      This is optional, if you already have a DH exchange object previously fetched via a call to `getDHExchange`, you can provide it.

    - `receiverId`\
      Same as for `verifyClaim`.

    - `useAttachedTrustChain`\
	  Same as for `verifyClaim`.
      
    **Usage** 
    
    ```javascript
    const {verification,claim,error} = await certisfy.verifyDHExchangeClaim(userCode,dhExchange,alicePrivateKey,receiverId,claim.trustChain || true);
    ```
	
    The `verification` same as for `verifyClaim`. The `claim` is the decrypted claim extracted via the DH exchange.
    
    DH claim exchange is intended to help mitigate the risk of a person getting a claim from someone else and
    passing it off as their own, contrary to the claim creator's expectation. 
    
    DH Exchange ensures that at least if there is data in the claim that is intended only for the expected recipient of a claim then an intermediary
    cannot get their hands on it. DH exchange doesn't however eliminate the problem of a transparent intermediary
    who may be trying to fraudulently facilitate the transmission of a claim, but it is an important capability that can be useful
    in other mitigation strategies.
    
    It should be considered best practice to accept claims only through DH exchange.
    
    **Note** that DH exchange is facilitated via the PKI API and the API requires a subscription.
    See DH exchange API [documentation](https://cipheredtrust.com/doc/#trp-dhx-api).

3. `initiateDHExchange(receiverId,useKeyPair)`

    This fuction will initiate a DH exchange playing the role of *Alice*. 
    
    **Arguments**

    - `receiverId`\
      The receiver id. 

    - `useKeyPair`\
      This is an optional ECDSA P-256 key pair, if not specified a key pair will be generated. 
      This object must take the form `{publicKeyBase64,privateKeyBase64}`.

    **Usage** 
    
    ```javascript
    const {status,message,dhExchange} = await certisfy.initiateDHExchange(receiverId);
    const {create_date,user_code,private_key} = dhExchange;
    ```    
    `user_code` is lookup code to be used for executing/completing exchange. `private_key` is base64
    encoded.
    
4. `verifyVouches(claim,verification)`

    This function is used to verify any vouching associated with a claim. 
    Vouching allows additional claims to be attached to a main claim as vouched-for claims. 
    For instance someone credible can vouch that you attended a hackathon and you are a great dev, 
    you can then attach such a vouch to a claim that you present to others.

    **Arguments**

    - `claim`\
      Claim from Certisfy 

    - `verification`\
      This is the verification object returned from the call to `verifyClaim`. 
      This contains details from the verification process. 
      You can look at the console app UI code [index.html](https://github.com/CipherTrustee/certisfy-verifier/blob/41d105d44af4652f427ed5967b295340c891c3dd/index.html) to see how it maps to verification UI result.

    **Usage** 
    
    This is optional, if there are vouched-for claims, verify them. 
    First ensure presenting claim is valid then you can care about any embedded vouched-for claims.
    
    ```javascript
    if(certisfy.isClaimTrustworthy(verification)) 
       await certisfy.verifyVouches(claim,verification);
    ```

	`verifyVouches` modifies the `verification` object to integrate vouch verification results.
    
5. `getVerificationResult(verification)`

    This function transforms the `verification` object into something easier to use. 
    The resulting object maps to the intuitive UI presentation. 
    In the verifier console you can see this object attached as `verificationResult`.

    **Arguments**

    - `verification`\
      This is the `verification` object that results from calling `verifyClaim`.

    **Usage** 
    
    ```javascript
    const verificationResult = certisfy.getVerificationResult(verification);
    ```
	
    The `verificationResult` object represents a simplier and more intuitive object that shows the result of verification.
    Review the console results to see what it looks like.
    
    Below is an example showing the structure of the resulting object.
    
    [verificationResult.json](https://github.com/CipherTrustee/certisfy-verifier/blob/6c0708eb1fc03b16e4ab7d2c7dd366218e508cd7/verificationResult.json)
    
6. `isClaimTrustworthy(verification)`

    This function determines whether a verification can be considered trustworthy. The determination involves
    confirming cryptographic signature integrity and certificate validity. The function makes the determination
    strictly based on the result of verification, ie it doesn't do any additional verification itself.
    
    Review the implementatation code to see what the exact conditions are for trustworthiness determination.
    
	https://github.com/CipherTrustee/certisfy-verifier/blob/877cfacff228724a117c87e6cd57ab886c32405d/js/certisfy/certisfy.js#L2861

    **Arguments**

    - `verification`\
      This is the `verification` object that results from calling `verifyClaim`.

    **Usage** 
    
    ```javascript
    const isTrustworthy = certisfy.isClaimTrustworthy(verification);
    ```
	
    The result is `true` or `false`.    