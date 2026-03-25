# Platofrm/Adapter/Post-Platform Authentication

## Problem statement
We need to align the authentication model and expirience of a post-platform with that of a person.
A method is required to allow a user "broswing" to the frontend of a platform to verify its authenticity;
a system that provides correlation (if not identical) between that identity and the ename/evault of the paltform;
a mechanism that allows platforms to prove to evaults they access their autenticity.
For all the above cases we want to support both one-time transactions and long lived "sessions".

## Details
Note: we will describe the use case of auth against person and auth against evault as separate,
but it is entirely possible that both can be implemented similarly.

### Platform -> Person
The user interacts with a platform by browsing to its frontend. Since we require secure connections,
the frontend already presents a certificate as part of TLS handshake.
We need to make sure that either the certificate or additional signed material is available 
to allow the end user to verify it deals with certified *and* expected platform.

If we feel that the verification can be too complicated for the casual user, we can add a verification screen to our eid app.

Note: If it is a mobile app, then the autenticity must be verified at the time of installation. 

### Platform -> eVault
In the prototype every request to an eVault comes with a token in the http header.
The code of eVault can traverse the chain of trust to verify that the token represents "some certified platform".
The expiration of the token is decided by issuing authority (Merul). There may be a revocation list.

A different flow is suggested.
The platform still presents a token in the http header to the eVault. But this token is unique and can only be used to interact with one particular eVault.
It is signed by eVault's _autonomous_ private key or by 3rd party auth provider this eVault trusts explicitly. The expiration is short and decided by eVault.
That token allows the eVault to know that it deals not just with a certified platform, but which platform it is exactly, including what is the eName of the platform.

For a platform to obtain such a token we suggest an auth endpoint on the evault to either handle the handshake or redirect to 3rd party auth provider it trusts.
In the handshake the platform provides its ename only. The eVault/auth provider uses the registry to resolve the eName and get the public key of the platform from platform's eVault.
They run a simple challenge and if the platform passed, it is presented with a token.
