---
layout: default
title: Web Enrollment Service (WES)  
has_toc: false
nav_order: 2
---

{% include header.html %}
<BR>  

# Web Enrollment Service

The DigitalPersona Web Enrollment Service (WES) provides the methods necessary for  
- Creating users in the DigitalPersona LDS database  
- Enrolling both DigitalPersona AD and DigitalPersona LDS users  
- Managing credentials and other data relating to users

Web Enrollment features are accessed through the WES, implemented as a Web API. It should be deployd inside the enterprise’s firewall, and will direct enrollment requests to an existing DigitalPersona Server. It also requires one or more open web endpoints listening on the HTTPS protocol for intranet and/or internet use.  

A sample program is available [here](https://hidglobal.github.io/digitalpersona-native-api/docs/sample-applications.html), which provides a simple GUI-based application  illustrating the main features of the service provided through this API.  

# Open API schemas

Open API (Swagger) schemas are available [here](swagger/DPWebEnroll.json).

# IDPWebEnroll interface  

The IDPWebEnroll interface is a Windows Foundation Class (WCF) interface, and is described below.  

~~~
namespace WebServices.DPWebEnroll
{
	[ServiceContract]
	public interface IDPWebEnroll
	{
		/* Return information which credentials are enrolled for specific user
		*/
		[OperationContract()]
		[WebInvoke(Method = "GET",
			ResponseFormat = WebMessageFormat.Json,
			BodyStyle = WebMessageBodyStyle.Wrapped,
			UriTemplate ="GetUserCredentials?user={userName}&type={userNameType}")]
		Object GetUserCredentials(String userName, UInt16 userNameType);

		/* Return credential specific public enrollment information
		*/
		[OperationContract()]
		[WebInvoke(Method = "GET",
			ResponseFormat = WebMessageFormat.Json,
			BodyStyle = WebMessageBodyStyle.Wrapped,
			UriTemplate = "GetEnrollmentData?user={userName}&type={userNameType}&cred_id={credentialId}")]
		String GetEnrollmentData(String userName, UInt16 userNameType, String credentialId);

		/* Creates specific user
		*/
		[OperationContract()]
		[WebInvoke(Method = "PUT",
			ResponseFormat = WebMessageFormat.Json,
			BodyStyle = WebMessageBodyStyle.Wrapped,
			UriTemplate = "CreateUser")]
		void CreateUser(Ticket secOfficer, User user, String password);

		/* Deletes specific user
		*/
		[OperationContract()]
		[WebInvoke(Method = "DELETE",
			ResponseFormat = WebMessageFormat.Json,
			BodyStyle = WebMessageBodyStyle.Wrapped,
			UriTemplate = "DeleteUser")]
		void DeleteUser(Ticket secOfficer, User user);

		/* Enroll specific credentials for specific user
		*/
		[OperationContract()]
		[WebInvoke(Method = "PUT",
				ResponseFormat = WebMessageFormat.Json,
				BodyStyle = WebMessageBodyStyle.Wrapped,
				UriTemplate = "EnrollUserCredentials")]
		void EnrollUserCredentials(Ticket secOfficer, Ticket owner, Credential credential);

		/* Delete specific credentials for specific user
		*/
		[OperationContract()]
		[WebInvoke(Method = "DELETE",
			ResponseFormat = WebMessageFormat.Json,
			BodyStyle = WebMessageBodyStyle.Wrapped,
			UriTemplate = "DeleteUserCredentials")]
		void DeleteUserCredentials(Ticket secOfficer, Ticket owner, Credential credential);

		/* Enroll specific credentials for Non AD user without user authentication
		*/
		[OperationContract()]
		[WebInvoke(Method = "PUT",
			ResponseFormat = WebMessageFormat.Json,
			BodyStyle = WebMessageBodyStyle.Wrapped,
			UriTemplate = "EnrollAltusUserCredentials")]
		void EnrollAltusUserCredentials(Ticket secOfficer, User user, Credential credential);

		/* Delete specific credentials for Non AD user without user authentication
		*/
		[OperationContract()]
		[WebInvoke(Method = "DELETE",
			ResponseFormat = WebMessageFormat.Json,
			BodyStyle = WebMessageBodyStyle.Wrapped,
			UriTemplate = "DeleteAltusUserCredentials")]
		void DeleteAltusUserCredentials(Ticket secOfficer, User user, Credential credential);

		/* Get specific attribute for specific user
		*/
		[OperationContract()]
		[WebInvoke(Method = "POST",
			ResponseFormat = WebMessageFormat.Json,
			BodyStyle = WebMessageBodyStyle.Wrapped,
			UriTemplate = "GetUserAttribute")]
		Attribute GetUserAttribute(Ticket ticket, User user, String attributeName);

		/* Put specific attribute to specific user
		*/
		[OperationContract()]
		[WebInvoke(Method = "PUT",
			ResponseFormat = WebMessageFormat.Json,
			BodyStyle = WebMessageBodyStyle.Wrapped,
			UriTemplate = "PutUserAttribute")]
	void PutUserAttribute(Ticket ticket, User user, String attributeName,
			AttributeAction action, Attribute attributeData);

		/* Self Unlock user account
		*/
		[OperationContract()]
		[WebInvoke(Method = "POST",
			ResponseFormat = WebMessageFormat.Json,
			BodyStyle = WebMessageBodyStyle.Wrapped,
			UriTemplate = "UnlockUser")]
		void UnlockUser(User user, Credential credential);

		/* Call for credential specific Custom Action.
		*/
		[OperationContract()]
		[WebInvoke(Method = "POST",
			ResponseFormat = WebMessageFormat.Json,
			BodyStyle = WebMessageBodyStyle.Wrapped,
			UriTemplate = "CustomAction")]
		String CustomAction(Ticket ticket, User user, Credential credential, UInt16 actionId);

		/* Check if enrollment allowed for specific user
		*/
		[OperationContract()]
		[WebInvoke(Method = "POST",
			ResponseFormat = WebMessageFormat.Json,
			BodyStyle = WebMessageBodyStyle.Wrapped,
			UriTemplate = "IsEnrollmentAllowed")]
		void IsEnrollmentAllowed(Ticket secOfficer, User user, String credentialId);
		}
}
~~~  

# Methods
The methods available through the Web Enrollment Servicec API are as follows.  

## GetUserCredentials method  
The GetUserCredentials method allows the caller to request information about the credentials that have been enrolled by a specified user. GetUserCredentials should be implemented as HTTP GET using JSON as the response format.

### Syntax

~~~
List<String> GetUserCredentials(String userName, UInt16 userNameType);
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left" colspan="2">Description</th>
  </tr>
  <tr>
  <td valign="top">username</td>
  <td valign="top" colspan="2">Name of the user whose credential needs to be verified.</td>
  </tr>
  <tr>
  <td valign="top">userNameType</td>
  <td valign="top" colspan="2">The specific format of the user name is provided in the first parameter. Valid formats are: </td>
  </tr>
  <tr>
  <td valign="top">&nbsp;</td>
  <td valign="top">3</td>
  <td valign="top">Windows NT® 4.0 account name (for example, digital_persona\klozin). The domainonly version includes trailing backslashes (\\).</td>  
  </tr>  
  <tr>
  <td valign="top">&nbsp;</td>
  <td valign="top">4</td>
  <td valign="top">Account name format used in Microsoft® Windows NT® 4.0. For example, "someone".</td>  
  </tr>
  <tr>
  <td valign="top">&nbsp;</td>
  <td valign="top">5</td>
  <td valign="top">GUID string that the IIDFromString function returns (for example, {4fa050f0-f561-11cfbdd9-00aa003a77b6}).</td>  
  </tr>  
  <tr>
  <td valign="top">&nbsp;</td>
  <td valign="top">7</td>
  <td valign="top">User Principal Name (UPN). For example, someone@mycompany.com.</td>  
  </tr>      
  <tr>
  <td valign="top">&nbsp;</td>
  <td valign="top">8</td>
  <td valign="top">User SID string (for example, S-1-5-21-1004).</td>  
  </tr>  
  <tr>
  <td valign="top">&nbsp;</td>
  <td valign="top">9</td>
  <td valign="top">DigitalPersona user name format (user name associated with DigitalPersona identity database).</td>  
  </tr>                
</table>

### Return values
List of credential IDs of all credentials enrolled by a user. For full details on all supported credential IDs, see [this section](wes-cred-format.md).  

### Examples  

~~~
https://www.somecompany.com/DPWebEnrollervice.svc/						GetUserCredentials?
user=john.doe@somecompany.com&type=6
~~~  

Here the user name is john.doe@somecompany.com, the user name type is 6 which means UPN.  

The example response would be the following:
~~~
{"GetUserCredentialsResult":
	[
		"D1A1F561-E14A-4699-9138-2EB523E132CC",
		"AC184A13-60AB-40e5-A514-E10F777EC2F9",
		"8A6FCEC3-3C8A-40c2-8AC0-A039EC01BA05"
	]
}
~~~  

This response means that the user john.doe@somecompany.com has password, fingerprint and PIN credentials enrolled.  

## GetEnrollmentData method  

The GetEnrollmentData method is a utility method which allows the caller to get credential enrollment specific data.  

For example, Live Question authentication may require knowing which particular questions were enrolled, for fingerprint - fingerprint positions enrolled, etc.
GetEnrollmentData should be implemented as HTTP GET using JSON as response format.  

### Syntax  

~~~  
String GetEnrollmentData(String userName, UInt16 userNameType, String credentialId);
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left" colspan="2">Description</th>
  </tr>
  <tr>
  <td valign="top">username</td>
  <td valign="top" colspan="2">Name of the user whose enrollment data needs to be obtained.</td>
  </tr>
  <tr>
  <td valign="top">userNameType</td>
  <td valign="top" colspan="2">The specific format of the user name is provided in the first parameter. Valid formats are: </td>
  </tr>
  <tr>
  <td valign="top">&nbsp;</td>
  <td valign="top">3</td>
  <td valign="top">Windows NT® 4.0 account name (for example, digital_persona\klozin). The domainonly version includes trailing backslashes (\\).</td>  
  </tr>  
  <tr>
  <td valign="top">&nbsp;</td>
  <td valign="top">4</td>
  <td valign="top">Account name format used in Microsoft® Windows NT® 4.0. For example, "someone".</td>  
  </tr>
  <tr>
  <td valign="top">&nbsp;</td>
  <td valign="top">5</td>
  <td valign="top">GUID string that the IIDFromString function returns (for example, {4fa050f0-f561-11cfbdd9-00aa003a77b6}).</td>  
  </tr>  
  <tr>
  <td valign="top">&nbsp;</td>
  <td valign="top">7</td>
  <td valign="top">User Principal Name (UPN). For example, someone@mycompany.com.</td>  
  </tr>      
  <tr>
  <td valign="top">&nbsp;</td>
  <td valign="top">8</td>
  <td valign="top">User SID string (for example, S-1-5-21-1004).</td>  
  </tr>  
  <tr>
  <td valign="top">&nbsp;</td>
  <td valign="top">9</td>
  <td valign="top">DigitalPersona user name format (user name associated with DigitalPersona identity database).</td>  
  </tr>   
  <tr>
    <td valign="top">credentialId</td>
    <td valign="top" colspan="2">Unique ID of credential whose data needs to be returned. For full details on all supported credential IDs, see [this section](wes-cred-format.md).</td>
  </tr>                   
</table>

### Return values
Base64Url encoded enrollment data. The format of such enrollment data is credential-specific and will be detailed in separate document(s).  

### Example  

~~~
https://www.somecompany.com/DPWebEnrollService.svc/GetEnrollmentData?
user=john.doe@somecompany.com&type=6&cred_id=AC184A13-60AB-40e5-A514-E10F777EC2F9
~~~

Here the user name is john.doe@somecompany.com, the user name type is 6 which means UPN name and the credential ID is {AC184A13-60AB-40e5-A514-E10F777EC2F9}, which means information about a fingerprint credential was requested.  

### Note  

The use of braces {} is considered unsafe in URLs (see RFC 1738), which is why "braceless" GUID representation is used in the API.
The example response would be the following:  

~~~
{"GetEnrollmentDataResult":"Z3NhZGhhc2Rma0FTREZLYWZyZGtB"}  
~~~

This response shows fingerprint enrollment data for the user john.doe@somecompany.com.

## CreateUser method

The CreateUser method creates a user account in the DigitalPersona database. This method makes sense only if DigitalPersona is used as the  backend server. In DigitalPersona AD user account is created in Active Directory and Administrator must use standard Active Directory tools to create it.  

CreateUser should be implemented as HTTP PUT using JSON as the response format.

### Syntax

~~~
void CreateUser(Ticket secOfficer, User user, String password);
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left" colspan="2">Description</th>
  </tr>
  <tr>
  <td valign="top">secOfficer</td>
  <td valign="top" colspan="2">JSON Web Token of Security Officer. Security Officer should use the DigitalPersona Web AUTH Service to authenticate himself and acquire this token. Token must be valid to call succeeded. To be valid token must be:<BR><BR>1. Issued no longer than 10 minutes before the operation,<BR>2. One of the Primary credentials must be used to acquire this token and<BR>3. The token owner must have the necessary rights to create the user account in the DigitalPersona AD/LDS  database.</td>
  </tr>
  <tr>
  <td valign="top">user	</td>
  <td valign="top" colspan="2">User account that needs to be created. See the definition of the User class  <A HREF="https://hidglobal.github.io/digitalpersona-access-management-services/was.html#user-class">here.</A></td>
  </tr>  
  <tr>
  <td valign="top">password</td>
  <td valign="top" colspan="2">String which represents the initial password for newly created user account. We cannot create user account without setting initial. Password must satisfy password complexity policy set for AD LDS database otherwise call will fail.</td>
  </tr>    
</table>

### Example

~~~
https://www.somecompany.com/DPWebEnrollService.svc/CreateUser
~~~

Below is an example of an HTTP BODY of CreateUser request.

~~~
{
	"secOfficer":{"jwt":"Z3NhZGhhc2Rma0FTREZLYWZyZGtB"},
	"user":
	{
		"name":"john.doe@somecompany.com",
		"type":6
	},
	"password":"aaaAAA123"
}
~~~

The call above creates an account in DigitalPersona (AD LDS) database for Active Directory user with UPN name john.doe@somecompany.com.

## DeleteUser method  

The DeleteUser method deletes a user account from the DigitalPersona database.  

This method makes sense only if DigitalPersona is used as the backend server. In DigitalPersona AD, the user account is deleted in Active Directory and an Administrator should use the standard Active Directory tools to do so.  

DeleteUser should be implemented as HTTP DELETE using JSON as the response format.  

### Syntax  
~~~
void DeleteUser(Ticket secOfficer, User user);
~~~  

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">secOfficer</td>
  <td valign="top">JSON Web Token of Security Officer. Security Officer should use the DigitalPersona Web AUTH Service to authenticate himself and acquire this token. Token must be valid for call to succeed. To be valid, the  token must be:<BR><BR>1. Issued no longer than 10 minutes before the operation,<BR>2. One of the Primary credentials must be used to acquire this token and<BR>3. The token owner must have the necessary rights to create the user account in the DigitalPersona AD/LDS  database.</td>
  </tr>
  <tr>
  <td valign="top">user</td>
  <td valign="top">The user account that needs to be deleted. See the definition of the User class <A HREF="https://hidglobal.github.io/digitalpersona-access-management-services/was.html#user-class">here.</A></td>
  </tr>  
</table>  

### Examples  

Below is example of a URL that can be used to activate a DeleteUser request:

~~~
https://www.somecompany.com/DPWebEnrollService.svc/DeleteUser
~~~

Below is an example of the HTTP BODY for a  DeleteUser request:

~~~
{
	"secOfficer":{"jwt":"Z3NhZGhhc2Rma0FTREZLYWZyZGtB"},
	"user":
	{
		"name":"john.doe@somecompany.com",
		"type":6
	}
}
~~~

The call above deletes an account from DigitalPersona (AD LDS) database for Active Directory user with UPN name john.doe@somecompany.com.  

### EnrollUserCredentials method  

The EnrollUserCredentials method enrolls (or re-enrolls) specific credentials for a named user and stores their credential data in the DigitalPersona AD database. This method will work for both DigitalPersona AD and DigitalPersona LDS backend servers.
EnrollUserCredentials should be implemented as HTTP PUT using JSON as response format.  

### Syntax

~~~
void EnrollUserCredentials(Ticket secOfficer, Ticket owner, Credential credential);
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">secOfficer</td>
  <td valign="top">JSON Web Token of the Security Officer. The Security Officer should use the DigitalPersona Web AUTH Service to authenticate himself and acquire this token. The token must be valid for the call to succeed. To be valid, a token must be:
	<BR><BR>
	1. Issued no longer than 10 minutes before the operation,<BR>2. One of the Primary credentials must be used to acquire this token and<BR>3. The token owner must have the necessary rights to create the user account in the DigitalPersona AD/LDS  database.<BR><BR>
	<b>NOTE</b>: This parameter is optional. If the user has the necessary rights to enroll himself (i.e. self-enrollment is allowed), the caller may provide "null" to this parameter.</td>
	</tr>
	<tr>
	<td>owner</td>
	<td>JSON Web Token of the owner of credentials. User should use DigitalPersona Web AUTH Service to authenticate itself and acquire this token. Token must be valid to call succeeded. To be valid, a token must be:
	<BR><BR>1. Issued no longer than 10 minutes before the operation, and<BR>
	2. One of the Primary credentials (or the same credentials) must be used to acquire this token.</td>
  </tr>
	<tr><td>credential</td>
	<td>Credential to be enrolled. Note that the Data field of this parameter is specific to each credential. See the definition of the Credential class  <A HREF="https://hidglobal.github.io/digitalpersona-access-management-services/wes-cred-format.html#credentials-class">here.</A></td></tr>
</table>  

### Examples
Below is an example of a URL which can be used to PUT the EnrollUserCredentials request:
~~~
https://www.somecompany.com/DPWebEnrollService.svc/EnrollUserCredentials
Below is example of HTTP BODY of EnrollUserCredentials request:
{
	"secOfficer":{"jwt":"Z3NhZGhhc2Rma0FTREZLYWZyZGtB"},
	"owner":{"jwt":"Z3NhZGhhc2Rma0FTREZLYWZyZGtB"},
	"credential":
	{
		"id":"AC184A13-60AB-40e5-A514-E10F777EC2F9",
		"data":"Z3NhZGhhc2Rma0FTREZLYWZyZGtB"
	}
}
~~~  

The call above enrolls fingerprint credentials for the user specified in the "owner" token.

## DeleteUserCredentials method

The DeleteUserCredentials method deletes (un-enrolls) specific credentials for a user and removes the credential data from the DigitalPersona database. This method will work for both DigitalPersona AD and DigitalPersona LDS backend servers.  

DeleteUserCredentials should be implemented as HTTP DELETE using JSON as the response format.

#### Syntax
~~~
void DeleteUserCredentials(Ticket secOfficer, Ticket owner, Credential credential);
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">secOfficer</td>
  <td valign="top">JSON Web Token of the Security Officer. The Security Officer should use the DigitalPersona Web AUTH Service to authenticate himself and acquire this token. The token must be valid for the call to succeed. To be valid, a token must be:
	<BR><BR>
	1. Issued no longer than 10 minutes before the operation,<BR>2. One of the Primary credentials must be used to acquire this token and<BR>3. The token owner must have the necessary rights to create the user account in the DigitalPersona AD/LDS  database.<BR><BR>
	<b>NOTE</b>: This parameter is optional. If the user has the necessary rights to enroll himself (i.e. self-enrollment is allowed), the caller may provide "null" to this parameter.</td>
	</tr>
	<tr>
	<td>owner</td>
	<td>JSON Web Token of the owner of credentials. User should use DigitalPersona Web AUTH Service to authenticate itself and acquire this token. Token must be valid to call succeeded. To be valid, a token must be:
	<BR><BR>1. Issued no longer than 10 minutes before the operation, and<BR>
	2. One of the Primary credentials (or the same credentials) must be used to acquire this token.</td>
  </tr>
	<tr><td>credential</td>
	<td>Credential to be deleted. Note that the Data field of this parameter is specific to each credential. See the definition of the Credential class <A HREF="https://hidglobal.github.io/digitalpersona-access-management-services/wes-cred-format.html#credentials-class">here.</A></td>
	</tr>
</table>

### Example
Below is an example of a URL which can be used to DELETE DeleteUserCredentials request:
~~~
https://www.somecompany.com/DPWebEnrollService.svc/DeleteUserCredentials
Below is example of HTTP BODY of DeleteUserCredentials request:
{
	"secOfficer":{"jwt":"Z3NhZGhhc2Rma0FTREZLYWZyZGtB"},
	"owner":{"jwt":"Z3NhZGhhc2Rma0FTREZLYWZyZGtB"},
	"credential":
	{
		"id":"AC184A13-60AB-40e5-A514-E10F777EC2F9",
		"data":"Z3NhZGhhc2Rma0FTREZLYWZyZGtB"
	}
}
~~~
The call above deletes any fingerprint credentials for the user specified in the "owner" token.  

## EnrollAltusUserCredentials method
The EnrollAltusUserCredentials method enrolls (or re-enrolls) specific credentials for specific user and store credential data in the DigitalPersona database.  

This method will work only for Non AD users and the DigitalPersona LDS Server backend. For AD users, the function will return Access Denied.  

This method is different from EnrollUserCredentials in that it allows enrolling user credentials without the user being previously authenticated. Only authentication of the Security Officer is required.  

By default the DigitalPersona Server requires the user to be authenticated to enroll credentials, so this functionality must be enabled through the GPO setting: AllowSecurityOfficerEnrollment.
- If this GPO is not configured or is set to 0, the EnrollAltusUserCredentials function will always return an 'Access Denied' error.  
- If this GPO set to 1 and Security Officer has rights to enroll this particular user, enrollment will be performed.  

### Syntax
~~~
void EnrollAltusUserCredentials(Ticket secOfficer, User user, Credential credential);
~~~


<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">secOfficer</td>
  <td valign="top">JSON Web Token of the Security Officer. The Security Officer should use the DigitalPersona Web AUTH Service to authenticate himself and acquire this token. The token must be valid for the call to succeed. To be valid, a token must be:
	<BR><BR>
	1. Issued no longer than 10 minutes before the operation,<BR>2. One of the Primary credentials must be used to acquire this token and<BR>3. The token owner must have the necessary rights to create the user account in the DigitalPersona LDS  database.</td>
	</tr>
	<tr>
	<td>user</td>
	<td>User which credentials needs to be enrolled. Only Non AD users can be accepted by this function. See the definition of the User class <A HREF="https://hidglobal.github.io/digitalpersona-access-management-services/was.html#user-class">here.</A></td>
  </tr>
	<tr><td>credential</td>
	<td>Credential to be enrolled. Note that the Data field of this parameter is specific to each credential. See the definition of the Credential class  <A HREF="https://hidglobal.github.io/digitalpersona-access-management-services/wes-cred-format.html#credentials-class">here.</A></td></tr>
</table>

EnrollAltusUserCredentials should be implemented as HTTP PUT using JSON as the response format.  

Below is an example of a URL which can be used to PUT the  EnrollAltusUserCredentials request:  
~~~
https://www.digitalpersona.com/DPWebEnrollService.svc/EnrollAltusUserCredentials
Below is example of HTTP BODY of EnrollAltusUserCredentials request:
{
	"secOfficer":{"jwt":"Z3NhZGhhc2Rma0FTREZLYWZyZGtB"},
	"user":
		{
			"name":"someone",
			"type":9
		},
		"credential":
		{
			"id":"AC184A13-60AB-40e5-A514-E10F777EC2F9",
			"data":"Z3NhZGhhc2Rma0FTREZLYWZyZGtB"
		}
}
~~~
The call above enrolls fingerprint credentials for the Non AD user "someone".  

NOTE: This method can be used to Reset a Non AD user’s password without user intervention. It can aslo be used to Randomize a user's password. To randomize a user password the caller must provide "null" in the data parameter of the "credential" class. Below is an example of a password randomization request:
~~~
{
	"secOfficer":{"jwt":"Z3NhZGhhc2Rma0FTREZLYWZyZGtB"},
	"user":
		{
			"name":"someone ",
			"type":9
		},
		 "credential":
		{
			"id":" D1A1F561-E14A-4699-9138-2EB523E132CC ",
			"data":null
		}
}
~~~

## DeleteAltusUserCredentials method  

The DeleteAltusUserCredentials method deletes (un-enrolls) specific credentials for a specific Non AD user and removes the associated credential data from the DigitalPersona database. This method will work only for Non AD users. For AD users it will return Access Denied.  

This method is different from the DeleteUserCredentials in that it allows deleting user credentials without the user being previously authenticated. Only authentication of Security Officer is required.  

By default the DigitalPersona Server requires the user to be authenticated to delete credentials, so this functionality must be enabled through the GPO setting: AllowSecurityOfficerEnrollment.
- If this GPO is not configured or is set to 0, the DeleteAltusUserCredentials function will always return an 'Access Denied' error.  
- If this GPO set to 1 and the Security Officer has rights to delete this particular user, credential deletion will be performed.  

### Syntax
~~~
void DeleteAltusUserCredentials(Ticket secOfficer, User user, Credential credential);
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">secOfficer</td>
  <td valign="top">JSON Web Token of the Security Officer. The Security Officer should use the DigitalPersona Web AUTH Service to authenticate himself and acquire this token. The token must be valid for the call to succeed. To be valid, a token must be:
	<BR><BR>
	1. Issued no longer than 10 minutes before the operation,<BR>2. One of the Primary credentials must be used to acquire this token and<BR>3. The token owner must have a rights to enroll user in the DigitalPersona database (LDS version) or in Active Directory (AD version).<BR><BR>
	NOTE: This parameter is optional. If the user has the rights to enroll himself (i.e. self-enrollment is allowed), the caller may provide "null" to this parameter.</td>
	</tr>
	<tr>
	<td>user</td>
	<td>User which credentials needs to be deleted. See the definition of the User class <A HREF="https://hidglobal.github.io/digitalpersona-access-management-services/was.html#user-class">here.</A>.</td>
  </tr>
	<tr><td>credential</td>
	<td>Credential to be deleted. Note that the Data field of this parameter is specific to each credential. See the definition of the Credential class <A HREF="https://hidglobal.github.io/digitalpersona-access-management-services/wes-cred-format.html#credentials-class">here.</A></td>
	</tr>
</table>

DeleteAltusUserCredentials should be implemented as HTTP DELETE using JSON as the response format.  

Below is an example of a URL which can be used to submit a  DeleteAltusUserCredentials request:
~~~
https://www.digitalpersona.com/DPWebEnrollService.svc/DeleteAltusUserCredentials
Below is example of HTTP BODY of DeleteAltusUserCredentials request:
{
	"secOfficer":{"jwt":"Z3NhZGhhc2Rma0FTREZLYWZyZGtB"},
	"user":
	{
		"name":"someone ",
		"type":9
  },
	"credential":
	{
		"id":"AC184A13-60AB-40e5-A514-E10F777EC2F9",
		"data":"Z3NhZGhhc2Rma0FTREZLYWZyZGtB"
	}
}
~~~
The call above deletes the fingerprint credentials for the Non AD user "someone".  

## GetUserAttribute method

The GetUserAttribute method requests some public (biographic) information about a user, such as their user surname, date of birth, e-mail address, etc.  

GetUserAttribute should be implemented as HTTP POST using JSON as the response format.  

### Syntax
~~~
Attribute GetUserAttribute(Ticket ticket, User user, String attributeName);
~~~
<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">ticket</td>
  <td valign="top">JSON Web Token of user requesting the information. This could be an attribute owner, Security Officer, Administrator or any user who has rights to read this information. The token must be valid for the call to succeed. To be valid, a token must be:
	<BR><BR>
	1. Issued no longer than 10 minutes before the operation,<BR>2. One of the Primary credentials must be used to acquire this token and<BR>3. The token owner must have the necessary rights to read this user's attribute in the  DigitalPersona (AD LDS) or DigitalPersona AD (Active Directory) database.</td>
  </tr>
	<tr>
	<td valign="top">user</td>
	<td valign="top">The user requesting the information. See the definition of the User class <A HREF="https://hidglobal.github.io/digitalpersona-access-management-services/was.html#user-class">here.</A></td>
	</tr>
	<tr>
	<td valign="top">attributeName</td>
	<td valign="top">Name of the attribute requested. Both AD and AD LDS are LDAP databases, so this name must be the Ldap-Display-Name of the Attribute Schema in the User object of the LDAP database.</td>
	</tr>
</table>

### Return values  

JSON representation of object of Attribute class will be returned if the call succeeds. For details on the Attribute class, see the [Attribute class](#attribute-class topic.  

### Examples
Below is an example of a URL which can be used to POST a GetUserAttribute request.  
~~~
https://www.somecompany.com/DPWebEnrollService.svc/GetUserAttribute
Below is example of HTTP BODY of GetUserAttribute request:
{
	"ticket":{"jwt":"Z3NhZGhhc2Rma0FTREZLYWZyZGtB"},
	"user":
	{
		"name":"john.doe@somecompany.com",
		"type":6
	},
	"attributeName":"sn"
}
~~~

The call above requests the "Surname" attribute for an Active Directory user with a UPN name of john.doe@somecompany.com.  

An example of the return value for this call might be:  

~~~
{"GetUserAttributeResult":
{
"type":3,
"values":["Lozin"]
}
}
~~~
## PutUserAttribute method  

The PutUserAttribute method writes, updates or clears specific public data (attribute) for the named user.  

This method makes sense only when using DigitalPersona as the backend server (AD LDS). For Active Directory, the Administrator must use standard AD tools to manage attributes (with the exception of DP specific attributes).  

PutUserAttribute should be implemented as HTTP PUT using JSON as the response format.  

### Syntax
~~~
void PutUserAttribute(Ticket ticket, User user, String attributeName,
		 AttributeAction action, Attribute attributeData);
~~~		 


<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">ticket</td>
  <td valign="top">JSON Web Token of user requesting modification of the attribute. This could be an attribute owner, Security Officer, Administrator or any user who has rights to write this information. The token must be valid for the call to succeed. To be valid, a token must be:
	<BR><BR>
	1. Issued no longer than 10 minutes before the operation,<BR>2. One of the Primary credentials must be used to acquire this token and<BR>3. The token owner must have the necessary rights to write this user's attribute to  the DigitalPersona (AD LDS) or DigitalPersona AD (Active Directory) database.</td>
  </tr>
	<tr>
	<td valign="top">user</td>
	<td valign="top">The user whose attribute is to be modified. See the definition of the User class <A HREF="https://hidglobal.github.io/digitalpersona-access-management-services/was.html#user-class">here.</A></td>
	</tr>
	<tr>
	<td valign="top">attributeName</td>
	<td valign="top">Name of the attribute requested. Both AD and AD LDS are LDAP databases, so this name must be the Ldap-Display-Name of the Attribute Schema in the User object of the LDAP database.</td>
	</tr>
	<tr>
	<td valign="top">action</td>
	<td valign="top">Action that needs to be taken. It could be Append, Update, Delete or Clear. For additional information, see the AttributAction parameters in the table <A HREF="https://hidglobal.github.io/digitalpersona-access-management-services/wes.html#attribute-action-params">here</A>.</td>
	</tr>
	<tr>
	<td valign="top">atttributeData</td>
	<td valign="top">Attribute data that needs to be written. For details on the Attribute class, see the topic <A HREF="https://hidglobal.github.io/digitalpersona-access-management-services/wes.html#attribute-class">Attribute class</A></td>
	</tr>
</table>

### Examples

Below is an example of a URL which can be used to submit a PutUserAttribute request:  
~~~
https://www.somecompany.com/DPWebEnrollService.svc/PutUserAttribute
Below is example of HTTP BODY of PutUserAttribute request:
{
	"ticket":{"jwt":"Z3NhZGhhc2Rma0FTREZLYWZyZGtB"},
	"user":
	{
		"name":"john.doe@somecompany.com",
		"type":6
	},
	"attributeName":"sn",
	"action":2,
	"attributeData":
	{
		"type":3,
		"values":["smith"]
	}
}
~~~

The call above requests an update to the "Surname" attribute for the Active Directory user with a UPN name of john.doe@somecompany.com. The new value should be "smith".

# Data Contracts
Below are the Data Contracts used in the Web Enrollment Service (WES) API. Only the data specific to Web Enrollment is described below. For a description of additional data contracts used in the Web Authentication Service (WAS) API, see  <A HREF="https://hidglobal.github.io/digitalpersona-access-management-services/was.html#was-data-contracts">this section</A>.  

## AttributeAction enumeration  

AttributeAction enumeration specifies the action to be taken with an attribute through the PutUserAttribute call.

~~~
[DataContract]
public enum AttributeAction
{
	[EnumMember]
	Clear = 1,
	[EnumMember]
	Update = 2,
	[EnumMember]
	Append = 3,
	[EnumMember]
	Delete = 4,
}
~~~
<A NAME="attribute-action-params"></A>

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  	<td valign="top">clear</td>
  	<td valign="top">Attribute must be cleared. The attributeData argument of the PutUserAttribute method will be ignored and can be "null".</td>
	</tr>
	<tr>
		<td valign="top">Update</td>
		<td valign="top">Attribute will be updated. All previous data in the attribute will be cleared.</td>
	</tr>
	<tr>
  	<td valign="top">Append</td>
  	<td valign="top">The data will be appended to data which already exists in the attribute. Makes sense only for multivalued attributes.</td>
  </tr>
	<tr>
		<td valign="top">Delete</td>
		<td valign="top">The data provided in the attributeData argument of the PutUserAttribute method will be deleted from the attribute. Makes sense only for multivalued attributes.</td>
	</tr>
</table>  

## AttributeType enumeration   
The AttributeType enumeration specifies the value type of the attribute.

~~~
DataContract]
public enum AttributeType
{
	[EnumMember]
	Boolean = 1,
	[EnumMember]
	Integer = 2,
	[EnumMember]
	String = 3,
	[EnumMember]
	Blob = 4,
}
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Value</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">Boolean</td>
  <td valign="top">The attribute has Boolean value(s).</td>
  </tr>
	<tr>
	<td valign="top">Integer</td>
	<td valign="top">The attribute has Integer value(s).</td>
	</tr>
	<tr>
	<td valign="top">String</td>
	<td valign="top">The attribute has String value(s).</td>
	</tr>
	<tr>
	<td valign="top">Blob</td>
	<td valign="top">The attribute has Blob value(s).</td>
	</tr>
</table>

## <A NAME="attribute-class"></A> Attribute class  

The Attribute class is the Attribute representation in the Web Enrollment API.

~~~
[DataContract]
	public class Attribute
	{
		[DataMember]
		public AttributeType type { get; set; }
		[DataMember]
		public List<Object> values { get; set; }
	}
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameters</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">type</td>
  <td valign="top">Type of Attribute value(s).</td>
  </tr>
	<tr>
	<td valign="top">values</td>
	<td valign="top">	Values of attribute. We assume all attributes are multivalued because singlevalued attributes are a subset  of multivalued attributes.<BR><BR>
	Below we give details of Json representation for attributes of different types.</td>
	</tr>
</table>

### Boolean attributes
For Boolean attributes, the Json representation is Json Boolean. Below is an example of the "isDeleted" attribute in Active Directory.

~~~
{
	"type":1,
	"values":[true]
}
~~~

The attribute above claims user is deleted from Active Directory.
### Integer attributes

For Integer attributes, the Json representation is Json Integer. It is used for all types of integers, such as  Uin8, Uint16, Uint32 and Uint64. Timestamps are  represented as long integers (Uint64). Below is an example of the "userAccountControl" attribute in Active Directory.

~~~
{
	"type":2,
	"values":[65536]
}
~~~

The attribute above claims users’ password never expires.  

### String attributes  

For String attributes, the Json representation is Json String.  

Below is an example of the "otherMailbox" (users' e-mail addresses) attribute in Active Directory.  

~~~
{
	"type":3,
	"values":["john.doe@somecompany.com","john.doe@mycompany.com"]
}
~~~

The attribute above contains all of the user's e-mail addresses.  

### Blob attributes

For Blob attributes,  the Json representation is the Json String. To convert the Blob to a string, we use  Base64UrlEncoding. Below is an example of the  "thumbnailPhoto" attribute in Active Directory.

~~~
{
	"type":4,
	"values":["Z3NhZGhhc2Rma0FTREZLYWZyZGtB"]
}
~~~

The attribute above has the Base64UrlEncoded user's  thumbnail photo.

## CustomAction method

The CustomAction method performs credential specific operations (custom actions) for a specified user.

CustomAction should be implemented as HTTP POST using JSON as the response format. For further details, see the WAS CustomAction method described <A HREF="https://hidglobal.github.io/digitalpersona-access-management-services/was.html#was-custom-action">here.</A>.  

### Syntax

~~~
void CustomAction(Ticket ticket, User user, Credential credential, UInt16 actionId);
~~~  

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">ticket</td>
  <td valign="top">ticket	JSON Web Token of the person initiating the CustomAction operation. This parameter is optional since not all CustomActions require Access Control. Therefore the caller may provide "null" to this parameter.</td>
  </tr>
	<tr>
   <td valign="top">user</td>
   <td valign="top">The user whose record the CustomAction is performed upon. This parameter is optional since not all CustomAction operations are performed upon a specific user. Therefore the caller may provide "null" to this parameter.</td>
   </tr>
	 <tr>
 <td valign="top">credential</td>
 <td valign="top">Credential to which CustomAction needs to be performed. Id attribute of Credential class should point to valid DigitalPersona Credential. Data attribute of Credential class should point to Base64Url encoded credential specific data and could be set to "null".</td>
 </tr> 	
</table>

Below is an example of a URL used to POST a CustomAction request.

~~~
https://www.digitalpersona.com/DPWebEnrollService.svc/CustomAction
~~~

Below is example of the HTTP BODY of the CustomAction request.

~~~
{
	"ticket":{"jwt":"Z3NhZGhhc2Rma0FTREZLYWZyZGtB"},
	"user":
	{
		"name":"someone",
		"type":9
	},
	"credential":
	{
		"id":"AC184A13-60AB-40e5-A514-E10F777EC2F9",
		"data":"Z3NhZGhhc2Rma0FTREZLYWZyZGtB"
	},
	"actionId":9
}
~~~

The call above sends a CustomAction request to fingerprint credentials with actionId 6 for DigitalPersona user "someone".
This call should return Base64Encoded output data of CustomAction call. The returned information is credential specific **<mark style="color:Red;"> and should be provided "Web Authentication Service Credentials Data Format" document.</mark>**  

## UnlockUser method  

The UnlockUser method performs self unlocking for a currently locked user.  

### Syntax  

~~~
void UnlockUser(User user, Credential credential);
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">user</td>
  <td valign="top">The user whose account needs to be unlocked.</td>
  </tr>
	<tr>
  <td valign="top">credential</td>
  <td valign="top">A valid user credential (including the  Recovery Questions credential) that is to be verified before the account gets unlocked.</td>
  </tr> 	
</table>

UnlockUser should be implemented as an HTTP POST using JSON as the response format.  

Below is an example of a URL  used to POST an UnlockUser request.  

~~~
https://www.digitalpersona.com/DPWebEnrollService.svc/UnlockUser
~~~

**NOTE**: For a user to be able to unlock their own account, the following GPO must be enabled on the DigitalPersona AD or LDS server.  

*Allow users to unlock their Windows account using DigitalPersona Recovery Questions.*
