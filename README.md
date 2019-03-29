CryptBundle
===========

[![Build Status](https://travis-ci.org/GlobalTradingTechnologies/crypt-bundle.svg?branch=master)](https://travis-ci.org/GlobalTradingTechnologies/crypt-bundle)
[![Latest Stable Version](https://poser.pugx.org/gtt/crypt-bundle/version)](https://packagist.org/packages/gtt/crypt-bundle)
[![Latest Unstable Version](https://poser.pugx.org/gtt/crypt-bundle/v/unstable)](//packagist.org/packages/gtt/crypt-bundle)
[![License](https://poser.pugx.org/gtt/crypt-bundle/license)](https://packagist.org/packages/gtt/crypt-bundle)

Provides a simple way to configure Symfony services for data encryption and decryption based on well-known encryption algorithms.
With the help of this CryptBundle you can encrypt or decrypt your data as simple as operate with elementary [EncryptorInterface](https://github.com/GlobalTradingTechnologies/crypt-bundle/blob/master/Encryption/EncryptorInterface.php) or [DecryptorInterface](https://github.com/GlobalTradingTechnologies/crypt-bundle/blob/master/Encryption/DecryptorInterface.php):
```php
use Gtt\Bundle\CryptBundle\Encryption\EncryptorInterface;
use Gtt\Bundle\CryptBundle\Encryption\DecryptorInterface;

class MyMagicService
{    
    /**
     * Encryptor
     *
     * @var EncryptorInterface
     */
    protected $encryptor;
 
    /**
     * Decryptor
     *
     * @var DecryptorInterface
     */
    protected $decryptor;
    
    public function __construct(EncryptorInterface $encryptor, DecryptorInterface $decryptor)
    {
        $this->encryptor = $encryptor;
        $this->decryptor = $decryptor;
    }
    
    public function doSomeMagic()
    {
        $someStringData = "Crypt me!";
        $encrypted = $this->encryptor->encrypt($someStringData);
        $decrypted = $this->decryptor->decrypt($encrypted);
        
        return $someStringData == $decrypted;
    }
}
```
Implementations of [EncryptorInterface](https://github.com/GlobalTradingTechnologies/crypt-bundle/blob/master/Encryption/EncryptorInterface.php) or [DecryptorInterface](https://github.com/GlobalTradingTechnologies/crypt-bundle/blob/master/Encryption/DecryptorInterface.php) would be provided by CryptBundle. The choice of encryption algorithm is up to you - you can specify it in bundle config.

Requirements
============

Requires only PHP 5.6+ and symfony/framework-bundle.

Installation
============

Bundle should be installed via composer

```
composer require gtt/crypt-bundle
```
After that you need to register the bundle inside your application kernel.

Also you probably need to install specific crypto libraries such as

```
composer install zendframework/zend-crypt
composer install defuse/php-encryption
```
(You can add the libraries that you need. All of them are optional)

Encryption
==========
Under the hood bundle provides bridges to well-known php components for encrypting data as implementations of
[encryptor](https://github.com/GlobalTradingTechnologies/crypt-bundle/blob/master/Encryption/EncryptorInterface.php) and [decryptor](https://github.com/GlobalTradingTechnologies/crypt-bundle/blob/master/Encryption/DecryptorInterface.php) interfaces.
This implementations are registered as a Symfony services that can be used by your code. See [Usage section](#Usage) for details.


Configuration
=============
Bundle configuration defines one or several cryptor-sections grouped by encryption-type (for now supported types are aes and rsa) and the name of the pair of encryptor and decryptor (_aes_binary_cryptor_, _aes_log_cryptor_ and _rsa_default_cryptor_ in example below).
Each cryptor-section contains options for defining the pair of encryptor and decryptor of certain encryption type.
You can see example that holds configs for 2 pairs of aes encryptor and decryptor and one pair of rsa encryptor and decryptor:
```yml
gtt_crypt:
    cryptors:
        aes:
            aes_binary_cryptor:
                key_size: 128
                key_path: "/tmp/keys/aes/first.key"
                binary_output: true
            aes_log_cryptor:
                key_size: 128
                key_path: "/tmp/keys/aes/second.key"
                binary_output: false
        rsa:
            rsa_default_cryptor:
                private_key: "/tmp/keys/rsa/priv.key"
                public_key: "/tmp/keys/rsa/pub.key"
                binary_output: false
                padding: 4
```
You can see reference of configuration options for supported encryption types:
### RSA
- private_key - path to RSA private key
- pass_phrase - RSA private key passphrase
- public_key - path to RSA public key
- binary_output - should be result of encryption encoded with base64 algorithm or should be input string base64-decoded before decryption
- padding - number of distinct practices which all include adding data to the message prior to encryption.
 Should be one of constants from list below:
-- OPENSSL_PKCS1_PADDING
-- OPENSSL_SSLV23_PADDING
-- OPENSSL_NO_PADDING
-- OPENSSL_PKCS1_OAEP_PADDING

### AES
- key_size - AES key size. Should be 128 for 1.x or 256 for 2.x version of [defuse/php-encryption](https://github.com/defuse/php-encryption/)
- key_path - path to AES private key. Can be generated by built in [crypt:aes:generate-key](https://github.com/GlobalTradingTechnologies/crypt-bundle/blob/master/Command/GenerateKeyCommand.php) command
- binary_output - should be result of encryption encoded with base64 algorithm or should be input string base64-decoded before decryption

Usage
=====
In order to use encryptos and decryptors in your code you have 3 availabilities:
### Tag your service (recommended)
The prefered way to receive encryptor or decryptor in you service is to implement in the service's class very simple [EncryptorAwareInterface](https://github.com/GlobalTradingTechnologies/crypt-bundle/blob/master/Encryption/EncryptorAwareInterface.php) or [DecryptorAwareInterface](https://github.com/GlobalTradingTechnologies/crypt-bundle/blob/master/Encryption/DecryptorAwareInterface.php).
You can also use traits in most cases if you are too lazy: [SingleDecryptorAwareTrait](https://github.com/GlobalTradingTechnologies/crypt-bundle/blob/master/Encryption/SingleDecryptorAwareTrait.php) and [SingleEncryptorAwareTrait](https://github.com/GlobalTradingTechnologies/crypt-bundle/blob/master/Encryption/SingleEncryptorAwareTrait.php).
After that you should tag the service with `gtt.crypt.encryptor.aware` or `gtt.crypt.decryptor.aware` tag (depends on whether you want to get encryptor or decryptor) and specify cryptor name in tag attribute `cryptor_name`.
For example if you use configuration such as in [Configuration section](#Configuration) the _cryptor_name_ attribute value can be one of _aes_binary_cryptor_, _aes_log_cryptor_ or _rsa_default_cryptor_.
The setter injection (setters are defined in EncryptorAwareInterface/DecryptorAwareInterface interfaces) would be done by [CryptorInjectorPass](https://github.com/GlobalTradingTechnologies/crypt-bundle/blob/master/DependencyInjection/Compiler/CryptorInjectorPass.php).

```yml
services:
    gtt_encryptor_holder:
        class: Your\Class\That\Wants\To\Receive\Encryptor
        tags:
            - { name: gtt.crypt.encryptor.aware, cryptor_name: "aes_binary_cryptor" }
    gtt_decryptor_holder:
        class: Your\Class\That\Wants\To\Receive\Decryptor
        tags:
            - { name: gtt.crypt.decryptor.aware, cryptor_name: "aes_binary_cryptor" }
```
### Inject cryptors directly by service id
Each encryptor or decryptor configured by CryptBundle is a service with id constructed in accordance with the following pattern:
`gtt.crypt.encryptor.<name>` for encryptors and `gtt.crypt.decryptor.<name>`, where <name> holds corresponding cryptor name defined in bundle config.
For example if you use configuration such as in [Configuration section](#Configuration) the <name> value can be one of _aes_binary_cryptor_, _aes_log_cryptor_ or _rsa_default_cryptor_.
You can simply inject these services in DI-configs of your bundles.
### Use cryptor registry
Crypt-bundle also registers [CryptorRegistry](https://github.com/GlobalTradingTechnologies/crypt-bundle/blob/master/CryptorRegistry.php) service with id _gtt.crypt.registry_ that collects all the encryptors and decryptors configured.
You can use it to get cryptors by calling getEncryptor or getDecryptor methods with name of the encryptor or decryptor specified.
For example if you use configuration such as in [Configuration section](#Configuration) the name can be one of _aes_binary_cryptor_, _aes_log_cryptor_ or _rsa_default_cryptor_.

Supported encryption components
===============================
* RSA (Based on [zendframework/zend-crypt](https://github.com/zendframework/zend-crypt))
* AES (Based on [defuse/php-encryption](https://github.com/defuse/php-encryption/))