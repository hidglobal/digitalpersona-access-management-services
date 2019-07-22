---
layout: default
title: Web Smart Card Support  
has_toc: false
nav_order: 8
---

{% include header.html %}

# Web Smart Card Support  
<BR>  

This topic describes describes the use of contacted PKI Smart Cards in DigitalPersona  v1.3 and above for WEB authentication, including the authentication and enrollment algorithms, workflows, implementation and data formats.  

## Overview  

Contacted PKI Smart Card is a powerful authentication token. The combination of two authentication factors (the card belonging to the user and a PIN known only to the user) along with using the card’s RSA private key (which never leaves the card), makes it a perfect, strong solution for Web authentication.

The format of database user record of enrolled Smart Card credentials is defined <A HREF="smart-card-data-format.md">here.</A>. This format has not changed since the early DigitalPersona (Altus) versions and still is used for the DigitalPersona (non-web) client Smart Card authentication and enrollment.  

The PKI SC WEB authentication token for DigitalPersona (Altus) v1.3 and above is built according to the requirements described in <A HREF="was.md">this topic</A>.  

## Authentication algorithm  

Unlike the commonly used PKI SSL protocol, which needs multiple transactions between the client and server to establish a secure WEB connection, the Altus SC authentication algorithm is built to use only one operation, the single call to the server. This remarkably simplifies the operation, especially in the multi-server environment, with impacting security.  \

The idea of Altus WEB SC (WASSC) authentication is as follows.  

1. On the client, where the SC is inserted into the reader, the AWEBSC client enumerates the asymmetric key pairs on the card and uses all the available private keys from the card to sign the nonce data.  

  - We use all the private keys from the card because in general the client has no information which key from the card was used for user enrollment:  

    PrK[]  = { PrK1 , … PrKn };  

    If the client has the information, which key pair must be used for authentication (for ex., the user has selected one of the card certificates in the Card Authentication UI page or one of the enrolled smart card tokens in the list of enrolled tokens), then only this selected key can be used in the following algorithm.  

    The data structure sent to the server is presented as an array, even if it contains only one key data:  

    PrK[]  = { PrK1 };  

  - As a nonce data, the current time is used in the UTC (8 bytes array) format:  

    Nonce = GetCurrentTime(UTC);  

  - For every private key, the hash of the public key paired to this private key is added to the nonce, and this combination is signed with this private key:  

    Hash = Hash(PuKi);  
    Sigi = Signature( PrKi )( Nonce + Hash );  

  - For every private key, the data sent to the server is a combination of the Nonce, the hash of the public key and the Signature:  

    AuthDatai = Nonce + Hash + Sigi;
  - The array of AuthData is sent to the server:
    AuthData[] = {AuthData1, …, AuthDatab}  

2. On the server:  

  - First, the timestamp of the received data is checked. The server checks if the Nonce provided by the client complies with the limitations in time difference between the server and the client, i.e.  

    If ( ( Nonce1 - GetCurrentTime(UTC) )  < AheadTimeThreshold ) AND
      ( GetCurrentTime(UTC)  - Nonce1 )  < BehindTimeThreshold ) )

      Unlike the time-based OTP solutions, we consider here that both cases: client time is behind the server and client time is ahead of the server are possible, so both thresholds are the same:  

      AheadTimeThreshold = BehindTimeThreshold = 3 min.  

      This is a default value, but it may be changed by the server’s settings.  

      All the Nonces in the AuthData array are the same, so only first Nonce is used for calculations.  

      This operation is done first, as it’s less resource consuming comparing to the cryptographic operations described below.  

      If the timestamp of the client data does not match the current server time, the authentication fails with “Out of time” error message.  

  - The user’s SC enrollment record is opened and the proper public key found on this record is used to verify the signatures sent by the client. We have to try all the public keys from the SC enrollment record against all the hashes sent by the client because multiple smart cards (i.e. multiple keys) are allowed to be enrolled for the user:  

    ~~~
    PuKs[]  = { PuKs1 , … PuKsm };
    For every Hashi in AuthData[]
    {
			For every PuKsj in PuKs[]
		    {
					If (Hashi = = Hash(PuKsj) )
					{
		     		Proceed authentication (i, j);
					 Exit;
					}
	 			}		
		}
    ~~~

    If the hash of one of the public keys stored in the user’s SC enrollment record matches one of the hashes sent by the client, then the authentication process is continued with this public key and this nonce/signature data. If no matches were found, the authentication fails with a “Not enough information to authenticate” error.  

  - The server verifies the signature of the data sent by the client, using the found public key from the user SC enrollment record and the found Nonce + Hash record from the authentication data sent by the client:  

    ~~~
If ( Sigi  = = VeryfySignature( PuKj )( Noncei + Hashi ) )
{
				Authentication succeeded!
}
  ~~~

    If the signature verification succeeds, the Jason authentication Web Token (JWT) is returned to the client, as described in “Altus Web AUTH Service.docx”. . If the signature verification fails, the “Access denied” error is returned.  

#### Authentication Implementation  

The Altus WEB SC (WASSC) authentication token is implemented according to < A HREF="was-cred-format.md">this credentials format</A>, to comply with the demands of the IDPWebAuth interface and Credential class. The Credential class is defined as follows.  

~~~
    [DataContract]
    public class Credential
    {
        [DataMember]
        public String id { get; set; }   // unique id (Guid) of credential
        [DataMember]
        public String data { get; set; } //  credential data
    }  
~~~

The following ID is defined for the Smart Card Credential.  

{D66CC98D-4153-4987-8EBE-FB46E848EA98}  

The following methods of the IDPWebAuth interface are supported by the WASSC authentication token.  

### WebAthenticate  

To call this method, the client should send IDPProWebAUTHMgr -> AuthenticateUser request to the server.  

The data for the Smart Card credential is a Base64url encoded UTF-8 representation of the JSON array of the DPJsonSCAuthToken classes. Every public key is represented by the CDPJsonSCAuthToken class.  

~~~
[DataContract]
    public class CDPJsonSCAuthToken
    {
        [DataMember]
        public Byte version { get; set; }   						// version
        [DataMember]
        public UInt64 timeStamp { get; set; }   						// current time
        [DataMember]
        public String keyHash { get; set; } 						// public key’s hash.
        [DataMember]
        public String signature { get; set; }						// signature
    }

~~~  

where:  

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Data Member</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">Byte <i>version</i></td>
  <td valign="top">Version of CDPJsonSCAuthToken object, must be set to 1 for the current implementation.   
	</td>
  </tr>
  <tr>
  <td valign="top">UInt64 <i>timeStamp</i></td>
  <td valign="top"> – Nonce, the UTC time of the moment when the object is created (64-bit value 	representing the number of 100-nanosecond intervals since January 1, 1601).</td>
  </tr>
  <tr>
  <td valign="top">String <i>keyHash</i></td>
  <td valign="top"> – Public key’s hash, Base64url UTF-8 encoded string.</td>
  </tr>
</table>

The public key from the Smart Card must be imported in the <A HREF=" http://msdn.microsoft.com/en-us/library/aa387459(VS.85).aspx>PUBLICKEYBLOB format</A>. After that, the <A HREF="https://en.wikipedia.org/wiki/Secure_Hash_Algorithm>RSA256 hash</A> of the key must be calculated. The resulting 32 bytes must be Base64url encoded to the string.  

~~~
KeyHash = Base64urlEncode(RSA256 Hash (PUBLICKEYBLOB (PuK))  
String signature – Timestamp and Public key’s hash, Base64url UTF-8 encoded string.  
~~~

The Timestamp (8 bytes) and Public key’s RSA256 Hash (32 bytes) must be combined into a 40 byte array, where the first 8 bytes is the Timestamp and the rest is the hash.  

This 40 byte blob must be hashed again with RSA256 and then signed with the Smart Card’s Private Key. The signature algorithm used is specified when the Smart Card key pair is originally created, usually it’s RSA.  

The resulting signature blob must be Base64url encoded into the string.  

~~~  
KeyHash = Base64urlEncode(Sign(PrK)( RSA256 Hash( Timestamp + RSA256 Hash (PUBLICKEYBLOB (PuK))))  
~~~

To create the Smart Card Credential for authentication, perform these steps on the client.  

1. Enumerate the asymmetric key pairs on the Smart Card or select the exact key pair to use.  

2. Create a JSON representation of the CDPJsonSCAuthToken class for every key pair from step #1.  

3. Combine the CDPJsonSCAuthToken classes into a JSON array, even if there’s only one DPJsonSCAuthToken.  

4. Base64url UTF-8 encode the representation of the string created in step #3.  

5. Finally, create a JSON representation of the Credential class using Smart Card Credential ID as the id member and the string created in step #4 as a data member.  

### WebIdentify  

The Smart Card token does not support user identification.  

### WebGetData  

This method returns the list of Smart Card credentials enrolled for the user. It can be used to select the Smart Card token for authentication or to delete with the WebDelete method. There’s no need to insert the Smart Card into the reader to call this method.  

To call this method, the client should send a  IDPProWebAUTHMgr -> GetEnrollmentData request to the server.  

As result of this call, a string presenting the Base64url encoded UTF-8 representation of the JSON array of the CDPJsonSCEnrolledToken classes will be returned.  
Every enrolled Smart Card credential is represented by the CDPJsonSCEnrolledToken class.  

~~~
[DataContract]
    public class CDPJsonSCEnrolledToken
    {
        [DataMember]
        public Byte version { get; set; }    						// version
        [DataMember]
        public UInt64 timeStamp { get; set; }    						// enrollment time
        [DataMember]
        public String keyHash { get; set; }  						// public key’s hash.
        [DataMember]
        public String nickname { get; set; } 						// token’s nickname
    }
~~~  

where:  

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Data Member</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
    <td valign="top"></td>
    <td valign="top">Byte version - version of CDPJsonSCEnrolledToken object, must be set to 1 for current implementation.   
  </td>
  </tr>
  <tr>
    <td valign="top">UInt64 <i>timeStamp</i></td>
    <td valign="top">The UTC time of the moment when the token was enrolled (64-bit value representing the number of 100-nanosecond intervals since January 1, 1601).
    </td>
  </tr>
  <tr>
    <td valign="top">String <i>keyHash</i></td>
    <td valign="top">Public key’s hash, Base64url UTF-8 encoded string.<BR><BR>The public key from the Smart Card is imported in the PUBLICKEYBLOB format. After that, the RSA256 	hash of the key is calculated. The resulting 32 bytes are Base64url encoded to the string:</td>
  </tr>
  <tr>
    <td valign="top">Byte <i>version</i></td>
    <td valign="top">Version of CDPJsonSCEnrolledToken object, must be set to 1 for current implementation.   
  	</td>
  </tr>
  <tr>
    <td valign="top">UInt64 <i>timeStamp</i></td>
    <td valign="top">The UTC time of the moment when the token was enrolled (64-bit value representing 	the number of 100-nanosecond intervals since January 1, 1601).
  	</td>
  </tr>
  <tr>
    <td valign="top">String <i>keyHash</i></td>
    <td valign="top">Public key’s hash, Base64url UTF-8 encoded string.<BR><BR>
  	The public key from the Smart Card is imported in the PUBLICKEYBLOB format. After that, the RSA256 	hash of the key is calculated. The resulting 32 bytes are Base64url encoded to the string:
  	KeyHash = Base64urlEncode(RSA256 Hash (PUBLICKEYBLOB (PuK))<BR><BR>
  	This unique string can be used to unambiguously indicate the card token in the WebDelete method.
  	</td>
  </tr>
  <tr>
     <td valign="top">String <i>nickname</i></td>
     <td valign="top">The nickname of the smart card token.<BR><BR>
   	This string along with enrollment time can be used in the UI to list the enrolled tokens.</td>
   </tr>   
</table>

### Enrollment algorithm  

On WEB Smart Card enrollment, one of the public keys stored on the card must be selected and sent to the server.  

1. On the client, where the SC is inserted into the reader:  

  - The enrollment UI must enumerate the appropriate key pairs (or X.509 certificates) on the card, and the user must select one of them. Please note that this operation does not need the card PIN.  

  - The user must provide a text string - the nickname for the enrolling smart card token to distinguish it from other smart card tokens enrolled for this particular user. By default, the nickname usually contains the card name, like “SmartCafe Expert 72K DI v3.2”.  

    Please note that nicknames are not unique in the WASSC, so it’s up to the user to make a good practical nickname for the enrolled token.  

2. On the server:  

  - The server enrollment procedure uses the card public key to encrypt the user sensitive data and create a database user record of enrolled Smart Card, as defined in the “Smart Card Enrollment Data Format” document. Public key encryption does not need the Smart Card presence at that time.  

  The time of the enrollment is added to the record, alone with the token’s nickname.  

# Enrollment Implementation  

The WEB enrollment of the WASSC authentication token is implemented according to the details in the < HREF="wes.md>Web Enrollment Service</A> topic to comply with the demands of the IDPWebEnroll interface and the  Credential class.  

The Credential class is defined as follows.  

~~~
[DataContract]
public class Credential
{
  [DataMember]
  public String id { get; set; }   // unique id (Guid) of credential
  [DataMember]
  public String data { get; set; } //  credential data
}  
~~~  

The following ID is defined for the Smart Card Credential.  

{D66CC98D-4153-4987-8EBE-FB46E848EA98}  

The following methods of the IDPWebEnroll interface are supported by the WASSC authentication token.  

#### WebEnroll  

To call this method, the client should send a IDPProWebEnrollMgr -> EnrollUserCredentials request to the server.  

The data for the Smart Card credential is a Base64url encoded UTF-8 representation of the JSON representation of the CDPJsonSCEnrollData class.  

The CDPJsonSCEnrollData class is defined as follows.

~~~
[DataContract]
public class CDPJsonSCEnrollData
  {
    [DataMember]
    public Byte version { get; set; }    // version
    [DataMember]
    public String key { get; set; }      // public key
    [DataMember]
    public String nickname { get; set; } // token’s nickname
  }  
~~~

where:  

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Data Member</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">Byte <i>version</i></td>
  <td valign="top">Version of CDPJsonSCEnrollData object, must be set to 1 for current implementation.</td>
  </tr>
  <tr>
  <td valign="top">String <i>key</i></td>
  <td valign="top">Public key, Base64url UTF-8 encoded string.
	The public key from the Smart Card must be imported in the PUBLICKEYBLOB format. After that, it 	must be Base64url encoded to the string:<BR><BR>
	Key = Base64urlEncode( (PUBLICKEYBLOB (PuK) )</td>
  </tr>
  <tr>
  <td valign="top">String <i>nickname</i></td>
  <td valign="top">The nickname of the Smart Card token. It’s suggested to use the card name, like 			“SmartCafe Expert 72K DI v3.2”, as a part of the nickname. The server limits the length of the nickname to 255 symbols, any additional symbols will be cut off.</td>
  </tr>
</table>

To create the Smart Card Credential for enrollment, perform the following steps on the client.  

1. Enumerate the asymmetric key pairs on the Smart Card and select the exact key pair to use. WASSC supports RSA keys of any length but at least 1024 bit length key is suggested for security reasons.  
2. Create a JSON representation of the CDPJsonSCEnrollData class for the public key selected on step #1.  
3. Base64Url encode string created in step #2.  
4. Finally, create a JSON representation of the Credential class using the Smart Card Credential ID as the id member and the string created in step #3 as a data member.  

### WebDelete  

To call this method, the client should send a  IDPProWebEnrollMgr -> DeleteUserCredentials request to the server.  

To delete a particular Smart Card credential, the input data for this call must be a string presenting the Smart Card Public key’s hash, calculated as described in <mark style="color:Red;">4.1 or received from the server as described in 4.3</mark>.  

To delete all the Smart Card credentials enrolled for this particular user, the input data for this call must be an empty string (not a NULL pointer).
