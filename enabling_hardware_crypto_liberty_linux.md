# Enabling Hardware Cryptography for Liberty for Linux with Semeru Java 17

## Summary

This technote provides configuration steps for integrating Liberty for Linux with hardware cryptographic devices. These steps were successfully tested by a customer using a Thales Luna HSM with Semeru Java 17 and have been shared with their permission. The steps should be applicable for other Java versions as well, with minor adjustments if needed. The configuration should also be adaptable to other hardware security modules that support the PKCS#11 interface.

## Objective

This document provides the steps necessary to enable hardware cryptography with Liberty for Linux. The configuration enables Liberty to leverage hardware security modules for enhanced security in cryptographic operations.

## Environment

The customer's successful implementation used:
- Liberty for Linux with IBM Semeru Runtime Certified Edition for Linux Version 17 (Java 17)
- Thales Luna HSM

These steps should be applicable for:
- Liberty for Linux with other Java versions (Java 11, Java 21, etc.)
- Other HSMs with PKCS#11 interface support

## Steps

### 1. Install HSM Client Libraries

**General Step**: Install the client libraries that enable communication between your application and the HSM.

**Implementation**: The Luna client libraries were installed on the Linux server using installation packages provided by Thales.

### 2. Configure Authentication Between Client and HSM

**General Step**: Configure authentication between the Linux server and the HSM to establish secure communication.

**Implementation**: Authentication was configured using commands provided with the Luna client installation. A self-signed client certificate was created and registered with the Luna HSM, establishing mutual TLS as the authentication method.

### 3. Create a PKCS#11 Configuration File

**General Step**: Create a configuration file that defines how Java will interact with the HSM through the PKCS#11 interface.

**Implementation**: The following PKCS11Config.cfg file was created with assistance from Luna Support and saved in the Liberty server configuration directory:

```
description = Thales Luna HSM/PKCS11 Configuration
name = PKCS11Config
library = /opt/safenet/lunaclient/lib/libCryptoki2_64.so
slot = 0
showInfo = true
attributes(*,CKO_SECRET_KEY,*) = {
 CKA_CLASS=4
 CKA_PRIVATE= true
 CKA_KEY_TYPE = 21
 CKA_SENSITIVE= true
 CKA_ENCRYPT= true
 CKA_DECRYPT= true
 CKA_WRAP= true
 CKA_UNWRAP= true
}
attributes(*,CKO_PRIVATE_KEY,*) = {
 CKA_CLASS=3
 CKA_LABEL=true
 CKA_PRIVATE = true
 CKA_DECRYPT=true
 CKA_SIGN=true
 CKA_UNWRAP=true
}
attributes(*,CKO_PUBLIC_KEY,*) = {
 CKA_CLASS=2
 CKA_LABEL=true
 CKA_ENCRYPT = true
 CKA_VERIFY=true
 CKA_WRAP=true
}
```

### 4. Update Java Security Configuration

**General Step**: Update the Java security configuration to include the PKCS#11 provider.

**Implementation**: A java.security file was created in the same directory as the server.xml with the following content to add the SunPKCS11 provider pointing to the PKCS11Config.cfg file:

```
security.provider.1=SunPKCS11 ${server.config.dir}/PKCS11Config.cfg
security.provider.2=SUN
security.provider.3=SunRsaSign
security.provider.4=SunEC
security.provider.5=SunJSSE
security.provider.6=SunJCE
security.provider.7=SunJGSS
security.provider.8=SunSASL
security.provider.9=XMLDSig
security.provider.10=SunPCSC
security.provider.11=JdkLDAP
security.provider.12=JdkSASL
```

Additionally, a jvm.options file was created in the same directory with the following content to point to the custom java.security file:

```
-Djava.security.properties=${server.config.dir}/java.security
```

### 5. Create Required Certificates

**General Step**: Create or import the necessary certificates for your application's security requirements.

**Implementation**: Required application certificates (TLS server certificate and client certificates for various use cases) were created using Java keytool.

### 6. Create a Traditional Truststore

**General Step**: Create a standard file-based truststore to hold trusted CA certificates.

**Implementation**: A traditional PKCS12 file-based keystore was created to serve as the general "truststore" in Liberty, which included CA certificates to be trusted.

### 7. Configure Liberty Server.xml

**General Step**: Configure the Liberty server.xml file to use both the HSM-backed keystore and the file-based truststore.

**Implementation**: The Liberty server.xml was configured with two keystore elements:

```xml
<!-- Standard file-based truststore -->
<keyStore
    id="defaultTrustStore"
    location="${server.config.dir}/truststore.p12"
    password="truststore_password"
    type="PKCS12"
/>

<!-- Luna HSM-backed PKCS11 keystore -->
<keyStore
    id="defaultKeyStore"
    fileBased="false"
    readOnly="true"
    location="${server.config.dir}/PKCS11Config.cfg"
    password="crypto_user_pwd_for_luna_hsm_partition"
    type="PKCS11"
/>

<!-- Enable SSL feature -->
<feature>transportSecurity-1.0</feature>
<!-- or -->
<feature>ssl-1.0</feature>
```

### 8. Configure Server Environment

**General Step**: Set up the necessary environment variables for the Liberty server.

**Implementation**: The following properties were set in the Liberty server.env file:

```
JAVA_HOME=/etc/alternatives/jre_17
JVM_ARGS="--add-exports=jdk.crypto.cryptoki/sun.security.pkcs11=ALL-UNNAMED"
```

The `--add-exports` JVM argument is necessary because in Java 17, the Java module system restricts access to internal packages. The PKCS#11 implementation requires access to classes in the `sun.security.pkcs11` package, which is part of the `jdk.crypto.cryptoki` module. This argument explicitly allows Liberty to access these internal classes that are needed for the HSM integration to work properly.

Note: Similar JVM arguments may be required for other Java versions that use the module system (Java 9 and later). The specific arguments might vary slightly depending on the Java version.

### 9. Configure JVM Options (if needed)

**General Step**: Configure any JVM options required for your specific HSM compatibility.

**Implementation**: The following settings were added to Liberty jvm.options to disable unsupported algorithms:

```
-Djdk.tls.disabledAlgorithms=RSASSA-PSS,RSAPSS
```

### 10. Start Liberty Server and Verify Configuration

**General Step**: Start your Liberty server and verify that the HSM integration is working correctly.

**Implementation**: After completing the above steps, the Liberty integration worked successfully with the Luna HSM.

## Sample Use Cases

The HSM integration with Liberty enables enhanced security for various cryptographic operations. Here are some sample use cases:

- **Secure Web Services with HSM-Protected Keys**: Liberty serves as a secure web service endpoint using TLS with the server's private key stored in the HSM. The TLS handshake uses the HSM-protected key, ensuring the private key never leaves the secure hardware boundary, even if the Liberty server is compromised.

- **Mutual TLS Authentication for Microservices**: Liberty stores client certificates in the HSM for outbound connections requiring mutual TLS. The HSM handles the cryptographic operations for client authentication, protecting the private key material.

- **Digital Signature Services**: Applications can use the HSM to sign data (like JSON Web Tokens) with keys that are generated and stored within the HSM. The signing operation occurs inside the hardware, maintaining the integrity of the signing process.

- **Secure Data Encryption and Decryption**: Encryption keys are securely stored in the HSM, and encryption/decryption operations are performed by the hardware, providing an additional layer of security for sensitive data.

- **Certificate Authority Operations**: Organizations running their own CA can store root and intermediate certificate private keys in the HSM, with certificate signing operations performed within the hardware to ensure the highest level of protection for these critical security assets.

## Troubleshooting

If you encounter issues with your HSM integration, consider enabling PKCS#11 debug logging by adding to jvm.options:

```
-Djava.security.debug=sunpkcs11,pkcs11
```

For specific HSM-related issues, consult your HSM vendor's documentation and support resources.

## Java Version Considerations

While these steps were written and tested with Semeru Java 17, they should be applicable to other Java versions with potential minor adjustments:

- The module-related JVM arguments (`--add-exports`) may vary depending on the Java version
- The exact path to the Java installation will differ
- Security provider configurations may have slight differences between Java versions

## Acknowledgements

Special thanks to our customer [Peter Vannman](link) for providing these detailed configuration steps and use cases for Liberty integration with Thales Luna HSM, and for allowing us to share this information with the broader Liberty community.