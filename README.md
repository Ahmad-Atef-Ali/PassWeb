# PassWeb

If you use the Internet much, you have a hundred different passwords for things like email accounts, subscriptions, banking, social media, and the like.
The best passwords are long and complex (combining letters, numbers, symbols, and punctuation) and never reused across different accounts.
With so many to remember, some people use a [password manager](http://en.wikipedia.org/wiki/Password_manager) to securely store everything behind a single, memorable master password of suitable complexity.
Password managers make it easy to use strong passwords for every account - but they also introduce a single point of failure.
Much has been said on both sides of the argument and the decision to use a password manager should be made carefully.

If you decide a password manager is right for you, there are different kinds and many options to choose from.
For my purposes, a [cloud-based](http://en.wikipedia.org/wiki/Password_manager#Advantages) password manager seemed best.
Thinking about what's important to me, I wanted something:

* Trustworthy
* Open source
* Cross-platform
* Cross-device
* Offline-enabled
* Simple

I couldn't find a perfect match, so I wrote my own cloud-based password manager: **PassWeb**.
From time to time, someone asks to try it out, so I've open-sourced the implementation for anyone to evaluate, use, and improve.


## Disclaimer

I've tried to ensure PassWeb is safe and secure for normal use in low-risk environments, but **do not trust me**.
Before using PassWeb, you should evaluate it against your unique needs, priorities, threats, and comfort level.
If you find a problem or a weakness, please let me know so I can address it - but ultimately you use PassWeb **as-is** and **at your own risk**.


## FAQ

**What is PassWeb?**
PassWeb is a simple online/offline web application to securely manage passwords. Data is encrypted locally and stored in the cloud so it's available from anywhere. Unencrypted data never leaves the machine, so YOU are in total control.

**How do I use PassWeb?**
Click an entry's title to open its web site. Click the name/password field to copy (where supported) or select it for you to copy+paste. Click the padlock to generate a random, complex password for each site. Notes store additional info.

**How do I create a login?**
Contact the administrator with the user name you want and he/she will create a new account with a temporary password. Log in, change the master password to something only you know (and won't ever forget!), then create entries for all your accounts.

**What if I'm not online?**
Checking the "Cache encrypted passwords" box makes your data available offline. Changes are synchronized with the server next time you use PassWeb online. Simple updates merge seamlessly; overlapping updates should be avoided.

**What if I leave PassWeb open?**
It's okay: PassWeb logs you out after three minutes of inactivity to protect your data. Names and passwords unmasked for copy+paste are re-masked after ten seconds to prevent anyone nearby from reading them.

**Why shouldn't I use untrusted devices?**
Untrusted machines (like a library kiosk or a friend's laptop) may have malware installed that records keystrokes. Typing your master password on such a device would compromise it, allowing an attacker to use your PassWeb account.

**What if I forget the master password?**
Sorry, your data is irretrievably lost! PassWeb's encryption algorithm is government-grade and there aren't any backdoors or secondary passwords. It's up to you to remember the master password - and keep it secure!

**What browsers can I use?**
Because it's simple and standards-based, PassWeb works cross-platform on modern browsers like recent releases of Internet Explorer, Chrome, Firefox, and Safari. If you see a problem, please email me detailed steps to reproduce it.

**How was PassWeb developed?**
The client is built with HTML, CSS, and JavaScript and uses the [jQuery](http://jquery.com/), [Knockout](http://knockoutjs.com/), [crypto-js](http://code.google.com/p/crypto-js/), and [lz-string](http://pieroxy.net/blog/pages/lz-string/index.html) libraries. The server's REST API runs on ASP.NET or Node.js. Encryption is 256-bit AES in CBC mode. Hashing is SHA-512.


## Configuration

* The client for PassWeb is a simple HTML application and can be hosted on any web server or file server.
  * For [offline mode](http://en.wikipedia.org/wiki/Cache_manifest_in_HTML5) to work, the `offline.appcache` file must be served as type `text/cache-manifest` and should not be cached.
* The server for PassWeb is a simple [REST API](http://en.wikipedia.org/wiki/Representational_state_transfer) that stores and retrieves blobs of data.
  * Implementations are provided for both [ASP.NET](http://www.asp.net/) and [Node.js](http://nodejs.org/).
  * Data is stored as files under an `App_Data` directory, so the account used by the web server needs `modify` permissions for that location.
* Choose the ASP.NET or Node.js server based on your hosting options or background; the two implementations are equivalent.
  * The provided [`Web.config`](Web.config) file handles everything when hosted by [IIS on Windows](http://en.wikipedia.org/wiki/Internet_Information_Services).
  * Setting up the Node.js server requires familiarity with package management and some manual configuration.
  * The ASP.NET code has been exercised more extensively; a test suite helps ensure both implementations behave the same.
* With the default settings for the server, creation of new blobs is blocked to prevent unwanted users; the administrator should temporarily unblock when creating a login for a new user.
  * In the ASP.NET implementation, this is done by commenting-out the following line in `App_Code\RemoteStorage.cs`:

    ```
    // Remove to allow the creation of new files
    #define BLOCK_NEW
    ```

  * In the Node.js implementation, this is done by changing the following variable to false in `NodeJs\server.js`:

    ```
    // Set to block the creation of new files
    var BLOCK_NEW = true;
    ```


## Implementation

**Offline Use**
* The encrypted data file is read from and written to both the server API and [HTML local storage](http://en.wikipedia.org/wiki/Web_storage) after every change.
* When the server can't be reached (e.g., when offline), changes can be made locally.
* When the server is reachable during login, local and remote changes are synchronized and both locations are updated.
* Non-conflicting changes to different accounts merge seamlessly; conflicting edits to a single account are resolved by keeping the most recent entry.

**Account data**
* All account data is stored in a single, compressed, encrypted file.
* That file is encrypted using the [Advanced Encryption Standard (AES)](http://en.wikipedia.org/wiki/Advanced_Encryption_Standard) in [Cipher-Block Chaining mode (CBC)](http://en.wikipedia.org/wiki/Block_cipher_mode_of_operation) mode.
* The 256-bit encryption key is derived from the master password used to log into PassWeb.
* The file name is derived from the master user name and password by hashing them with [SHA-512](http://en.wikipedia.org/wiki/Secure_Hash_Algorithm).
* The data is only ever decrypted on the client; the server **never** sees the user name, password, or unencrypted data

**Storage API**
* The default configuration blocks creation of new files to prevent unwanted accounts.
* The default configuration keeps a backup of every version of the encrypted data file as a safety measure.

**Communication**
* Because it is always encrypted, the account data file can be transmitted via the storage API over an unsecure HTTP connection.
* Turning on [HTTPS](http://en.wikipedia.org/wiki/Https) for the storage API prevents tampering and hides the hash of the user name/password (this is enabled by default).
* Enabling HTTPS for the PassWeb client files prevents tampering (you need to enable this on the web server).
* Mixing HTTP/HTTPS requires the storage API support [Cross-Origin Resource Sharing (CORS)](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing) and requires an update to `offline.appcache`.


## Manifest

File | Purpose
-----|--------
default.htm <br/> default.js <br/> default.css | PassWeb implementation
offline.appcache | Offline cache manifest
jquery-2.1.4.min.js <br/> knockout-3.4.0.js <br/> lz-string.min.js <br/> aes.js <br/> sha512.js <br/> | External libraries
favicon.ico <br/> Resources\\\*.png <br/> Resources\\\*.svg <br/> | Image resources
App_Code\RemoteStorage.cs <br/> Web.config <br/> *App_Data\\PassWeb\\...* | ASP.NET server
NodeJs\\server.js <br/> NodeJs\\package.json <br/> *NodeJs\\App_Data\\...* | Node.js server
Readme.md | This file
LICENSE | License


## Ideas

* Add ability to import data from other password managers
* Convert to the [Web Cryptography API](http://www.w3.org/TR/WebCryptoAPI/) (when widely available)
* Translate user interface into other languages


## References

* [Secure your passwords (Google)](https://www.google.com/intl/en_US/goodtoknow/online-safety/passwords/)
* [A guide to staying safe and secure online (Google)](https://www.google.com/intl/en_US/goodtoknow/)
* [Protect your passwords (Microsoft)](http://www.microsoft.com/security/pc-security/protect-passwords.aspx)
* [Create strong passwords (Microsoft)](https://www.microsoft.com/security/pc-security/password-checker.aspx)
* [Safety & Security Center (Microsoft)](http://www.microsoft.com/security/default.aspx)
* [SDL - Process Guidance - Phase 2: Design](http://msdn.microsoft.com/en-us/library/windows/desktop/cc307414.aspx)
* [Banned Crypto and the SDL](http://blogs.msdn.com/b/sdl/archive/2009/07/16/banned-crypto-and-the-sdl.aspx)


## License

[MIT](LICENSE)
