# Algorithm Confusion Attacks --

**Caused by : Improper JWT Library implementation**	

**Concept** :  This attack exploits assumptions that developers make; that a particular algorithm will be used to verify the signature of a JWT.  We can force a server that handles JWT tokens improperly to verify the signature of a token using another unintended algorithm.
				
Understanding the types of Keys :

 > Symmetric :  A single key is used to sign and verify the token. The key needs to be kept secret and not should not be guessable/ brute-forcable. 
   Example of key algorithm : HS256
						
> Asymmetric : two different keys are used to sign and verify the signature. A private key kept securly on a server signs the token and a public key that anyone has acces to, verifies the token. 
  Example of key algorithm : RS256

	
The flawed implementation of JWT libraries  by developers is what causes algorithm confusion attacks to be possible. The verification process differs according to the algorithm used ... but the process of deciding which algorithm to use to verify the signatrue relies on reading the value of the key _alg_. Below is a code snippet of a JWT library that is  used to decide the algorithm to be  used. 
				
				
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
function verify(token, secretOrPublicKey){
    algorithm = token.getAlgHeader();
    if(algorithm == "RS256"){
        // Use the provided key as an RSA public key
    } else if (algorithm == "HS256"){
        // Use the provided key as an HMAC secret key
    }
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

				
The assumption made,  is that an asymetric algorithm like RS256 will be used to handle the signing of a JWT resulting in a fixed public key always being sent to the code method (hardcoded public key) . If the algorithm in the key _alg_ is changed to HS256, then the function will treat the hardcoded public key as a HMAC secret key. What results is, we can sign a token using  HS256 and the discovered public key and the server will make use of the same public key in order to verify the signature. There are different ways in which the public key can be obtained. 

> checking standard endpoints such as /jwks.json or /.well-known/jwks.json


> using a pair of valid JWTs to extract the public key


**Challenges with this attack** : 

The public key used needs to be completely identical to the public key that is stored on the server, from its format to any non-printing characters (e.g new lines). As a result, some experimentation is required to get the right formatting in order to succed wth the attack 


Execution of this Attack Vector
						
- Requirements 
   + servers public key
					
- Steps	

 1. The server may expose the public key but it will still use the hardcoded public key - this may be stored in a different format. The public exposed and the one hardcoded need to be an exact match in order for this attack to work (every single byte and spaces) 
								
Sometimes the key may need to be converted into a different format such as PEM. Use Burp, in the JWT Editor Keys tab (needs JWT Editor to be installed) 
										
 - Create "New RSA key"
 - Paste the exposed public key taking care not to copy characters outside the keys array
 - Select “PEM” option 
 - Copy the PEM key. 
 - In Decoder, Base64-encode the PEM 
 - In the JWT Editor Keys tab, choose "New Symmetric Key" 
 - Generate a new key (it will be in JWK format )
 - the key value of k needs to be replaced with the base64-encoded PEM KEY  
 - Save the key								
											

3. Change the key alg into HS256 and make any needed changes to the token & request .
							
4. Sign the token using the generated symmetric key. 
											
											
					
					
### Getting a Public Key from existing tokens 
					
Using a pair of existing JWTs it may be possible to extract the used key in order to test for an algorithm confusion attack.
					
 Tools : jwt_forgery.py 
       : docker run --rm -it portswigger/sig2n <token 1>  <token 2>
								 
	|__ jwt_forgery.py available on https://github.com/silentsignal/rsa_sign2n
	|__ calculates one or more potential values of key n (parameter seen when looking at certain headers in tokens ) 
	|__ Output of the tools is 
	 - a PEM key that is base64-encoded that is in X.509 and PKCS1 formats
	 - a forged JWT signed using each of the keys. 
						
The correct key can be identified by sending a request that contains each of the forget JWTs. Only one of them will be accepted by th server. 
The key that is found can then be used to carry out an algorithm confusion attack. 

   Steps 
					
   1. Extracting the public key from existing tokens 
    > Log into two different accounts or log out and log back in. 
    > Obtain the JWTs of each of the logins and run them through the tools to generate forged JWTs
    > Copy the generated JWT ffrom the tools output and enter it into a request caputred by Burp. This request needs to be the original because if the token is invalid, we should be redirected to a login page (302 response). If its valid, we should proceed as normal to the requested page (200 response). 
								
--the steps below assume that the key is in PEM format--
								
   - For the correct forget JWT, copy its base64-encode X.509 key
   - In the JWT Editor Keys tab, choose "New Symmetric Key" 
   - Generate a new key (it will be in JWK format )
   - The key value of k needs to be replaced with the base64-encoded PEM KEY that is found by the tools  
   - Save the key	
						
						
 3. Change the key alg into HS256 and make any needed changes to the token & request .
						
 4. Sign the token using the generated symmetric key. 
