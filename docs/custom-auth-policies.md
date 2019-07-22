---
layout: default
title: Custom Authentication Policies
has_toc: false
nav_order: 10
---

{% include header.html %}

# Custom Authentication Policies

This section describes how to create and work with DigitalPersona custom authentication policies. The following information is applicable to applications developed for either Windows or the web.

*Note: This material is for advanced developers only.*

Technical support queries regarding custom authentication policies should be sent to ProSDKCAPsupport@digitalpersona.com

[comment]: <> (Need new email address.)

rather than the usual DigitalPersona technical support.  

If you need DigitalPersona to authenticate users and return user secrets, then you will need to satisfy the authentication policy defined by the DigitalPersona administrator. You can also choose to define your own custom authentication policy but if you do, your custom policy may not be sufficient for secret release. DigitalPersona will not release secrets unless you satisfy the authentication policy defined by the DigitalPersona administrator.  

The DigitalPersona administrator must define the authentication policy or policies. Some examples of authentication policies might be:  

- Users can authenticate with either a fingerprint or a password but we don’t need both fingerprint AND password.
- Users can authenticate with password OR with a smartcard; if they use their smartcard, then they must also enter their PIN.  

Consult the DigitalPersona AD or LDS Administrator Guide (available at [https://www.crossmatch.com/company/support/documentation/](https://www.crossmatch.com/company/support/documentation/) for more information on policies.  

Note that an authentication policy is defined for a user on a specified workstation. Users may have different policies, and a policy for one user may not work if you try to use it to retrieve a secret for another user.  

# How an Authentication Policy is Represented  

An authentication policy is represented by an array of credential masks.  

To create a data representation of the DigitalPersona administrator’s authentication policy, we make a list of all credentials or credential combinations that are permitted. Then we create a credential mask for each valid credential or combination.   

Each mask has a bit set for every credential that is required in this combination. As long as the user supplies one valid combination of credentials (i.e., satisfies at least one credential mask) they will be authenticated.  

Each credential mask is a 64-bit value with each bit representing one valid credential. The bits are defined as:

**Definition of bits in each credential mask**

<table style="width:55%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:30%" ALIGN="left">Hex Bit Mask</th>
    <th style="width:35%" ALIGN="left">Credential</th>
  </tr>
  <tr>
    <td valign="top">
    0x01
</td>
    <td valign="top">Password</td>
  </tr>
  <tr>
    <td valign="top">0x02</td>
    <td valign="top">
    	Fingerprint
    </td>
  </tr>
  <tr>
    <td valign="top">0x04</td>
    <td valign="top">	Smart card
    </td>
  </tr>
  <tr>
    <td valign="top">0x10</td>
    <td valign="top">	Face</td>
  </tr>
  <tr>
    <td valign="top">0x20</td>
    <td valign="top">  	Contactless card</td>
  </tr>
  <tr>
    <td valign="top">0x80</td>
    <td valign="top">      	PIN
          </td>
  </tr>
  <tr>
    <td valign="top">0x100</td>
    <td valign="top">	Proximity card
    </td>
  </tr>
  <tr>
    <td valign="top">0x200</td>
    <td valign="top">	Bluetooth</td>
  </tr>     
</table>

The simplest authentication policy consists of a single (binary) credential mask:  

0000000000000000000000000000000000000000000000000000000000000001  

which represents a policy that requires users to provide a password (the password bit is set).  

As another example, if the authentication policy requires BOTH a password and fingerprint, the authentication policy would consist of this credential mask (password and fingerprint bits both set):  

0000000000000000000000000000000000000000000000000000000000000011  

If the user can authenticate either with a password OR a fingerprint, the authentication policy would consist of an array containing the following two credential masks.   

The first credential mask has the bit set for password access and the second mask has the bit set for fingerprint access. As long as a user satisfies ONE of these credentials masks, the user will be authenticated.  

0000000000000000000000000000000000000000000000000000000000000001  
0000000000000000000000000000000000000000000000000000000000000010  

As another example, the policy below shows the two credential masks for the authentication policy that requires users to authenticate by providing both their fingerprint and their password OR by providing their smartcard.  

0000000000000000000000000000000000000000000000000000000000000011  
0000000000000000000000000000000000000000000000000000000000000100  

A fairly typical default authentication policy is shown below. This policy allows the user to use ANY one of: password, fingerprint, or smart card.  

0000000000000000000000000000000000000000000000000000000000000001  
0000000000000000000000000000000000000000000000000000000000000010  
0000000000000000000000000000000000000000000000000000000000000100  

The order of credential masks within the Policy array is unimportant. The bits that correspond to each credential type are consistent for all credential masks (i.e., bit 0 always represents that a password is required).  

# Extending an Authentication Policy  

If you want to extend the DigitalPersona administrator’s authentication policy, you can read the current policy by calling DPAlReadAuthPolicy. You would then modify the credentials masks or add new credentials masks to the authentication policy array and pass your authentication policy to calls to DPAlAuthenticate or DPAlIdentAuthenticate. This would result in your authentication policy being used instead of the policy defined by the DigitalPersona administrator.  

As a best practice, we recommend that this feature only be used to make the DigitalPersona administrator’s authentication policy more strict.  

*DigitalPersona will not release secrets if your policy is less stringent than the existing DigitalPersona authentication policy.*  

For example, consider the case where the existing DigitalPersona authentication policy is to allow fingerprints or passwords.  

In that case, the authentication policy would consist of these two credential masks:  

0000000000000000000000000000000000000000000000000000000000000001  
0000000000000000000000000000000000000000000000000000000000000010  

If you require that users also use face recognition in addition to either a fingerprint or password, you would update the authentication policy’s credential masks to this:  

0000000000000000000000000000000000000000000000000000000000010001  
0000000000000000000000000000000000000000000000000000000000010010  

In this case, DigitalPersona will release secrets because the policy is stricter than the original. However if you update the authentication policy to allow a different kind of credential entirely (for example, by adding a new credential mask that allows smart cards), then DigitalPersona will not release secret data.  

# Creating a New Authentication Policy  

You can also create your own custom policy. To do this, simply create the appropriate credentials masks and pass your authentication policy to DPAlAuthenticate or DPAlIdentAuthenticate.
