
**In Android, how would you design a secure way to store and use an authentication token that you receive from a server after login?**

I would use *Android Keystore* to securely store any tokens or encryption keys. Once stored, they are difficult to extract - even from rooted devices, and you can require authentication to use these keys, restricting when and how they are used. Apps must specify the authorised use of their keys, which is then enforced by Android.

Another option is to use *EncryptedSharedPreferences*, which is a version of *SharedPreferences* that encrypts it's contents.