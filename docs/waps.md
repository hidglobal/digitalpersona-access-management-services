---
layout: default
title: Web Authentication Policy Service  (WAPS)
has_toc: false
nav_order: 5
---

###### [DigitalPersona Access Management API ](https://hidglobal.github.io/digitalpersona-access-management-api/)/ DigitalPersona Access Management Services / Web Authentication Policy Service (WAPS)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[\| View Repo \|](https://github.com/hidglobal/access-management-services)  

![](assets/HID-DPAM-access-mgmt-svcs.png)       

## Web Authentication Policy Service

The goal of the Web Authentication Policy Service (WAPS) is to specify the authentication policy required for a specific user to manage a Resource.  

It is implemented as a GUI-less Web Service that  can be located inside or outside the firewall, depending on the logic of the specific WAPS implementation. Location inside the firewall is more common since WAPS may need connection to Active Directory, the DigitalPersona Server or other customer Databases.  

WAPS can use any protocol the customer chooses for listening. However, it is important that the Service Provider (SP) using WAPS must always have access to it.  

WAPS can be hosted in any supported environment and developed using any Web development framework as long as the SP can communicate with it.  

### IDPWebPolicy interface  

The DigitalPersona Web Authentication Policy Service (WAPS) is implemented as a WCF service. The IDPWebPolicy interface has only one method, GetPolicyList(), as described in the next topic.  

~~~
namespace DPWcfPolicyService
{
	[ServiceContract]
  public interface IDPWebPolicy
  {
  /*
  * Return policy for specific resource, specific user and specific action
  */
	[WebInvoke(Method = "GET",
		ResponseFormat = WebMessageFormat.Json,
		BodyStyle = WebMessageBodyStyle.Wrapped,
		UriTemplate = "GetPolicyList?user={userName}&type={nameType}&uri=
			{resourceUri}&action={action}")]
	List<Policy> GetPolicyList(String userName, UInt16 nameType,
		String resourceUri, ResourceActions action);
	/*
	* Return policy for specific resource, specific user, specific action using
		contextual/behavior information
	*/
	[OperationContract()]
	[WebInvoke(Method = "POST",
		ResponseFormat = WebMessageFormat.Json,
		BodyStyle = WebMessageBodyStyle.Wrapped,
		UriTemplate = "GetPolicyListEx")]
	List<Policy> GetPolicyListEx(String userName, UInt16 nameType, String
		resourceUri, ResourceActions action, ContextualInfo info);
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
    public List<PolicyElement> policy{ get; set; }
    }
	[DataContract]
	public class ContextualInfo
	{
		[DataMember]
		public Boolean behavior { get; set; }
		[DataMember]
		public Boolean ip { get; set; }
		[DataMember]
		public Boolean device { get; set; }
	 [DataMember]
		public Boolean altusInstalled { get; set; }
		[DataMember]
		public String computer { get; set; }
		[DataMember]
		public String domain { get; set; }
		[DataMember]
		public String user { get; set; }
		[DataMember]
		public Boolean insideFirewall { get; set; }
		[DataMember]
		public Boolean remoteSession { get; set; }
	}
}
~~~

### Methods  

#### GetPolicyList method

The GetPolicyList() method returns a list of policies representing authentication options; at least one of them must be met in order to access (read, write or delete) the specific Secret of the user.

##### Syntax  

~~~List<Policy> GetPolicyList(String userName, UInt16 nameType, String resourceUri,
ResourceActions action);
~~~


<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">userName</td>
  <td valign="top">The name of the user whose Secret is to be managed.</td>
  </tr>
  <tr>
  <td valign="top">userNameType</td>
  <td valign="top">The format of the user name provided in the userName parameter. See “User class” <mark style="color:Red;">on page 68</mark> for a detailed description of user name formats.</td>
  </tr>
  <tr>
  <td valign="top">resourceUri</td>
  <td valign="top">	Uri which represent resource to be accessed. For example, a Secret name like “SystemLogonInfo.”
  </td>
  </tr>  
  <tr>
  <td valign="top">action</td>
  <td valign="top">The action requested for this resource. Currently the following actions are supported: 1) Read, 2) Write and 3) Delete.</td>
  </tr>       
</table>

##### Returns  

Returns a list of policies of which one must be met in order to access the resource.  

##### Notes  

GetPolicyList () should be implemented as HTTP GET using JSON as the response format.
The following is an example of using GetPolicyList().  

~~~http://www.mycompany.com/DPWebPolicy/DPWebPolicyService.svc/GetPolicyList?
user=someone@mycompany.com&type=6&uri=SystemLogonInfo&action=Read
~~~

This URL requests a list of policies required to read a secret with the name SystemLogonInfo, which belongs to a user with the name someone@mycompany.com.  

The example response would be the following.  

~~~
{"GetPolicyListResult":
	[
		{"policy":[
			{"cred_id":"AC184A13-60AB-40e5-A514-E10F777EC2F9"},
			{"cred_id":"8A6FCEC3-3C8A-40c2-8AC0-A039EC01BA05"}
		]},
		{"policy":[
			{"cred_id":"AC184A13-60AB-40e5-A514-E10F777EC2F9"},
			{"cred_id":"E750A180-577B-47f7-ACD9-F89A7E27FA49"}
		]}
	]
}
~~~

This response means that the user someone@mycompany.com needs to provide “Fingerprint AND PIN” or “Fingerprint AND Bluetooth” to be allowed to read the SystemLogonInfo Secret.  

#### GetPolicyListEx method  

The GetPolicyListEx() method is an extension of the  GetPolicyList() method which adds Contextual/Behavior information and returns a list of policies which are  authentication options of which at least one must met in order to access (read, write or delete) the specific Secret of the specific user.  

##### Syntax  

~~~
List<Policy> GetPolicyListEx(User user, String resourceUri, ResourceActions
action, ContextualInfo info);
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">userName</td>
  <td valign="top">The name of the user whose Secret is to be managed.</td>
  </tr>
  <tr>
  <td valign="top">resourceUri</td>
  <td valign="top">AUri which represents the resource to be accessed. For example, a Secret name like “SystemLogonInfo.”
  </td>
  </tr>  
  <tr>
  <td valign="top">action</td>
  <td valign="top">The action requested for this resource. Currently the following actions are supported: 1) Read, 2) Write and 3) Delete.</td>
  </tr>    
  <tr>
  <td valign="top">info</td>
  <td valign="top">Contextual/Behavior information provided (see detailed description below).</td>
  </tr>      
</table>

##### Returns  

Returns a list of policies of which one must be met to access the resource.

##### Examples  

GetPolicyListEx () should be implemented as HTTP POST using JSON as response format.
The following is an example of the use of GetPolicyListEx().

~~~
http://www.digitalpersona.com/DPWebPolicy/DPWebPolicyService.svc/GetPolicyListEx
~~~

Below is an example of an HTTP request body.

~~~
{
	"user":{"name":"someone@mycompany","type":6},
	"resourceUri":"SystemLogonInfo",
	"action":1,
	"info":
	{
		"behavior":true,
		"ip":true,
		"device":true,
		"altusInstalled":true,
		"computer":"computername.mycompany.net",
		"domain":"mycompany.net",
		"user":"someone@mycompany.com",
		"insideFirewall":true,
		"remoteSession":false
	}
}
~~~

This URL requests the policies required to read a secret (in this example Secret is requested resource) with the name SystemLogonInfo which belongs to a user with the name someone@mycompany.com.

The example response would be the following.

~~~
{"GetPolicyListExResult":
	[
		{"policy":[
			{"cred_id":"AC184A13-60AB-40e5-A514-E10F777EC2F9"},
			{"cred_id":"8A6FCEC3-3C8A-40c2-8AC0-A039EC01BA05"}
		]},
		{"policy":[
			{"cred_id":"AC184A13-60AB-40e5-A514-E10F777EC2F9"},
			{"cred_id":"E750A180-577B-47f7-ACD9-F89A7E27FA49"}
		]}
	]
}
~~~

This response means that the user someone@mycompany.com needs to provide "Fingerprint AND PIN" or "Fingerprint AND Bluetooth" to be allowed to read the SystemLogonInfo Secret.

### Data Contracts  

#### ResourceActions enumerator  

ResourceActions enumerates the possible actions the caller may apply to a particular resource.

~~~
public enum ResourceActions
{
	[EnumMember]
	Read = 0,
  [EnumMember]
  Write = 1,
  [EnumMember]
  Delete = 2,
}
~~~

We support only three actions in this version of WAPS.
- Read
- Write
- Delete

#### PolicyElement class  

The PolicyElement class describes one policy element.

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
  <td valign="top">The unique Id of a credential.</td>
  </tr>
</table>

The following credentials are supported.

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

Below is an example of JSON representation of a fingerprint credential. The relationship between policy elements within one policy is AND. Note that we must use “braceless” GUID representation in our API in URLs and JSON representation.  

~~~
{"cred_id":"AC184A13-60AB-40e5-A514-E10F777EC2F9"}
~~~

#### Policy class  

This class describes one authentication policy. To access a particular resource, multiple authentication policies could be applied. In this case the relationship between the authentication policies would be OR.

~~~
public class Policy
{
	public Policy()
  {
	  policy = new List<PolicyElement>();
  }
  [DataMember]
  public List<PolicyElement> policy{ get; set; }
}
~~~

Below is an example of the JSON representation of a policy. This policy describes an authentication policy which requires “Fingerprint AND PIN” credentials to be provided.  

~~~
{"policy":[
	{"cred_id":"AC184A13-60AB-40e5-A514-E10F777EC2F9"},
	{"cred_id":"8A6FCEC3-3C8A-40c2-8AC0-A039EC01BA05"}
]},
~~~

#### ContextualInfo class  

This class describes Contextual/Behavior information.  

~~~
[DataContract]
public class ContextualInfo
{
	[DataMember]
	public Boolean behavior { get; set; }
	[DataMember]
	public Boolean ip { get; set; }
	[DataMember]
	public Boolean device { get; set; }
	[DataMember]
	public Boolean altusInstalled { get; set; }
	[DataMember]
	public String computer { get; set; }
	[DataMember]
	public String domain { get; set; }
	[DataMember]
	public String user { get; set; }
	[DataMember]
	public Boolean insideFirewall { get; set; }
	[DataMember]
	public Boolean remoteSession { get; set; }
    }
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Parameter</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">behavior</td>
  <td valign="top">	Boolean value which is true if user behavior is matched, false otherwise.</td>
  </tr>
  <tr>
  <td valign="top">ip</td>
  <td valign="top">	Boolean value which is true if user IP is matched with the trained data, false otherwise.</td>
  </tr>
  <tr>
  <td valign="top">device</td>
  <td valign="top">	Boolean value which is true if user device is matched with the trained data, false otherwise.</td>
  </tr>
  <tr>
  <td valign="top">altusInstalled</td>
  <td valign="top">	Boolean value which is true if DigitalPersona client software is installed on client machine, false otherwise.</td>
  </tr>
  <tr>
  <td valign="top">computer</td>
  <td valign="top">	DNS name of the client machine.</td>
  </tr>
  <tr>
  <td valign="top">domain</td>
  <td valign="top">	AD domain client computer belongs to.</td>
  </tr>       
  <tr>
  <td valign="top">user</td>
  <td valign="top">	UPN name of the user currently logged on client machine.</td>
  </tr>
  <tr>
  <td valign="top">insideFirewall</td>
  <td valign="top">	Boolean value which is true if client machine located inside corporate firewall, false otherwise.</td>
  </tr>     
  <tr>
  <td valign="top">remoteSession</td>
  <td valign="top">	Boolean value which is true user access client machine remotely, false otherwise.</td>
  </tr>    
</table>

Below is an example of the JSON representation of ContextualInfo.

~~~
{
	"behavior":true,
	"ip":true,
	"device":true,
	"altusInstalled":true,
	"computer":"kloizinD7.crossmatch.net",
	"domain":"crossmatch.net",
	"user":"Kirill.Lo@crossmatch.com",
	"insideFirewall":true,
	"remoteSession":false
}
~~~

### Policy Configuration  

#### dpWebDefaultPolicies configuration element  

The dpWebDefaultPolicies element in the Web.config file contains general configuration information about the Authentication Policies.  

Put the dpWebDefailtPolicies  element within the configuration element in a Web.config. The dpWebDefailtPolicies element has no attributes.  

The dpWebDefaultPolicies  element may include the following elements.

##### dpWebPolicies configuration element  

The dpWebPolicies element in the Web.config file contains configuration information about standard Authentication Policies (authentication policies when step-up authentication is not triggered).  

Put the dpWebPolicies element within the dpWebDefaultPolicies element in a Web.config. The dpWebPolicies  element has no attributes.  

The dpWebPolicies  element may include the following elements.

###### dpWebPolicy configuration element.

The dpWebPoliciy element in the Web.config file contains configuration information about one particular Authentication Policy.  

The following table describes the attributes of the dpWebPoliciy  element.  


<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:15%" ALIGN="left">Attribute</th>
    <th style="width:15%" ALIGN="left">Data type</th>
    <th style="width:65%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">name</td>
  <td valign="top">String</td>
  <td valign="top">		Readable name of authentication policy. This attribute is required.</td>  
  </tr>
  <tr>
  <td valign="top">tokens</td>
  <td valign="top">String</td>
  <td valign="top">		List of authentication tokens separated by semicolon (;) in token GUID format. This attribute is required.</td>  
  </tr>   
</table>

Below are a few examples of the dpWebPolicies  element.

~~~
<dpWebPolicy name="Password" tokens="D1A1F561-E14A-4699-9138-2EB523E132CC" />
~~~

The element above represents a Password only policy.  

~~~
<dpWebPolicy name="Fingerprint" tokens="AC184A13-60AB-40E5-A514-E10F777EC2F9" />
~~~

The element above represents a Fingerprint only policy.  

~~~
<dpWebPolicy name="Fingerprint AND Password" tokens="AC184A13-60AB-40E5-
A514-E10F777EC2F9; D1A1F561-E14A-4699-9138-2EB523E132CC" />
~~~

The element above represents a Fingerprint AND Password policy.

######Example of dpWebPolicies Configuration Element  

Below is an example of dpWebPolicies Configuration Element.

~~~
<dpWebPolicies>
	<dpWebPolicy name="Password" tokens="D1A1F561-E14A-4699-9138-2EB523E132CC" />
	<dpWebPolicy name="Fingerprint" tokens="AC184A13-60AB-40E5-A514-E10F777EC2F9" />
	<dpWebPolicy name="Contactless" tokens="F674862D-AC70-48CA-B73E-64A22F3BAC44" />
	<dpWebPolicy name="Smart Card" tokens="D66CC98D-4153-4987-8EBE-FB46E848EA98" />
	<dpWebPolicy name="One-time Password" tokens="324C38BD-0B51-4E4D-BD75-200DA0C8177F" />
</dpWebPolicies>
~~~

The above policies should allow using Password OR Fingerprint OR Contactless cards OR Smart Cards OR OTP if step up authentication is not triggered.  

###### dpWebStepUpPolicies configuration element  

The dpWebStepUpPolicies element in the Web.config file contains configuration information about step-up Authentication Policies (authentication policies when step-up authentication is triggered).  

Put the dpWebStepUpPolicies  element within the dpWebDefailtPolicies element in a Web.config file. The dpWebStepUpPolicies  element has no attributes.  

The dpWebStepUpPolicies  element may include one or more  dpWebPoliciy  elements.  

Below is an example of dpWebStepUpPolicies Configuration Element.

~~~
<dpWebStepUpPolicies>
	<dpWebPolicy name="Fingerprint AND Password" tokens="AC184A13-60AB-40E5-
		A514-E10F777EC2F9; D1A1F561-E14A-4699-9138-2EB523E132CC" />
	<dpWebPolicy name="Fingerprint AND PIN" tokens="AC184A13-60AB-40E5-A514-
		E10F777EC2F9; 8A6FCEC3-3C8A-40c2-8AC0-A039EC01BA05" />
</dpWebStepUpPolicies>
~~~

Those policies would force the user to use Fingerprint AND Password OR Fingerprint AND PIN if step-up authentication is triggered.  

##### dpWebStepUpTriggers configuration element  

The dpWebStepUpTriggers element in the Web.config file contains configuration information about triggers which can trigger Step-up Authentication.  

Put the dpWebStepUpTriggers  element within the dpWebDefailtPolicies element in a Web.config. The dpWebStepUpTriggers  element has no attributes.  

The dpWebStepUpTriggers  element may include the following elements.

###### dpWebStepUpTrigger Configuration Element
The dpWebStepUpTrigger element in the Web.config file contains configuration information about one particular Step Up Authentication trigger.  

The following table describes the attributes of the dpWebStepUpTrigger element.  



<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:15%" ALIGN="left">Attribute</th>
    <th style="width:15%" ALIGN="left">Data type</th>
    <th style="width:65%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">name</td>
  <td valign="top">String</td>
  <td valign="top">Trigger name. This attribute is required.</td>  
  </tr>
</table>

Below are few examples of using the dpWebStepUpTrigger  element.

~~~
<dpWebStepUpTrigger name="behavior" />
~~~

The element above represents the Behavior trigger, which initiates step-up authentication if previously captured user behavior is not matched.

~~~
<dpWebStepUpTrigger name="insideFirewall" />
~~~
The element above represents the inside firewall trigger, which initiates step up authentication if the user's machine is located outside the corporate network.

###### List of Triggers supported  

In this release we support the following triggers.  


<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:15%" ALIGN="left">Parameter</th>
    <th style="width:80%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">behavior</td>
  <td valign="top">	Trigger step up authentication if user behavior does not match.</td>
  </tr>
  <tr>
  <td valign="top">ip</td>
  <td valign="top">	Trigger step up authentication if user IP does not match with the trained data stored in the BehavioSec cloud.</td>
  </tr>
  <tr>
  <td valign="top">device</td>
  <td valign="top">	Trigger step up authentication if user device does not match with the trained data stored in the BehavioSec cloud.</td>
  </tr>
  <tr>
  <td valign="top">altusInstalled</td>
  <td valign="top">	Trigger step up authentication if DPCA is not installed on client machine.</td>
  </tr>
  <tr>
  <td valign="top">computer</td>
  <td valign="top">	Trigger step up authentication if client machine is not trusted.</td>
  </tr>
  <tr>
  <td valign="top">domain</td>
  <td valign="top">	Trigger step up authentication if client machine does not belong to trusted AD domain.</td>
  </tr>
  <tr>
  <td valign="top">user</td>
  <td valign="top">	Trigger step up authentication user who is trying to access web server is not currently logon user on local machine.</td>
  </tr>    
  <tr>
  <td valign="top">insideFirewall</td>
  <td valign="top">	Trigger step up authentication if local machine is not located in corporate network (not inside corporate firewall).</td>
  </tr>   
  <tr>
  <td valign="top">remoteSession</td>
  <td valign="top">	Trigger step up authentication if user connected to local machine remotely.</td>
  </tr>            
</table>

######Example of dpWebStepUpTriggers Element  

Below is an example of the dpWebStepUpTriggers Configuration Element.

~~~
<dpWebStepUpTriggers>
	<dpWebStepUpTrigger name="behavior" />
	<dpWebStepUpTrigger name="insideFirewall" />
</dpWebStepUpTriggers>
~~~

The above settings will trigger step-up authentication if user behavior does not match or when the local machine is  located outside of the firewall.

###### Example of dpWebDefaultPolicies Element  

Below is an example of the dpWebDefaultPolicies configuration element.

~~~
<dpWebDefaultPolicies>

	<dpWebPolicies>
		<dpWebPolicy name="Password" tokens="D1A1F561-E14A-4699-9138-2EB523E132CC" />
		<dpWebPolicy name="Fingerprint" tokens="AC184A13-60AB-40E5-A514-		E10F777EC2F9" />
  </dpWebPolicies>

	<dpWebStepUpPolicies>
    <dpWebPolicy name="Fingerprint AND Password" tokens="AC184A13-60AB-40E5-
			A514-E10F777EC2F9; D1A1F561-E14A-4699-9138-2EB523E132CC" />
  </dpWebStepUpPolicies>

	<dpWebStepUpTriggers>
    <dpWebStepUpTrigger name="behavior" />
    <dpWebStepUpTrigger name="insideFirewall" />
  </dpWebStepUpTriggers>

</dpWebDefaultPolicies>
~~~

The policy above will allow a user to authenticate with their Fingerprint OR Password if step-up authentication is not triggered, it will force the user to use Fingerprint AND Password if step-up authentication is triggered, and user behavior and computer location outside firewall will trigger step-up authentication.
