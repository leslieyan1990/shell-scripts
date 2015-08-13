# Secrets generation

This guide generates the secrets used in the container services. 

## Generate a self-signed SSL certificate for nginx:

```sh
mkdir -p secrets/ssl

openssl req -nodes -new -x509 \
	-keyout secrets/ssl/dev.test.key \
	-out secrets/ssl/dev.test.crt
```

## Generate the SSH keypair and known hosts file for git access:

```sh
mkdir -p secrets/ssh

ssh-keygen -t rsa -C deploy -N "" -f secrets/ssh/id_rsa

ssh-keyscan -t rsa github.com $(dig +short github.com) \
  >> secrets/ssh/known_hosts
ssh-keyscan -t rsa bitbucket.org $(dig +short bitbucket.org) \
  >> secrets/ssh/known_hosts
```

## Create the environment variables config file:

```sh
echo '
# Each line of the expenv configuration must have the following format:
# VARIABLE_NAME command [args...]
# Examples:
# PASSWORD echo secret
# KEY cat path/to/key_file
# Empty lines and lines starting with a hash (#) will be ignored.

SSMTP_AUTH_USER echo mail@dev.test
SSMTP_AUTH_PASS echo password

SSL_CRT cat secrets/ssl/dev.test.crt
SSL_KEY cat secrets/ssl/dev.test.key

SSH_PRIVATE_KEY cat secrets/ssh/id_rsa
SSH_PUBLIC_KEY cat secrets/ssh/id_rsa.pub
SSH_KNOWN_HOSTS cat secrets/ssh/known_hosts
' > .expenv
```