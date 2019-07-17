---
layout: default
title: Web Secret Management Service (WSMS)  
has_toc: false
nav_order: 3
---

{% include header.html %}

## Web Secret Management Service

The Web Secret Management Service (WSMS) is part of the DigitalPersona Web Secret Service Provider (WSSP), a set of Web Services whose goal is to provide Secret Management functionality such as writing, reading and deleting protected data (Secrets). Generally speaking the main goal is to give access to Secret Management by applications running on computers outside the firewall. The actual secrets are stored protected in the DigitalPersona Server Database and to access them WSSP needs to send a request to the DigitalPersona Server.  

The goal of the DigitalPersona Web Secret Management Service (WSMS) is to provide Secret Management. By Secret management we mean the following operations:

- Read Secret content  
- Create and Modify Secret content
- Delete Secret

WSMS is implemented as a GUI-less Web Service. It
should be located inside the firewall where it will direct secret management requests to the DigitalPersona Server over DCOM. The DigitalPersona Server is the only one who can process secret management requests.  

WSMS allows secret management only to authenticated users whose authenticated credentials satisfy the authentication policy provided by the Web Authentication Policy Service.  

WSMS opens one or several Web endpoints listening on the HTTPS protocol. These endpoints may be designated as accessible from an intranet only or as additionally available from the internet.  

HTTPS (Transport Layer Security) is used by the Web Service in order to simplify client side implementation and protect user Secrets from man-in-the-middle attacks during transactions. This requires the Web Service hosting environment to have a Certificate which is accepted by the Client.  

## IDPWebSecretMgr interface
We define the IDPWebSecretMgr interface as a WCF interface.

~~~
namespace DPWcfSecretService
{
	[ServiceContract]
	public interface IDPWebSecretMgr
		{
		/*
		* Return policy for specific secret, specific user and specific action
		*/
		[OperationContract()]
		[WebInvoke(Method = "GET",
			ResponseFormat = WebMessageFormat.Json,
			BodyStyle = WebMessageBodyStyle.Wrapped,
			UriTemplate = "GetAuthPolicy?user={userName}&type={nameType}&secret=
				{secretName}&action={action}")]
			List<Policy> GetAuthPolicy(String userName, UInt16 nameType,
				String secretName, ResourceActions action);
		/*
		* Return policy for specific secret, specific user and specific action
		*/
		[OperationContract()]
		[WebInvoke(Method = "GET",
			ResponseFormat = WebMessageFormat.Json,
			BodyStyle = WebMessageBodyStyle.Wrapped,
			UriTemplate = "DoesSecretExist?user={userName}&type={nameType}&secret=
				{secretName}")]
      bool DoesSecretExist(String userName, UInt16 nameType, String secretName);
			/*
			* Read content of specific secret
			*/
     [OperationContract()]
     [WebInvoke(Method = "POST",
        ResponseFormat = WebMessageFormat.Json,
        BodyStyle = WebMessageBodyStyle.Wrapped,
        UriTemplate = "ReadSecret")]
        String ReadSecret(Ticket ticket, String secretName);
			/*
			* Write content to specific secret
			*/
     [OperationContract()]
     [WebInvoke(Method = "PUT",
				ResponseFormat = WebMessageFormat.Json,
       BodyStyle = WebMessageBodyStyle.Wrapped,
       UriTemplate = "WriteSecret")]
      void WriteSecret(Ticket ticket, String secretName, String secretData);
			/*
     * Delete content of specific secret
     */
     [OperationContract()]
     [WebInvoke(Method = "DELETE",
	     ResponseFormat = WebMessageFormat.Json,
       BodyStyle = WebMessageBodyStyle.Wrapped,
       UriTemplate = "DeleteSecret")]
			void DeleteSecret(Ticket ticket, String secretName);
		}

	[DataContract]
  public enum ResourceActions
		{
     [EnumMember]
     Read = 0,
     [EnumMember]
     Write = 1,
     [EnumMember]
     Delete = 2,
    }
	[DataContract]
  public class PolicyElement
		{
			public PolicyElement(String id)
        {
						cred_id = id;
        }
			[DataMember]
			public String cred_id { get; set; }
   }
	[DataContract]
	public class Policy
    {
				public Policy()
				{
					policy = new List<PolicyElement>();
       }
        [DataMember]
       	 public List<PolicyElement> policy { get; set; }
    }
    [DataContract]
    public class Ticket
    {
        [DataMember]
        public String jwt { get; set; } // JSON Web Token
    }
}
~~~
## Methods  

### GetAuthPolicy method  

The GetAuthPolicy() method returns a list of policies (credentials) which are required in order to access (read, write or delete) specific Secret of a named user.

#### Syntax  

~~~
List<Policy> GetAuthPolicy(String userName, UInt16 nameType, String secretName,
ResourceActions action);
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">userName</td>
  <td valign="top">The name of the user whose Secret is being managed.</td>
  </tr>
  <tr>
  <td valign="top">userNameType</td>
  <td valign="top">The format of the user name provided in the userName parameter. See **<mark style="color:Red;">“User class” on page 65</mark>** for a detailed description of user name formats.</td>
  </tr>
  <tr>
  <td valign="top">secretName</td>
  <td valign="top">The name of the Secret  to be accessed. For example, “SystemLogonInfo”.</td>
  </tr>
  <tr>
  <td valign="top">action</td>
  <td valign="top">The action to be performed on this Secret, i.e. Read, Write or Delete.</td>
  </tr>        
</table>

#### Return values   

Returns a list of policies (credentials) required to access the Secret.  

#### Notes  

GetAuthPolicy () is implemented as an HTTP GET using JSON as the response format.  

The following is an example of using GetAuthPolicy().

~~~
http://www.mycompany.com/DPWebSecretService.svc/GetAuthPolicy?user=someone@mycompany.com&type=6&secret=SystemLogonInfo&action=Read
~~~        

This above URL requests a list of policies required to read a secret named SystemLogonInfo, belonging to a user with the name someone@mycompany.com.  

An example of a response might be the following:

~~~
{"GetAuthPolicyResult":
	[
		{"policy":[
			{"cred_id":"AC184A13-60AB-40e5-A514-E10F777EC2F9"},
			{"cred_id":"8A6FCEC3-3C8A-40c2-8AC0-A039EC01BA05"}
		]},
		{"policy":[
			{"cred_id":"AC184A13-60AB-40e5-A514-E10F777EC2F9"},
			{"cred_id":"E750A180-577B-47f7-ACD9-F89A7E27FA49 "}
		]}
	]
}
~~~

This response means that the user someone@mycompany.com needs to provide either their “Fingerprint AND PIN” or their “Fingerprint AND Bluetooth” credentials in order to be allowed to read the SystemLogonInfo Secret. Note that “braceless” GUID representations must be used in all of our APIs for URLs and JSON representation.

### DoesSecretExist method

The DoesSecretExist method verifies whether or not a specific secret exists for a specified user.

#### Syntax

~~~
bool DoesSecretExist(String userName, UInt16 nameType, String secretName);
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">userName</td>
  <td valign="top">The name of the user whose Secret is  to be verified.
</td>
  </tr>
  <tr>
  <td valign="top">userNameType</td>
  <td valign="top">The format of the user name provided in the userName parameter. See

<mark style="color:Red;">“User class” on page 65  </mark>
for a detailed description of user name formats.
    </td>
  </tr>
  <tr>
  <td valign="top">secretName</td>
  <td valign="top">The name of the Secret to be verified. For example, "SystemLogonInfo".</td>
  </tr>     
</table>

#### Return values  

true – If secret with the specified name exists in the DigitalPersona database.  

false – If secret with the specified name exists in the DigitalPersona database.  

#### Notes  

DoesSecretExist () is implemented as an HTTP GET using JSON as the response format.  

The following is an example of using DoesSecretExist ().

~~~
http://www.mycompany.com/ DPWebSecretService.svc/ DoesSecretExist?user=
someone@mycompany.com&type=6&secret=SystemLogonInfo  
~~~

The URL above requests information about whether a secret with the name “SystemLogonInfo” exists for the user someone@mycompany.com.  

The following is an example of the HTTP response to the request.  

~~~
{"DoesSecretExistResult":true}  
~~~

This response means that a secret with the name “SystemLogonInfo” exists for the user someone@mycompany.com.  

### ReadSecret method  

The ReadSecret method returns the content of a specific secret for the user if the authentication provided in the ticket satisfies the policy.  

#### Syntax  

~~~
String ReadSecret(Ticket ticket, String secretName);
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">ticket</td>
  <td valign="top">The JSON Web Token returned by the Authentication Service.</td>
  </tr>
  <tr>
  <td valign="top">secretName</td>
  <td valign="top">The name of the Secret to be read.</td>
  </tr>   
</table>

#### Returns  

A string which represents the Base64url encoded secret content.  

#### Notes  

ReadSecret () is implemented as HTTP POST using JSON as the response format.  

The following URL is an example of using ReadSecret ().

~~~
http://www.mycompany.com/DPWebSecretService.svc/ReadSecret
~~~

Below is an example of an HTTP BODY of a ReadSecret request.  

~~~
{"secretName":"SystemLogonInfo","ticket":{"jwt":"eyJ0eXAiOiJKV1QiLA0KICJhbGc
iOiJIUzI1NiJ9.eyJpc3MiOiJqb2UiLA0KICJleHAiOjEzMDA4MTkzODAsDQogImh0dHA6Ly9leG
FtcGxlLmNvbS9pc19yb290Ijp0cnVlfQ.dBjftJeZ4CVP-B92K27uhbUJU1p1rwW1gFWFOEjXk"}}
~~~

Below is an example of a response.

~~~
{"ReadSecretResult":"eyJ0eXAiOiJKV1QiLA0KICJhbGciOiJIUzI1NiJ9"}   
~~~

### WriteSecret method  

The WriteSecret method writes the content of a user's specific secret if the authentication provided in the  ticket satisfies the policy.  

If a secret with the specified name does not exist, it will be created. There is no separate CreateSecret method or separate authorization for secret creation.

#### Syntax  

~~~
void WriteSecret(Ticket ticket, String secretName, String secretData);
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">ticket</td>
  <td valign="top">The JSON Web Token returned by the Authentication Service.</td>
  </tr>
  <tr>
  <td valign="top">secretName</td>
  <td valign="top">The name of the Secret to be written. For example, "SystemLogonInfo".</td>
  </tr>  
  <tr>
  <td valign="top">secretData</td>
  <td valign="top">Base64url encoded data to be written to the secret.</td>
  </tr>     
</table>

#### Returns  

None  

#### Notes  

WriteSecret () is implemented as an HTTP PUT using JSON as the response format.  

The following URL is an example of using WriteSecret ().

~~~
http://www.mycompany.com/ DPWebSecretService.svc/WriteSecret
Below is an example of HTTP BODY of the WriteSecret request.
{
	"secretName":"SystemLogonInfo",
	"secretData":"eyJ0eXAiOiJKV1QiLA0KICJhbGciOiJIUzI1NiJ9",
	"ticket":
		{"jwt":"eyJ0eXAiOiJKV1QiLA0KICJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJqb2UiLA0KICJl
		eHAiOjEzMDA4MTkzODAsDQogImh0dHA6Ly9leGFtcGxlLmNvbS9pc19yb290Ijp0cnVlfQ.dB
		jftJeZ4CVP-B92K27uhbUJU1p1rwW1gFWFOEjXk"}
}
~~~

### DeleteSecret method  

The DeleteSecret method will clear the secret content and delete the secret object (if it exists). The caller must provide a ticket which satisfies the policy.  

#### Syntax  

~~~
void DeleteSecret(Ticket ticket, String secretName);  
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">ticket</td>
  <td valign="top">The JSON Web Token returned by the Authentication Service.</td>
  </tr>
  <tr>
  <td valign="top">secretName</td>
  <td valign="top">The name of the Secret to be deleted. For example, "SystemLogonInfo".</td>
  </tr>  
</table>

#### Returns  

None.

#### Notes  

DeleteSecret () should be implemented as an HTTP DELETE using JSON as the response format.  

The following URL is an example of using DeleteSecret ().  

~~~  
http://www.mycompany.com/DPWebSecretService.svc/DeleteSecret
~~~  

Below is an example of HTTP BODY of a DeleteSecret request.

~~~
{
	"secretName":"SystemLogonInfo",
	"ticket":{"jwt":"eyJ0eXAiOiJKV1QiLA0KICJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJqb2Ui
	LA0KICJleHAiOjEzMDA4MTkzODAsDQogImh0dHA6Ly9leGFtcGxlLmNvbS9pc19yb290Ijp0cn
	VlfQ.dBjftJeZ4CVP-B92K27uhbUJU1p1rwW1gFWFOEjXk"}
}
~~~

## Data Contracts

### ResourceActions enumerator  

ResourceActions enumerates possible actions the caller may perform with a particular resource (Secret).
public enum ResourceActions
{
	[EnumMember]
	Read = 0,
 [EnumMember]
	Write = 1,
	[EnumMember]
  Delete = 2,
}

We support only these actions in the current version of the API.  
- Read  
- Write  
- Delete  

### PolicyElement class  

The PolicyElement class describes one policy element or credential.  

~~~  
public class PolicyElement
	{
  public PolicyElement(String id)
  {
		cred_id = id;
  }
  [DataMember]
  public String cred_id { get; set; }
  }  
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">cred_id</td>
  <td valign="top">The unique ID of a credential.</td>
  </tr>
</table>

The following credentials are supported in this version.  

<table style="width:100%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:27%" ALIGN="left">Credential</th>
    <th style="width:70%" ALIGN="left">GUID</th>
  </tr>
  <tr>
  	<td valign="top">User password</td>
  	<td valign="top">D1A1F561-E14A-4699-9138-2EB523E132CC</td>
  </tr>
	<tr>
   	<td valign="top">PIN</td>
   	<td valign="top">8A6FCEC3-3C8A-40c2-8AC0-A039EC01BA05</td>
  </tr>
	<tr>
	  <td valign="top">Smart Card</td>
	  <td valign="top">D66CC98D-4153-4987-8EBE-FB46E848EA98</td>
	</tr>
	<tr>
	  <td valign="top">Contactless/Proximity card</td>
	  <td valign="top">F674862D-AC70-48ca-B73E-64A22F3BAC44</td>
	</tr>
	<tr>
		<td valign="top">Recovery Questions</td>
		<td valign="top">B49E99C6-6C94-42DE-ACD7-FD6B415DF503</td>
	</tr>
	<tr>
		<td valign="top">Bluetooth</td>
		<td valign="top">E750A180-577B-47f7-ACD9-F89A7E27FA49</td>
	</tr>
	<tr>
		<td valign="top">? Any missing ?</td>
		<td valign="top"></td>
	</tr> 				 			 	
</table>

#### Note

You must use the “braceless” GUID representation in our API in URLs and JSON representation

#### Example  

Below is an example of the JSON representation for a credential. This policy element describes a Fingerprint credential.

~~~
{"cred_id":"AC184A13-60AB-40e5-A514-E10F777EC2F9"}
~~~

### Policy class

This class describes one authentication policy. To access a resource, multiple authentication policies can be applied. In this case, the relationship between the authentication policies would be OR.  

~~~ public class Policy
{
  public Policy()
  {
     policy = new List<PolicyElement>();
  }
  [DataMember]
  public List<PolicyElement> policy{ get; set; }
}
~~~  

#### Example  

Below is an example of the JSON representation of a policy. This policy describes an authentication policy requiring that “Fingerprint AND PIN” credentials must be provided.  

~~~  
{"policy":  
	[
		{"cred_id":"AC184A13-60AB-40e5-A514-E10F777EC2F9"},
		{"cred_id":"8A6FCEC3-3C8A-40c2-8AC0-A039EC01BA05"}
	]
},
~~~  

### Ticket class

The result of a successful authentication is a ticket. The format of the ticket is a JSON Web Token (JWT). Additional details about JWT are provided in the topic “JSON Web Token (JWT)” <mark style="color:Red;">on page 71</mark>.

~~~ public class Ticket
	{
  [DataMember]
   public String jwt { get; set; } // JSON Web Token
	}
~~~
