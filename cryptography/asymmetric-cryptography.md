# Asymmetric cryptography (Public-key cryptography)

I learned a lot about cryptography when I was getting into blockchain and distributed ledger technology. One of the most fundamental cryptographic systems in our modern technology is asymmetric cryptography a.k.a. public-key cryptography.

## How does it work?

In asymmetric encryption, each encryption has two keys. A private key and a public key. The private key should be stored very safely. The public key (in most of the cases) can and should be exchanged with others. These keys can be used to crypt, decrypt and sign messages and data.

The trick in this *asymmetric* system is that a message encrypted with the public key, can only be decrypted with the private key. To decrypt a message thats encrypted (signed) by the private key, you need the public key. But note that anyone who got the private key can generate the corresponding public key from it. The asymmetric cryptography is consequently only secure if it's ensured that the private key never fall into the wrong hands.

### A real example

There are many well-known protocols where the asymmetric cryptography is used by default or optional. A very good and modern example can be found in blockchain technology. There are wallets and transactions. Wallets are the place where some sort of value (e.g. a coin of an cryptocurrency) is stored. This value can be transfered to another wallet by an transaction.

Your wallet is basically just a private key. Any wallet is public accessible with the public key. But a value can only be transfered from one wallet to another with an transaction that is signed with the private key of the sender.

The transaction consists of the amount, the senders public key, the receivers public key and the signature which is created by the senders private key. This signature can by verified by anyone with the public key of the sender.

## How to do it technically

### Generate key pair

The most basic requirement to work with asymmetric cryptography is the key pair. The generation of a keypair always begins with the generation of the private key. Then the public key will be generated with the private key. 

Its also possible to generate a fingerprint which is more or less just a simpler presentation of the public key but cannot be used for encryption itself. 

#### Generate private key

There are different algorithms that can be used to generate a private key. They differ mainly in the grade of encryption strenght but there are also algorithms that are giving you some helpful features. Some popular algorithms are RSA and ECC. Just to name a few of them.

Private keys usualy created completely random. In some tools you even have to move your mouse while generating to ensure that the key is generated completely random. But in some usecases it makes sense to be able to easily restore a private key. This brings some downsides in security. But imagine you got a wallet with crypto assets with real money value. You want to be able to restore your wallet if the device with your privatekey isn't working anymore.

For this case hierachical deterministic keys were invented. This gives a possibility to generate a seed (usually a combination of words, also known as mnemonic phrase) which can be used to generate one and the same private key again.

Keys also can have different sizes. The higher the size of a key, the higher is the rate of encryption strength. But most algorithms only support a set of sizes and a maximum size.

There are many tools to work with keys. But here I'll use openssl since its available on many systems by default.

The following command can be used to generate a pretty basic RSA private key with a size of 2048 bits.

```
openssl genrsa -out privatekey.pem 2048
```

Note that there are also different formats. In this case we've created a key in pem. Some systems work with different formats. There are tools to convert a key from one format to another.

A private key can also be protected with a passpharse. In this case the private key gets an additional encryption. But you have to enter the passpharse each time before using the private key.

#### Generate public key

As mentioned, the public key is generated out of the private key. The following command is used to do this:

```
openssl rsa -in privatekey.pem -pubout > publickey.pem
```

#### Generate fingerprint

Since the generated keys are very long, some systems work with fingerprints to represent a key. That safes space and comes with just a few downsides.

For example, a public key is stored in a database and you have to search for it. You can do this in two ways. Either the public key is stored as it is or it is stored as a fingerprint.

The first is easy to do since you just search with the whole public key but gets slower the more data are in the database. With the second way the key is hashed before searching and the stored keys are also hashed. This will be much faster since the size of a hash is smaller then the size of the key.

Don't confuse a fingerprint with a digital signature. A fingerprint is just a reference to a specific kind of data. A digital signature ensures that the creator of some data is the creator you're actually expecting.

Please keep also in mind that working with fingerprints got a few downsides like the possibility of hash collisions. There are also possible exploits like the length extension attack. Most of the downsides can be eliminated. But you have to know them. For example, the Bitcoin blockchain uses a double hashed fingerprint to prevent for the length extension attack. 

### Encryption, decryption and signing

The following part explains encrypting and signing. I've learned a lot about that in this stackoverflow https://stackoverflow.com/questions/18257185/how-does-a-public-key-verify-a-signature question. Especially from Jaakko's reply. So I got highly inspired by his or her reply, for the following section.

#### Encrypting (Public key encrypts, private key decrypts)

Encrypting is the process of encrypting a message with a public key, that only can be decrypted by the private key.

You can do so with openssl:

```
openssl rsautl -encrypt -inkey publickey.pem -pubin -in message.txt -out message.ssl
```

Decrypt the message with the private key:

```
openssl rsautl -decrypt -inkey privatekey.pem -in message.ssl -out messagedecrypted.txt
```

#### Signing (Private key encrypts, public key decrypts)

The process of encrypting with the private key and decrypting with the public key is called signing. This way isn't really intended to be safe and basically just ensures that the sender of a message got the matching private key that's corresponding with your public key. So you can be sure that the creator of the message is the expected creator.

Sign a message with a private key:

```
openssl rsautl -sign -inkey privatekey.pem -in message.txt -out message.ssl
```

Decrypt the signature with the public key:

```
openssl rsautl -inkey publickey.pem -pubin -in message.ssl -out messagedesigned.txt
```

Some protocols like a JWT got signing by design. In addition to the data, they send a signature. This signature represents the complete content of the data. You know who should be the sender and the corresponding public key. So you can decrypt the signature and if this content equals the sended data you can be sure that you got the data from the correct sender.