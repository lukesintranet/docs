# Authentication with Luke's Intranet
## Account-Auth (Auth 1)
This system only works with access to the database, and therefore only with Luke's Intranet applications. For older internal applications and some Luke's Intranet local services, this is the only method of authentication. AuthPage has been added as an additional method for authentication for most of these services.

This is the basis for all futher authentication methods, and works by hashing your password, and only storing the salt and hash for additional security. Luke's Intranet acount passwords are encrypted with modern methods to ensure complete security, safeguarding your account.
## AuthPage (Auth v2)
To use the legacy system, you must create a script that is accessible to the client. It will take in  the following arguments:
```
username: <their Luke's Intranet username>
password: <their Luke's Intranet app-password*>
```

\* Note on passwords: the same password is used for every website on Auth v2 (this is fixed in Auth v3)

Your website may be a local application as long as your user may access it, and other uses for Auth v2 are not reccomended, due to lower security than Auth v3.

Redirect your user to the following AuthPage URL to begin the authentication process for youir website:

```
https://account.lukesintranet.com/authpage/?returnurl=<Your authentication script's URL>&title=<App name>&message=<Message displayed to user>
```

When they login, create, or select an account, and proceed to the next step, they are redirected to the Luke's Intranet Auth Page API to verify their username and password.

The user is then redirtected to the Return URL (that of your script), and your sacript will deal with the user further.

The username and password (of the user's account) are stored in plaintext in local storage (user storage) for future use.

Newer local and web applications on Luke's Intranet only use this method due to its simplicity to implement, and security when being used for internal purposes. Internal applications are given the nornal account password to verify with the database.

## Popup Auth (Auth v3)
### Popup Auth v1 (Current method)

This system works by generating a popup on the local website. Your app has public tokens and data tokens, to collect neccesary data. This increases security because it helps prevent phising attacks. A secret token is generated by Luke's Intranet, and not stored on Luke's Intranet, and sent to the client and server. Authentication is handled by AuthPage on the popup, but a new unique password is generated for each site, ensuring higher levels of security.

First, your app will need to make a popup to this website:
```
https://account.lukesintranet.com/popup-auth/v1/?aptk=<your APTK (application public token)>&adtk=<ADTK (application data token)>
```

Both tokens can be obtained at the [Luke's Intranet Developer Website](https://developer.lukesintranet.com/get-tokens/index.php)


Code to generate the popup is shown below:

(Code written in JavaScript)
```
var popupWin;

function popup() {
    popupWin = window.open ("https://account.lukesintranet.com/popup-auth/v1/?aptk=lukesintranet_test&adtk=lukesintranet_test","Login with Luke's Intranet", "width=800,height=800");
}

let messages_recived = 0;
let edata = "";
let success = false;

// Create IE + others compatible event handler
var eventMethod = window.addEventListener ? "addEventListener" : "attachEvent";
var eventer = window[eventMethod];
var messageEvent = eventMethod == "attachEvent" ? "onmessage" : "message";

// Listen to message from child window
eventer(messageEvent,function(e) {
    console.log('origin: ', e.origin)
	  
    // Check if origin is proper
    

    console.log('parent received message!: ', e.data);
    messages_recived++;
    if (messages_recived >= 2) {
        edata = e.data;
        popupWin.close();
        messages_recived = 1;
        success = true;
    }
    if (success == true) {
        let parsed_response = JSON.parse(edata);
        document.write("Authentication successful! Username: " + parsed_response.username + "" + " Secret: '" + parsed_response.secret + "'" + ' Reauthenticate: <button onclick="location.reload()">click me</button>');
    }
}, false);
```

This code listens for a message sent by the Luke's Intranet Account website, which sends a message back to the main website (all front-end)

The username and authentication secret are sent to the client. The client will need to send the server the secret token to authenticate. In the above demo, the code simply outputs it to the screen.

You can get the "Login with LiNET" button here: https://account.lukesintranet.com/LoginButton.png

This can be implemented into your website without permission.
