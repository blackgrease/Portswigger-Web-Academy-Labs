# JWT Header Parameter Injections for Self-signed JWTS--
			
The JWS spcification states that only the key alg is compulsory. There are many JWT headers that can be included in a JSON string. The below user controllable parameters are very intresting as potential attack vectors as they tell the server which key ti use when verifying the signature. 
		
	- jwk : JSON Web Key. is a embedded JSON object representing a key (nested)
	- jku : JSON Web Key set URL, provides a URL where a server can fetch a set of keys
	- kid : Key ID. When there are multiple keys in use, this key allows a server to identify the correct key. 
			  
	----other critical paramters
						
	- cty : Content Type. When a way tp bypass signature verification is found, it can be use to state a media type for the content in the JWT payload. This can introduce vectors for XXE and deserialization attacks
						
		- x5c : X.509 Certificate Chain : is sometimes used to pass the X.509 public key certificate or certificate chain of the key that was used to sign the JWT. It can be used to inject self-signed certificates. The complexity of the x.509 format along with its extentions means that vulnerabilities arise. 
			|__ More info :  https://talosintelligence.com/vulnerability_reports/TALOS-2017-0293 &&  https://mbechler.github.io/2018/01/20/Java-CVE-2018-2633
			
			
			
			
### Using the jwk paramter to inject self-signed JWTs
**Caused by** : _Server misconfiguration _
				
**Concept**
						
The JWS Specification allows a jwk header parameter that a server can use to embed its public key (usaully shared so that anyone can verify the tokens issued by the server)  directly into the token itself using the JWK format. 
				
A misconfigured server can use any key that is embedded within the jwk parameter by failing to check if it came from a trusted source. Exploiting this miconfiguration involves, signing a modified JWT using my own RSA Private key and then embedding the corresponding public key in the jwk header.  
					
	Steps
		- Generate a new RSA key
		- Modify the tokens payload as required
		- “Attack” > “Embedded JWK” & send to see how server responds. 
							
							
Redemidiation : The best practice is to have a whitelist of public keys that a server can use in order to verify JWT signatures. 		
							
							
							
							
							
							
### Using the jku paramter to inject self-signed JWTs
					
The jku key can be used to reference by URL a JWK Set that contains the keys. The relevant key is fetched by the server when verifying the signature.  JWK Sets can be exposed publicly at endpoints such as /.well-known/jwks.json 
					
Most websites that are secure, will only fetch keys from trusted domains but by taking advantage of URL parsing discrepanices, this filter can be bypassed. 
							
	- emedding credentials before the hostname using @
								
			- indicate a URL fragment using #
									
			- leverage the DNS naming heirarchy to place the required input into a fully qualifed DSN name that is under my control 
									
			- confuse the URL-parsing code by URL-encoding characters. A useful feature when the filter on there is discrepanices between front & backend code handling.
									
		  - all the techniques combined together 
									
						
	
What this attack looks like :  
	- Requirements : 
			+ a server
			+ a RSA Key
									
	1 - With a request that contains a JWT, have the arbitray parameters / URL set. 
	2 - Generate a new RSA key using the JWT Editor extension. 					 
	3 - On the server, have page set to a key set - [e.g https://mydomain/keys.json ] May have the below format
											
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

{
    "keys": [
        {
            "kty": "RSA",
            "e": "AQAB",
            "kid": "75d0ef47-af89-47a9-9061-7c02a610d5ab",
            "n": "o-yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9mk6GPM9gNN4Y_qTVX67WhsN3JvaFYw-fhvsWQ"
        },
        {
            "kty": "RSA",
            "e": "AQAB",
            "kid": "d8fDFo-fS9-faS14a9-ASf99sa-7c1Ad5abA",
            "n": "fc3f-yy1wpYmffgXBxhAUJzHql79gNNQ_cb33HocCuJolwDqmk6GPM4Y_qTVX67WhsN3JvaFYw-dfg6DH-asAScw"
        }
    ]
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

											
	4 - in the header of the JWT, modify the key kid ( key identifier) to mimic the key kid in the generated RSA key. 
				
  5 - Add a  key jku parameter into the header that points to my URL.  
							
	6 - Sign the token using my generated RSA key and send it 
							

Why it works : the server does not verrify that my domain is a trusted source. Due to this being a form of self - signing attack, I am signing the keys myself by forcing the server to use a my key that is hosted on my domain. 
					
					
**Remediation** : Have a white-list of trusted domains that a server can use in order 
					
					
					

## #Using the kid paramter to inject self-signed JWTs

**Caused by** : _combination of loose paramater constraints (specifications ) and 	developers actions _

**Concept**
					
Due to servers using cryptograohic keys to sign different types of data - not only JWTs - the kid, Key Identifier is used to aid the server in identifying which key to use when a signature is being verified. Verification keys are often stored as a JWK Set. As a result, the server will look for the JWK (JSON Web Key) that shares  the value of the kid with the token. The structure of the value of the key kid is not well defined in the JWS specification and therefore the value can be anything that the developer chooses; can even point to a file within the database or a file name.
					
	 > If the key kid points to a file, it can introduce vulnerabilities such as directory traversal. I can force the server to use a different file to verify the key. If a symmetric algorithm is used to sign JWTs, this dangerous due to the ability to point to a static file such as /dev/null will cause the token to be signed using a base64-encoded null byte meaning the signature will be valid. 
							
	> If the keys are stored in a database, it is possible the kid key becomes a vector for SQL Injection  
					
What this looks : 
						
	- Requirements :
			+ a generated symmetric key
			+ a valid JWT token
											
	- Exploiting 			
		- Generate a new symmetic key using the JWT Editor keys. 
		- Replace the value of the key k with a base64-encoded version of a null byte = (AA==) . 
		- Attempt a path traversal injection by replacing the key kid with /../../../../../dev/null
		- Change the necessary values of the token / URL and sign the token
		- Test the attack vector
								 
								
