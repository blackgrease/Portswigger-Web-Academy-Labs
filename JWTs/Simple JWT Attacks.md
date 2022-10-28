# > Accepting modified signatures
			
### Caused by : Improper implementation
			
	In JWT libraries, there is a method to decode a token 
    - decode() : only performs decoding ofthe base64url encoded token without verifying its signature
    
    and a method to verify the token 
    
    - verify() : will decode the token along with verify the signature. 
    
    The functions are dependent on the langauge used. Some developers confuse the two  workings of the two methods which means that if the decode() is used, the token passed, does not have its signature verified at all and taken as it is. 
			
			What this looks like : changing the JWT token context to a different user such as admin and accessing a different page. Different users can be discovered via different enumeration techniques and endpoints can be discovered by various content discovery tools such as ffuf.  
					|__ why it works : the token has been modified but its authenticity is not checked by the server due to the code that it is passed to.  
					
			Remidiation : Make sure you understand and use the proper code methods
				
				
				
				
# Accepting tokens that have no signatures
			
## Caused by : Improper implementation
			
	A JWT header usually contains the key alg that informs the server of the type of algorithm that was used to sign the token. This will help the server know what algorithm it needs to use when the signature is verified. This flawed because due to the server not keeping any information about the token, it needs to trust the value of the key parameter - which can be influenced by me. 
			
	Various algorithms can be used to sign a token. A token that is not signed is known as a unsecured JWT. This is done by use none as the value for the key alg. A properly implemented server using JSON will reject a token that is unsecured. This mechanism relies on string parsing which can be bypassed using different obfuscation techniques such as mixing capitals along with unexpected encodings. 
				
			 An unsigned token still needs to follow the syntax, the payload section needs to have a trailing dot. 
				
			What this looks like : sending a request that is not signed in order to access parts of an application that would normally be unathorised
						|__ _why it work_s : the server has been misconfigured to accept unsigned tokens. 
						
			Steps : 
						1. Verify your own account can be be reached if a non-signed token is used. 
            2. If it does work, abuse it and try and to access another user or endpoint that would have normally been unathorized. 
						
					
			Remidiation: Reject unsigned tokens 
					
					
					
					
					
# Brute-forcing Secret Keys 
			
## Caused by : Human Error and bad practices 
			

There are some algorithms that use a standalone key as the secret key. Similarly to a password, it is important that an attacker is unable to guess / bruteforce the secret. An example is HS256 (HMAC + SHA-256). If I can guess the secret , then it is possible to create my own JWTs with any header / payload values and re-sign the token with a valid key. 
			
This vector arises due to developers forgetting to change default or placeholder secrets during implentation of JWTs. Some times, code snippets are copy & pasted into an application and this is hardcoded; bruteforcing this with a list of well known secrets is easy. 
			
		Known secrets wordlist : https://github.com/wallarm/jwt-secrets/blob/master/jwt.secrets.list
			
		Tools : Hashcat 
					> hashcat -a 0  -m 16500 <jwt>  <wordlist>
								
		Requirements : a valid and signed JWT from the target server + wordlist. 
			
    Additional : 
			+ use --show if the command is run more than once. 
			+ Due to running locally, it is much faster as requests arent sent to a server. 
      + Once the secret key has been discovered, a valid signature can be created for any JWT header and payload. 
					
					
		STEPS 
			1. Add the cracked secret to a key 
					- based64 encode the found secret
	  			- generate a symmetric key and change the value in the key k with the encoded secret. 
							
			2. Sign the request in the JSON Web Token message editor.  
						
			3. Change to an arbitrary / unintedted functionality (e.g another users account)



