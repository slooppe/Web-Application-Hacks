* [What is JWT](#JWT)

## .JWT Vulnerabilities

* [Using None Algorithm](#None)
* [HMAC Algorithm Attack](#HMAC-V)
* [Weak Secret Key](#Secret-w)
* [Lack Of Verification](#No-Verify)
* [Directory Traversal in Kid](#kid-1)
* [Sql Injection in Kid](#kid-2)
* [JWKS Spofing](#JWK-1)
* [TOOLS And Refernces](#Tools-Refernces)







## JWT

```
A JSON Web Token (JWT) is an open standard (RFC 7519) that used for securely transmitting information between parties as a JSON object. 
This information can be verified and trusted because it is digitally signed.
JWTs can be signed using a secret or a public/private key pair

````
### JSON Web Token Structure:

```
Json Web Token consist of :
   Header
   Payload
   Signature
  

Header : - In this part algorithm is defined . which alogorithm  is using to generate the token like:
  
  {
   "alg": "RS256",
   "typ": "JWT"
}

Payload : - User information that is anything like username , email etc.

{ "user": "effortlessdevsec"

}
  
Signature: -  Signature is an random string that is created using header payload and secret   

JWT follow the following pattern:

Base64(Header).Base64(payload).Base64(Signature)

Signature = sign (Base64(Header).Base64(payload),key)

For Example: 
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiZWZmb3J0bGVzc2RldnNlYyJ9.q1gLIkSPBFAKPxe_Bj_Lhmwjsxjxuu0-mX-E9XgeYnA
             (Header)                      (Payload)                           signature


Multiple signature methods can be used to ensure the integrity of JWT:

      RSA based  -- Private key for sign and public key for verify token
       Elliptic curves
        HMAC   --  Same Secret key is used to sign and verify token
        None    --  No signature required
```
##
![JWT ](https://github.com/effortlessdevsec/Web-Application-Hacks/blob/master/Images/jwt.png)

### None
```
Like SSL (with the NULL Cipher), JWT support a None algorithm for signature. This was probably introduced to debug applications.In None algorithm
no any signature required .

Nowdays None algorithm is disbaled  by default , developer use none algorithm for debugging purpose, if developer forget to disable None algorithm .
then  the token not required  signature any more:

-----------------------------------------------------------Hack Steps --------------------------------------------------------------------------

                            1. Decode the JWT 
                            2.Change the Algorithm to None
                            3.Remove the signature 
                            4.do   Base64(Header).base64(Payload).
                            5. example : eyJhbGciOiJOb25lIiwidHlwIjoiSldTIn0.eyJsb2dpbiI6ImFtaXQiLCJpYXQiOiIxNTk2MTc3MDkzIn0.
                            6. If token worked properly . :_None algorithm supported , now proceed with furhter attacks.

```

### HMAC-V

```
   If application  uses RSA algorithm and also HMAC supported then an attacker can bypass the Signature verfication and can proceed their attack:
   
   LETS DIG INTO THIS: lets process all scenerio :
   
   you know 
   : RSA based algorithm uses private key for sign the token and public key for verify the token
   : Hmac based algorithm used same secret key for sign and verify the token
   
   WHAT THE PROBLEM:
   
   IF application used RSA And Also HMAC Supported that the porblem: why?
   
   When application uses RSA. You can even download the public key if you want to be able to verify the token. Since you don't have the private key, 
   you cannot generate a valid token (in theory). In a real scenario, you may get to get the public key from a JavaScript script or from a mobile application.

  ~ you can change the algorithm used by the application (RSA - RS256) to tell it to use HMAC (HS256).
  ~ The application will call the method verify when you send the cookie. Since the code is written to use RSA, it will call verify(public_key, data).
  ~ But since the algorithm is set to HMAC, it will end up calling HMAC(public_key,data).
  ~The application will verify the signature with the public key but since you are forcing the application to use HMAC, it will actually verify the signature with    HMAC(public_key, data).
  
  -----------------------------------------------------------Hack Steps --------------------------------------------------------------------------
                     1. decode the JWT
                     2. change the algorithm used for the signature
                     3. compute the new signature with public key
                     4. verify the token
                     
```

### Secret-w

```
The issue here is very simple. The integrity of the token relies on the strength of the secret used to sign the token.
 If an attacker can find the secret, she/he will be able to forge her own malicious token
 
 Hashcat is powerful tool for bruteforcing the secret:
 
 USAGES : hashcat  -a 3 -m 16500 -o abc.txt   target.txt  dict.txt
 
```
![Hashcat ](https://github.com/effortlessdevsec/Web-Application-Hacks/blob/master/Images/hashcaht.png)

### No-Verify
```
Sometimes application does not verify the signature ,so whatever signature is not matter..

  -----------------------------------------------------------Hack Steps --------------------------------------------------------------------------
                    1. Decode The Jwt
                    2. Modify The Jwt.
                    3. Leave the signature as it is.
                    4. Verify the token

```

### kid-1
```
This Vulnerability comes from one of the fields in the header: kid.

kid is an optional header claim which holds a key identifier, particularly useful when you have multiple keys to sign the tokens 
and you need to look up the right one to verify the signature,If  kid is used without proper escaping to retrieve the key.this vulnerability arises when
developer kid use as a file path.

  -----------------------------------------------------------Hack Steps --------------------------------------------------------------------------
                        1. Decode the Jwt
                        2. change the kid value with known pathname like for example 
                                 {
                                   "typ": "JWT",
                                  "alg": "HS256",
                                  "kid": "css/bootstrap.css"
                                      }
                         3. create signature with css/bootstrap.css content
                         4. verify the token 

```
### JWK-1

```
JWT allows users to link to a public key (using the jku header) inside the header of the token. 
JKU stands for JWK Set URL
JKU links to a json web keys (JWK). when developer not verifying the jku url then an attacker can create a forged token and proceed further .

An attacker can  generates a public-private key pair and creates a forged token using the generated private key and replace the “jku” parameter’s value 
with the URI of this newly generated JWK Set JSON file (hosted on an HTTP server),

  -----------------------------------------------------------Hack Steps --------------------------------------------------------------------------
                      
                    1. create  a public-private key pair:
                        
                        openssl genrsa -out keypair.pem 2048
                        openssl rsa -in keypair.pem -pubout -out publickey.crt
                        openssl pkcs8 -topk8 -inform PEM -outform PEM -nocrypt -in keypair.pem -out pkcs8.key
                        
                     2. Extract n and e value from public key and put n and e value in jwks.json file and host the  file on a server 
                     3 Set the jwks.json file url in jku header
                     3.create a forged token using the public-private key pair
                     4.Verify the token
                        
  
                        

```

### Tools-Refernces

```
https://jwt.io/
https://github.com/ticarpi/jwt_tool
https://www.pingidentity.com/en/company/blog/posts/2019/jwt-security-nobody-talks-about.html
BURP EXTENSION : JOSEPH

```
