---
layout: default
title: Web Authentication Service (WAS)
has_toc: false
nav_order: 4
---

{% include header.html %}
<BR>

# Web Authentication Service

The purpose of the DigitalPersona Web Authentication Service (WAS) is user authentication and identification. WAS is a GUI-less web service which should be located inside the enterprise firewall, where it directs authentication (identification) requests to the DigitalPersona Server over DCOM. The DigitalPersona Server is the only one who can process authentication requests.  

The result of successful authentication is a JSON token signed by the Web Service.  

## IDPWebAuth interface  

The DigitalPersona Web Authentication Service is a WCF (Windows Communication Foundation) service. The IDPWebAuth interface is the WCF interface.  

~~~  
namespace DPWcfAuthService
{
	[ServiceContract]
	public interface IDPWebAuth
	{
		[OperationContract()]
		/*
		* Return list of credentials enrolled by user
		*/
		[WebInvoke(Method = "GET",
			ResponseFormat = WebMessageFormat.Json,
			BodyStyle = WebMessageBodyStyle.Wrapped,
			UriTemplate = "GetUserCredentials?user={userName}&type={userNameType}")]
			List<String> GetUserCredentials(String userName, UInt16 userNameType);
		/*
		* Return credential specific public enrollment information
		*/
		[WebInvoke(Method = "GET",
			ResponseFormat = WebMessageFormat.Json,
			BodyStyle = WebMessageBodyStyle.Wrapped,
			UriTemplate = "GetEnrollmentData?user={userName}&type={userNameType}&cred_id={credentialId}")]
			String GetEnrollmentData(String userName, UInt16 userNameType, String credentialId);
		/*
		* Identify user based on credential provided
		*/
		[WebInvoke(Method = "POST",
			ResponseFormat = WebMessageFormat.Json,
			BodyStyle = WebMessageBodyStyle.Wrapped,
			UriTemplate = "IdentifyUser")]
			Ticket IdentifyUser(Credential credential);
		/*
		* Authenticate user based on user name and credential provided
		*/
		[WebInvoke(Method = "POST",
			ResponseFormat = WebMessageFormat.Json,
			BodyStyle = WebMessageBodyStyle.Wrapped,
			UriTemplate = "AuthenticateUser")]
			Ticket AuthenticateUser(User user, Credential credential);
		/*
		* Authenticate user based on previously issued ticket and credential provided
		*/
		[WebInvoke(Method = "POST",
			ResponseFormat = WebMessageFormat.Json,
			BodyStyle = WebMessageBodyStyle.Wrapped,
			UriTemplate = "AuthenticateUserTicket")]
			Ticket AuthenticateUserTicket(Ticket ticket, Credential credential);
	}
	[DataContract]
	public class User
	{
		[DataMember]
		public String name { get; set; } 																		// name of the user
		[DataMember]
		public UInt16 type { get; set; } 																		// form (or type) of user name
	}
	[DataContract]
	public class Credential
	{
		[DataMember]
		public String id { get; set; } 																		// unique id (Guid) of credential
		[DataMember]
		public String data { get; set; } 																		// Base64 encoded credential data
	}
	[DataContract]
	public class Ticket
	{
		[DataMember]
		public String jwt { get; set; } 																		// JSON Web Token
	}
}
~~~

## Methods  

The IDPWebAuth interface has 5 methods. Below we give a detailed description of each method.  

### GetUserCredentials method  

The GetUserCredentials method is a utility method which allows the caller to determine which particular credentials have been enrolled by a user.  

#### Syntax  

~~~  
List<String> GetUserCredentials(String userName, UInt16 userNameType);
~~~  
<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">userName</td>
  <td valign="top">The name of the user whose credential is to be verified. </td>
  </tr>
  <tr>
  <td valign="top">userNameType</td>
  <td valign="top">The format of the user name provided in the userName parameter. See <A HREF="was.md#user-class">User Class</A> for a detailed description of user name formats.</td>
  </tr>    
</table>

#### Return values  

Returns a list of credential IDs signifying the credentials enrolled by a user. The format of credential IDs is described [here](#credential-class).

#### Notes  

GetUserCredentials is implemented as an HTTP GET using JSON as the response format.  

The following is an example of the use of GetUserCredentials.  

~~~
https://www.mydomain.com/DPWebAuthService.svc/GetUserCredentials?user=someone@mycompany.com&type=6
~~~  

Here the user name is someone@mycompany.com and the user name type is 6 which means UPN.  

The example response would be the following.  

~~~{"GetUserCredentialsResult":
	[
	"D1A1F561-E14A-4699-9138-2EB523E132CC",
	"AC184A13-60AB-40e5-A514-E10F777EC2F9",
	"8A6FCEC3-3C8A-40c2-8AC0-A039EC01BA05"
	]
}
~~~

This response means that the user someone@mycompany.com has password, fingerprint and PIN credentials enrolled.  

### GetEnrollmentData method  

The GetEnrollmentData method is a utility method which allows the caller to get some credential enrollment specific data. For example Recovery Question authentication may require knowing which particular questions were enrolled.  

#### Syntax  

~~~
String GetEnrollmentData(String userName, UInt16 userNameType, String credentialId);  
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top"userName></td>
  <td valign="top">The name of the user whose enrollment data is to be returned.</td>
  </tr>
  <tr>
  <td valign="top">userNameType</td>
  <td valign="top">The format of the user name provided in the userName parameter. See <A HREF="was.md#user-class">User class</A> for a detailed description of the userName formats.</td>
  </tr>   
  <tr>
  <td valign="top">credentialId</td>
  <td valign="top">The unique ID of the credential whose data needs to be returned. The format of credential IDs is described in the section [Credential class](credential-class).</td>
  </tr>     
</table>

#### Return values  

Returns Base64 encoded enrollment data. The format of this enrollment data is credential-specific.  

#### Notes  

GetEnrollmentData is implemented as an HTTP GET using JSON as the response format.

#### Example  

This is an example of using GetEnrollmentData. The use of braces { } is considered unsafe in URLs (see RFC 1738), which is why we use a “braceless” GUID representations in our API.

~~~https://www.mydomain.com/DPWebAuthService.svc/GetEnrollmentData?
user=someone@mycompany.com&type=6&cred_id=AC184A13-60AB-40e5-A514-E10F777EC2F9
~~~

Here the user name is someone@mycompany.com, the user name type is 6 which means UPN name and the credential ID is {AC184A13-60AB-40e5-A514-E10F777EC2F9} which means that information about the fingerprint credential is being requested.

An example of a response might be the following.

~~~
{"GetEnrollmentDataResult":”Z3NhZGhhc2Rma0FTREZLYWZyZGtB”}
~~~
This response has fingerprint enrollment data for the user someone@mycompany.com.

### IdentifyUser method  

The IdentifyUser method allows identification of the user based on a provided credential. Note that not all credentials support identification. For example, fingerprints support identification but passwords do not.  

#### Syntax  

~~~
Ticket IdentifyUser(Credential credential);
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">credential</td>
  <td valign="top">The credential ID and data which needs to be used for identification.</td>
  </tr>
</table>

#### Return value

As a result of successful identification a Ticket will be returned. The format of this ticket is described in the section [JSON Web Token (JWT)](json-web-token-jwt).

#### Notes

IdentifyUser is implemented as an HTTP POST using JSON as the response format.  
Below is an example of a URL used to POST an IdentifyUser request.

~~~
https://www.mydomain.com/DPWebAuthService.svc/IdentifyUser
~~~

Below is an example of the HTTP BODY of an IdentifyUser request.

~~~
{
	“credential”:
	{
	“id":”AC184A13-60AB-40e5-A514-E10F777EC2F9”,
	“data”:”Z3NhZGhhc2Rma0FTREZLYWZyZGtB”
	}
}
~~~  
This is a request to identify the user with a provided fingerprint credential.  

The following is an example of an IdentifyUser response.  

~~~{"IdentifyUserResult":{“jwt”:”eyJ0eXAiOiJKV1QiLA0KICJhbGciOiJIUzI1NiJ9.eyJpc3
MiOiJqb2UiLA0KICJleHAiOjEzMDA4MTkzODAsDQogImh0dHA6Ly9leGFtcGxlLmNvbS9pc19yb29
0Ijp0cnVlfQ.dBjftJeZ4CVP-B92K27uhbUJU1p1rwW1gFWFOEjXk”}}
~~~

### AuthenticateUser method  

The AuthenticateUser method allows authentication of a user based on a provided credential.

#### Syntax  

~~~
Ticket AuthenticateUser(User user,Credential credential);
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">user</td>
  <td valign="top">User identity which needs to be authenticated.</td>
  </tr>
  <tr>
  <td valign="top">credential</td>
  <td valign="top">Credential ID and data to be used for identification.</td>
  </tr>
</table>

#### Return values  

As a result of successful authentication, a Ticket will be returned. The format of this ticket will be described below in detail.

#### Notes  

AuthenticateUser is implemented as an HTTP POST using JSON as the response format.   

Below is an example of a URL used to POST the AuthenticateUser request.  

~~~
https://www.mydomain.com/DPWebAuthService.svc/AuthenticateUser
~~~

Below is example of the HTTP BODY of an AuthenticateUser request. This is a request to authenticate the user “someone@mycompany.com” with a provided fingerprint credential.

~~~
{
	“user”:
	{
		“name”:”someone@mycompany.com”,
		“type”:6
	},
	“credential”:
	{
		“id":”AC184A13-60AB-40e5-A514-E10F777EC2F9”,
		“data”:”Z3NhZGhhc2Rma0FTREZLYWZyZGtB”
	}
}
~~~

The following is an example of an AuthenticateUser response.

~~~
{"AuthenticateUserResult":{“jwt”:”eyJ0eXAiOiJKV1QiLA0KICJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJqb2UiLA0KICJleHAiOjEzMDA4MTkzODAsDQogImh0dHA6Ly9leGFtcGxlLmNvbS9pc19yb290Ijp0cnVlfQ.dBjftJeZ4CVP-B92K27uhbUJU1p1rwW1gFWFOEjXk”}}
~~~

### AuthenticateUserTicket method

The AuthenticateUserTicket method allows authentication of a user through a previously issued ticket and the provided credential.

#### Syntax  

~~~Ticket AuthenticateUserTicket(Ticket ticket, Credential credential);
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">ticket</td>
  <td valign="top">A ticket previously issued by the authentication authority.</td>
  </tr>
  <tr>
  <td valign="top">credential</td>
  <td valign="top">The Credential ID and data to be used for identification.</td>
  </tr>
</table>

#### Return values  

As a result of successful authentication, a Ticket will be returned.  

AuthenticateUserTicket is implemented as an HTTP POST using JSON as the response format.  

Below is an example of a URL used to POST the AuthenticateUserTicket request.

~~~
https://www.mydomain.com/DPWebAuthService.svc/AuthenticateUserTicket  
~~~

Below is example of the HTTP BODY for an AuthenticateUserTicket request. This is a request to authenticate the user whose name is encoded in JWT with the provided fingerprint credential.
~~~
{
	“ticket”:
	{
		“jwt”:
		”eyJ0eXAiOiJKV1QiLA0KICJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJqb2UiLA0KICJleHAiOjE
		zMDA	4MTkzODAsDQogImh0dHA6Ly9leGFtcGxlLmNvbS9pc19yb290Ijp0cnVlfQ.dBjftJeZ4
		CVP-	B92K27uhbUJU1p1rwW1gFWFOEjXk”
	},
	“credential”:
	{
		“id":”AC184A13-60AB-40e5-A514-E10F777EC2F9”,
		“data”:”Z3NhZGhhc2Rma0FTREZLYWZyZGtB”
	}
}
~~~  
Below is an example of the AuthenticateUserTicket response.  

~~~
{"AuthenticateUserTicketResult":{“jwt”:”eyJ0eXAiOiJKV1QiLA0KICJhbGciOiJIUzI1
NiJ9.eyJpc3MiOiJqb2UiLA0KICJleHAiOjEzMDA4MTkzODAsDQogImh0dHA6Ly9leGFtcGxlLmN
vbS9pc19yb290Ijp0cnVlfQ.dBjftJeZ4CVP-B92K27uhbUJU1p1rwW1gFWFOEjXk”}}  
~~~
### <A NAME="was-custom-action></A>Custom Action method  

The CustomAction method allows calling some specific custom action for a designated authentication token.

#### Syntax  

~~~
String CustomAction(Ticket ticket, User user, Credential credential, UInt16 actionId);
~~~


<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">ticket</td>
  <td valign="top">	Ticket previously issued by the authentication authority.</td>
  </tr>
  <tr>
  <td valign="top">user	</td>
  <td valign="top">User to which credential custom action would need to apply.</td>
  </tr>
  <tr>
  <td valign="top">credential</td>
  <td valign="top">	Credential ID and data to be used for identification.</td>
  </tr>
  <tr>
  <td valign="top">actionId</td>
  <td valign="top">The actionId	of the specific action to be performed. This ID is specific for each authentication token.</td>
  </tr>   
</table>

#### Return values

CustomAction may or may not return any data. This information is specific to each authentication token.  

CustomAction is implemented as an HTTP POST using JSON as the response format.  

Below is example of URL which can be used to POST CustomAction request.

~~~
https://www.digitalpersona.com/DPWebAuthService.svc/CustomAction
~~~

Below is example of the HTTP BODY of an CustomAction request.
~~~
{
	"ticket":
	{
		"jwt":
		"eyJ0eXAiOiJKV1QiLA0KICJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJqb2UiLA0KICJleHAiOjE
		zMDA4MTkzODAsDQogImh0dHA6Ly9leGFtcGxlLmNvbS9pc19yb290Ijp0cnVlfQ.dBjftJeZ4
		CVP-B92K27uhbUJU1p1rwW1gFWFOEjXk"
	},
	"user":
	{
		"name":"someone@mycompany.com",
		"type":6
	},
	"credential":
	{
		"id":"AC184A13-60AB-40e5-A514-E10F777EC2F9",
		"data":"Z3NhZGhhc2Rma0FTREZLYWZyZGtB"
	},
	"actionId":6
}
~~~

The above illustrates a request to perform custom action #6 using a  fingerprint authentication token for the user someone@mycompany.com.

### CreateUserAuthentication method  

CreateUserAuthentication creates Extended Authentication for the designated  user and authentication token (credential).

#### Syntax  

~~~UInt32 CreateUserAuthentication(User user, String credentialId);
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">user</td>
  <td valign="top">	The user for whom extended authentication is to be created.</td>
  </tr>
  <tr>
  <td valign="top">credentialId </td>
  <td valign="top">The credential ID of the authentication token (credential) for which extended authentication is to be created.</td>
  </tr>
  <tr>
  <td valign="top"></td>
  <td valign="top"></td>
  </tr>   
</table>

#### Return values  

A unique extended authentication handle will be returned.

#### Example    

CreateUserAuthentication is implemented as an HTTP POST using JSON as the response format.

Below is example of a URL used to POST a CreateUserAuthentication request.

~~~
https://www.mycompany.com/DPWebAuthService.svc/CreateUserAuthentication
~~~

Below is example of the HTTP BODY of a CreateUserAuthentication request.

~~~
{
 "user":
{
"name":"klozin@digitalpersona.com",
"type":6
},
 "credentialId":"AC184A13-60AB-40e5-A514-E10F777EC2F9"
}
~~~

This above is a request to create extended authentication for a user someone@mycompany.com and for the fingerprint authentication token.
The following is an example of CreateUserAuthentication response.

~~~
{"CreateUserAuthenticationResult":52346}
~~~

### CreateTicketAuthentication method

The CreateTicketAuthentication method creates Extended Authentication based on an existing Ticket and a specific authentication token (credential).

####Syntax  

~~~
UInt32 CreateTicketAuthentication(Ticket ticket, String credentialId);
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">ticket</td>
  <td valign="top">	Ticket previously issued by the authentication authority.</td>
  </tr>
  <tr>
  <td valign="top">credentialId </td>
  <td valign="top">The credential ID of the authentication token (credential) for which extended authentication is to be created.</td>
  </tr>
  <tr>
  <td valign="top"></td>
  <td valign="top"></td>
  </tr>   
</table>

#### Return values

A unique extended authentication handle will be returned.

#### Example  

CreateTicketAuthentication is implemented as an HTTP POST using JSON as the  response format.

Below is an example of a URL used to POST a CreateTicketAuthenticationrequest.

~~~
https://www.mycompany.com/DPWebAuthService.svc/CreateTicketAuthentication
~~~

Below is an example of the HTTP BODY of a CreateTicketAuthentication request.  

~~~
{
	"ticket":
	{
	"jwt":
	"eyJ0eXAiOiJKV1QiLA0KICJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJqb2UiLA0KICJleHAiOjE
	zMDA	4MTkzODAsDQogImh0dHA6Ly9leGFtcGxlLmNvbS9pc19yb290Ijp0cnVlfQ.dBjftJeZ4
	CVP-B92K27uhbUJU1p1rwW1gFWFOEjXk"
	},
 	"credentialId":"AC184A13-60AB-40e5-A514-E10F777EC2F9"
}
~~~

The above is a request to create extended authentication based on the provided ticket and for fingerprint authentication token.  

The following is an example of a CreateTicketAuthentication response.

~~~
{"CreateTicketAuthenticationResult":52346}
~~~

### ContinueAuthentication method  

ContinueAuthentication performs a single step involved in Extended Authentication. Extended Authentication may require handshakes with two or  more steps, so ContinueAuthentication may be called multiple times.

#### Syntax  

~~~
ExtendedAUTHResult ContinueAuthentication(UInt32 authId, String authData);
~~~
<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">authId</td>
  <td valign="top">	Extended Authentication handle returned by CreateUserAuthentication or CreateTicketAuthentication call. </td>
  </tr>
  <tr>
  <td valign="top">authData	</td>
  <td valign="top">Authentication token specific authentication data.</td>
  </tr>
</table>

#### Return values  

An Extended Authentication Result will be returned
<mark style="color:Red;">(see 4.10.12 </mark>ExtendedAUTHResult data structure for details).  

ContinueAuthentication is implemented as an HTTP POST using JSON as the response format.  

Below is example of URL be used to POST ContinueAuthentication request.

~~~
https://www.mycompany.com/DPWebAuthService.svc/ContinueAuthentication
~~~

Below is an example of the HTTP BODY of a ContinueAuthentication request.

~~~
{
	"authId": 52346,
  "authData":
		"eyJ0eXAiOiJKV1QiLA0KICJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJqb2UiLA0KICJleHAiOj
		EzMDA4MTkzODAsDQogImh0dHA6Ly9leGFtcGxlLmNvbS9pc19yb290Ijp0cnVlfQ.dBjftJe
		Z4CVP-B92K27uhbUJU1p1rwW1gFWFOEjXk"
}
~~~
This above is a request for a step in extended authentication.  

The following is an example of a ContinueAuthentication response.

~~~
{"ContinueAuthenticationResult":
	{
		"status":2,
		"authData":null,
		"jwt":"eyJ0eXAiOiJKV1QiLA0KICJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJqb2UiLA0KICJl
		eHAiOjEzMDA4MTkzODAsDQogImh0dHA6Ly9leGFtcGxlLmNvbS9pc19yb290Ijp0cnVlfQ.dB
		jftJeZ4CVP-B92K27uhbUJU1p1rwW1gFWFOEjXk"
	}
}
~~~
In this example status 2 means Extended Authentication successfully completed and a JSON Web Token is returned in the "jwt" parameter.

### DestroyAuthentication method  

DestroyAuthentication destroys a previously created Extended Authentication and clears all associated resources. Note that it is not generally necessary to call DestroyAuthentication, as the Extended Authentication will be automatically destroyed upon successful (or unsuccessful) completion. You shuld call DestroyAuthentication only when you want to stop an authentication which has not completed yet.

#### Syntax  

~~~
void DestroyAuthentication(UInt32 authId);
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">authId</td>
  <td valign="top">	Extended Authentication handle returned by CreateUserAuthentication or CreateTicketAuthentication call.</td>
  </tr>
</table>

#### Return value  
None.  

#### Examples  

DestroyAuthentication is implemented as an HTTP DELETE using JSON as the  response format.

Below is an example of a URL used to submit a DestroyAuthentication request.  

~~~
https://www.mycompany.com/DPWebAuthService.svc/DestroyAuthentication
~~~

Below is example of the HTTP BODY of a DestroyAuthentication request.

~~~
{
"authId": 52346

}
~~~

The above is a request to destroy an extended authentication with a handle of 52346.

## <A NAME="was-data-contracts"></A> Data Contracts

Below are the Data Contracts used in this API.

### <A NAME="user-class"></A>User class

User class is user representation in this API.

~~~
public class User
	{
	[DataMember]
	public String name { get; set; } // name of the user
	[DataMember]
	public UInt16 type { get; set; } // form (or type) of user name
	}
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">name
</td>
  <td valign="top">	String which represents the name of the user.</td>
  </tr>
  <tr>
  <td valign="top">type
</td>
  <td valign="top">Integer which represents the format of the user name string.</td>
  </tr>   
</table>

The following user name formats are currently supported.

<A NAME="user-name-formats"></A>

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th colspan="2" style="text-align:center">User Name Formats</th>
  </tr>
	<tr>
    <th style="width:5%" ALIGN="left">Type</th>
    <th style="width:90%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">3</td>
  <td valign="top">Windows NT® 4.0 account name (for example, digital_persona/\johnsmith). The domain-only version includes trailing backslashes (/\\).</td>
  </tr>
  <tr>
  <td valign="top">4</td>
  <td valign="top">	Account name format used in Microsoft® Windows NT® 4.0. For example, "someone"</td>
  </tr>
  <tr>
    <td valign="top">5</td>
    <td valign="top">GUID string that the IIDFromString function returns (for example, {4fa050f0-f561-11cf-bdd9-00aa003a77b6}).</td>
  </tr>
  <tr>
    <td valign="top">7</td>
    <td valign="top">User principal name (for example, someone@mycompany.com).</td>
  </tr>
  <tr>
    <td valign="top">8</td>
    <td valign="top">User SID string (for example, S-1-5-21-1004).</td>
  </tr>
  <tr>
    <td valign="top">9</td>
    <td valign="top">DigitalPersona user name format (user name associated with the DigitalPersona identity database).</td>
  </tr>   
</table>

Below is an example of the JSON representation of a user.

~~~  
{
	“user”:
	{
	“name”:”someone@mycompany.com”,
	“type”:6
	}
}
~~~

### <A NAME="credential-class"></A>Credential class

Credential class is credential representation in this API.

~~~
public class Credential
	{
	[DataMember]
	public String id { get; set; } 																		// unique id (Guid) of credential
	[DataMember]
	public String data { get; set; } 																		// Base64 encoded credential data
	}
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">id</td>
  <td valign="top">	Unique Id of credential.</td>
  </tr>
  <tr>
  <td valign="top">data	</td>
  <td valign="top">Base64url encoded credential data. The format of this data depends on the credential and is explained in the <A HREF="https://hidglobal.github.io/digitalpersona-access-management-services/was-cred-format.html">WAS Credentials Data Format</A> topic.</td>
  </tr>
</table>

The following credentials are currently supported.

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Credential</th>
    <th style="width:35%" ALIGN="left">GUID</th>
  </tr>
  <tr>
  <td valign="top">User password</td>
  <td valign="top">{D1A1F561-E14A-4699-9138-2EB523E132CC}</td>
  </tr>
  <tr>
  <td valign="top">Fingerprints</td>
  <td valign="top">{AC184A13-60AB-40e5-A514-E10F777EC2F9}</td>
  </tr>
  <tr>
  <td valign="top">PIN</td>
  <td valign="top">{8A6FCEC3-3C8A-40c2-8AC0-A039EC01BA05}</td>
  </tr>
	<tr>
  <td valign="top">Smart Cards</td>
  <td valign="top">{D66CC98D-4153-4987-8EBE-FB46E848EA98}</td>
  </tr>
	<tr>
  <td valign="top">Proximity cards</td>
  <td valign="top">{1F31360C-81C0-4EE0-9ACD-5A4400F66CC2}</td>
  </tr>
	<tr>
  <td valign="top">Contactless cards</td>
  <td valign="top">	{7BF3E290-5BA5-4C2D-AA33-24B48C189399}</td>
  </tr>
	<tr>
  <td valign="top">Recovery Questions</td>
  <td valign="top">{B49E99C6-6C94-42DE-ACD7-FD6B415DF503}</td>
  </tr>
	<tr>
  <td valign="top"> Bluetooth</td>
  <td valign="top">{E750A180-577B-47f7-ACD9-F89A7E27FA49}</td>
  </tr> 	
	<tr>
  <td valign="top"> ?? MISSING CREDENTIAS ??</td>
  <td valign="top"></td>
  </tr> 		 			
</table>

#### Examples

You must use “braceless” GUID representation in our API in URLs and JSON representation. Below is an example of the JSON representation of a credential.

~~~
{
	“credential”:
	{
		“id":”AC184A13-60AB-40e5-A514-E10F777EC2F9”,
		“data”:”Z3NhZGhhc2Rma0FTREZLYWZyZGtB”
	}
}
~~~
### Ticket class
The result of successful authentication is a Ticket.

~~~
public class Ticket
	{
	[DataMember]
	public String jwt { get; set; } 																		// JSON Web Token
	}
~~~
The format of the Ticket is a JSON Web Token (JWT). See the next section for further details on JWT.  

### AuthenticationStatus enumeration

AuthenticationStatus is enumeration for status of extended authentication operation.  

~~~ public enum AuthenticationStatus
{
	/// <summary>
	/// Error happened. Authentication attempt failed.
	/// </summary>
	[EnumMember]
	Error = 0,

	/// <summary>
	/// Authentication step succeeded but authentication continuation is required.
	/// </summary>
	[EnumMember]
	Continue = 1,

	/// <summary>
	/// Authentication successfully completed.
	/// </summary>
	[EnumMember]
	Completed = 2,
}  
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Status</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">Error</td>
  <td valign="top">Extended Authentication operation failed.</td>
  </tr>
  <tr>
  <td valign="top">Continue	</td>
  <td valign="top">Extended Authentication operation  succeeded but continuation is required.</td>
  </tr>
  <tr>
  <td valign="top">Completed	</td>
  <td valign="top">Extended Authentication operation is successfully completed.</td>
  </tr>
</table>

### <A NAME="extended-auth-result"></A>ExtendedAUTHResult class  

ExtendedAUTHResult class represents result of Extended Authentication operation.

~~~
public class ExtendedAUTHResult
{
	[DataMember]
	public AuthenticationStatus status { get; set; } // Status of extended
		authentication operation.
	[DataMember]
	public String authData { get; set; } // Authentication data returned by the
		Server.
	[DataMember]
	public String jwt{ get; set; } 																			// JWT of successful authentication
		operation.
    }
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">status</td>
  <td valign="top">	Status of Extended Authentication operation <mark style="color:Red;"> (see 4.11.4 for details).</mark></td>
  </tr>
  <tr>
  <td valign="top">authData	</td>
  <td valign="top">Authentication data returned by server. This data is required to proceed with an authentication handshake. This is an optional parameter, and would be returned if the operation status is Continue, i.e. handshake continuation is required.</td>
  </tr>
  <tr>
  <td valign="top">jwt</td>
  <td valign="top">	JSON Web Token returned by the server upon successful authentication. This is an optional parameter and will be returned if the operation status is Completed, i.e. the authentication handshake is suucessfuly completed).</td>
  </tr>
</table>

## <A NAME="json-web-token-jwt"></A> JSON Web Token (JWT)  

The format of a returned Ticket upon successful authentication is a JSON Web Token.

The official specification of JWT is detailed at the following location.

https://tools.ietf.org/html/draft-jones-json-web-token-10.  

The JWT token returned by the DigitalPersona Web Authentication Service is constructed as follows.

The standard JWT has three parts:
1. Header  
2. Claims
3. Signature  

All three parts are created separately as UTF-8 strings, then they are Base64url encoded and finally concatenated in the order above with periods between the parts, yielding the complete JWT.

**Base64url Encoding** - For the purposes of this specification, this term always refers to the URL- and filename-safe Base64 encoding described in RFC 4648 [RFC4648], Section 5, with the (non URL-safe) '=' padding characters omitted, as permitted by Section 3.2.  

### JWT Header
In the current version of the API, we use the following JWT header.

~~~
{"typ":"JWT",
 "alg":" RS256"} –
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Value</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">"typ":"JWT"</td>
  <td valign="top">Means this is a JSON Web Token.</td>
  </tr>
  <tr>
  <td valign="top">"alg":" RS256" </td>
  <td valign="top">Means we will use RSA with a SHA-256 hash algorithm for the signature.</td>
  </tr>
</table>

### JWT Claims  

The following claims must be included in our JWT, although the claims order is not defined so they can be presented in any order.

#### "jti" (JWT ID) Claim  

The "jti" (JWT ID) claim provides a unique identifier for the JWT. The identifier value must be assigned in a manner that ensures that there is a negligible probability that the same value will be accidentally assigned to a different data object. The "jti" claim can be used to prevent the JWT from being replayed. The "jti" value is case sensitive. Its value must be a string.  

The “jti” claim should be a string representation of the GUID randomly generated by the DigitalPersona Server during JWT creation.

Below is an example of this claim.

~~~
"jti":"AC184A13-60AB-40e5-A514-E10F777EC2F9"  
~~~

#### "iss" (Issuer) Claim

The "iss" (issuer) claim identifies the principal that issued the JWT.  

For this API, the issuer will always be the DigitalPersona Server, so we include the DigitalPersona Server DNS name in this claim.  

Below is an example of this claim.  

~~~
"iss":"altus-01.mydomain.com"  
~~~

### "dom" (Issuer Domain) Claim  

The "dom" (issuer domain) claim identifies the domain that issued the JWT. For DigitalPersona (AD LDS) users, the issuer domain is the DigitalPersona (AD LDS) instance. For DigitalPersona AD users, the issuer domain is the NETBIOS Domain name that the user belong to.  

Below is an example of this claim.  

~~~
"dom":"Software_Department"  
~~~

### "iat" (Issued At) Claim  

The "iat" (issued at) claim identifies the date and time when the JWT was issued. This claim can be used to determine the age of the token. Its value must be a number containing an IntDate value.  

The time is always in UTC Unix time format, an integral value representing the number of seconds elapsed since 00:00 hours, i.e. Jan 1, 1970 UTC.  

Below is an example of this claim.

~~~
"iat”:1300819380
~~~

### "exp" (Expiration Time) Claim  

The exp (expiration time) claim identifies the expiration time on or after which the JWT must not be accepted for processing. Use of this claim is optional.

The processing of the exp claim requires that the current date/time MUST be before the expiration date/time listed in the exp claim.  

Implementers may provide for some small leeway, usually no more than a few minutes, to account for clock skew. Its value must be a number containing a IntDate value.

The time is always in UTC Unix time format - it is generally implemented as an integral value representing the number of seconds elapsed since 00:00 hours, Jan 1, 1970 UTC.  

Below is example of this claim:  

~~~
"exp":1300819380
~~~

### sub" (Subject) Claim  

The "sub" (subject) claim identifies the principal that the JWT was issued to. This should be a readable string specifying the identity of the authenticated user; for example, the user’s display name or account name.  

Below is an example of this claim.  

~~~
"sub":”Tom Jones”
~~~

### "uid" (Subject Unique ID) Claim  

The "uid" (subject UID) claim uniquely identifies the principal (inside a particular DigitalPersona instance) that the JWT is issued to. This is the objectGUID of the user record in the DigitalPersona (AD LDS) database.  

Below is an example of this claim.

~~~
"uid":”6E2A0211-59E0-4EFA-89C5-68F75E6CE8B7”
~~~

### "crd" (Credentials) Claim  

The "crd" (Credentials) claim must list all the credentials used for the authentication, providing the credential id and the authentication timestamp in the Unix time format.  

Below is an example of this claim.  

~~~
"crd":[
			{“id”:"D1A1F561-E14A-4699-9138-2EB523E132CC",”time”: 1300819380},
			{“id”:”AC184A13-60AB-40e5-A514-E10F777EC2F9",”time”: 7865233679}
			]
~~~

The claim above means that two types of credentials were used for authentication at two separate times.  

#### NOTE

The JWT claim does not include an expiration time, providing the flexibility for the Service Provider to decide to accept this ticket or reject it based on their own logic. It is assumed that the SP clock is in sync with the DigitalPersona Server clock.  

### "wan" (Windows Account Name) Claim

The "wan" (Windows Account Name) claim uniquely identifies the user that the JWT is issued to. This claim is optional and will be issued only for Active Directory users, providing the user SAM account name of the user with the format <NETBIOS Domain Name\user account name>. For DigitalPersona Non AD users this claim will be omitted.  

Below is an example of this claim:  

~~~
"wan":"mydomain\someone"  
~~~

### "t24" (T24 Name) Claim  

The "t24" (Windows Account Name) claim uniquely identifies the principal to whom JWT is issued inside the T24 database. This claim is optional and will be issued only for T24 users.  

Below is an example of this claim.  

~~~
"t24":"someone"
~~~

### Example of JWT claims section  

Below is an example of the claims part of a JWT.

~~~
{"jti":"AC184A13-60AB-40e5-A514-E10F777EC2F9",
"dom":"Software_Department",
"iss":"DP-01.mycompany.com",
"iat":1300819380,
“exp”:1300819623
"sub":”John Doe”,
"uid":”6E2A0211-59E0-4EFA-89C5-68F75E6CE8B7”
"crd":[
			{"id":"D1A1F561-E14A-4699-9138-2EB523E132CC","time": 1300819380},
			{"id":"AC184A13-60AB-40e5-A514-E10F777EC2F9","time": 7865233679}
			]}
~~~

The token above claims the user John Doe@mycompany.com” was authenticated using their password and fingerprint credentials on the  DigitalPersona server “DP-01.mycompany.com”.

### JSON Web Signature (JWS)  

The detailed description for the JWS (JSON Web Signature) is available at the following location.

https://tools.ietf.org/html/draft-jones-json-web-signature-04  

### Rules for Creating and Validating a JWS  

To create a JWS, you need to perform the following steps.  

1. Create the content to be used as the JWS Payload (JWT Claims in our case).  
2. Base64url encode the bytes of the JWS Payload.  This encoding becomes the Encoded JWS Payload.
3. Create a JWS Header containing the desired set of header parameters (JWT Header in our case). Note that white space is explicitly allowed in the representation and no canonicalization is performed before encoding.  
4. Base64url encode the bytes of the UTF-8 representation of the JWS Header to create the Encoded JWS Header.
5. Compute the JWS Signature in the manner defined for the particular algorithm being used. The JWS Signing Input is always the concatenation of the Encoded JWS Header, a period ('.') character, and the Encoded JWS Payload. The "alg" header parameter must be present in the JSON Header, with the algorithm value accurately representing the algorithm used to construct the JWS Signature.  
6.Base64url encode the representation of the JWS Signature to create the Encoded JWS Signature.  

When validating a JWS, the following steps must be taken. If any of the listed steps fails, then the signed content must be rejected.  

1. The Encoded JWS Header must be successfully base64url decoded following the restriction given in this specification that no padding characters have been used.  
2. The JWS Header must be completely valid JSON syntax conforming to RFC 4627 [RFC4627].  
3. The JWS Header must be validated to only include parameters and values whose syntax and semantics are both understood and supported.  
4. The Encoded JWS Payload must be successfully base64url decoded following the restriction given in this specification that no padding characters have been used.  
5. The Encoded JWS Signature must be successfully base64url decoded following the restriction given in this specification that no padding characters have been used.  
6. The JWS Signature must be successfully validated against the JWS Header and JWS Payload in the manner defined for the algorithm being used, which must be accurately represented by the value of the "alg" header parameter, which must be present.  

Processing a JWS inevitably requires comparing known strings to values in the header. For example, in checking what the algorithm is, the Unicode string encoding "alg" will be checked against the member names in the JWS Header to see if there is a matching header parameter name. A similar process occurs when determining if the value of the "alg" header parameter represents a supported algorithm.  

Comparisons between JSON strings and other Unicode strings must be performed as specified below.  

1. Remove any JSON applied escaping to produce an array of Unicode code points.  
2. Unicode Normalization [USA15] must NOT be applied at any point to either the JSON string or to the string it is to be compared against.  
3. Comparisons between the two strings must be performed as a Unicode code point to code point equality comparison.

### Creating a JWS with RSA SHA-256  

This section defines the use of the RSASSA-PKCS1-v1_5 signature algorithm as defined in RFC 3447 [RFC3447], Section 8.2 (commonly known as PKCS#1), using SHA-256, SHA-384, or SHA-512 as the hash function. The RSASSA-PKCS1-v1_5 algorithm is described in FIPS 186-3 [FIPS.186-3], Section 5.5, and the SHA-256, SHA-384, and SHA-512 cryptographic hash functions are defined in FIPS 180-3 [FIPS.180-3].  

The "alg" header parameter values "RS256" are used in the JWS Header to indicate that the Encoded JWS Signature contains a base64url encoded RSA signature using the respective hash function.  

The public keys employed will be distributed using methods that are outside the scope of this document.  

A 2048-bit or longer key length must be used with this algorithm.
The RSA SHA-256 signature is generated as follows.  

1. Generate a digital signature of the UTF-8 representation of the JWS Signing Input using RSASSA-PKCS1-V1_5-SIGN and the SHA-256 hash function with the desired private key.  The output will be a byte array.  
2. Base64url encode the resulting byte array. The output is the Encoded JWS Signature for that JWS.  

The RSA SHA-256 signature for a JWS is validated as follows.  

1. Take the Encoded JWS Signature and base64url decode it into a byte array. If decoding fails, the signed content must be rejected.
2. Submit the UTF-8 representation of the JWS Signing Input and the public key corresponding to the private key used by the signer to the RSASSA-PKCS1-V1_5-VERIFY algorithm using SHA-256 as the hash function.   
3. If the validation fails, the signed content must be rejected.  

### JWT Example

The following example JWS Header declares that the data structure is a JSON Web Token (JWT) [JWT] and the JWS Signing Input is signed using the RSA SHA-256 algorithm.  Note that white space is explicitly allowed in JWS Header strings and no canonicalization is performed before encoding.

~~~
{"typ":"JWT",
	"alg":" RS256"}
~~~

The following byte array contains the UTF-8 characters for the JWS Header.

~~~
[123,34,116,121,112,34,58,34,74,87,84,34,44,10,32,34,97,108,103,34,58,34,32,
82,83,50,53,54,34,125]  
~~~

Base64url encoding this UTF-8 representation yields this Encoded JWS Header value.  

~~~
eyJ0eXAiOiJKV1QiLAogImFsZyI6IiBSUzI1NiJ9  
~~~

The JWS Payload used in this example follows. Note that the payload can be any base64url encoded content, and need not be a base64url encoded JSON object.  

~~~
{"jti":"AC184A13-60AB-40e5-A514-E10F777EC2F9",
"dom":"Software_Department",
"iss":"DP-01.mycompany.com",
"iat":1300819380,
"sub":John Doe”,
"uid":”6E2A0211-59E0-4EFA-89C5-68F75E6CE8B7”
"crd":[
			{"id":"D1A1F561-E14A-4699-9138-2EB523E132CC","time": 1300819380},
			{"id":"AC184A13-60AB-40e5-A514-E10F777EC2F9","time": 7865233679}
			]}  
~~~

The following byte array contains the UTF-8 characters for the JWS Payload.  

~~~
[123,34,106,116,105,34,58,34,123,65,67,49,56,52,65,49,51,45,54,48,65,66,45,52,48,101,53,45,65,53,49,52,45,69,49,48,70,55,55,55,69,67,50,70,57,125,34,44,10,34,105,115,115,34,58,34,97,108,116,117,115,45,48,49,46,100,105,103,105,116,97,108,112,101,114,115,111,110,97,46,99,111,109,34,44,10,34,105,97,116,34,58,49,51,48,48,56,49,57,51,56,48,44,10,34,115,117,98,34,58,123,34,110,97,109,101,34,58,34,107,108,111,122,105,110,64,100,105,103,105,116,97,108,112,101,114,115,111,110,97,46,99,111,109,34,44,34,116,121,112,101,34,58,54,125,44,10,34,99,114,100,34,58,91,10,123,34,105,100,34,58,34,123,68,49,65,49,70,53,54,49,45,69,49,52,65,45,52,54,57,57,45,57,49,51,56,45,50,69,66,53,50,51,69,49,51,50,67,67,125,34,44,34,116,105,109,101,34,58,32,49,51,48,48,56,49,57,51,56,48,125,44,10,123,34,105,100,34,58,34,123,65,67,49,56,52,65,49,51,45,54,48,65,66,45,52,48,101,53,45,65,53,49,52,45,69,49,48,70,55,55,55,69,67,50,70,57,125,34,44,34,116,105,109,101,34,58,32,55,56,54,53,50,51,51,54,55,57,125,10,93,125]  
~~~  


Base64url encoding the above yields the Encoded JWS Payload value (with line breaks for display purposes only).  
~~~
eyJqdGkiOiJ7QUMxODRBMTMtNjBBQi00MGU1LUE1MTQtRTEwRjc3N0VDMkY5fSIsCiJpc3MiOiJhbHR1cy0wMS5kaWdpdGFscGVyc29uYS5jb20iLAoiaWF0IjoxMzAwODE5MzgwLAoic3ViIjp7Im5hbWUiOiJrbG96aW5AZGlnaXRhbHBlcnNvbmEuY29tIiwidHlwZSI6Nn0sCiJjcmQiOlsKeyJpZCI6IntEMUExRjU2MS1FMTRBLTQ2OTktOTEzOC0yRUI1MjNFMTMyQ0N9IiwidGltZSI6IDEzMDA4MTkzODB9LAp7ImlkIjoie0FDMTg0QTEzLTYwQUItNDBlNS1BNTE0LUUxMEY3NzdFQzJGOX0iLCJ0aW1lIjogNzg2NTIzMzY3OX0KXX0  
~~~

Concatenating the Encoded JWS Header, a period character, and the Encoded JWS Payload yields this JWS Signing Input value (with line breaks for display purposes only).  

~~~
eyJ0eXAiOiJKV1QiLAogImFsZyI6IiBSUzI1NiJ9
.
eyJqdGkiOiJ7QUMxODRBMTMtNjBBQi00MGU1LUE1MTQtRTEwRjc3N0VDMkY5fSIsCiJpc3MiOiJh
bHR1cy0wMS5kaWdpdGFscGVyc29uYS5jb20iLAoiaWF0IjoxMzAwODE5MzgwLAoic3ViIjp7Im5h
bWUiOiJrbG96aW5AZGlnaXRhbHBlcnNvbmEuY29tIiwidHlwZSI6Nn0sCiJjcmQiOlsKeyJpZCI6
IntEMUExRjU2MS1FMTRBLTQ2OTktOTEzOC0yRUI1MjNFMTMyQ0N9IiwidGltZSI6IDEzMDA4MTkz
ODB9LAp7ImlkIjoie0FDMTg0QTEzLTYwQUItNDBlNS1BNTE0LUUxMEY3NzdFQzJGOX0iLCJ0aW1l
IjogNzg2NTIzMzY3OX0KXX0  
~~~  

Running the RSA SHA-256 algorithm on the UTF-8 representation of the JWS Signing Input with this key yields the following byte array.  

~~~
[116, 24, 223, 180, 151, 153, 224, 37, 79, 250, 96, 125, 216, 173, 187, 186,
22, 212, 37, 77, 105, 214, 191, 240, 91, 88, 5, 88, 83, 132, 141, 121]  
~~~

Base64url encoding the above output yields the Encoded JWS Signature value.  

~~~
dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk  
~~~

The following string represents the final JWT.  

~~~
eyJ0eXAiOiJKV1QiLAogImFsZyI6IiBSUzI1NiJ9
.
eyJqdGkiOiJ7QUMxODRBMTMtNjBBQi00MGU1LUE1MTQtRTEwRjc3N0VDMkY5fSIsCiJpc3MiOiJhbHR1cy0wMS5kaWdpdGFscGVyc29uYS5jb20iLAoiaWF0IjoxMzAwODE5MzgwLAoic3ViIjp7Im5hbWUiOiJrbG96aW5AZGlnaXRhbHBlcnNvbmEuY29tIiwidHlwZSI6Nn0sCiJjcmQiOlsKeyJpZCI6IntEMUExRjU2MS1FMTRBLTQ2OTktOTEzOC0yRUI1MjNFMTMyQ0N9IiwidGltZSI6IDEzMDA4MTkzODB9LAp7ImlkIjoie0FDMTg0QTEzLTYwQUItNDBlNS1BNTE0LUUxMEY3NzdFQzJGOX0iLCJ0aW1lIjogNzg2NTIzMzY3OX0KXX0
.
dBjftJeZ4CVPmB92K27uhbUJU1p1r_wW1gFWFOEjXk  
~~~

## Error Handling  

If the DigitalPersona Server failed process authentication request, HTTP status of the request would be set to 404 (Not found) and WebFaultException of the WebFault class will be fired so the HTTP response will have Json representation of the WebFault class.

For further details on the WebFaultException, see the following Microsoft article.  

https://msdn.microsoft.com/en-us/library/system.servicemodel.web.webfaultexception(v=vs.110).aspx  

~~~
[DataContract]
public class WebFault
	{
	public WebFault(int err, String desc)
	{
		error_code = err;
		description = desc;
	}
[DataMember]
public int error_code { get; set; }
[DataMember]
public String description { get; set; }
}
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Value</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">error_code</td>
  <td valign="top">Error code returned by the DigitalPersona Server, for example -2147024891 (0x80070005) for an access denied error.</td>
  </tr>
  <tr>
  <td valign="top">description</td>
  <td valign="top">	Detailed description of the error, for example "Access is denied" for access denied error.
</td>
  </tr>
</table>


Below is an example of an error returned by WAS.

~~~  
{
	"error_code"=-2147024891,
	"description"="Access is denied."
}
~~~
