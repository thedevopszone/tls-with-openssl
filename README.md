# tls-with-openssl

- https://jamielinux.com/docs/openssl-certificate-authority/sign-server-and-client-certificates.html

- https://www3.rocketsoftware.com/rocketd3/support/documentation/Uniface/10/uniface/security/certificates/createIntermediateCertificate.htm?TocPath=Protecting%20Your%20Application%7CTransport%20Protocols%20and%20Certificates%7CDigital%20Certificates%7CCreating%20Certificates%7CActing%20as%20a%20Certificate%20Authority%7C_____2


# Root CA
1. Create private key (ca.key)
2. Create public certificate file (ca.crt)

# Intermediate CA
1. Create private key (mid-ca.key)
2. Create certificate signing request (mid-ca.csr)
3. Create public certificate file (mid-ca.crt)

# Servers
1. Create private key (server.key)
2. Create certificate signing request (xxx.csr)
3. Create public certificate (xxx.crt)
4. Create PFX file for Windows OS (xxx.pfx)



mkdir practice
cd practice

mkdir -p {ca,mid-ca,server}/{private,certs,newcerts,crl,csr}
# Generate private key to create the root certificate (type passphrase)
chmod -v 700 {ca,mid-ca,server}/private

touch {ca,mid-ca}/index

openssl rand -hex 16 > ca/serial
openssl rand -hex 16 > mid-ca/serial


Are the same as: /usr/lib/ssl/openssl.conf

vi ca/ca.conf
```
#
# openSSL example configuration file.
# This is mostly being used for generation of certificate requests.
#

####################################################################
[ ca ]
default_ca	= CA_default		# The default ca section

####################################################################
[ CA_default ]

dir		= ca			# Where everything is kept
certs		= $dir/certs		# Where the issued certs are kept
crl_dir		= $dir/crl		# Where the issued crl are kept
database	= $dir/index		# database index file.
certificate	= $dir/certs/ca.crt
serial		= $dir/serial
private_key	= $dir/private/ca.key
#
#unique_subject	= no			# Set to 'no' to allow creation of
					# several certs with same subject.
new_certs_dir	= $dir/newcerts		# default place for new certs.

#certificate	= $dir/cacert.pem 	# The CA certificate
#database	= $dir/CA/index.txt
#serial		= $dir/serial 		# The current serial number
crlnumber	= $dir/crlnumber	# the current crl number
					# must be commented out to leave a V1 CRL
crl		= $dir/crl/ca.crl	# The current CRL

x509_extensions	= usr_cert		# The extensions to add to the cert

# Comment out the following two lines for the "traditional"
# (and highly broken) format.
name_opt 	= ca_default		# Subject Name options
cert_opt 	= ca_default		# Certificate field options

# Extension copying option: use with caution.
# copy_extensions = copy

# Extensions to add to a CRL. Note: Netscape communicator chokes on V2 CRLs
# so this is commented out by default to leave a V1 CRL.
# crlnumber must also be commented out to leave a V1 CRL.
crl_extensions	= crl_ext

default_days	= 365			# how long to certify for
default_crl_days= 30			# how long before next CRL
default_md	= sha256		# use public key default MD
preserve	= no			# keep passed DN ordering

# A few difference way of specifying how similar the request should look
# For type CA, the listed attributes must be the same, and the optional
# and supplied fields are just that :-)
policy		= policy_anything


####################################################################
# For the CA policy
[ policy_match ]
countryName		= match
stateOrProvinceName	= match
organizationName	= match
organizationalUnitName	= optional
commonName		= supplied
emailAddress		= optional


####################################################################
# For the 'anything' policy
# At this point in time, you must list all acceptable 'object'
# types.
[ policy_anything ]
countryName		= optional
stateOrProvinceName	= optional
localityName		= optional
organizationName	= optional
organizationalUnitName	= optional
commonName		= supplied
emailAddress		= optional


####################################################################
[ req ]
default_bits		= 2048
default_keyfile 	= privkey.pem
distinguished_name	= req_distinguished_name
#attributes		= req_attributes
x509_extensions		= v3_ca	# The extensions to add to the self signed cert

# Passwords for private keys if not present they will be prompted for
# input_password = secret
# output_password = secret

# This sets a mask for permitted string types. There are several options.
# default: PrintableString, T61String, BMPString.
# pkix	 : PrintableString, BMPString (PKIX recommendation before 2004)
# utf8only: only UTF8Strings (PKIX recommendation after 2004).
# nombstr : PrintableString, T61String (no BMPStrings or UTF8Strings).
# MASK:XXXX a literal mask value.
# WARNING: ancient versions of Netscape crash on BMPStrings or UTF8Strings.
string_mask = utf8only


####################################################################
# req_extensions = v3_req # The extensions to add to a certificate request
[ req_distinguished_name ]
countryName			= Country Name (2 letter code)
countryName_min			= 2
countryName_max			= 2
stateOrProvinceName		= State or Province Name (full name)
localityName			= Locality Name (eg, city)
0.organizationName		= Organization Name (eg, company)
organizationalUnitName		= Organizational Unit Name (eg, section)
commonName			= Common Name (e.g. server FQDN or YOUR name)
commonName_max			= 64
emailAddress			= Email Address
emailAddress_max		= 64


####################################################################
[ req_attributes ]
challengePassword		= A challenge password
challengePassword_min		= 4
challengePassword_max		= 20
unstructuredName		= An optional company name


####################################################################
[ server_cert ]
# Extensions for server certificates
basicConstraints                =   CA:FALSE
nsCertType                      =   server
nsComment                       =   "Self-Signed Certificate generated by OpenSSL (ca.conf)"
subjectKeyIdentifier            =   hash
authorityKeyIdentifier          =   keyid,issuer:always
keyUsage                        =   critical, digitalSignature, keyEncipherment
extendedKeyUsage                =   serverAuth


####################################################################
[ v3_ca ]
# Extensions for a typical CA
# PKIX recommendation.
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid:always,issuer
basicConstraints = critical,CA:true
# Key usage: this is typical for a CA certificate. However since it will
# prevent it being used as an test self-signed certificate it is best
# left out by default.
# keyUsage = cRLSign, keyCertSign


####################################################################
[ v3_mid_ca ]
# Extensions for a typical CA
# PKIX recommendation.
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid:always,issuer
basicConstraints = critical,CA:true,pathlen:0
# Key usage: this is typical for a CA certificate. However since it will
# prevent it being used as an test self-signed certificate it is best
# left out by default.
# keyUsage = cRLSign, keyCertSign



```



vi mid-ca/mid-ca.conf
```
#
# OpenSSL example configuration file.
# This is mostly being used for generation of certificate requests.
#

####################################################################
[ ca ]
default_ca	= CA_default		# The default ca section

####################################################################
[ CA_default ]

dir		= mid-ca		# Where everything is kept
certs		= $dir/certs		# Where the issued certs are kept
crl_dir		= $dir/crl		# Where the issued crl are kept
database	= $dir/index		# database index file.
serial		= $dir/serial

certificate	= $dir/certs/mid-ca.crt
private_key	= $dir/private/mid-ca.key

#unique_subject	= no			# Set to 'no' to allow creation of
					# several certs with same subject.
new_certs_dir	= $dir/newcerts		# default place for new certs.

#certificate	= $dir/cacert.pem 	# The CA certificate
#database	= $dir/CA/index.txt
#serial		= $dir/serial 		# The current serial number
crlnumber	= $dir/crlnumber	# the current crl number
					# must be commented out to leave a V1 CRL
crl		= $dir/crl/mid-ca.crl	# The current CRL

x509_extensions	= usr_cert		# The extensions to add to the cert

# Comment out the following two lines for the "traditional"
# (and highly broken) format.
name_opt 	= ca_default		# Subject Name options
cert_opt 	= ca_default		# Certificate field options

# Extension copying option: use with caution.
# copy_extensions = copy

# Extensions to add to a CRL. Note: Netscape communicator chokes on V2 CRLs
# so this is commented out by default to leave a V1 CRL.
# crlnumber must also be commented out to leave a V1 CRL.
crl_extensions	= crl_ext

default_days	= 365			# how long to certify for
default_crl_days= 30			# how long before next CRL
default_md	= sha256		# use public key default MD
preserve	= no			# keep passed DN ordering

# A few difference way of specifying how similar the request should look
# For type CA, the listed attributes must be the same, and the optional
# and supplied fields are just that :-)
policy		= policy_anything


####################################################################
# For the CA policy
[ policy_match ]
countryName		= match
stateOrProvinceName	= match
organizationName	= match
organizationalUnitName	= optional
commonName		= supplied
emailAddress		= optional


####################################################################
# For the 'anything' policy
# At this point in time, you must list all acceptable 'object'
# types.
[ policy_anything ]
countryName		= optional
stateOrProvinceName	= optional
localityName		= optional
organizationName	= optional
organizationalUnitName	= optional
commonName		= supplied
emailAddress		= optional


###################################################################
[ req ]
default_bits		= 2048
default_keyfile 	= privkey.pem
distinguished_name	= req_distinguished_name
# attributes		= req_attributes 	# Adding an additional password which is not required
x509_extensions		= v3_ca			# The extensions to add to the self signed cert

# Passwords for private keys if not present they will be prompted for
# input_password = secret
# output_password = secret

# This sets a mask for permitted string types. There are several options.
# default: PrintableString, T61String, BMPString.
# pkix	 : PrintableString, BMPString (PKIX recommendation before 2004)
# utf8only: only UTF8Strings (PKIX recommendation after 2004).
# nombstr : PrintableString, T61String (no BMPStrings or UTF8Strings).
# MASK:XXXX a literal mask value.
# WARNING: ancient versions of Netscape crash on BMPStrings or UTF8Strings.
string_mask = utf8only


###################################################################
# req_extensions = v3_req 	# The extensions to add to a certificate request
[ req_distinguished_name ]
countryName			= Country Name (2 letter code)
countryName_default		= AU
countryName_min			= 2
countryName_max			= 2
stateOrProvinceName		= State or Province Name (full name)
stateOrProvinceName_default	= NSW
localityName			= Locality Name (eg, city)
localityName_default		= Sydney
0.organizationName		= Organization Name (eg, company)
0.organizationName_default	= Practice Pty
organizationalUnitName		= Organizational Unit Name (eg, section)
organizationalUnitName_default	= Customer Support
commonName			= Common Name (e.g. server FQDN or YOUR name)
commonName_max			= 64
emailAddress			= Email Address
emailAddress_max		= 64


####################################################################
[ req_attributes ]
challengePassword		= A challenge password
challengePassword_min		= 4
challengePassword_max		= 20
unstructuredName		= An optional company name


####################################################################
[ v3_ca ]
# Extensions for a typical CA
# PKIX recommendation.
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid:always,issuer
basicConstraints = critical,CA:true,pathlen:0
# Key usage: this is typical for a CA certificate. However since it will
# prevent it being used as an test self-signed certificate it is best
# left out by default.
# keyUsage = cRLSign, keyCertSign



####################################################################
####################################################################
####################################################################
[ server_cert ]
# Extensions for server certificates
basicConstraints                =   CA:FALSE
nsCertType                      =   server
nsComment                       =   "Self-Signed Certificate generated by OpenSSL (mid-ca.conf)"
subjectKeyIdentifier            =   hash
authorityKeyIdentifier          =   keyid,issuer:always
keyUsage                        =   critical, digitalSignature, keyEncipherment
extendedKeyUsage                =   serverAuth, clientAuth



####################################################################
[ server_cert_wildcard ]
# Extensions for server certificates
basicConstraints                =   CA:FALSE
nsCertType                      =   server
nsComment                       =   "Self-Signed Certificate generated by OpenSSL (mid-ca.conf)"
subjectKeyIdentifier            =   hash
authorityKeyIdentifier          =   keyid,issuer:always
keyUsage                        =   critical, digitalSignature, keyEncipherment
extendedKeyUsage                =   serverAuth, clientAuth
subjectAltName                  =   DNS:*.veg.cloud



####################################################################
[ server_cert_multi ]
# Extensions for server certificates
basicConstraints                =   CA:FALSE
nsCertType                      =   server
nsComment                       =   "Self-Signed Certificate generated by OpenSSL (mid-ca.conf)"
subjectKeyIdentifier            =   hash
authorityKeyIdentifier          =   keyid,issuer:always
keyUsage                        =   critical, digitalSignature, keyEncipherment
extendedKeyUsage                =   serverAuth, clientAuth
subjectAltName			=   @alt_names

[ alt_names ]
DNS.1 = DEV-WEB-EN001.veg.cloud
DNS.2 = DEV-WEB-EN002.veg.cloud
DNS.3 = DEV-WEB-EN003.veg.cloud
DNS.4 = DEV-WEB-EN004.veg.cloud
DNS.5 = DEV-WEB-EN005.veg.cloud
```


cat /usr/lib/ssl/openssl.cnf


# Now create the root certificate step: 1

```
# Create the private key
openssl genrsa -aes256 -out ca/private/ca.key 4096

chmod 400 ca/private/ca.key

# Create public certificate for CA
openssl req -config ca/ca.conf -key ca/private/ca.key -new -x509 -days 3650 -sha256 -extensions v3_ca -out ca/certs/ca.crt
openssl req -config ca/ca.conf -key ca/private/ca.key -new -x509 -days 3650 -extensions v3_ca -out ca/certs/ca.crt
Common Name: Practice - Root CA

chmod 444 ca/certs/ca.crt


```



Create certificate for intermediate CA (Step. 2)
```
# Create private key for intermediate certificate
openssl genrsa -aes256 -out mid-ca/private/mid-ca.key 4096


chmod 400  mid-ca/private/mid-ca.key


# Create a Certificate Signing Request (CSR) for “Intermediate CA
openssl req -config ca/ca.conf -new -key mid-ca/private/mid-ca.key -sha256 -out mid-ca/csr/mid-ca.csr
Common Name: Practice - Intermediate CA
chmod 400 mid-ca/csr/mid-ca.csr


# Create a certificate file for “Intermediate CA" Public Certificate
openssl -config ca/ca.conf -extensions v3_mid_ca -days 3650 -notext -in mid-ca/csr/mid-ca.csr -out mid-ca/certs/mid-ca.crt
Password from root certificate



Under ca/newcerts/ we have a backup of the intermedia certificate
In the index file of the ca directory there is some informations we can use later
```
Now also the intermediate certificate is done





# Create a Single Domain Certificate (Step. 3) for the servers

```
# Create a key file for “Single Domain Certificate”
openssl genrsa -out server/private/DEV-WEB-EN001.key 2048


# Create a Certificate Signing Request (CSR) for “Sigle Domain Certificate”
openssl req -config mid-ca/mid-ca.conf -key server/private/DEV-WEB-EN001.key -new -sha256 -out server/csr/DEV-WEB-EN001.csr


# Create a certificate file for “Sigle Domain Certificate”
openssl ca -config mid-ca/mid-ca.conf -extensions server_cert -days 3650 -notext -in server/csr/DEV-WEB-EN001.csr -out server/certs/DEV-WEB-EN001.crt


# Create a PFX Certificate file for “Sigle Domain Certificate
openssl pkcs12 -inkey server/private/DEV-WEB-EN001.key -in server/certs/DEV-WEB-EN001.crt -export -out server/certs/DEV-WEB-EN001.pfx

# How to check the contents of a certificate file (.crt)
openssl x509 -noout -text -in server/certs/DEV-WEB-EN001.crt



```



# Create a Wildcard Certificate

```
# Create a key file for “Wildcard Certificate”
openssl genrsa -out server/private/Wildcard.key 2048


# Create a Certificate Signing Request (CSR) for “Wildcard Certificate”
openssl req -config mid-ca/mid-ca.conf -key server/private/Wildcard.key -new -sha256 -out server/csr/Wildcard.csr
Common name: Wildcard Certificate - veg.cloud

# Create a certificate file for “Wildcard Certificate”
openssl ca -config mid-ca/mid-ca.conf -extensions server_cert_wildcard -days 3650 -notext -in server/csr/Wildcard.csr -out server/certs/Wildcard.crt


# Create a PFX Certificate file for “Wildcard Certificate”
openssl pkcs12 -inkey server/private/Wildcard.key -in server/certs/Wildcard.crt -export -out server/certs/Wildcard.pfx


# How to check the contents of a certificate file (.crt)
openssl x509 -noout -text -in server/certs/Wildcard.crt


```



# Create a Multi Domain Certificate

```
# Create a key file for “Multi Domain Certificate”
openssl genrsa -out server/private/Multi-Domain.key 2048


# Create a Certificate Signing Request (CSR) for “Multi Domain Certificate”
openssl req -config mid-ca/mid-ca.conf -key server/private/Multi-Domain.key -new -sha256 -out server/csr/Multi-Domain.csr


# Create a certificate file for “Multi Domain Certificate”
openssl ca -config mid-ca/mid-ca.conf -extensions server_cert_multi -days 3650 -notext -in server/csr/Multi-Domain.csr -out server/certs/Multi-Domain.crt


# Create a PFX Certificate file for “Multi Domain Certificate”
openssl pkcs12 -inkey server/private/Multi-Domain.key -in server/certs/Multi-Domain.crt -export -out server/certs/Multi-Domain.pfx


# How to check the contents of a certificate file (.crt)
openssl x509 -noout -text -in server/certs/Multi-Domain.crt


```



# Create a CA-bundle

```
#  Merge 2 certificates (mid-ca.crt and ca.crt) using “cat” command
# The order is “Intermidiate certificate” and “CA certficate”. 
cat mid-ca/certs/mid-ca.crt ca/certs/ca.crt > mid-ca/certs/ca-bundle.crt


# Make the certificate “Read Only” for everyone
chmod 444 mid-ca/certs/ca-bundle.crt


# Verify the certificate
openssl verify -CAfile ca/certs/ca.crt mid-ca/certs/ca-bundle.crt






```











####

openssl req -config intermediate/openssl.cnf \
      -key intermediate/private/www.home54.de.key.pem \
      -new -sha256 -out intermediate/csr/www.home54.de.csr.pem


openssl ca -config intermediate/openssl.cnf \
      -extensions server_cert -days 375 -notext -md sha256 \
      -in intermediate/csr/www.home54.de.csr.pem \
      -out intermediate/certs/www.home54.de.cert.pem
