---
layout: post
category : opinion
title: "Two-factor security with TOTP"
comments: true
tags : [security]
---

As a follow-up on my last article I looked into how easy it would be to incorporate Google Authenticator into your application. As it turns out, it's dead simple.

Google Authenticator adheres to the HOTP standard, which is an actual standard described in [RFC 6238](http://tools.ietf.org/html/rfc6238). The algoritms uses he 
HMAC SHA1 algorithm to calculate a 6 digit code for a secret in combination with a time interval (the time interval replaces the counter from the standard HOTP algorith).
Java has out-of-the-box support for HMAC SHA1 in it's cryptography algorithms (HmacSHA1), so implementing this is extremely easy.

The examples use Groovy, but all major languages and frameworks like PHP, Ruby or Node.JS  support the HMAC SHA1 hashing algorithm.<!--more-->

### Step one

Add a secret key to your accounts. Google Authenticator requires a 16-character key, so generating a secret key can for example be done by randomly generating an array of 10 bytes and then 
BASE64 encoding that array, resulting in a 16-character string.
 
``` groovy
static def generateSecret() {
    def buffer = new byte[10];
    new SecureRandom().nextBytes(buffer);
    return new String(new Base32().encode(buffer));
}
```

You can off course use your own generation algorithm.

### Generate a QR code for your application

Google authenticator uses a QR codes to add an account to it's application. To generate a QR code for a TOTP, you need to create one for the following key URI format:

``` plaintext
    otpauth://totp/issuer:user@host?secret=xxx&issuer=yyy
```

You can use Google Charts or ZXing to generate QR codes. As an example, I'll use Google Charts, so to generate a QR code for user fred for host myapplication 
with a secret like NAR5XTDD3EQU22YU, the key URI would be

``` plaintext
    otpauth://totp/fred@myapplication?secret=NAR5XTDD3EQU22YU
```

With Google Charts, this will generate the needed QR code: 

``` plaintext
    http://chart.apis.google.com/chart?cht=qr&chs=300x300&chl=otpauth%3A//totp/fred%40myapplication%3Fsecret%3DNAR5XTDD3EQU22YU
```

Scan this code with Google Authenticator and you will have added fred@myapplication to the repository of OTP codes. Link the secret secret to your users' account and you've
created the foundation for OTP authentication.

Google Authenticator allows for more information in the otpauth url, you can find the information [here](https://code.google.com/p/google-authenticator/wiki/KeyUriFormat).

### Add verification code to your authentication mechanism

Generating the current code for a timestamp is done by using the algorithm as described in the RFC. A good explanation of this algorithm is found on 
[wikipedia](http://en.wikipedia.org/wiki/Time-based_One-time_Password_Algorithm). The code itself is just as simple as generating the secret. First of all, you 
need a time index.

``` groovy
public static long getTimeIndex() {
    return System.currentTimeMillis()/1000/30;
}
```

This generates the amount of milliseconds since the UNIX epoch. Then you generate the OTP code for that time index.

``` groovy
static def getCode(byte[] secret, long timeIndex) 
              throws NoSuchAlgorithmException, InvalidKeyException {
    SecretKeySpec signKey = new SecretKeySpec(secret, "HmacSHA1");
    ByteBuffer buffer = ByteBuffer.allocate(8);
    buffer.putLong(timeIndex);
    byte[] timeBytes = buffer.array();
    Mac mac = Mac.getInstance("HmacSHA1");
    mac.init(signKey);
    byte[] hash = mac.doFinal(timeBytes);
    int offset = hash[19] & 0xf;
    long truncatedHash = hash[offset] & 0x7f;
    for (int i = 1; i < 4; i++) {
        truncatedHash <<= 8;
        truncatedHash |= hash[offset + i] & 0xff;
    }
    return (truncatedHash %= 1000000);
}
```


### Enjoy two-factor security

Two-factor security is simple, it's authenticating through 2 things:

* something the user knows (his username)
* something the user has (his OTP generating device)

Instead of using basic passwords, you can now use an OTP instead of a static password (or you can use both, but I don't see any added value by adding a second thing
the user should 'know'). When the user logs in using his username and OTP, you retrieve the secret key from that user, generate the current OTP and verify the OTP the 
user entered with the one you generated. 

### Look out for time zones and clock differences

If you're using the UNIX epoch, you won't need to worry about time zones, but should you generate the time index using something using time zones, always make sure 
both the OTP generator and verifier are using the same time zone or you won't be able to log in.

A more difficult item is clock differences. If for some reason the clocks of the generator and verifier are out of sync, you can get yourself into a situation in which 
it is impossible to log in to the system. This can be managed by allowing some time leniency into your verification code by also generating the next and previous 3 codes
and checking the user's entered OTP code against those 7 codes.
This allows for a 90 second discrepancy between the OTP verificator and generator.

### Support for hardware tokens and alternatives to Google Authenticator

Sometimes, software solutions like Google Authenticator may not be an ideal solution. Luckily, there are a lot of vendors out there that provide hardware tokens that
are compatible with RFC 6238. The difference between them and something like Google Authenticator is that the key size can be different and or another hashing algorithm
like SHA-256.

There are also a lot more applications other than Google Authenticator that are RFC 6238 compliant. Just take a look into your app store for TOTP applications.

### Conclusion

Looking back, this solution is so simple, you start to wonder why passwords are still so much in use. This solution is a lot safer than static passwords (you need the server
side secret key in order to generate the OTP and previous OTP's are useless). Ok, you require your users to use a physical device to log in, but most of our users already
own a smartphone. Those who don't you can provide with a hardware token (or a cheap Android smartphone for that matter).

Personally I don't see any reason why a company wouldn't want to adopt a system like this. I don't know of any security officers who wouldn't smile at the prospect of not
having to lose sleep over password policies or security breaches due to someone using 'god' or the username written backwards as password. Now that we have BYOD, maybe it's time
for BYOP (Bring Your Own Password).
