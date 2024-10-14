+++
date = 2020-10-13T19:00:00Z
lastmod = 2020-10-13T19:00:00Z
title = "TPM2.0"
subtitle = ""
+++


TPM2.0 Book
https://web.archive.org/web/20210629155650/https://link.springer.com/content/pdf/10.1007%2F978-1-4302-6584-9.pdf

Things to be worried about when using Microsoft's Reference simulator:
- It's not production ready
- Side channel attacks
- Random numbers
- Key derivation

- David Wooten
  - TPM Code for commands
  - Interface to crypto library
  - module rpovded by platform manufacturer
    - Time of day
    - Random number generator
    - Anything in the module call platform is supposed to be replaced by actual platform or hardware code
  - SImulator piece
    - TCP/IP interface to talk to the TPM

    
- William (Bill) Roberts (notes from his presentation to 
  - TPM2-TSS
    - SAPI (shouldn't be used people aren't going to use it much)
    - ESAPI Full control but encrypted. If you want full low level control but want to do it with encryption
    - FAPI high level automatatic verified sessions with TPM and key storage
    - RcDecode Return code to error string (strerror for TPM2)
    - MU Marshalling and unmarshaling library
    - TctiLdr They are similar to a network layer in that they move bytes to and from the TPM over an arbitrary interface. Could talk to simulator, or /dev/tpm0
  - pytss
    - All based on cffi
    - Classes
    - Default arguments
    - Type coercions (Python types to TPM types, TPM2B, TPMT_PUBLIC)
      - Can take dictionaries
      - Can take subclass types directly
      - Copy constructors work
      - Comparison with string, euqivilent Python types, work
    - tpm2-tools -G strings work
      - -G, -a, -n tpm2-tools strings work
      - `.parse()` supported on TPM2T and TPM2B
      - `TPMT_PUBLIC.parse(alg="rsa2048:rsapss", objectAttributes="fixedtpm|sign")`
      - Defaults to tpm2_create over tpm2_createprimary
    - TPM2_ALG.parse("sha") --> TPM2_ALF.SHA or 0x04
      - For all structures
    - Can load tpm2-tool context files
      - `TPMS_CONTEXT.from_tools(ctxbytes)`
    - Exceptions run return code thorugh RcDecode for error message
      - `TSS2_Exception`
      - Handle, parameter, session, and return code are provided
    - ESAPI
      - `with ESAPI(TCTILdr("device")) as e: e.get_random()`
      - Context managers are your friend, they help you close the underlying TCTI
      - Will be able to pass TCTI loader string into `ESAPI()` call
      - Will choose sensible defaults
      - Will make you choose auth objects
    - Converting PEM/DER/SSH keys to TPM
      - `TPM2B_Public.from_pem(rsa_public_key)`
      - `TPM2B_SENSITIVE.from_pem(rsa_private_key)`
      - `e.load_extrenal(priv, pub)`
      - Conversions are being done using Python cryptography package
      - Other attributes can be set during call to `.from_pem()`
    - Importing keys ot TPM
      - You must wrap before import.
      - `.wrap()` let's you add keys
    - `make_credential()` will create a TPM credential which doesn't require talking to the TPM
      - Many people have asked for this
    - Calculate Name Without TPM
      - You can call `.get_name()` to get the name of a `TPM2B_PUBLIC` without going to the TPM
    -  TSS2 PEM/DER Format Support
      - Can grab keys from openssl engine / provider
      - Could create keys with `key.create_rsa(esapi).to_pem()`
    - FAPI
      - Very close to C API, doesn't use any of the native TPM types
        - Strings byte arrays, PEM files
     - Next Steps
       - Test against more versions of the TSS
       - policy engine
         - Eventually pull this back into the Python bindings
     - Examples
       - Check the test code
       - Will be adding to the docs
       - test/test_esapi.py
         - `get_random()` in two lines
     - Conclusions
       - Default arguments make commnds simple and less cluttered than the C code
       - Docs get updated automatically on read the docs
       - Can pass all your normal session handles
         - They'll be validated
