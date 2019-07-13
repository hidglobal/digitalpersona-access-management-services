---
layout: default
title: WAS Credential Format
has_toc: false
nav_order: 7
---

###### [DigitalPersona Access Management API ](https://hidglobal.github.io/digitalpersona-access-management-api/)/ DigitalPersona Access Management Services / WAS Credential Format&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[\| View Repo \|](https://github.com/hidglobal/access-management-services)  

![](assets/HID-DPAM-access-mgmt-svcs.png)    
## WAS Credential Format

In the [IDPWebAuth interface](was.md#idpwebauth-interface), we define a number of methods for user authentication and identification which have as an input parameter an object of the Credential class. This topic describes the data members associated with each credential.

<table style="width:95%;margin-left:auto;margin-right:auto;">

  <tr>
  <td style="width:50%" ALIGN="left" valign="top"><A HREF="#fingerprint-credential">Fingerprint Credential</A>
  </td>
  <td valign="top"><A HREF="#wia-credential">Windows Integrated Authentication (WIA) Credential</A></td>
  </tr>
  <tr>
  <td valign="top"><A HREF="#password-credential">Password Credential</A>
</td>
  <td valign="top"><A HREF="#pin-credential">  PIN Credential</A></td>
  </tr>
  <tr>
  <td valign="top"><A HREF="#email-credential">
  Email Credential</A>
    </td>
  <td valign="top"><A HREF="#contacless-card-credential">Contactless Card Credential</A></td>
  </tr>
  <tr>
  <td valign="top"><A HREF="#recovery-credential">Recovery  Questions Credential</A>
</td>
  <td valign="top"><A HREF="#prox-card-credential">  Proximity Card Credential</A>
  </td>
  </tr>
  <tr>
  <td valign="top"><A HREF="#smart-card-credential">Smart Card Credential</A>
    </td>
  <td valign="top"><A HREF="#face-credential">
  Face Credential</A>
  </td>
  </tr>
  <tr>
  <td valign="top"><A HREF="#totp-credential">Time-Based OTP (TOTP) Credential</A></td>
  <td valign="top"><A HREF="#fido-credential">
  FIDO Device Credential</A></td>
  </tr>   
</table>

The Credential class is defined as follows.

~~~
[DataContract]
public class Credential
{
	[DataMember]
  public String id { get; set; } // unique id (Guid) of credential
  [DataMember]
  public String data { get; set; } //  credential data
}
~~~
It is obvious that the data member of the Credential class should be different for different types of credentials. For example, the data member of the Fingerprint Credential is different from the same member of the Password Credential.

### <A NAME="fingerprint-credential"></A>Fingerprint Credential

The following ID is defined for Fingerprint credentials.  

~~~
{AC184A13-60AB-40e5-A514-E10F777EC2F9}
~~~

The data for fingerprint credential is defined in WCF terminology as follows.  

~~~
[DataContract][Flags]
public enum BioFactor
{
	[EnumMember]
	MULTIPLE =              0x0001,
  [EnumMember]
  FACIAL_FEATURES =       0x0002,
  [EnumMember]
  VOICE =                 0x0004,
  [EnumMember]
  FINGERPRINT =           0x0008,
  [EnumMember]
  IRIS =                  0x0010,
  [EnumMember]
  RETINA =                0x0020,
  [EnumMember]
  HAND_GEOMETRY =         0x0040,
  [EnumMember]
  SIGNATURE_DYNAMICS =    0x0080,
  [EnumMember]
  KEYSTROKE_DYNAMICS =     0x0100,
  [EnumMember]
	LIP_MOVEMENT =          0x0200,
  [EnumMember]
  THERMAL_FACE_IMAGE =    0x0400,
  [EnumMember]
  THERMAL_HAND_IMAGE =    0x0800,
  [EnumMember]
  GAIT =                  0x1000,
}

[DataContract]
public class BioSampleFormat
{
	public BioSampleFormat(UInt16 owner, UInt16 id)
  {
  		FormatOwner = owner;
    FormatID = id;
	}
	[DataMember]
	public UInt16 FormatOwner { get; set; }
		// Biometric owner ID registered with IBIA (http://www.ibia.org/base/
				cbeff/_biometric_org.phpx).
		// 51 for DigitalPersona, 49 for Neurotechnologija
	[DataMember]
	public UInt16 FormatID;   // Vendor specific format ID
}

[DataContract][Flags]
public enum BioSampleType
{
	[EnumMember]
  RAW             = 0x01,   																// Fingerprint image
  [EnumMember]
  INTERMEDIATE    = 0x02,   																// Fingerprint feature set
  [EnumMember]
  PROCESSED	    = 0x04,   																	// Fingerprint template
	[EnumMember]
  RAW_WSQ_COMPRESSED    = 0x08, // WSQ compressed image
  [EnumMember]
  ENCRYPTED	    = 0x10,
  [EnumMember]
   SIGNED		    = 0x20,
}

[DataContract]
public enum BioSamplePurpose
{
	[EnumMember]
	ANY = 0,        // Any purpose
	[EnumMember]
	VERIFY = 1,     // Purpose is verification
	[EnumMember]
  IDENTIFY = 2,   // Purpose is identification
  [EnumMember]
  ENROLL =   3,   // Purpose is enrollment
  [EnumMember]
  ENROLL_FOR_VERIFICATION_ONLY = 4, //  Purpose is enrollment for verification
	[EnumMember]
	ENROLL_FOR_IDENTIFICATION_ONLY =  5,  // Purpose is enrollment for identification
	[EnumMember]
  AUDIT = 6,      // Purpose is audit
}

[DataContract]
public enum BioSampleEncryption
{
	[EnumMember]
	NONE = 0,       // Data is not encrypted
  [EnumMember]
  XTEA = 1,       // XTEA encryption with well-known key
}

[DataContract]
public class BioSampleHeader
{
	[DataMember]
  public BioFactor Factor { get; set; }                          
		// Biometric factor. Must be set to 8 for fingerprint
	[DataMember]
  public BioSampleFormat Format { get; set; }             
		// Format owner (vendor) information
  [DataMember]
  public BioSampleType Type { get; set; }                 
		// type of biometric sample
	[DataMember]
  public BioSamplePurpose Purpose { get; set; }           
		// Purpose of biometric sample
  [DataMember]
  public SByte Quality { get; set; }                      
		// Quality of biometric sample.
		// If we don't support quality it should be set to -1.
	[DataMember]
  public BioSampleEncryption Encryption { get; set; }    
		// Encryption of biometric sample.
}

[DataContract]
public class BioSample
{
	public BioSample()
  {
		Header = new BioSampleHeader();
  }
  [DataMember]
  public UInt16 Version { get; set; }                
		// header version. Must be set to 1 in this version
	[DataMember]
	public BioSampleHeader Header { get; set; }        
		// Biometric sample header
	[DataMember]
  public String Data { get; set; }                   
		// Base64url encoded biometric sample data
}
~~~
#### BioSample class  

The BioSample class represents a general biometric sample.  

~~~
[DataContract]
public class BioSample
{
	public BioSample()
  {
		Header = new BioSampleHeader();
  }
  [DataMember]
  public UInt16 Version { get; set; }                
		// header version. Must be set to 1 in this version
  [DataMember]
  public BioSampleHeader Header { get; set; }        
		// Biometric sample header
	[DataMember]
  public String Data { get; set; }                   
		// Base64url encoded biometric sample data
}
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Data Member</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">Version	</td>
  <td valign="top">Version of Biometric Sample format. Must be 1 for this API version.</td>
  </tr>
  <tr>
  <td valign="top">Header</td>
  <td valign="top">	Base64url encoded biometric sample data. See Base64url encoded biometric sample data. See <b>BioSampleHeader class</b> below.</td>
  </tr>
  <tr>
  <td valign="top">Data</td>
  <td valign="top">	Base64url encoded biometric sample data. See <A HREF="https://hidglobal.github.io/digitalpersona-access-management-services#biometric-sample-data">Biometric Sample Data</A>.</td>
  </tr>
</table>

### BioSampleHeader class  

The BioSampleHeader provides detailed information about a Biometric sample.   

~~~
[DataContract]
public class BioSampleHeader
{
	[DataMember]
  public BioFactor Factor { get; set; }                          
		// Biometric factor. Must be set to 8 for fingerprint
  [DataMember]
  public BioSampleFormat Format { get; set; }             
		// Format owner (vendor) information
  [DataMember]
  public BioSampleType Type { get; set; }  // type of biometric sample
  [DataMember]
  public BioSamplePurpose Purpose { get; set; }           
		//Purpose of biometric sample
	[DataMember]
  public SByte Quality { get; set; }                 
 		// Quality of biometric sample.
		// If we don't support quality, it should be set  to -1.
	[DataMember]
  public BioSampleEncryption Encryption { get; set; }
		// Encryption of biometric sample.
}
~~~
<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Data Member</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">Factor</td>
  <td valign="top">	Biometric factor presented in this Biometric sample. This version only supports fingerprint, so this member should be set to 8.</td>
  </tr>
  <tr>
  <td valign="top">Format	</td>
  <td valign="top">Format of Biometric sample. See <i>BioSampleFormat class</i> below</td>
  </tr>
  <tr>
  <td valign="top">Type</td>
  <td valign="top">	Type of Biometric sample. See <A HREF="#biosampletype-enum">BioSampleType enumeration</A>.
</td>
  </tr>
  <tr>
  <td valign="top">Purpose</td>
  <td valign="top">
  	Purpose of Biometric sample. See <A HREF="#biosamplepurpose-enum"> BioSamplePurpose enumeration</A>.</td>
  </tr>
  <tr>
  <td valign="top">Quality</td>
  <td valign="top">
  	Quality of Biometric sample. Unsupported in this version. Set to -1.</td>
  </tr>
  <tr>
  <td valign="top">Encryption</td>
  <td valign="top">
  	Define what kind of encryption algorithm was used to protect the Biometric sample. See <A HREF = "biosample-encryption-enumeration"> BioSampleEncryption enumeration</A>.</td>
  </tr>   
</table>

### BioSampleFormat class  

BioSampleFormat describes the vendor-specific format of the Biometric sample.

~~~
[DataContract]
public class BioSampleFormat
{
	public BioSampleFormat(UInt16 owner, UInt16 id)
  {
		FormatOwner = owner;
    FormatID = id;
  }
  [DataMember]
	public UInt16 FormatOwner { get; set; }        
		// Biometric owner ID registered with IBIA (http://www.ibia.org/base/
		//	 cbeff/_biometric_org.phpx).
		// 51 for DigitalPersona, 49 for Neurotechnologija
	[DataMember]
  	public UInt16 FormatID;  // Vendor specific format ID
}
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:50%" ALIGN="left">Data Member</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">FormatOwner</td>
  <td valign="top">	Represents the vendor which produced this Biometric sample. Values are assigned and registered by the International Biometric Industry Association (IBIA) (http://www.ibia.org/base/cbeff/_biometric_org.phpx). The supported values are:<BR>&nbsp;&nbsp;&nbsp;
  51 – for DigitalPersona<BR>&nbsp;&nbsp;&nbsp;
  49 – for Neurotechnology</td>
  </tr>
  <tr>
  <td valign="top">FormatID</td>
  <td valign="top">	This value is assigned by the Format Owner and may optionally be registered by the IBIA. Unsupported in this version, so it should be set to 0.</td>
  </tr>
</table>

### <A NAME="biosampletype-enum"></A>BioSampleType enumeration

The BioSampleType enumeration defines the type of Biometric sample.

~~~
[DataContract][Flags]
public enum BioSampleType
{
	[EnumMember]
  RAW              = 0x01,   // Fingerprint image
  [EnumMember]
  INTERMEDIATE     = 0x02,   // Fingerprint feature set
  [EnumMember]
  PROCESSED	= 0x04,           		// Fingerprint template
  [EnumMember]
  ENCRYPTED	= 0x10,
  [EnumMember]
  SIGNED		= 0x20,
}
~~~

<table style="width:90%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:10%" ALIGN="left">Data Member</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">RAW</td>
  <td valign="top">	The sample is a raw (unprocessed) biometric sample. For fingerprints, this means the sample is a Fingerprint Image.</td>
  </tr>
  <tr>
  <td valign="top">
  INTERMEDIATE	</td>
  <td valign="top">The sample is a partially processed biometric sample.For fingerprints, this means the sample is a Fingerprint Feature Set.</td>
  </tr>
  <tr>
  <td valign="top">
  PROCESSED	</td>
  <td valign="top">The sample is a fully processed biometric sample. For fingerprints, this means the sample is a Fingerprint Template.</td>
  </tr>
  <tr>
  <td valign="top">
  ENCRYPTED	</td>
  <td valign="top">Not supported in this version.</td>
  </tr>
  <tr>
  <td valign="top">
  SIGNED	</td>
  <td valign="top">Not supported in this version.</td>
  </tr>   
</table>

### BioSamplePurpose enumeration  

The BioSamplePurpose details the purpose of this biometric sample.

~~~
[DataContract]
public enum BioSamplePurpose
{
	[EnumMember]
  ANY = 0,                          																				// Any purpose
  [EnumMember]
  VERIFY = 1,                       																				// Verification
  [EnumMember]
  IDENTIFY = 2,                     																				// Identification
  [EnumMember]
  ENROLL = 3,                       																				// Enrollment
  [EnumMember]
  ENROLL_FOR_VERIFICATION_ONLY = 4, 																				// Enrollment for verification    
  [EnumMember]
  ENROLL_FOR_IDENTIFICATION_ONLY = 5, // Enrollment for identification
  [EnumMember]
  AUDIT = 6,                        																				// Audit
}
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Data Member</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">ANY</td>
  <td valign="top">	The purpose of this biometric sample is undefined and it can be used for any purpose.</td>
  </tr>
  <tr>
  <td valign="top">VERIFY</td>
  <td valign="top">This biometric sample was created for verification (authentication) purposes only.</td>
  </tr>
  <tr>
  <td valign="top">IDENTIFY</td>
  <td valign="top">This biometric sample was created for identification purposes only.</td>
  </tr>
  <tr>
  <td valign="top">ENROLL</td>
  <td valign="top">This biometric sample was created for enrollment purposes.</td>
  </tr>
  <tr>
  <td valign="top"> 	
  ENROLL_FOR_VERIFICATION_ONLY	</td>
  <td valign="top">This biometric sample was created for audit purposes.</td>
  </tr>
</table>

#### Notes  

If a fingerprint image is sent as a  biometric sample, the ANY purpose should be set.  

If feature extraction is done by the DigitalPersona engine, with any flag except FT_PRE_REG_FTR and FT_REG_FTR, it can be used for both verification and identification so ANY purpose could be provided.

#### <A NAME="biosample-encryption-enumeration">BioSampleEncryption enumeration</A>

BioSampleEncryption specifies the encryption algorithm that was used to protect the biometric sample.

~~~
[DataContract]
public enum BioSampleEncryption
{
	[EnumMember]
  NONE = 0,                               
	[EnumMember]
  XTEA = 1,     // XTEA encryption with well-known key
}
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Enum Member</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">NONE</td>
  <td valign="top">	The biometric sample provided is not encrypted.</td>
  </tr>
  <tr>
  <td valign="top">XTEA</td>
  <td valign="top">	The biometric sample provided is encrypted using the DigitalPersona implementation of XTEA with a hardcoded key.</td>
  </tr>
</table>

#### Notes  

We strongly recommend using XTEA encryption for biometric samples (as we do in DigitalPersona products), but we recognize the complexity of XTEA implementation on different platforms (such as Android) so unencrypted samples are accepted as well.  

### Biometric Sample Data  

The following data formats for fingerprint samples are supported.  
- Fingerprint Feature Set  
-	Raw Fingerprint Image  

#### Fingerprint Feature Set  

To specify a Fingerprint Feature Set in the Biometric sample, the following values in BioSampleHeader must be set.  


<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Value</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">Factor</td>
  <td valign="top">Must be set to 8. (FINGERPRINT).</td>
  </tr>
  <tr>
  <td valign="top">FormatOwner</td>
  <td valign="top">	If the DigitalPersona engine was used for feature extraction, FormatOwner must be set to 51.<BR><BR> If the Neurotechnologija engine was used for feature extraction, FormatOwner must be set to 49.</td>
  </tr>
  <tr>
  <td valign="top">Type</td>
  <td valign="top">Must be set to 2. (INTERMEDIATE).</td>
  </tr>
  <tr>
  <td valign="top">Purpose</td>
  <td valign="top">Can be set to 0 (ANY).</td>
  </tr>
  <tr>
  <td valign="top">Quality</td>
  <td valign="top">Should be set to -1.</td>
  </tr>
  <tr>
  <td valign="top">Encryption</td>
  <td valign="top">	If XTEA encryption was used, Encryption must be set to 1. Otherwise, set to 0.</td>
  </tr>       
</table>

Because we treat the Fingerprint Feature Set as an opaque blob, we must provide the Base64Url encoded feature set blob as the Data member of BioSample.  

Below is an example of JSON representation of a Fingerprint Feature Set.

~~~
{
	"Version":1,
	"Header":
	{
		"Factor":8,          	// Fingerprint
		"Format":
		{
			"FormatOwner":51,    // DigitalPersona engine
			"FormatID":0
		},
		"Type":2,		           // Feature Set
		"Purpose":0,        	// Any purpose
		"Quality":-1,
		"Encryption":0  	    // Unencrypted
	},
	"Data":"eyJ0eXAiOiJKV1QiLA0KICJhbGciOiJIUzI1NiJ9"
		// Base64url encoded Feature Set
}
~~~
#### Fingerprint image  

To specify a Fingerprint Image in the Biometric sample, the following values in BioSampleHeader must be set.  


<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Value</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">Factor</td>
  <td valign="top">Must be set to 8 (FINGERPRINT).
  </td>
  </tr>
  <tr>
  <td valign="top">FormatOwner</td>
  <td valign="top">Is ignored and could be set to 51.</td>
  </tr>
  <tr>
  <td valign="top">Type</td>
  <td valign="top">Must be set to 1 (RAW).</td>
  </tr>
  </tr>
  <tr>
  <td valign="top">Purpose</td>
  <td valign="top">Is ignored and can be set to 0 (ANY).</td>
  </tr>
  </tr>
  <tr>
  <td valign="top">Quality</td>
  <td valign="top">Should be set to -1.</td>
  </tr>  
  <tr>
  <td valign="top">Encryption</td>
  <td valign="top">	If XTEA encryption was used, Encryption must be set to 1. Otherwise, set to 0.</td>
  </tr>    
</table>

#### Notes  

We cannot treat fingerprint image as an opaque blob because the image has many parameters which need to be passed, such as image size, image resolution, etc.  

Therefore, we use the FD_Image format which we are using in “C” code as a prototype for Web image representation to simplify the image conversion on the DigitalPersona Server.  

The following classes are defined in WBF terminology for specifying an image in the biometric sample.

~~~
/* uImageType */
[DataContract]
public enum FpImageType
{
	[EnumMember]
  UNKNOWN =                       0,
  [EnumMember]
  BLACK_WHITE =                   1,
  [EnumMember]
  GRAY_SCALE  =                   2,
  [EnumMember]
  COLOR       =                   3,
}
/* uPadding */
[DataContract]
public enum FpImagePadding
{
	[EnumMember]
  NO_PADDING  =                     0,
  [EnumMember]
  LEFT_PADDING =                    1,
  [EnumMember]
  RIGHT_PADDING =                   2,
}
/* uPolarity */
[DataContract]
public enum FpImagePolarity
{
	[EnumMember]
  UNKNOWN_POLARITY     =            0,
  [EnumMember]
  NEGATIVE_POLARITY    =            1,
  [EnumMember]
  POSITIVE_POLARITY    =            2,
}
/* uRGBcolorRepresentation */
[DataContract]
public enum FpImageColorRepresantation
{
	[EnumMember]
  NO_COLOR_REPRESENTATION = 0,
  [EnumMember]
  PLANAR_COLOR_REPRESENTATION = 1,
  [EnumMember]
  INTERLEAVED_COLOR_REPRESENTATION = 2,
}
[DataContract]
public class FpDataHeader
{
	[DataMember]
  public Byte uDataType { get; set; }            
/* must be 1 which represents 2D image		*/
  [DataMember]
  public UInt64 DeviceId { get; set; }
		/* reserved for future use		            */
	  /* FD_DEVICE_TYPE provides the following information:																																		*/
	  /* 0000     -  05BA      - 0001       - 01             - 04																																						*/
  		/* reserved    vendor Id  														product Id 							major revision  									minor revision		*/
	[DataMember]
  public UInt64 DeviceType { get; set; }                           
		/* image acquisition or device testing may take some time.
		/* Progress indicator													*/
   /* gives a feed back on the progress of the transaction.								*/
  [DataMember]
  public Byte iDataAcquisitionProgress { get; set; } 	/* should be 100					*/
}
[DataContract]
public class FpImageFormat
{
	[DataMember]
  public Byte uDataType { get; set; }               
	/* is 1 which represents 2D image			 */
		[DataMember]
    public FpImageType uImageType { get; set; }              
		/* B&W, gray or color images                 */
    [DataMember]
    public Int32 iWidth { get; set; }                 
		/* image width [pixel]                       */
    [DataMember]
    public Int32 iHeight { get; set; }                 
		/* image height [pixel]                      */
    [DataMember]
    public Int32 iXdpi { get; set; }                   
		/* X resolution [DPI]                        */
    [DataMember]
    public Int32 iYdpi { get; set; }                   
		/* Y resolution [DPI]                        */
    [DataMember]
    public UInt32 uBPP { get; set; }   /*number of bits per pixel																																		*/
    [DataMember]
    public FpImagePadding uPadding { get; set; }                
		/* right or left padding                     */
    [DataMember]
    public UInt32 uSignificantBpp { get; set; }         
		/* number of significant bits per pixel      */
   [DataMember]
   public FpImagePolarity uPolarity { get; set; }               
		/* positive=black print on white background  */
   [DataMember]
   public FpImageColorRepresantation uRGBcolorRepresentation { get; set; }
   [DataMember]
   public UInt32 uPlanes { get; set; }   /* color planes.*/
}
[DataContract]
public enum FpImageCompression
{
	[EnumMember]
  NONE = 0,                     // Data is not compressed
  [EnumMember]
  JASPER_JPEG = 1,              // Jasper JPEG compression
  [EnumMember]
  WSQ = 2,                      // WSQ compression
}
[DataContract]
public class FpImage
{
	public FpImage()
  {
		Header = new FpDataHeader();
    Format = new FpImageFormat();
  }
  [DataMember]
  public Byte Version { get; set; }
	[DataMember]
  public FpDataHeader Header { get; set; }
  [DataMember]
  public FpImageFormat Format { get; set; }
  [DataMember]
  public FpImageCompression Compression { get; set; }
  [DataMember]
  public String Data { get; set; }
}
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Value</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
    <td valign="top">Version</td>
    <td valign="top">	Version of Fingerprint Image format. Must be 1 in this version.</td>
  </tr>
  <tr>
    <td valign="top">Header</td>
    <td valign="top">Header which specifies details of the fingerprint imaging device.</td>
  </tr>
  <tr>
    <td valign="top">Format</td>
    <td valign="top">Format of the fingerprint image. It details image size, image resolution, etc.
  	</td>
  </tr>
  <tr>
    <td valign="top">Compression</td>
    <td valign="top">Compression algorithm used to compress the fingerprint image. In this version the only supported compression algorithm is Jasper JPEG.</td>
  </tr>
  <tr>
    <td valign="top">Data	</td>
    <td valign="top">Base64url encoded fingerprint image.</td>
  </tr>   
</table>

Below is an example of fingerprint image JSON representation.

~~~
{
	"Version":1,
	"Header":
	{
		"uDataType":1,	       // 2D image
		"DeviceId":0,
		"DeviceType":49264417347272704, // U.are.U 5200 device
		"iDataAcquisitionProgress":100
	},
	"Format":
	{
		"uDataType":1,       // 2D image
		"uImageType":2,      // Gray scale
		"iHeight":400,       	// 400 pix width
		"iWidth":400,        	// 400 pix height
		"iXdpi":500,         	// 500 dpi
		"iYdpi":500,         	// 500 dpi
		"uBPP":8,		            // 8 bits per pixel
		"uPadding":2,	        // right padding
		"uSignificantBpp":8  // 8 significant bits per pixel
		"uPolarity":2,	       // positive polarity
		"uPlanes":1,	         // 1 plane
		"uRGBcolorRepresentation":0   // no color representation
	},
	"Compression":0,      												// uncompressed
	"Data":"eyJ0eXAiOiJKV1QiLA0KICJhbGciOiJIUzI1NiJ9" // Base64url encoded image
}
~~~

#### Creating a JSON representation of a BioSample from a fingerprint image  

To create a JSON representation of BioSample from a fingerprint image, perform the following steps.  

1. Capture a fingerprint image using any supported capturing device.  
2. Base64url encode the image raw bytes. These raw bytes should not include any metadata like image size or resolution; those data will be provided in the JSON Header and Format sections.
3.	Knowing the fingerprint image metadata information, create a JSON representation of the FpImage class.
4.	Base64url encode UTF-8 representation of JSON representation of FpImage;
5.	Create JSON representation of BioSample as described in [BioSample class](#biosample-class), setting the string from step #5 as the Data member of BioSample.

  Below we give an example of how to create a JSON representation of BioSample from a fingerprint image.  

  Let’s assume you capture a fingerprint image using a U.are.U 5200 device. The raw image bytes are the following.

  [123,34,116,121,112,34,58,34,74,87,84,34,44,10,32,34,97,108,103,34,58,34,32,82,83,50,53,54,34,125]  

  The Base64url representation of this image is:  

  eyJ0eXAiOiJKV1QiLAogImFsZyI6IiBSUzI1NiJ9

  Knowing the image width and height, resolution, etc we can create a JSON representation of the FpImage class. (Note that the sequence of nodes in JSON representation is unimportant.)  

  ~~~
{"Compression":0,"Data":"eyJ0eXAiOiJKV1QiLA0KICJhbGciOiJIUzI1NiJ9","Format":{"iHeight":400,"iWidth":400,"iXdpi":500,"iYdpi":500,"uBPP":8,"uDataType":1,"uImageType":2,"uPadding":2,"uPlanes":1,"uPolarity":2,"uRGBcolorRepresentation":0,"uSignificantBpp":8},"Header":{"DeviceId":0,"DeviceType":49264417347272704,"iDataAcquisitionProgress":100,"uDataType":1},"Version":1}
  ~~~
Base64url encoding UTF-8 representation of image above, we get the following string:

  ~~~
eyJDb21wcmVzc2lvbiI6MCwiRGF0YSI6ImV5SjBlWEFpT2lKS1YxUWlMQTBLSUNKaGJHY2lPaUpJVXpJMU5pSjkiLCJGb3JtYXQiOnsiaUhlaWdodCI6NDAwLCJpV2lkdGgiOjQwMCwiaVhkcGkiOjUwMCwiaVlkcGkiOjUwMCwidUJQUCI6OCwidURhdGFUeXBlIjoxLCJ1SW1hZ2VUeXBlIjoyLCJ1UGFkZGluZyI6MiwidVBsYW5lcyI6MSwidVBvbGFyaXR5IjoyLCJ1UkdCY29sb3JSZXByZXNlbnRhdGlvbiI6MCwidVNpZ25pZmljYW50QnBwIjo4fSwiSGVhZGVyIjp7IkRldmljZUlkIjowLCJEZXZpY2VUeXBlIjo0OTI2NDQxNzM0NzI3MjcwNCwiaURhdGFBY3F1aXNpdGlvblByb2dyZXNzIjoxMDAsInVEYXRhVHlwZSI6MX0sIlZlcnNpb24iOjF9
  ~~~

Finally, we create a JSON representation of BioSample.

~~~
{
	"Version":1,
	"Header":
	{
		"Factor":8, 	
		"Format":
		{
			"FormatOwner":51,
			"FormatID":0
		},
	"Type":1,		
	"Purpose":0, 	
	"Quality":-1,
	"Encryption":0
	},
	"Data":"eyJDb21wcmVzc2lvbiI6MCwiRGF0YSI6ImV5SjBlWEFpT2lKS1YxUWlMQTBLSUNKaG
	JHY	2lPaUpJVXpJMU5pSjkiLCJGb3JtYXQiOnsiaUhlaWdodCI6NDAwLCJpV2lkdGgiOjQwMCwi
	aVhkcGkiOjUwMCwiaVlkcGkiOjUwMCwidUJQUCI6OCwidURhdGFUeXBlIjoxLCJ1SW1hZ2VUe
	XBlIjoyLCJ1UGFkZGluZyI6MiwidVBsYW5lcyI6MSwidVBvbGFyaXR5IjoyLCJ1UkdCY29sb3JS
	ZXByZXNlbnRhdGlvbiI6MCwidVNpZ25pZmljYW50QnBwIjo4fSwiSGVhZGVyIjp7IkRldmlj
	ZUlkIjowLCJEZXZpY2VUeXBlIjo0OTI2NDQxNzM0NzI3MjcwNCwiaURhdGFBY3F1aXNpdGlv
	blByb2dyZXNzIjoxMDAsInVEYXRhVHlwZSI6MX0sIlZlcnNpb24iOjF9"
}
~~~

#### Creating Fingerprint Credentials from BioSample(s)  

Follow these steps to create a JSON representation of the Credential class which we can send in one of the IDPWebAuth interface methods from one or more biometric samples.  

1.	Base64url encode the binary representation of any fingerprint feature set(s) or encode any fingerprint image(s) as described in the previous topic.
2.	Create a JSON representation BioSample(s).
3.	Combine one or more BioSamples in a JSON array using square brackets []. NOTE: we support multiple biometric samples in one Credential to handle two (multi) finger identification/verification.
4.	Base64url encode UTF-8 representation of JSON array of BioSamples in a string;
5.	Create a JSON representation of the credentials using the Fingerprint ID as the id member and the string created in step 4 as the data member.  

  For example we have a fingerprint feature set which we would like to send to the DigitalPersona Server for identification. This feature set is created using the DigitalPersona fingerprint engine and the following bytes array represents this feature set.  

  [123,34,116,121,112,34,58,34,74,87,84,34,44,10,32,34,97,108,103,34,58,34,32,82,83,50,53,54,34,125]  

  The Base64url encoded representation of this fingerprint feature set is:  

  eyJ0eXAiOiJKV1QiLAogImFsZyI6IiBSUzI1NiJ9  

  The JSON representation of BioSample class for this feature set is:  

  ~~~
  {
	"Version":1,
	"Header":
	{
		"Factor":8,
		"Format":
		{
			"FormatOwner":51,
			"FormatID":0
		},
	"Type":2,
	"Purpose":0,
	"Quality":-1,
	"Encryption":0
	},
	"Data":"eyJ0eXAiOiJKV1QiLA0KICJhbGciOiJIUzI1NiJ9"				
}
  ~~~
  Combining the BioSample(s) in an array we have the following JSON representation:

  ~~~
  [{
	"Version":1,
	"Header":
	{
		"Factor":8,
		"Format":
		{
			"FormatOwner":51,
			"FormatID":0
		},
		"Type":2,
		"Purpose":0,
		"Quality":-1,
		"Encryption":0
		},
	"Data":"eyJ0eXAiOiJKV1QiLA0KICJhbGciOiJIUzI1NiJ9"				
}]
  ~~~

  The Base64url encoded UTF-8 representation of the JSON array above is:  

  ~~~
W3sNCiJWZXJzaW9uIjoxLA0KIkhlYWRlciI6DQp7DQoiRmFjdG9yIjo4LA0KIkZvcm1hdCI6DQp7DQoiRm9ybWF0T3duZXIiOjUxLA0KIkZvcm1hdElEIjowDQp9LA0KIlR5cGUiOjIsDQoiUHVycG9zZSI6MCwNCiJRdWFsaXR5IjotMSwNCiJFbmNyeXB0aW9uIjowIA0KfSwNCiJEYXRhIjoiZXlKMGVYQWlPaUpLVjFRaUxBMEtJQ0poYkdjaU9pSklVekkxTmlKOSIJCQkJDQp9XQ0K  
  ~~~
6. Finally we create a JSON representation of the Credential class which we can send for identification (verification).

  ~~~
{"id":"AC184A13-60AB-40e5-A514-E10F777EC2F9",
"data":"W3sNCiJWZXJzaW9uIjoxLA0KIkhlYWRlciI6DQp7DQoiRmFjdG9yIjo4LA0KIkZvcm1hdCI6DQp7DQoiRm9ybWF0T3duZXIiOjUxLA0KIkZvcm1hdElEIjowDQp9LA0KIlR5cGUiOjIsDQoiUHVycG9zZSI6MCwNCiJRdWFsaXR5IjotMSwNCiJFbmNyeXB0aW9uIjowIA0KfSwNCiJEYXRhIjoiZXlKMGVYQWlPaUpLVjFRaUxBMEtJQ0poYkdjaU9pSklVekkxTmlKOSIJCQkJDQp9XQ0K"}  
  ~~~
#### AuthenticateUser  

To call the AuthenticateUser() method, a fingerprint credential must already have been created as described in the previous section.  

Below is an example of HTTP Body for fingerprint authentication with the fingerprint credential created in the previous section.  

~~~
{
	"user":
	{
		"name":"someone@mycompany.com",
		"type":6
	},
	"credential":
	{
		"id":"AC184A13-60AB-40e5-A514-E10F777EC2F9",
		"data":"W3sNCiJWZXJzaW9uIjoxLA0KIkhlYWRlciI6DQp7DQoiRmFjdG9yIjo4LA0KIkZvc
		m1hdCI6DQp7DQoiRm9ybWF0T3duZXIiOjUxLA0KIkZvcm1hdElEIjowDQp9LA0KIlR5cGUiOj
		IsDQoiUHVycG9zZSI6MCwNCiJRdWFsaXR5IjotMSwNCiJFbmNyeXB0aW9uIjowIA0KfSwNCi
		JEYXRhIjoiZXlKMGVYQWlPaUpLVjFRaUxBMEtJQ0poYkdjaU9pSklVekkxTmlKOSIJCQkJDQp
		9XQ0K"
	}
}
~~~

#### IdentifyUser  

To call the IdentifyUser() method, the caller must create a fingerprint credential as described previously.  

Below is an example of HTTP Body for fingerprint identification with the fingerprint credential created previously.

~~~
{
	"credential":
	{
		"id":"AC184A13-60AB-40e5-A514-E10F777EC2F9",
		"data":"W3sNCiJWZXJzaW9uIjoxLA0KIkhlYWRlciI6DQp7DQoiRmFjdG9yIjo4LA0KIkZvc
		m1hdCI6DQp7DQoiRm9ybWF0T3duZXIiOjUxLA0KIkZvcm1hdElEIjowDQp9LA0KIlR5cGUiO
		jIsDQoiUHVycG9zZSI6MCwNCiJRdWFsaXR5IjotMSwNCiJFbmNyeXB0aW9uIjowIA0KfSwNCi
		JEYXRhIjoiZXlKMGVYQWlPaUpLVjFRaUxBMEtJQ0poYkdjaU9pSklVekkxTmlKOSIJCQkJDQp
		9XQ0K"
	}
}
~~~

#### GetEnrollmentData  

To ask information about enrolled fingerprints, the client should send an IDPWebAuth-> GetEnrollmentData request to the server. Below is an example of such a request.

~~~
https://www.mycompany.com/DPWebAuthService.svc/GetEnrollmentData?user=
someone@mycompany.com&type=6&cred_id=AC184A13-60AB-40e5-A514-E10F777EC2F9  
~~~

The result of a successful GetEnrollmentData request should be a Base64url encoded UTF-8 representation of a list (array) of fingerprints (fingerprint positions) enrolled by the user.  

Below is the ANSI 381 list of valid fingerprint positions.  

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:27%" ALIGN="left">Fingerprint position		</th>
    <th style="width:5%" ALIGN="left">Value</th>
    <th style="width:2%" ALIGN="left"></th>
    <th style="width:27%" ALIGN="left">Fingerprint position</th>
    <th style="width:5%" ALIGN="left">Value</th>            
  </tr>
  <tr>
    <td valign="top">Unknown</td>
    <td valign="top">0  </td>
    <td valign="top"></td>
    <td valign="top"></td>
    <td valign="top"></td>
  </tr>
  <tr>
  <td valign="top">Right thumb</td>
  <td valign="top">		1</td>
  <td valign="top"></td>
  <td valign="top">Left thumb</td>
  <td valign="top">6</td>
  </tr>
  <tr>
  <td valign="top">Right index finger	</td>
  <td valign="top">	2
  	</td>
  <td valign="top"></td>
  <td valign="top">Left index finger</td>
  <td valign="top">		7		</td>
  </tr>
  <tr>
  <td valign="top">Right middle finger</td>
  <td valign="top">3</td>
  <td valign="top"></td>
  <td valign="top">Left middle finger</td>
  <td valign="top">8</td>
  </tr>
  <tr>
  <td valign="top">Right ring finger</td>
  <td valign="top">4</td>
  <td valign="top"></td>
  <td valign="top">Left ring finger</td>
  <td valign="top">9</td>
  </tr>
  <tr>
  <td valign="top">Right little finger</td>
  <td valign="top">5</td>
  <td valign="top"></td>
  <td valign="top">Left little finger</td>
  <td valign="top">10</td>
  </tr>    
</table>

The following GetEnrollmentDataResult indicates that the user has their right thumb, right index finger and left middle fingers enrolled.  

[{"position":1},{"position":2},{"position":8}]  

#### CustomAction
CustimAction is not currently supported for the Fingerprint Credential.  

### Password Credential  

The following ID is defined for Password Credential.  

{D1A1F561-E14A-4699-9138-2EB523E132CC}  

Follow these steps to create a Password Credential.

1. Base64url encode UTF-8 representation of the password. NOTE: It is not necessary to include null terminating character to the UTF-8 representation.
2. Create a JSON representation of the Password credential, setting the Password Credential ID as the id member and the string we created in step #1 as the data member.  

  For example, we create a JSON representation of the following password: P@ssw0rd. The Base64url encoded UTF-8 representation of this password is:

  UEBzc3cwcmQ  

3. Finally we create a JSON representation of the Credential class which we can send for password authentication to the DigitalPersona Server.  

~~~
{"id":"D1A1F561-E14A-4699-9138-2EB523E132CC",
"data":"UEBzc3cwcmQ"}  
~~~

#### AuthenticateUser
To call the AuthenticateUser() method, the caller must create a password credential as described above.  

Below is an example of HTTP Body for password authentication with the password credential created above.

~~~
{
	"user":
	{
		"name":"someone@mycompany.com",
		"type":6
	},
	"credential":
	{
		"id":"D1A1F561-E14A-4699-9138-2EB523E132CC",
		"data":"UEBzc3cwcmQ"
	}
}
~~~

#### IdentifyUser  

The Password credential does not support user identification, so an IdentifyUser() call with a Password credential will return a "Not implemented" error.  

#### GetEnrollmentData  

The password credential does not support the GetEnrollmentData() call and will return a "Not implemented"error.  

#### CustomAction
The following CustomAction operations are supported for the Password Credential.
- Password Randomization
- Password Reset

##### Password Randomization  

For Password Randomization, the Action ID is "4".  

A valid ticket has to be provided to perform Password Randomization. The ticket owner has to have the rights to randomize the user password.  

A valid user for whom the password needs to be randomized needs to be provided.  

The data parameter of the Credential class should be set to "null".  

Below is a valid example of HTTP Body of Password Randomization request.  

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
		"id":"D1A1F561-E14A-4699-9138-2EB523E132CC ",
		"data":null
	},
	"actionId":4
}
~~~

This call will send a Password Randomization request for a Non AD user with the account name of "someone".  

**Note**: We can guarantee successful Password Randomization only for Non AD Users in DigitalPersona LDS installations. We can randomize the  password for an AD user only in a  DigitalPersona AD installation.

##### Password Reset

Password Reset Action ID is "13".  

A valid ticket has to be provided to perform Password Reset. The ticket owner has to have rights to *set user password*.

A valid user to whom password needs to be reset needs to be provided and the
Data parameter of the Credential class should be set to Base64Url encoded UTF8 representation of the user's new password.  

Below is a valid example of the HTTP Body for a Password Reset request.  

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
"id":"D1A1F561-E14A-4699-9138-2EB523E132CC ",
"data":"UEBzc3cwcmQ"
},
"actionId":13
}
~~~

This call will send Password Reset request for Altus user with account name "someone".  

**Note**: We can only guarantee a successful Password Reset  for Altus Users in Altus LDS installations and can only reset the password for AD users in Altus AD installations.  

### PIN Credential  

The following ID is defined for the PIN Credential.  

{8A6FCEC3-3C8A-40c2-8AC0-A039EC01BA05}  

Follow these steps to create a PIN Credential.  

1. Base64url encode the UTF-8 representation of the PIN. Note that it is not necessary to include null terminating character to the UTF-8 representation.  
2.	Create a JSON representation of the PIN credential, setting the PIN Credential ID as the id member and the string created in step #1 as the data member.  

  For example, we create a JSON representation of the following PIN: 1234. The Base64url encoded UTF-8 representation of this PIN is:  

  MTIzNA  

3. Finally we create a JSON representation of Credential class which we can send for PIN authentication to the DigitalPersona Server.  

~~~{"id":"8A6FCEC3-3C8A-40c2-8AC0-A039EC01BA05",
"data":"MTIzNA"}
~~~

#### AuthenticateUser  

To call the AuthenticateUser() method, the caller must create a PIN credential as described above.  

Below is an example of the HTTP Body for PIN authentication with the previously created PIN credential.  

~~~
{
	"user":
	{
		"name":"someone@mycompany.com",
		"type":6
	},
	"credential":
	{
		"id":"8A6FCEC3-3C8A-40c2-8AC0-A039EC01BA05",
		"data":"MTIzNA"
	}
}
~~~

#### IdentifyUser  

The PIN credential does not support user identification, so an IdentifyUser() call with a PIN credential will return a "Not implemented" error.  

#### GetEnrollmentData  

The PIN credential does not support the GetEnrollmentData() call, and will return a "Not implemented" error.  

#### CustomAction  

CustomAction is not currently supported for the PIN Credential.

### Recovery Questions Credential  

The following ID is defined for the Recovery Questions Credential.  

{B49E99C6-6C94-42DE-ACD7-FD6B415DF503}  

Before a user can answer their  Recovery Questions, the software needs to know which questions to ask the user, the user language id, etc. To ask for this information, the software sends a IDPWebAuth-> GetEnrollmentData request to the server.  

Below is an example of such a request.  

~~~
https://www.digitalpersona.com/DPWebAuthService.svc/GetEnrollmentData?user=
someone@mycompany.com&type=6&cred_id=B49E99C6-6C94-42DE-ACD7-FD6B415DF503
~~~
In the following topic, details are provided about the data returned as a result of this request.  

#### GetEnrollmentData  

The result of a successful GetEnrollmentData request should be a Base64url encoded UTF-8 representation of the list (array) of questions enrolled by the user. The LiveQuestion class represents every question in the list.  

~~~
[DataContract]
public enum LiveQuestionType
{
	[EnumMember]
  REGULAR = 0,         																								// Regular Question
  [EnumMember]
  CUSTOM = 1,          																								// Custom Question
}
[DataContract]
public class LiveQuestion
{
	[DataMember]
  public Byte version { get; set; }    
		// version of Live Question class. Should be set to 1.
  [DataMember]
  public Byte number { get; set; }     																								// Question number
  [DataMember]
  public LiveQuestionType type { get; set; } // Question type
  [DataMember]
  public Byte lang_id { get; set; }        
		// Language ID should be used to display this question
	[DataMember]
	public Byte sublang_id { get; set; }    
		// Sublanguage ID should be used to display this question
  [DataMember]
	public UInt32 keyboard_layout { get; set; }  
		// Keyboard layout should be used to type answer
	[DataMember]
  public String text { get; set; }     
		// Custom question text. For custom questions only.
}
~~~
<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Value</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">version</td>
  <td valign="top">	Version of Live Question format. Must be 1 in this version.</td>
  </tr>
  <tr>
  <td valign="top">number</td>
  <td valign="top">
  	Live Question number. This number has a meaning only for regular questions (see below).</td>
  </tr>
  <tr>
  <td valign="top">type</td>
  <td valign="top">
  	Type of Live Question. We support two types of Live Question: 1) Regular and 2) Custom.</td>
  </tr>
  <tr>
  <td valign="top">lang_id</td>
  <td valign="top">	This primary language id was used to display the question during enrollment. The client application should (if possible) use the same primary language id to display the question.</td>
  </tr>
  <tr>
  <td valign="top">sublang_id</td>
  <td valign="top">
  	This sublanguage id was used to display the question during enrollment. The client application should (if possible) use the same sublanguage id to display the question.</td>
  </tr>
  <tr>
  <td valign="top">keyboard_layout</td>
  <td valign="top">	This keyboard layout was used for typing an answer to the question during enrollment. The client application must use the same keyboard layout for the user to enter the answer to this question.</td>
  </tr>
  <tr>
  <td valign="top">text</td>
  <td valign="top">
  	Question text. Text should be provided for both Custom and Regular questions and must be in the appropriate language (correspond lang_id and sublang_id as described above).</td>
  </tr>      
</table>

Below is an example of the JSON representation of a Regular Live Question.

~~~
{
	"version":1,														// must be set to 1
	"number":2,														// question number in regular question list
	"type":0,														// type - regular question
	"lang_id":9,														// language - English
	"sublang_id":1,														// Sublanguage - English US
	"keyboard_layout":1033,														// Keyboard layout - English US
	"text”:”What was the name of the first school you attended?”
															// text of the question
}
~~~

Below is an example of JSON representation of a Custom Live Question.

~~~
{
	"version":1,														// must be set to 1
	"number":102,														// question number
	"type":1,														// type - custom question
	"lang_id":9,														// language - English
	"sublang_id":1,														// Sublanguage - English US
	"keyboard_layout":1033,														// Keyboard layout - English US
	"text":"Date of your employment."																			// custom question text
}
~~~
##### Parsing GetEnrollmentDataResult  

These are the steps to process (parse) GetEnrollmentDataResult.  

1. Get the string provided in GetEnrollmentDataResult.
2.	Base64url decode the string to get the UTF-8 representation of the Live Questions list.
3.	Decode the UTF-8 string to a format compatible with the JSON parser.
4.	Using the JSON parser, parse the string to objects of the LiveQuestion class.  

The following is an example of <A NAME="getenrollmentdata-parsing">GetEnrollmentDataResult parsing</A>.  

1. The client application makes the following call.  

  ~~~
  https://www.mycompany.com/DPWebAuthService.svc/GetEnrollmentData?user=
someone@mycompany.com&type=6&cred_id=B49E99C6-6C94-42DE-ACD7-FD6B415DF503
and receives the following result:
{"GetEnrollmentDataResult":"W3sia2V5Ym9hcmRfbGF5b3V0IjoxMDMzLCJsYW5nX2lkIjo5L
CJudW1iZXIiOjIsInN1YmxhbmdfaWQiOjEsInRleHQiOm51bGwsInR5cGUiOjAsInZlcnNpb24iOj
F9LHsia2V5Ym9hcmRfbGF5b3V0IjoxMDMzLCJsYW5nX2lkIjo5LCJudW1iZXIiOjYsInN1Ymxhbmd
aWQiOjEsInRleHQiOm51bGwsInR5cGUiOjAsInZlcnNpb24iOjF9LHsia2V5Ym9hcmRfbGF5b3V0I
joxMDMzLCJsYW5nX2lkIjo5LCJudW1iZXIiOjYsInN1YmxhbmdfaWQiOjEsInRleHQiOiJEYXRlIG
9mIHlvdXIgZW1wbG95bWVudC4iLCJ0eXBlIjoxLCJ2ZXJzaW9uIjoxfV0"}
  ~~~
2. Parsing the result would give us the following string.  

  ~~~
  W3sia2V5Ym9hcmRfbGF5b3V0IjoxMDMzLCJsYW5nX2lkIjo5LCJudW1iZXIiOjIsInN1Ymxhbmdfa
WQiOjEsInRleHQiOm51bGwsInR5cGUiOjAsInZlcnNpb24iOjF9LHsia2V5Ym9hcmRfbGF5b3V0Ij
oxMDMzLCJsYW5nX2lkIjo5LCJudW1iZXIiOjYsInN1YmxhbmdfaWQiOjEsInRleHQiOm51bGwsInR
5cGUiOjAsInZlcnNpb24iOjF9LHsia2V5Ym9hcmRfbGF5b3V0IjoxMDMzLCJsYW5nX2lkIjo5LCJu
dW1iZXIiOjYsInN1YmxhbmdfaWQiOjEsInRleHQiOiJEYXRlIG9mIHlvdXIgZW1wbG95bWVudC4iL
CJ0eXBlIjoxLCJ2ZXJzaW9uIjoxfV0
  ~~~
3. Base64url decoding the string above will give us the following JSON representation.

    ~~~
[{"keyboard_layout":1033,"lang_id":9,"number":2,"sublang_id":1,
"text":”What was the name of the first school you attended?”,
"type":0,"version":1},{"keyboard_layout":1033,"lang_id":9,"number":6,
"sublang_id":1,"text":”Who was your first employer?”,
"type":0,"version":1},{"keyboard_layout":1033,"lang_id":9,"number":102,
"sublang_id":1,"text":"Date of your employment.","type":1,"version":1}]
  ~~~
4. Using the JSON parser, we have the following Live Questions.  

  ~~~
{
	"version":1,
	"number":2,		
	"type":0,		
	"lang_id":9,
	"sublang_id":1,
	"keyboard_layout":1033,
	"text":”What was the name of the first school you attended?”
}
{
	"version":1,
	"number":6,
	"type":0,
	"lang_id":9,
	"sublang_id":1,
	"keyboard_layout":1033,
	"text":”Who was your first employer?”
}
{
	"version":1,
	"number":102,
	"type":1,
	"lang_id":9,
	"sublang_id":1,
	"keyboard_layout":1033,
	"text":"Date of your employment."
}
  ~~~
##### Creating the Live Questions Credential  

The LiveAnswer class represents the answers to LiveQuestion.  

~~~
[DataContract]
public class LiveAnswer
{
	[DataMember]
  public Byte version { get; set; }    
		// Version of Live Question class. Should be set to 1.
	[DataMember]
  public Byte number { get; set; }     // Question number
  [DataMember]
  public String text { get; set; }     // Question answer
}
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Value</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">version</td>
  <td valign="top">	Version of Live Answer format. Must be 1 in this version.</td>
  </tr>
  <tr>
  <td valign="top">number</td>
  <td valign="top">
  	Live Question number. This number must correspond to the question number received in GetEnrollmentDataResult.</td>
  </tr>
  <tr>
  <td valign="top">text</td>
  <td valign="top">
  	Answer text.</td>
  </tr>
</table>

Below is the JSON representation of LiveAnswer to the question number 102 from the [Parsing GetEnrollmentDataResult](#getenrollmentdata-parsing).

~~~
{
	"version":1,		// must be set to 1
	"number":102,		// question number
	"text":"04/24/2009"	// custom question text
}
~~~~

##### Steps to create a Live Questions Credential  

To create the Live Questions Credential, perform the following steps.  

1. Collect user answers.
2. Create a JSON representation of LiveAnswer for every answer.
3. Combine those answers in a JSON array.
4. Base64url encode UTF-8 representation of string created in step #3 above.
5. Create a JSON representation of the Credential class using Live Question Credential ID as id member and the string created in step #4 as the data member.
  For example, assume the user gave the following answers to the Live Questions in the above example.  

  Canyon Middle  
  SampleCo  
  04/24/2009  

  Based on those answers, the client application must create the following JSON representation for the array of LiveAnswer objects:
  ~~~
[
{
	"version":1,
	"number":2,		
	"text":"Canyon Middle "
},
{
	"version":1,
	"number":6,
	"text":"SampleCo"
},
{
	"version":1,
	"number":102,
	"text":"04/24/2009"
}
]
  ~~~
  Base64url encoding UTF-8 representation of string above, creates the following value.

  ~~~
  W3sidmVyc2lvbiI6MSwibnVtYmVyIjoyLCJ0ZXh0IjoiTmV3IFlvcmsifSx7InZlcnNpb24iOjEsIm51bWJlciI6NiwidGV4dCI6IkNhbnlvbiBNaWRkbGUifSx7InZlcnNpb24iOjEsIm51bWJlciI6MTAyLCJ0ZXh0IjoiMDQvMjQvMjAwOSJ9XQ  
  ~~~
6. Finally we create the JSON representation of the Live Question Credentials.

  ~~~
{"id":"B49E99C6-6C94-42DE-ACD7-FD6B415DF503",
"data":"W3sidmVyc2lvbiI6MSwibnVtYmVyIjoyLCJ0ZXh0IjoiTmV3IFlvcmsifSx7InZlcnNpb24iOjEsIm51bWJlciI6NiwidGV4dCI6IkNhbnlvbiBNaWRkbGUifSx7InZlcnNpb24iOjEsIm51bWJlciI6MTAyLCJ0ZXh0IjoiMDQvMjQvMjAwOSJ9XQ"}
  ~~~

#### AuthenticateUser  

To call the AuthenticateUser() method, the caller must create the LiveQuestions credential as described above.  

Below is an example of an HTTP Body for LiveQuestions authentication with the LiveQuestion credential previously created.  

~~~
{
	"user":
	{
		"name":"someone@mycompany.com",
		"type":6
	},
	"credential":
	{
		"id":"B49E99C6-6C94-42DE-ACD7-FD6B415DF503",
		"data":"W3sidmVyc2lvbiI6MSwibnVtYmVyIjoyLCJ0ZXh0IjoiTmV3IFlvcmsifSx7InZl
		cnNpb24iOjEsIm51bWJlciI6NiwidGV4dCI6IkNhbnlvbiBNaWRkbGUifSx7InZlcnNpb24i
		OjEsIm51bWJlciI6MTAyLCJ0ZXh0IjoiMDQvMjQvMjAwOSJ9XQ"
	}
}
~~~

#### IdentifyUser  

The LiveQuestions credential does not support user identification, so an IdentifyUser() call with a LiveQuestions credential will return a "Not implemented" error.  

#### CustomAction  

CustomAction is not currently supported for the Live Questions Credential.  

### Proximity Card Credential  

The following ID is defined for the Proximity Card Credential.  

{1F31360C-81C0-4EE0-9ACD-5A4400F66CC2}  

We treat the Proximity Card Data (Prox Card ID) as an opaque blob.  

Follow these steps to create the Prox Card Credential.  

1. Base64url encode the Prox Card ID.
2. Create the JSON representation of the Credential class, setting the Proximity Card Credential ID as the id member and the string created in step #1 as the data member.  

  For example, the client gets the Prox Card ID and it is represented by the following byte array.  

  ~~~
  [123,34,116,121,112,34,58,34,74,87,84,34,44,10,32,34,97,108,103,34,58,34,32,82,83,50,53,54,34,125]  
  ~~~  

  The Base64url encoded value of the Prox Card ID is:  

  eyJ0eXAiOiJKV1QiLAogImFsZyI6IiBSUzI1NiJ9  

  Then we create a JSON representation of the Credential class which we can send for Prox Card identification or authentication to the DigitalPersona Server.  

  ~~~
  {"id":"1F31360C-81C0-4EE0-9ACD-5A4400F66CC2",
"data":"eyJ0eXAiOiJKV1QiLAogImFsZyI6IiBSUzI1NiJ9"}  
  ~~~
#### AuthenticateUser  

To call the AuthenticateUser() method, the caller must create a Proximity Card credential as described above.  

Below is an example of an HTTP Body for Proximity Card authentication with a previously created Proximity card credential.  

~~~
{
	"user":
	{
		"name":"someone@mycompany.com",
		"type":6
	},
	"credential":
	{
		"id":"1F31360C-81C0-4EE0-9ACD-5A4400F66CC2",
		"data":"eyJ0eXAiOiJKV1QiLAogImFsZyI6IiBSUzI1NiJ9"
	}
}
~~~

#### IdentifyUser  

To call the IdentifyUser() method, the caller must create a Proximity Card credential as described above.  

Below is an example of HTTP Body for Proximity Card identification with the previously created Proximity card credential.  

~~~
{
	"credential":
	{
		"id":"1F31360C-81C0-4EE0-9ACD-5A4400F66CC2",
		"data":"eyJ0eXAiOiJKV1QiLAogImFsZyI6IiBSUzI1NiJ9"
	}
}
~~~

#### GetEnrollmentData  

The Proximity Card credential does not support the GetEnrollmentData() call, and will return a "Not implemented" error.  

#### CustomAction  

CustomAction is not currently supported for the Proximity Card Credential.  

### Time-Based OTP (TOTP) Credential  

The following ID is defined for the TOTP Credential.

{324C38BD-0B51-4E4D-BD75-200DA0C8177F}  

We treat the OTP code as a regular string.  

Follow these steps to create an OTP Credential.  

1. Base64url encode the UTF-8 representation of the OTP code. Note thatit is not necessary to include a null terminating character in the UTF-8 representation.
2. Create a JSON representation of the OTP credential, setting the TOTP Credential ID as the id member and the string  created in step #1 as the data member.  

  For example, to create a JSON representation of the following OTP: 123456:

  The Base64url encoded UTF-8 representation of such OTP is:  

  MTIzNAdT  

  We then create a JSON representation of the Credential class which we can send for OTP authentication to the DigitalPersona Server.

  ~~~
  {"id":"324C38BD-0B51-4E4D-BD75-200DA0C8177F","data":"MTIzNAdT"}
  ~~~

**Note**: To use Push Notification, the OTP caller must use the word "push" instead of the OTP code.  

Below is example of JSON representation of Credential class which we can send for Push Notification OTP authentication to the DigitalPersona Server.

~~~
{"id":"324C38BD-0B51-4E4D-BD75-200DA0C8177F",
"data":"cHVzaA"}
~~~

#### AuthenticateUser  

To call the AuthenticateUser() method, the caller must create a TOTP credential as described above.  

Below is an example of HTTP Body for TOTP authentication with a previously created TOTP credential.  

~~~
{
	"user":
	{
		"name":"somebody@mycompany.com",
		"type":6
	},
	"credential":
	{
		"id":"324C38BD-0B51-4E4D-BD75-200DA0C8177F",
		"data":"MTIzNAdT"
	}
}
~~~  

Below is an example of HTTP Body for Push Notification OTP authentication with TOTP credential created above.  

~~~
{
	"user":
	{
		"name":"someone@mycompany.com",
		"type":6
	},
	"credential":
	{
		"id":"324C38BD-0B51-4E4D-BD75-200DA0C8177F",
		"data":"cHVzaA"
	}
}
~~~  

#### IdentifyUser  

The TOTP credential does not support user identification, so an IdentifyUser() call with a TOTP credential will return a "Not implemented" error.  

#### GetEnrollmentData  

The result of a successful GetEnrollmentData request should be a Base64url encoded UTF-8 representation of our OTP-based information. The  OTPEnrollmentData class  represents this information.

~~~
[DataContract]
public class OTPEnrollmentData
{
	[DataMember]
	public String pn_tenant_id { get; set; }    // Push Notification Tenant Id.
	[DataMember]
	public String pn_api_key { get; set; }      // Push Notification API Key.
	[DataMember]
	public String nexmo_api_key { get; set; } 																							// Nexmo API Key.
	[DataMember]
	public String nexmo_api_secret { get; set; }	// Nexmo API Secret
	[DataMember]
	public String phoneNumber { get; set; } // Last 4 digits of registered Phone number
	[DataMember]
	public String serialNumber { get; set; } // Last 4 digits of HD OTP Serial number
}
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Value</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
    <td valign="top">pn_tenant_id</td>
    <td valign="top">	Push Notification Tenant Id. This is an optional parameter and if the customer does not subscribe to  DigitalPersona Push Notification this parameter will be set to null.</td>
  </tr>
  <tr>
    <td valign="top">pn_api_key</td>
    <td valign="top">	Push Notification API Key. This is an  optional parameter and if the customer does not subscribe for DigitalPersona Push Notification this parameter will be set to null.</td>
  </tr>
  <tr>
    <td valign="top">nexmo_api_key</td>
    <td valign="top">	API Key for Nexmo SMS Gateway. The real API key would not be returned here. If the Nexmo API Key is set by the customer, the word "set" would be returned here. Otherwise this parameter would be omitted</td>
  </tr>
  <tr>
    <td valign="top">nexmo_api_secret</td>
    <td valign="top">	API Secret for Nexmo SMS Gateway. The real API Secret would not be returned here. If the Nexmo API Secret is set by the customer, the word "set" would be returned here. Otherwise this parameter would be omitted.</td>
  </tr>
  <tr>
    <td valign="top">phoneNumber</td>
    <td valign="top">	Last 4 digits of registered phone number which would be used for SMS OTP.</td>
  </tr>
  <tr>
    <td valign="top">serialNumber</td>
    <td valign="top">	Last 4 digits of hardware OTP token serial number assigned to this user.</td>
  </tr>
</table  

Below is an example of JSON representation of GetEnrollmentDataResult.  

~~~
{
	"pn_tenant_id":"crossmatch.com",
	"pn_api_key":"25317484582342934",
	"nexmo_api_key":"2345467865",
	"nexmo_api_secret":"2345467865",
	"phoneNumber":"7304",
	"serialNumber":"7895",
}
~~~

##### Parsing GetEnrollmentDataResult  

Perform the following steps to process (parse) GetEnrollmentDataResult.  

1. Get the string provided in GetEnrollmentDataResult.  
2. Base64url decode the string above to get the UTF-8 representation of OTPEnrollmentData.  
3. Decode the UTF-8 string to a format compatible with JSON parser
<mark style="color:Red;">(Unicode?)</mark>.
4	Using a JSON parser, parse the string to objects of the OTPEnrollmentData class.  

#### CustomAction  

The following CustomAction operations are currently supported for the TOTP Credential.  

- Send SMS OTP Request  
- Send E-Mail OTP Request  

##### Send SMS OTP Request  

There are two possible options for an SMS OTP request.  

- Send SMS request for Enrollment  
- Send SMS request for Authentication  

######	Send SMS Request for Enrollment  

Send SMS Request operation Action ID is "513".  

The caller does not need to provide a  valid ticket to perform this operation, so the ticket parameter may be set to "null".  

A valid user to whom SMS needs to be send needs to be provided.  

The Data parameter of the Credential class should be set to the Base64Url encoded Json Representation of OTPEnrollData class.

The following class represents the  enrollment sample for TOTP credentials.

~~~
[DataContract]
public class OTPEnrollData
{
	[DataMember]
	public String otp{ get; set; }      // String with OTP verification code
	[DataMember]
	public String key { get; set; }     // String with Base64Encoded OTP key
	[DataMember]
	public String phoneNumber { get; set; }     // User phone number
}
~~~

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Value</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">otp</td>
  <td valign="top">	String with OTP verification code. In case of Send SMS action this parameter should omitted or set to "null".</td>
  </tr>
  <tr>
  <td valign="top">key</td>
  <td valign="top">	Bease64Url Encoded TOTP key.</td>
  </tr>
  <tr>
  <td valign="top">phoneNumber</td>
  <td valign="top">
  	User's phone number.</td>
  </tr>   
</table>

Below is an example of an HTTP Body of Send SMS request for Enrollment.

~~~
{
	"ticket":{"jwt":null},
	"user":
	{
		"name":"someone",
		"type":9
	},
	"credential":
	{
		"id":"324C38BD-0B51-4E4D-BD75-200DA0C8177F",
		"data":"eyJ0eXAiOiJKV1QiLAogImFsZyI6IiBSUzI1NiJ9"
	},
	"actionId":513
}
~~~

This call will send Enrolment Send SMS request for DigitalPersona user with account name "someone".  

###### Send SMS Request for Authentication  
Send SMS Request operation Action ID is "513".  

The caller does not need to provide a  valid ticket to perform this operation, so the ticket parameter may be set to "null".  

A valid user to whom SMS needs to be sent has to be provided.  

The Data parameter of the Credential class should be set "null".  

Below is a example of an HTTP Body of Send SMS request for Authentication.

~~~
{
	"ticket":{"jwt":null},
	"user":
	{
		"name":"someone",
		"type":9
	},
	"credential":
	{
		"id":"324C38BD-0B51-4E4D-BD75-200DA0C8177F",
		"data":null
	},
	"actionId":513
}
~~~

This call will send Authentication Send SMS request for DigitalPersona user with account name "someone".  

###### Send E-Mail OTP Request

The Send E-Mail OTP Request operation Action ID is "514".  

The caller does not need to provide a  valid ticket to perform this operation, so the ticket parameter may be set to "null".  

A valid user to whom the email needs to be sent has to be provided. The user email address would be retrieved from the "E-mail-Addresses" attribute (Ldap-Display-Name is "mail") of the user account object in Active Directory.  

If the "mail" attribute is not set in the user account in AD, the call will fail. If multiple mail attributes are  set in AD, the OTP code will be sent to all of the addresses.  

A valid user to whom the OTP code requests are to be mailed must be provided.  

The Data parameter of the Credential class should be set "null".  

Below is an example of an HTTP Body for a  Send E-Mail OTP request.  

~~~
{
	"ticket":{"jwt":null},
	"user":
	{
		"name":"someone@mycompany.com",
		"type":6
	},
	"credential":
	{
		"id":"324C38BD-0B51-4E4D-BD75-200DA0C8177F",
		"data":null
	},
	"actionId":514
}
~~~

This call will send the OTP code over email for the AD user with the UPN name "someone@mycompany.com".  

### Smart Card Credential  

The following ID is defined for Smart Card Credential.  

{D66CC98D-4153-4987-8EBE-FB46E848EA98}  

#### AuthenticateUser  

The data for a Smart Card credential is a Base64url encoded UTF-8 representation of the JSON array of the CDPJsonSCAuthToken classes.

Every public key is represented by the CDPJsonSCAuthToken class.  

~~~
[DataContract]
public class CDPJsonSCAuthToken
{
	[DataMember]
	public Byte version { get; set; }       	// version
	[DataMember]
	public UInt64 timeStamp { get; set; }   // current time
	[DataMember]
	public String keyHash { get; set; }     // public key's hash.
	[DataMember]
	public String signature { get; set; }   // signature
}
~~~

where:  

<table style="width:95%;margin-left:auto;margin-right:auto;">
  <tr>
    <th style="width:20%" ALIGN="left">Value</th>
    <th style="width:35%" ALIGN="left">Description</th>
  </tr>
  <tr>
  <td valign="top">Byte <i>version</i></td>
  <td valign="top">	The version of the CDPJsonSCAuthToken object, which must be set to 1 for the current implementation.</td>
  </tr>
  <tr>
  <td valign="top">UInt64 <i>timeStamp</i></td>
  <td valign="top">	The UTC time when the object is created (64-bit value representing the number of 100-nanosecond intervals since January 1, 1601).</td>
  </tr>
  <tr>
  <td valign="top">String <i>keyHash</i></td>
  <td valign="top">	Public key's hash, Base64url UTF-8 encoded string.<BR><BR>  

  The public key from the Smart Card must be imported in the PUBLICKEYBLOB  format (see https://msdn.microsoft.com/en-us/library/ee442238.aspx). After that, the RSA256 hash (see https://en.wikipedia.org/wiki/Secure_Hash_Algorithm) of the key must be calculated. The resulting 32 bytes must be Base64url encoded to the string - KeyHash = Base64urlEncode(RSA256 Hash (PUBLICKEYBLOB (PuK))</td>
  </tr>
  <tr>
  <td valign="top"></td>
  <td valign="top"></td>
  </tr>   
</table>



 String signature 	Timestamp and Public key’s hash, Base64url UTF-8 encoded string.
The Timestamp (8 bytes) and Public key’s RSA256 Hash (32 bytes) must be combined into a 40 byte array, where the first 8 bytes is the Timestamp and the remainder is the hash.
This 40 bytes blob must be hashed again with RSA256 and then signed with the Smart Card’s Private Key.
The signature algorithm used is specified when the Smart Card key pair is originally created, usually RSA. The resulting signature blob must be Base64url encoded into the string:
KeyHash = Base64urlEncode(Sign(PrK)( RSA256 Hash( Timestamp + RSA256 Hash (PUBLICKEYBLOB (PuK))))

To create the Smart Card Credential for authentication, follow these steps on the client.
1	Enumerate the asymmetric key pairs on the Smart Card or select the exact key pair to use.
2	Create a JSON representation of the CDPJsonSCAuthToken class for every key pair from step #1.
3	Combine the CDPJsonSCAuthToken classes into JSON array, even if there's only one CDPJsonSCAuthToken.
4	Base64url UTF-8 encode the representation of the string created in step 3.
5	Create a JSON representation of the Credential class using the Smart Card Credential ID as the id member and the string created in step 4 as a data member.
IdentifyUser
This method is not supported. The Smart Card token does not support user identification.
GetEnrollmentData
This method returns the list of Smart Card credentials enrolled for the user. It can be used to select the Smart Card token for authentication or to delete the token using the DeleteUserCredentials method. Te method does not require that the Smart Card be inserted into the reader when it is called.
As result of this call, a string presenting the Base64url encoded UTF-8 representation of the JSON array of the CDPJsonSCEnrolledToken classes will be returned. Every enrolled Smart Card credential is represented by the CDPJsonSCEnrolledToken class:
[DataContract]
public class CDPJsonSCEnrolledToken
{
	[DataMember]
	public Byte version { get; set; }        // version
	[DataMember]
	public UInt64 timeStamp { get; set; }    // enrollment time
	[DataMember]
	public String keyHash { get; set; }      // public key's hash.
	[DataMember]
	public String nickname { get; set; }     // token's nickname
}
where:

 Parameter	 Description
Byte version	The version of the CDPJsonSCEnrolledToken object, which must be set to 1 for the current implementation.
UInt64 timeStamp	The UTC time when the token is enrolled (64-bit value representing the number of 100-nanosecond intervals since January 1, 1601).
String keyHash	Public key's hash, Base64url UTF-8 encoded string.
The public key from the Smart Card must be imported in the PUBLICKEYBLOB  format (see http://msdn.microsoft.com/en-us/library/aa387459(VS.85).aspx). After that, the RSA256 hash (see https://en.wikipedia.org/wiki/Secure_Hash_Algorithm) of the key is calculated. The resulting 32 bytes are Base64url encoded to the string - KeyHash = Base64urlEncode(RSA256 Hash (PUBLICKEYBLOB (PuK))
This unique string can be used to unambiguously indicate the card token in the DeleteUserCredentials method.
 String nickname 	The nickname of the smart card token.This string, along with enrollment time, can be used in the UI to list enrolled tokens.

CustomAction
CustomAction is not currently supported for the Smart Card Credential.
Face Credential
One of the following third-party SDKs can be used to support the Face Credential in your application.
•	Cognitec FVSDK ver. 9.1.0
•	Innovatrics IFace SDK ver. 3.1.0
All the DigitalPersona Servers and Clients in the environment must use the same SDK. Enrollment data is not compatible between the SDKs!
The following ID is defined for Face Credentials: {85AEAA44-413B-4DC1-AF09-ADE15892730A}
The BioSample class used in description below is declared in "Altus 1 1 WAS Credentials Format.docx"
AuthenticateUser
The data for Face Authentication is a Base64url encoded UTF-8 representation of the JSON array, containing BioSample objects(s).
The following data members for BioSample should be provided for authentication.

Data member	Description
BioFactor	Must be set to 2 (DP_BIO_FACTOR::FACIAL_FEATURES)
BioSamplePurpose	Must be set to 0 (Any purpose) or 1 (purpose Verification)
BioSampleEncryption	Must be 0 (not encrypted) or 1 (XTEA encryption)
BioSampleType	Must be one of the following:
	DP_BIO_SAMPLE_TYPE::PROCESSED
	DP_BIO_SAMPLE_TYPE::RAW

DP_BIO_SAMPLE_TYPE::PROCESSED image
Indicates a face template in the internal SDK format.
•		Cognitec FIR format;
•		Innovatrics Template format.
When the input credential data is already a face template (type 4), then only one BioSample object is expected in the array.
Data member	Description
BioSampleType	Must be 4.
BioSampleFormat->FormatOwner	"Organization Identifier" number from the International Biometrics Identity Association.
•	0x63 (99) for Cognitec
•	0x35 (53) for Innovatrics
Data	Base64url encoded JSON representation of the CDPJsonFIR class described below.

[DataContract]
public class CDPJsonFIR
{
	[DataMember]
	public Byte Version { get; set; }    // version
	[DataMember]
	public long SDKVersion { get; set; } // SDK version
	[DataMember]
	public String Data { get; set; }     // FIR object
}
where:

Data member	Description
Byte Version 	Specifies the version of the CDPJsonFIR object. It must be set to 1.   
ULONG SDKVersion	 Specifies the version of the SDK:
•	Cognitec FVSDK, must be not less than 0x90100 (ver. 9.1.0).
•	Innovatrics IFace SDK, must be not less than 0x30100 (ver. 3.1.0).
String Data	Contains a Base64url encoded BYTE array.
•		Cognitec SDK: this BYTE array is a serialized Cognitec FIR object, created using FIR::writeTo() method.
•		Innovatrics SDK: this BYTE array is a Template object, created using IFACE_CreateTemplate function.

If BioSampleEncryption is set to 1 (XTEA encryption), then this data is encrypted.
Below is an example of JSON representation of the CDPJsonFIR object containing the Cognitec FIR object:
{
"Version":1,
"SDKVersion":?590080??, //? ?0x90100?, ver. 9.1.0
"Data":"2lvbiI6MSwibnVtYmVyIjoyLCJ0ZXh0IjoiTmV3IFlvcmsifSx7InZlcnNpb24iOjEsI
m51bWJlciI6NiwidGV4dCI6IkNhbnlvbiBNaWRkbGUifSx7InZlcnNpb24iOjEsIm51bWJlciI6MT
AyLCJ0ZXh0IjoiMDQvMjQvMjAwOSJ9XQ" // Base64url encoded serialized FIR object
}
Example of JSON representation of the authentication array containing the face FIR template:
{
	[{
	"Version":1,
	"Header":
	{
		"Factor":2, 											// Facial features
		"Format":
		{
			"FormatOwner":99, 										// Cognitec
			"FormatID":0
		},
		"Type":4,	 										// Face template
		"Purpose":1, 	 										// Verification
		"Quality":-1,
		"Encryption":0 											// Unencrypted
	},
	"Data":"eyJ0eXAiOiJKV1QiLA0KICJhbGciOiJIUzI1NiJ9"
		// Base64url encoded CDPJsonFIR object
	}]
}
DP_BIO_SAMPLE_TYPE::RAW image
Indicates a raw face image. It's recommended to have a minimum of ten BioSample objects containing raw images in the authentication array for successful verification.
The only type of raw image supported by the current version is a jpeg file.
BioSampleType - must be 1.
BioSampleFormat->FormatOwner - not used, must be 0.
Data - Base64url encoded JSON representation of the CDPJsonFaceImage class described below:
[DataContract]
public class CDPJsonFaceImage
{
	[DataMember]
	public Byte Version   { get; set; }  // version
	[DataMember]
	public DP_FACE_IMAGE_TYPE ImageType  { get; set; } // type of image
	[DataMember]
	public String ImageData { get; set; } // face image
}
where:

Data member	Description
Byte Version 	Specifies the version of the CDPJsonFaceImage object. It must be set to 1.   
DP_FACE_IMAGE_TYPE ImageType	type of image. Must be set to 1, JPEG_FILE:
String ImageData	contains Base64Url encoded raw image data, according to the ImageType. In the current version, it's a jpeg file. If BioSampleEncryption is set to 1 (XTEA encryption), this data is encrypted.

typedef enum DP_FACE_IMAGE_TYPE
{
	JPEG_FILE = 1, // Base64Url encoded JPEG file
}DP_FACE_IMAGE_TYPE;
Below is an example of JSON representation of the CDPJsonFaceImage object containing the raw face image (jpeg file):
{
	"Version":1,
	"ImageType":1,	// JPEG_FILE              
	"ImageData":"W3sidmVyc2lvbiI6MSwibnVtYmVyIjoyLCJ0ZXh0IjoiTmV3IFlvcmsifSx7
	InZlcnNpb24iOjEsIm51bWJlciI6NiwidGV4dCI6IkNhbnlvbiBNaWRkbGUifSx7InZlcnNp
	b24iOjEsIm51bWJlciI6MTAyLCJ0ZXh0IjoiMDQvMjQvMjAwOSJ9XQ"
		// Base64url encoded jpeg file
}
Example of JSON representation of the authentication array containing the raw face images (jpeg files):
{
	[{
	"Version":1,
	"Header":
	{
		"Factor":2, 										// Facial features
		"Format":
		{
			"FormatOwner":0, // Not used
			"FormatID":0
		},
		"Type":1,										// Raw image
		"Purpose":1, 										// Verification
		"Quality":-1,
		"Encryption":0 										// Unencrypted
		},
	"Data":"WF0T3duZXIiOjUxLA0KIkZvcm1hdElEIjowDQp9LA0KIlR5cGUiOjIsDQoiUHVycG9z
	…		
	…
	ZSI6MCwNCiJRdWFsaX"											// Base64url encoded CDPJsonFaceImage object
	},
	…
	…
	{
	"Version":1,
	"Header":
	{
		"Factor":2, 										// Facial features
		"Format":
		{
			"FormatOwner":0, // Not used
			"FormatID":0
		},
		"Type":1,										// Raw image
		"Purpose":1, 										// Verification
		"Quality":-1,
		"Encryption":0 										// Unencrypted
		},
	"Data":"SGVhZGVyIjp7IkRldmljZUlkIjowLCJEZXZpY2VUeXBlIjo0OTI2NDQxNzM0NzI3Mjc
  …
  …
	wNCwiaURhdGFBY3F1a" 											// Base64url encoded CDPJsonFaceImage object
	}]
}
The following steps will be needed to create the Face authentication packet.
1	Create JSON representation of BioSample(s).
2	Combine those JSON representations in a JSON array ([]).
3	Base64Url encode the string created in step #2.
4	Create a JSON representation of the Credential class using Face Credential ID as id member and a string created in step #3 as a data member.
IdentifyUser
This method is not supported. The Face Credential does not support user identification.
GetEnrollmentData
This method is not supported.
CustomAction
CustomAction is not supported for the Face Credential.
Contactless Card Credentials
The following ID is defined for Contactless Card Credential:
{F674862D-AC70-48ca-B73E-64A22F3BAC44}
AuthenticateUser
The data for the contactless card credential is a Base64url encoded UTF-8 representation of the JSON CDPJsonCLCAuthToken class:
[DataContract]
public class CDPJsonCLCAuthToken
{
	[DataMember]
	public Byte version { get; set; } // version
	[DataMember]
	public String UID { get; set; }   // card UID
	[DataMember]
	public String OTP { get; set; }   // TOTP
}
where:

Data member	Description
Byte version 	Specifies the version of CDPJsonCLCAuthToken object, must be set to 1.
String UID 	Card's unique ID, array of 64 bytes, presented as Base64url UTF-8 encoded string.
String OTP 	6 symbols time based one-time password, generated by the client, presented as string.

To create the Contactless Card Credential for authentication, following steps must be performed on the client:
1	Read the card UID and the symmetric key stored on the Contactless card, create the SHA 256 hash of this key.
2	Use the key hash as a TOTP seed, generate the OTP.
3	Create a JSON representation of the CDPJsonCLCAuthToken class.
•	Use the OTP string created in step #2;
•	Base64url UTF-8 encode the card UID;
4	Base64Url encode the JSON representation of the CDPJsonCLCAuthToken class;
5	Create a JSON representation of the Credential class using Contactless Card Credential ID as id member and string created in step #4 as a data member.
IdentifyUser
To call IdentifyUser() method, you must first create the Contactless Card credential (CDPJsonCLCAuthToken) as described above. Below is an example of HTTP Body for Contactless Card identification with Contactless card credential created above.
{
	"credential":
	{
		"id":" F674862D-AC70-48ca-B73E-64A22F3BAC44",
		"data":"LAogImFs1QiLAogImFeyJ0eXAiOiJKVBSUzI18"
	}
}
GetEnrollmentData
This method is not supported.
CustomAction
CustomAction is not currently supported by Contactless Card Credential.
Windows Integrated Authentication (WIA) Credential
The following ID is defined for the WIA Credential.
{AE922666-9667-49BC-97DA-1EB0E1EF73D2}
The following functions are not supported by the WIA Credential:
•		GetEnrollmentData
•		IdentifyUser
•		AuthenticateUser
•		AuthenticateUserTicket
•		CustomAction
CreateUserAuthentication
The User parameter of CreateuserAuthentication is ignored, and the credentialId parameter should be set to AE922666-9667-49BC-97DA-1EB0E1EF73D2.
Below is an example of HTTP Body required to create Extended Authentication for WIA:
{
	"user":null,
	"credentialId":"AE922666-9667-49BC-97DA-1EB0E1EF73D2"
}
CreateTicketAuthentication
The ticket parameter of CreateTicketAuthentication must be a valid Ticket, and the credentialId parameter should be set to AE922666-9667-49BC-97DA-1EB0E1EF73D2.
Below is an example of HTTP Body required to create Extended Authentication for WIA:
{
	"ticket":{"jwt":"Z3NhZGhhc2Rma0FTREZLYWZyZGtB"},
	"credentialId":"AE922666-9667-49BC-97DA-1EB0E1EF73D2"
}
ContinueAuthentication
The authId parameter of ContinueAuthentication must be a valid authentication handle returned by CreateUserAuthentication or CreateTicketAuthentication.  The authData parameter is Based64Url encoded data returned by the WIA client.
Below is an example of HTTP Body for WIA authentication handshake:
{
	"authId":657854,
	"authData":"‘eypZ3NhZGhhc2Rma0FTREZLYWZyZGtB"
}
DestroyAuthentication
The authId parameter of DestroyAuthentication must be a valid authentication handle returned by CreateUserAuthentication or CreateTicketAuthentication.
Below is an example of HTTP Body to destroy WIA authentication:
{
	"authId":657854
}
Email Credential
The following ID is defined for the Email Credential:
{7845D71D-AB67-4EA7-913C-F81E75C3A087}
The Email credential is an auxiliary credential and cannot be used alone to log on to STS. It has to be combined with other Primary credential(s).
The email credential is some string (details of the string is TBD) which will be sent to the user over e-mail and user can present the very same string during E-mail authentication.
Upon initiating email verification (see the CustomAction defined on page 136), the user should receive an email similar to the following.
To verify your email address, please click the following link:
https://sts.yourdomain.com/verifymail&user=John.Doe@yourdomain.com&type=6
&data=COqe3faVtaeWdBD4ncmH4r3C9EQ
The URL in the example above has several components.
1	The URL of the service which will process the email verification. In the example it’s https://sts.yourdomain.com.
2	The name of the function that will be used for e-mail verification. In the example it’s verifymail.
3	The user name. In the example it’s user=John.Doe@yourdomain.com.
4	The user name type. In example it’s type=6 (which signifies a UPN name).
5	Email verification data. In the example it’s data=COqe3faVtaeWdBD4ncmH4r3C9EQ. This data (COqe3faVtaeWdBD4ncmH4r3C9EQ) has to be provided in the credential object’s data field when calling the AuthenticateUser request (see below). The data will be valid for 30 minutes afterthe email is sent.
AuthenticateUser
To call the AuthenticateUser() method, the caller must constrict the authentication request using the "data" field of the verification URL received through email as the data field of the credential object.
Below is an example of HTTP Body for email authentication with the Email credential described in the example above.
{
	"user":
	{
		"name":"Kirill.Lo@crossmatch.com",
		"type":6
	},
	"credential":
	{
		"id":"7845D71D-AB67-4EA7-913C-F81E75C3A087",
		"data":"COqe3faVtaeWdBD4ncmH4r3C9EQ"
	}
}
IdentifyUser
Not supported.
GetEnrollmentData
As result of successful GetEnrollmentData request should be a Base64url encoded UTF-8 representation E-mail based information. We introduce MailEnrollmentData class to represent this information:
[DataContract]
public class MailEnrollmentData
{
	[DataMember]
	public Boolean has_mail { get; set; }
	// true if user has mail data enrolled
}

Data member	Description
has_mail	is True if the user has email data enrolled and False if not. Usually a user email address is stored as the "mail" attribute in AD or LDS, but in the future we may use other mechanisms as well to find a user’s email address such as querying a custom SQL database.

Below is an example of JSON representation of GetEnroomentDataResult.
	{
		"has_mail":true
	}
Parsing GetEnrollmentDataResult
Use the following steps to process (parse) GetEnrollmentDataResult.
1	Get string provided in GetEnrollmentDataResult.
2	Base64url decode the string above to get UTF-8 representation of MailEnrollmentData.
3	Decode UTF-8 string to a format compatible with the JSON parser (Unicode?).
4	Using the JSON parser, parse the string to objects of the MailEnrollmentData class.
CustomAction
The following CustomAction operations are currently supported by the Email credential.
Send Email Verification Request;
The Send Email Verification Request operation Action ID is "16".
The caller does not need to provide a valid ticket to perform this operation, so the ticket parameter may be set to "null".
A valid user to whom the email should be sent needs needs to be provided.
The data parameter of the Credential class does not need to be provided and can be set to “null”.
Below is a valid example of HTTP Body of Send E-mail Verification request:
{
	"ticket":{"jwt":null},
	"user":
	{
		"name":"someone@mycompany.com",
		"type":9
	},
	"credential":
	{
		"id":"7845D71D-AB67-4EA7-913C-F81E75C3A087",
		"data":null
	},
	"actionId":16
}
This call will send an email verification request to the user John.Doe@yourdomain.com. An example of the email request was provided above.
NOTE: The user must have valid e-mail address stored in our database (AD or LDS) to be able to send the email. If the email address cannot be found in the user record, the following error will be returned.
E_ADS_PROPERTY_NOT_FOUND (0x8000500D)
The property was not found in the cache. The property may not have an attribute, or is invalid.
U2F Device Credential
The following ID is defined for the U2F Credential.
{5D5F73AF-BCE5-4161-9584-42A61AED0E48}
AuthenticateUser
This method is not supported.
IdentifyUser
This method is not supported.
GetEnrollmentData
This method is not supported.
CustomAction
The following CustomAction operations are currently supported by the FIDO U2F Credential.
Request Application Id from Server
Application Id request operation Action ID is "17".
The caller does not need to provide a valid ticket to perform this operation, so the ticket parameter may be set to "null".
The caller does not need to provide a valid user name or type to perform this operation, so the name parameter may be set to "null" and the type parameter can be set to 0.
The data parameter of Credential class does not need to be provided and can be set to “null”.
Below is a valid example of HTTP Body of Application Id request:
{
	"ticket":{"jwt":null},
	"user":
	{
		"name":null,
		"type":0
	},
	"credential":
	{
		"id":"5D5F73AF-BCE5-4161-9584-42A61AED0E48",
		"data":null
	},
	"actionId":17
}
Below is a valid example of HTTP Body of Application Id response:
{
	"AppId":"https://sts.crossmatch.com/AppId.json"
}
If the AppId is not set on the Server, an appropriate error will be returned.
CreateUserAuthentication
The credentialId parameter should be set to 5D5F73AF-BCE5-4161-9584-42A61AED0E48.
Below is an example of HTTP Body required to create Extended Authentication for FIDO U2F:
{
	"user":{"name":"John.Doe@yourdomain.com","type":6},
	"credentialId":" 5D5F73AF-BCE5-4161-9584-42A61AED0E48"
}
CreateTicketAuthentication
The ticket parameter of CreateTicketAuthentication must be a valid Ticket, and credentialId parameter should be set to 5D5F73AF-BCE5-4161-9584-42A61AED0E48.
Below is an example of HTTP Body required to create Extended Authentication for WIA:
{
	"ticket":{"jwt":"Z3NhZGhhc2Rma0FTREZLYWZyZGtB"},
	"credentialId":"5D5F73AF-BCE5-4161-9584-42A61AED0E48"
}
ContinueAuthentication
The authId parameter of ContinueAuthentication must be a valid authentication handle returned by CreateUserAuthentication or CreateTicketAuthentication.  The authData parameter is Based64Url encoded data of FIDO U2F handshake.
Below is an example of HTTP Body for FIDO U2F authentication handshake:
{
	"authId":657854,
	"authData":"Z3NhZGhhc2Rma0FTREZLYWZyZGtB"
}
Handshake data is defined as:
[DataContract]
public class U2FHandshake
{
	[DataMember]
	public HandshakeType handshakeType { get; set; }
		// Type of U2F specific handshake.
	[DataMember]
	public String handshakeData { get; set; }   
		// Base64Url encoded handshake data.
}

Where

/// <summary>
/// Handshake types supported by U2F.
/// </summary>
[DataContract]
public enum HandshateType
{
	/// <summary>
	/// Client request challenge from U2F server.
	/// </summary>
	[EnumMember]
	ChallengeRequest = 0,

	/// <summary>
	/// Server respond with U2F challenge.
	/// </summary>
	[EnumMember]
	ChallengeResponse = 1,

	/// <summary>
	/// Client request data verification.
	/// </summary>
	[EnumMember]
	AuthenticationRequest = 2,

	/// <summary>
	/// Server respond with result of data verification.
	/// </summary>
	[EnumMember]
	AuthenticationResponse = 3,
}
The value of handshakeData depends on the handshake type. There are four types of handshake.
•	Challenge Request
•	Challenge Respond
•	Authentication Request
•	Authentication Respond
Challenge Request
Client send Challenge Request as first step of U2F authentication handshakes. handshakeData of Challenge Request is ignored and should be set to null.
Below is an example of Json representation of Challenge Request:
{
	"handshakeType":0,
	"handshakeData":null
 }
Challenge Respond
Challenge Respond will be returned by the U2F Server as authData member of ExtendedAUTHResult returned by Challenge Request call.
Below is an example of Challenge Respond:
{
	"handshakeType":1,
	“handshakeData":"Z3NhZGhhc2Rma0FTREZLYWZyZGtB"
 }
handshakeData of Challenge Respond should be provided in the following format:
[DataContract]
public class U2FChallengeRespond
{
	[DataMember]
	public String version { get; set; } 																						// version.
	[DataMember]
	public String appId { get; set; } 			 																			// application ID.
	[DataMember]
	public String challenge { get; set; } 																						// Base64Url encoded challenge.
	[DataMember]
	public String keyHandle { get; set; } 																						// Base64Url encoded key handle.
}

Data member	Description
version	Version of supported FIDO standard, must be set to "U2F_V2" for current implementation.
appId	Application ID.
challenge	Base64Url encoded challenge.
keyHandle	Base64Url encoded key handle.

Authentication Request
The client sends an Authentication Request as the second step of U2F authentication handshaking. handshakeData of the Authentication Request has the following format.
[DataContract]
public class U2FAuthenticationRequest
{
	[DataMember]
	public String version { get; set; } 																							// version.
	[DataMember]
	public String appId { get; set; } 																							// application ID.
	[DataMember]
	public String serialNum { get; set; } 																							// serial number of U2F device.
	[DataMember]
	public String signatureData { get; set; }																							
		//		Base64Url encoded								 	signature of challenge.
	[DataMember]
	public String clientData { get; set; }																							
		//		U2F token authentication													counter.
}

Data member	Description
version	Version of supported FIDO standard, must be set to "U2F_V2" for current implementation.
appId	Application ID.
serialNum	U2F device serial number.
signatureData	Base64Url encoded signature of client data.
clientData	Base64Url encoded client data.

See the following webpage for details:
https://fidoalliance.org/specs/fido-u2f-v1.0-nfc-bt-amendment-20150514/fido-u2f-raw-message-formats.html
Client data contains challenge which was generated previously.
Authentication Respond
Authentication Respond is not supported at this time and reserved for future use. The result of Authentication Request will be returned in ExtendedAUTHResult. Json Web Token will be returned if authentication succeeded or error if authentication failed (see topic 4.9 of Web Authentication Service design document for details).
DestroyAuthentication
The authId parameter of DestroyAuthentication must be a valid authentication handle returned by CreateUserAuthentication or CreateTicketAuthentication.
Below is an example of HTTP Body to destroy FIDO U2F authentication:
{
	"authId":657854
}
