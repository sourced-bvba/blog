---
layout: post
category : article
title: "Easy RSA signatures and encryption with JWK"
comments: true
tags : [java]
---

Sometimes when you send data, it's required that you prove that you actually sent the data and not some malicious third party. That's when you have to sign some data. The most known type of signatures around currently are RSA signatures, with PGP being one of the most popular applications widely used around the world.

The same thing can be said with encryption. When you want to send certain data to a certain endpoint and you want to make sure only that endpoint can decrypt the data (not even the one that encrypted the data in the first place), you'll need assymetric encryption like RSA.

Using RSA isn't really that hard in Java, if you know your way around keystores and how to manage them. But in the REST age, interchanging keystores can be a serious pain in the ass, because it's not really REST friendly. This now changed with the advent of the JWK (JSON Web Key) standard. Most of us know what a JSON Web Token is, and if you're securing REST services, chances are you're already using them (if you're still using basic or digest authentication, shoot yourself now). JWTs can be signed using a certain key and can even be encrypted using a certain key. The problem is that most people tend to go for the easy solution and choose a simple symmetric algorithm. The problem with this is that anyone can sign data if they can verify that data, because they need to know the shared key, which opens up the possibility for forgery.

RSA is assymmetrical. This means you have a public and a private key (that can also be used to derive the public key). For encryption you encrypt using the public key and only the person with the private key can decrypt. For signatures this is vice versa, you need the private key to sign and the public key to verify.

So how does a JWK actually look like. Well, a RSA512 key with a key size of 2048 bits can look like this:

{% highlight json %}
{
	  "alg": "RS512",
	  "d": "zg42TgpUyzGx6Gs9VUsbgiDk41CDg7SOFs_56nNt_ZimjZWO48YBewQXeTD8HGIcKUyo0IlKqWxNrOZBYXKWy_ac2F-SAHUHrxLvNIoclphCyDl43H6y0eLeSu4QjylM3sKwUjAIaMxBuFiQ2lswzxUc4037YuYx1XzCZcByhQAw4nZ-aywBRYe9O50UgZbIl-4jyc9QD0Iioh4xPZh31DwGGf6q_3vrLCHXe3-AW530ogpgJBvz7vRX_FdFNxDlC-tbtJn8eFmi_QZujj6pUIRqyVIufEObhsUDYZkS22wNeOiHV8Z651pgfqAPQBBY5YEz21VviDhmtx02mKZMzMT6aCaY52mWCQVo1Q9jnO7nZQDN5I8G3JLQdVE-DTUNHHD8GYnX2oI3ihuLonFgp21XYXF40ATBU8isHQTGwc1RcRxokxOt0rfc-PVGzb8i7a-rViXkxmDjh6-5Rb4JTclh54EMdyAcDQRZo0kg54wLnZVHSCFLuOmrLecGjwDMSux8nIQblL7oepjQ8qjqPrYaGax_LJ3ujhoEArAjAOiGNNtUXztgoudoUqKUm3HOuWtnIAyAyik0bY4dDlNDHaP0Zb-NxerzC9uXG3lr2hFrrHD3wJA7q8Zpxex14V9_DzZ0Th_d0zy5aph6zaaY9c63v3Gz41hAgGvB1Vt4nQE",
	  "e": "AQAB",
	  "n": "05Nzt8JfbI4uRP_FOCAfPcvxf2TbHwYuDMltadj10hIGe4Kjl27UHxktkoONhR_uuAt2pDhnv08aKuREIo6qp8-uPSrAyQT-evV28op3gHmEmrknIClCw0dTVRuELqVBPJDiS5LeDTLGe6GfJl80vVn9u0YRc2DSFiyfyTUPfd34uAwJO0tB2BifIa4nvfqLxc49iLzvJxo8Bvtv5cq_8AlITbN8MkJwLvaHYyftZvatAO44fZqZGwGQYYpql-5eaRPhV0lmELOaU-unw6I5iUcrapNyH00NK0qKxw9V1HpdyQ0tNycCFsdYo72m86gy0Rf2U_fYbXrLsVKyyWds5tmMu_My2EUK9t0OdJGXC2pYClabRV-S3zzFq_SiuAXsIFKJGGHxeLSy3a-Nrlo0jGVfpEcV2Zzo1WCjXEQp2FYir2D47lX0MQqPEvxgmCqFbLuzwRUwfDxCFsYZsDDqViMGvfq22iJq2XUbnmN98LAtlup7J8S7hqBfkMr9nPvAkJ9Qstsro_BXAUHCMasUdiAiON7eQza7vGR7saQ4EvP-xga4LuI8VNx8O8FoiK_jOPB-C542bdNtGPhAY82SVY9UMCxnYG6n5DXf5KEs__p9VrQtXlpGapQAm6BnNieo69UQlN-uJIslMjrkK5qqCiMEVRP_5jf-81Xm79hFI-U",
	  "kty": "RSA",
	  "kid": "my-rsa-key"
	}
{% endhighlight %}

This is a private key, because of the presence of the `d` element (which is the private exponent of the RSA key). If you omit this element from the above JSON, you have the corresponding public key.

To use this in Java, you can use the Nimbus JWT library that is available. It supports all the current standards like JWT, JWE and off course JWK. 

To read this key into Java you need to put all the keys in a JSON structure that has a collection of `keys`, so something like this:

{% highlight json %}
{
  "keys": [
	{
	  "kid": "key-one",
	  ...
	},
	{
	  "kid": "key-two",
	  ...
	}
  ]
}
{% endhighlight %}

Once you have this, you just need a little bit of code to use this file:

{% highlight java %}
File keySetFile = ...;
JWKSet keySet = JWKSet.load(keySetFile);
{% endhighlight %}

Now to get an RSA key out of the keyset, you can just do this:

{% highlight java %}
RSAKey key = (RSAKey) keySet.getKeyByKeyId("my-rsa-key");
KeyPair keyPair = new KeyPair(key.toPublicKey(), key.toPrivateKey());
{% endhighlight %}

You can see I also make a `javax.security.KeyPair` from this `RSAKey`. This now allows me to use standard Java cryptographic APIs to sign and encrypt data.

For example, if I wanted to sign some data, I can do this:

{% highlight java %}
Signature signature = Signature.getInstance("SHA512withRSA");
signature.initSign(keyPair.getPrivate(), new SecureRandom());
String stringToBeSigned = "Hello there";
signature.update(stringToBeSigned.getBytes());
String sig = Base64.getEncoder().encodeToString(signature.sign());
System.out.println("This is the signature: " + sig);

signature.initVerify(keyPair.getPublic());
signature.update(stringToBeSigned.getBytes());
System.out.println("The signature is valid: " + signature.verify(Base64.getDecoder().decode(sig)));
{% endhighlight %}

This example will print out a SHA512 signature (Base64 encoded for readability) and also indicate that the signature is also valid according to the public key.

To encrypt some data, this is actually easy:

{% highlight java %}
String stringToBeEncrypted = "Hello there";
Cipher encrypter = Cipher.getInstance("RSA");
encrypter.init(Cipher.ENCRYPT_MODE, keyPair.getPublic());
encrypter.update(stringToBeEncrypted.getBytes());
String encrypted = Base64.getEncoder().encodeToString(encrypter.doFinal());
System.out.println(encrypted);

Cipher decrypter = Cipher.getInstance("RSA");
decrypter.init(Cipher.DECRYPT_MODE, keyPair.getPrivate());
decrypter.update(Base64.getDecoder().decode(encrypted));
String decrypted = new String(decrypter.doFinal());
System.out.println(decrypted);
{% endhighlight %}

This will encrypt a String using the RSA algorithm using the keysize you chose when generating the JWK and print it out (again Base64 encoded) and will then decrypt it using the private key and show you the decrypted string (which should be `Hello there`).

Bear in mind that RSA is not suitable for large data encryptions. A 2048 bit RSA key can at most encrypt 256 bytes of data. If you need to encrypt a lot of data, this is what you normally do:

- you encrypt the data using a symmetrical algorithm (like AES or Blowfish) with a shared key
- you encrypt the shared key using RSA and the public key of the recipient
- you send both the encrypted data and the encrypted key to the recipient

This way you can encrypt large amounts of data and be sure only the recipient will be able to decrypt the data.

As you can see, encrypting using assymmetrical algorithms is really not that hard anymore. If you for example want to allow external parties to send you encrypted data, you can easily now provide a REST endpoint to your public key. At the same time you can also ensure your external parties that they can verify any data you send them because they can check the signature using that public key (or another one you specifically use for signing).

Now you literally have no excuse anymore to store sensitive data unencrypted in your system or send sensitive unencrypted data over the wire. You only need to guard your RSA private key with your life just like you do with your private SSH key. Which is also an RSA key by the way.





