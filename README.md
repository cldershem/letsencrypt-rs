# letsencrypt-rs

[![Build Status](https://secure.travis-ci.org/onur/letsencrypt-rs.svg?branch=master)](https://travis-ci.org/onur/letsencrypt-rs)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/onur/letsencrypt-rs/master/LICENSE)
[![Crates.io](https://img.shields.io/crates/v/letsencrypt-rs.svg)](https://crates.io/crates/letsencrypt-rs)
[![docs.rs.io](https://docs.rs/acme-client/badge.svg)](https://docs.rs/acme-client)

Easy to use Let's Encrypt client and acme client library to issue, renew and
revoke TLS certificates.

## Installation

You can install letsencrypt-rs with:
`cargo install letsencrypt-rs`


## Usage

letsencrypt-rs is using the openssl library to generate all required keys
and certificate signing request. You don't need to run any openssl command.
You can use your already generated keys and CSR if you want and you don't need
any root access while running letsencrypt-rs.

letsencrypt-rs is using simple HTTP validation to pass Let's Encrypt's DNS
validation challenge. You need a working HTTP server to host the challenge file.


### Sign a certificate

```sh
letsencrypt-rs sign -D example.org -P /var/www -k domain.key -o domain.crt
```

This command will generate a user key, domain key and X509 certificate signing
request. It will register a new user account and identify the domain ownership
by putting the required challenge token into `/var/www/.well-known/acme-challenge/`.
If everything goes well, it will save the domain private key into `domain.key`
and the signed certificate into `domain.crt`.

You can also use the `--email` option to provide a contact adress on registration.



### Using your own keys and CSR

You can use your own RSA keys for user registration and domain. For example:

```sh
letsencrypt-rs sign \
  --user-key user.key \
  --domain-key domain.key \
  --domain-csr domain.csr \
  --domain example.org \
  -P /var/www \
  -o domain.crt
```

This will not generate any key and it will use provided keys to sign
the certificate.


### Using DNS validation

You can use `--dns` flag to trigger DNS validation instead of HTTP. This
option requires user to generate a TXT record for domain. An example DNS
validation:

```sh
$ letsencrypt-rs sign --dns -D onur.im -E onur@onur.im \
    -k /tmp/onur.im.key -o /tmp/onur.im.crt
Please create a TXT record for _acme-challenge.onur.im: fDdTmWl4RMuGqj9acJiTC13hF6dVOZUNm3FujCIz3jc
Press enter to continue
```


### Revoking a signed certificate

letsencrypt-rs can also revoke a signed certificate. You need to use your
user key and a signed certificate to revoke.

```sh
letsencrypt-rs revoke --user-key user.key --signed-crt signed.crt
```


## Options

You can get a list of all available options with `letsencrypt-rs sign --help`
and `letsencrypt-rs revoke --help`:

```
$ letsencrypt-rs sign --help
letsencrypt-rs-sign 
Signs a certificate

USAGE:
    letsencrypt-rs sign [FLAGS] [OPTIONS] --domain <DOMAIN>

FLAGS:
    -c, --chain      Chains the signed certificate with Let's Encrypt Authority
                     X3 (IdenTrust cross-signed) intermediate certificate.
    -d, --dns        Use DNS challenge instead of HTTP. This option requires
                     user to generate a TXT record for domain
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -B, --bit-length <BIT_LENGHT>
            Bit length for CSR. Default is 2048.
    -D, --domain <DOMAIN>
            Name of domain for identification. This option is required.
    -C, --domain-csr <DOMAIN_CSR>
            Path to domain certificate signing request.
    -K, --domain-key <DOMAIN_KEY_PATH>
            Domain private key path to use it in CSR generation.
    -E, --email <EMAIL>
            Contact email address (optional).
    -P, --public-dir <PUBLIC_DIR>
            Directory to save ACME simple http challenge. This option is
            required.
    -S, --save-csr <SAVE_DOMAIN_CSR>
            Path to save domain certificate signing request.
    -k, --save-domain-key <SAVE_DOMAIN_KEY>     Path to save domain private key.
    -o, --save-crt <SAVE_SIGNED_CERTIFICATE>
            Path to save signed certificate. Default is STDOUT.
    -u, --save-user-key <SAVE_USER_KEY>         Path to save private user key.
    -U, --user-key <USER_KEY_PATH>
            User private key path to use it in account registration.
```

```
$ letsencrypt-rs revoke --help
letsencrypt-rs-revoke 
Revokes a signed certificate

USAGE:
    letsencrypt-rs revoke --user-key <USER_KEY> --signed-crt <SIGNED_CRT>

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -C, --signed-crt <SIGNED_CRT>    Path to signed domain certificate to revoke.
    -K, --user-key <USER_KEY>        User or domain private key path.
```


## `acme-client` crate

`letsencrypt-rs` is powered by the acme-client library. You can read the
documentation in [docs.rs](https://docs.rs/acme-client). Example usage of
`AcmeClient`:

```rust
AcmeClient::new()
    .and_then(|ac| ac.set_domain("example.org"))
    .and_then(|ac| ac.register_account(Some("contact@example.org")))
    .and_then(|ac| ac.identify_domain())
    .and_then(|ac| ac.save_http_challenge_into("/var/www"))
    .and_then(|ac| ac.simple_http_validation())
    .and_then(|ac| ac.sign_certificate())
    .and_then(|ac| ac.save_domain_private_key("domain.key"))
    .and_then(|ac| ac.save_signed_certificate("domain.crt"));
```
