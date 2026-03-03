Certisfy (https://certisfy.com) is a trust projection and information verification toolkit. It can be used to meet a vast class of trust related needs for online use...even offline use.

This verifier is for use by developers to verify Certisfy claims within apps that use Certisfy for information verification. The verifier has no other dependencies except crypto functionality via pkijs (https://pkijs.org/), the required dependencies are already included.

The actual code required for verification is in the /js/crypto folder, the required module import and setup is:
<script type="module"> import * as certisfyCrypto from "./js/crypto/pkijs-impl-module.js" window.certisfyCrypto = certisfyCrypto; window.certisfyCrypto.SET_SDK_MODE(true); window.certisfyCrypto.loadTrustRoots() </script>

The verifier console app can be accessed here: https://certisfy.com/verifier/ The point of the console app is primarily to demonstrate claim verification independent of the Certisfy app (https://certisfy.com/app/). The way to use it is to generate claims from the Certisfy app, then use the verifier to verify the claims.

You must attach the trust chain when creating your claim in the Certisfy app in order to be able to verify via the verifier.

Developers can use the console app for basic testing of remote claim verification, it uses the /js/crypto folder code.

Currently a verifier is available only for Javascript.

The verifier exposes three functions to faciliate claim verification.

verifyClaim(claim,receiverId,useAttachedTrustChain)

claim>claim from Certisfy receiverId>The receiver id (ie persona) from which claim should have been generated. Not if you pass null for receiverId, it is assumed receiverId match is irrelivant and thus the corresponding flag for receiver match is set to true. Not the most intuitive logic but that is it is. Bottom line, this should not be an issue since you should always verify a claim against a given receiver id. useAttachedTrustChain>Set this to true so the verifier will accept trust chains attached to claims. You can also just pass a trust chain that should be used. If not set or set to false, this verifier will fail because of a lack of access to the trust chain. Within the Certisfy app, attached trust cains can be ignored since the Certisfy certificate registry can be consulted for trust chains, ie if the full trust chain is in fact in the registry.

usage: const verification = await certisfyCrypto.verifyClaim(claim,receiverId,claim.trustChain || true);

verifyVouches(claim,verification): This function is used to verify any vouching associated with a claim. Vouching allows additional claims to be attached to a main claim as vouched-for claims. For instance someone vouching that you attended a hackathon, you can then attached such a vouch to claim as way of using it.

claim>claim from Certisfy verification>This is the verification object returned from call verifyClaim. This contains details from the verification process. You can look at the console app ui code (index.html) to see how it maps to verification UI result.

usage: //optional- if there are vouched-for claims, verify them. //first ensure presenting claim is valid then you can care about any associated vouching if(certisfyCrypto.isClaimTrustworthy(verification)) await certisfyCrypto.verifyVouches(claim,verification);

getVerificationResult(verification):This function transforms the verification object into something easier to use. The resulting object maps to the intuitive ui presentation. In the verifier console you can see this object attached as verificationResult.

