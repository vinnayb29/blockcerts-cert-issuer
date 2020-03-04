[![Build Status](https://travis-ci.org/blockchain-certificates/cert-issuer.svg?branch=master)](https://travis-ci.org/blockchain-certificates/cert-issuer)
[![PyPI version](https://badge.fury.io/py/cert-issuer.svg)](https://badge.fury.io/py/cert-issuer)


# cert-issuer

The cert-issuer project issues blockchain certificates by creating a transaction from the issuing institution to the
recipient on the Bitcoin blockchain that includes the hash of the certificate itself.

## Web resources
For development or testing using web requests, check out the documentation at [docs/web_resources.md](./docs/web_resources.md).

# Quick start using Docker

## Getting the Docker image
This uses bitcoind in regtest mode. This route makes many simplifications to allow a quick start, and is intended for
experimenting only.

1. First ensure you have Docker installed. [See our Docker installation help](docs/docker_install.md).

2. Clone the repo and change to the directory

    ```
    git clone https://github.com/blockchain-certificates/cert-issuer.git && cd cert-issuer
    ```


3. From a command line in cert-issuer dir, build your docker container:

    ```
    docker build -t bc/cert-issuer:1.0 .
    ```

4. Read before running!

    - Once you launch the docker container, you will make some changes using your personal issuing information. This flow mirrors what you would if you were issuing real certificates.
    - To avoid losing your work, you should create snapshots of your docker container. You can do this by running:

        ```
        docker ps -l
        docker commit <container for your bc/cert-issuer> my_cert_issuer
        ```

5. When you're ready to run:

    ```
    docker run -it bc/cert-issuer:1.0 bash
    ```

## Create issuing address

__Important__: this is a simplification to avoid using a USB, which needs to be inserted and removed during the
standard certficate issuing process. Do not use these addresses or private keys for anything other than experimenting.

Ensure your docker image is running and bitcoind process is started

1. Create an 'issuing address' and save the output as follows:

    ```
    issuer=`bitcoin-cli getnewaddress`
    sed -i.bak "s/<issuing-address>/$issuer/g" /etc/cert-issuer/conf.ini
    bitcoin-cli dumpprivkey $issuer > /etc/cert-issuer/pk_issuer.txt
    ```

2. Don't forget to save snapshots so you don't lose your work (see step 3 of client setup)

## Issuing certificates

1. Add your certificate to /etc/cert-issuer/data/unsigned_certificates/.

    ```
    # To use a sample unsigned certificate as follows:
    cp /cert-issuer/examples/data-testnet/unsigned_certificates/3bc1a96a-3501-46ed-8f75-49612bbac257.json /etc/cert-issuer/data/unsigned_certificates/

    # If you created your own unsigned certificate using cert-tools (assuming you placed it under data/unsigned_certificates):
    cp <cert-issuer-home>/data/unsigned_certificates/<your-cert-guid>.json /etc/cert-issuer/data/unsigned_certificates/
    ```

2. Make sure you have enough BTC in your issuing address.

    a. You're using bitcoind in regtest mode, so you can print money. This should give you 50 (fake) BTC:

    ```
    bitcoin-cli generate 101
    bitcoin-cli getbalance
    ```

    b. Send the money to your issuing address -- note that bitcoin-cli's standard denomination is bitcoins not satoshis! (In our
    app, the standard unit is satoshis.) This command sends 5 bitcoins to the address

    ```
    bitcoin-cli sendtoaddress $issuer 5
    ```

3. Issue the certificates on the blockchain

    ```
    cert-issuer -c /etc/cert-issuer/conf.ini
    ```

4. Your Blockchain certificates are located in `/etc/cert-issuer/data/blockchain_certificates`. Copy these to your local machine, and add them to cert-viewer's `cert_data` folder to see your certificates in the Certificate Viewer.

```
docker ps  // shows the docker containerId
docker cp <containerId>:/etc/cert-issuer/data/blockchain_certificates <localPath>/cert-viewer/cert_data
```



## Mac scrypt problems

If your install on Mac is failing with a message like the following, try the [workaround described in this thread](https://github.com/ethereum/pyethereum/issues/292) or the [workaround described here](https://github.com/pyca/cryptography/issues/2692#issuecomment-272773481).

```
fatal error: 'openssl/aes.h'
      file not found
#include <openssl/aes.h>
```

# Contact

Contact us at [the Blockcerts community forum](http://community.blockcerts.org/).


