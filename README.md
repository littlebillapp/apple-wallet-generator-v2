# edutap.wallet_apple

this package provides

- [x] creation of apple pass files (.pkpass)
- [x] signing pass files
- [ ] provide pass delivery 
- [ ] update passes using apple push notifications


## Installation

Prerequisites:

- python >= 3.10
- SWIG (needed by the M2crypto python lib)

    ```bash
    sudo apt-get install swig
    ```

Normal installation for development via pip, it is recommended to use a virtual env.



```bash
pip install -e .[test]
```

### OSX

If you have problems installing M2Crypto on an Apple Silicon, you need to use `LDFALGS` and `CFLAGS`:

```console
LDFLAGS="-L$(brew --prefix openssl)/lib" CFLAGS="-I$(brew --prefix openssl)/include" SWIG_FEATURES="-I$(brew --prefix openssl)/include" pip install m2crypto
```


## Run the unittests

The unit tests can be run without the cert files:

```shell
pytest -m "not integration"
```

## Installation Cert stuff

PKPASS is a file format, used for storage and exchange of digital passes, developed by Apple for its Wallet application (Formerly known as PassBook until iOS 9)

For signing the .pkpass files we need certificate and key files that need to be created. Please follow exactly the steps described below. You need an Apple developer account to obtain the certificate for the pass identifier.
<!-- To run integration tests and the passbook server you need a certificate and a private key. The certificate is used to sign the passbook files and the private key is used to sign the push notifications. The certificate and the private key are stored in the config file of the passbook server. -->

this is the overall process to get the necessary certificates for issuing passes

```mermaid
flowchart TD
    B[create private key.pem]
    D[get/create Pass ID - apple.com]
    WWDR[download AppleWWDRCA.cer] -->WWDRPEM[convert to wwdr_certificate.pem]
    D --> E[request Certificate.cer based on Pass Id - apple.com]
    B[create key.pem] --> CSR[create CSR]
    CSR -->|upload CSR in form| F[create+download Certificate.cer - apple.com]
    E --> F
    F -->|x509| G[create Certificate.pem]
    G --> H[install Certificate.pem, private.key and wwdr_certificate.pem on server]
    WWDRPEM --> H
```

### prepare key and CSR for requesting a certificate from apple

- create your own private key
```shell
$ openssl genrsa -out private.key 2048
```

- create a certificate signing request (CSR) with the private key

    (this is only necessary when you create a new certificate, if you already have certificates in your account you can download them)
```shell
$ openssl req -new -key private.key -out request.csr -subj="/emailAddress=[your email addr],CN=[your full name],C=[your country ISO code]"
```

Name and email do not necessarily have to match with the account data of your apple developer account.

### Get a Pass Type Id and certificate from Apple

you need a developer account at apple to get a pass type id and a certificate for signing your passes. you can get a free developer account at [developer.apple.com](https://developer.apple.com/programs/)

* Visit the iOS Provisioning [Portal -> Pass Type IDs -> New Pass Type ID](https://developer.apple.com/account/resources/identifiers/list/passTypeId)
    - either create a new pass type id by clicking the blue (+) icon on top of the menu
    - or select one of the existing pass type id's
* In the screen labelled `Edit your Identifier Configuration` you can do one of
    - select an existing certificate and hit the `Download` button
    - or hit `Create Certificate` on the bottom of the page (there you need the above mentioned `pass.cer`)
* Use Keychain tool to export a Certificates.cer  file (need Apple Root Certificate installed)
* Convert the certificate.cer (X509 format) to a certificate.pem file by calling

```shell
    $ openssl x509 -inform der -in pass.cer -out certificate.pem
```

### Apple Worldwide Developer Relations (WWDR) root certificate


see [https://developer.apple.com/support/certificates/expiration/](apple support)

```shell
curl https://www.apple.com/certificateauthority/AppleWWDRCAG4.cer -o AppleWWDRCA.cer
```

an overview of downloadable apple certs:

https://www.apple.com/certificateauthority/

convert it to a pem file

```shell
openssl x509 -inform der -in AppleWWDRCA.cer -out wwdr_certificate.pem
```
then copy it into the 'certs' folder of the passbook server


see [documentation @ apple](https://developer.apple.com/documentation/walletpasses/building_a_pass)

check expiration date of certificate

```shell
openssl x509 -enddate -noout -in file.pem
```

## run the integration tests

> ⚠️ **Attention:**
>for running the integration tests, the above mentioned files (`certificate.pem`, `private.key`, `wwdr_certificate.pem`) have to be located in `tests/data/certs/private`

```shell
pytest -m integration
```

the test "test_passbook_creation_integration" will create a passbook file and display it with the passbook viewer. Displaying the pass works just under OSX since the passbook viewer is part of OSX.


# Notification

TODO

## Create a certificate for push notifications

TODO

## Further readings

- [apple doc for updating passes](https://developer.apple.com/documentation/walletpasses/adding_a_web_service_to_update_passes)

- [passninja docs](https://www.passninja.com/tutorials/apple-platform/how-does-pass-updating-work-on-apple-wallet)

## Credits

This project is inspired by the work of https://github.com/devartis/passbook


