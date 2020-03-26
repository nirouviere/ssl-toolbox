
# Abstract

This repository contains the common openssl commands I often use and always forget, especially when managing a CA.

## How to use it

Simply clone the repository and start using it.
The various subdirectories needed for the certificate and private keys (private, certs, etc.) all contains a `.gitignore` that will prevent their content from being tracked by git. It basically contains the following snippet

```
# Ignore everything in this directory
*

# Except this file
!.gitignore
```

# OpenSSL general commands

## Generating random numbers

Raw generation

```bash
$> openssl rand 32  # 32: numbers of bytes. Replace by any desired value
```

Hex encoding

```bash
$> openssl rand -hex 32
```

Base64 encoding

```bash
$> openssl rand -base64 32
```

## Encrypting data

## Cryptographic keys

### RSA keys

#### Generate a 4096 bits private RSA file

``` bash
$> openssl genrsa \
    -aes256 \               # Encrypt the private key with aes 256 cbc. Password will be prompted by OpenSSL
    -out private_key.pem \  # Output key to `private_key.pem`
    4096                    # The length of the RSA key
```

### EC keys

#### Generating a new prim256v1 key w/o parameters and print to file

Generate with the parameters:

```bash
$> openssl ecparam     \
      -name prime256v1 \    # Name of the algorithm to use
      -genkey          \    # Generate a new key
      -noout           \    # Do not print the EC parameter
      -out priv.pem         # Name of the file where the keys will be stored
```

#### Protecting the EC key with a password

The `ecparam` command does not have a built-in tool to password-protect the private key.

We'll need to use the `ec` command:

```bash
$> openssl ec                          \
      -in <private.key.pem>            \     # The unencrypted private key
      -out <private_encrypted.key.pem> \     # Encrypted private key
      -aes256                                # Encryption algorithm
```

#### Protecting the key without writing to the disk

```bash
$> openssl ecparam \
    -genkey \
    -name secp256k1 \
  | \
    openssl ec \
      -aes256 \
      -out privatekey.pem
```

## Interacting with server

### Connecting to client

### Display full certificate server chain

# Root CA

## Structure

## Commands

### Generating root CA certificate

```bash
$> openssl req \
        -config openssl.cnf \       # Root CA configuration file
        -key private.pem \          # Private key of the Root CA
        -new \                      # Generate a new request
        -x509 \                     # Instead of a req, output a x509 certificate
        -out ca.cert.pem            # File to which the certificate will be generated
```

# Intermediate CA

## Structure

## Commands

### Creating the intermediate CA csr

You will of course need a public/private key pair for this

```bash
$> openssl req
        -config <intermediate.cnf>   \      # Intermediate CA configuration file
        -new                         \      # Generate a new CSR
        -key <intermediate.key.pem>  \      # Intermediate CA private key
        -out <intermediate.csr.pem>         # File to output the CSR to
```

### Signing the intermediate CA certificate

# End-user certificates

## Create and sign certificate

The commands are basically going to be the same for all the end-user certificate. Just the configuration file will change and this will be put in another section

### Creating the private key

See above for the commands

### Creating a certificate CSR

```bash
$> openssl req \
      -config <end-user.cnf> \     # The config file for the CSR
      -key <private_key.pem> \     # The private key previously generated
      -new                   \     # Generate a new file
      -out <csr.pem>         \     # output PEM file
```

### Sign the certificate

```bash
$> openssl ca \
      -config <ca-config.conf>         \    # The CA config file
      -extensions <end-user-type.cnf>  \    # The extensions to be added to the cert.
                                            #  depends on the cert type
      -in <path_to_end_user_csr.pem>   \    # The cleint CSR
      -out <path_to_cert.pem>               # The signed certificate
```

# CRL

## Revoking a certificate

You only need the certificate you want to revoke. There are two ways to achieve this

  - You kept the signed certificate
  - OpenSSL keep signed certificates in `newcerts`. They are named by their indexes in the database. Just read the `index.txt` content to see the serial number and the DN of the certificate you want to revoke

```bash
$> openssl ca -revoke \
    cert.pem                  # Certificate you want to revoke
```


# References

Generating
