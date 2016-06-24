## JSON Web Tokens


This is a [middleware](https://github.com/iris-contrib/middleware/jwt).


[JWT.io](https://jwt.io) has a great [introduction](https://jwt.io/introduction/) to JSON Web Tokens.

In short, it's a signed JSON object that does something useful (for example, authentication). It's commonly used for Bearer tokens in Oauth 2. A token is made of three parts, separated by .'s. The first two parts are JSON objects, that have been base64url encoded. The last part is the signature, encoded the same way.

The first part is called the header. It contains the necessary information for verifying the last part, the signature. For example, which encryption method was used for signing and what key was used.

The part in the middle is the interesting bit. It's called the Claims and contains the actual stuff you care about. Refer to the RFC for information about reserved keys and the proper way to add your own.
