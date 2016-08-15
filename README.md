# Live Localizer

Live Localizer widget for Polymer [i18n-behavior](https://github.com/t2ym/i18n-behavior) (work in progress)

[API Docs](https://t2ym.github.io/live-localizer/components/live-localizer/) and [Live Demo](https://t2ym.github.io/live-localizer/components/live-localizer/demo/) on GitHub Pages (Chrome recommended for now)

## Live Localizer applied to [the Shop App](https://www.polymer-project.org/1.0/toolbox/case-study)

<img src="https://raw.githubusercontent.com/wiki/t2ym/live-localizer/live-localizer.gif" width="768px">

Sequential description of the animated screenshot above:

- Browse the Shop App with Live Localizer widget at the bottom left corner as a fab icon
- Open the Live Localizer widget
- Switch to the Japanese locale with pseudo-L10N
- Save the working strings of the Shop App as a local XLIFF file `bundle.ja.xlf`
- Open the local XLIFF with XLIFF editor [Virtaal](https://github.com/translate/virtaal) 
- Translate the local XLIFF (Actually, open the translated XLIFF `shop-bundle.ja.xlf`)
- Drag and drop the translated XLIFF from Windows Explorer to Live Localizer widget
- Load the dropped XLIFF to the running Shop App
- Browse the localized Shop App
- Switch to the English locale

## Use Cases

- Dispatch translation tasks via the running Web app itself
- Dispatch incremental translation tasks via the running Web app itself
- Check screenshots on the running Web app
- Check translation progress in statistics view
- Submit translations to Cloud (Firebase)
- Watch Cloud to update translations in real-time

## Features

- Export XLIFF from the running app to a local file
- Import XLIFF from a local file to the running app
- Save XLIFF to Browser Storage (IndexedDB)
- Load XLIFF from Browser Storage (IndexedDB)
- Upload XLIFF to Cloud Storate (Firebase)
- Download XLIFF from Cloud Storage (Firebase)
- Show translation statistics in list view
- Detect changes on Cloud Storage (Firebase) to trigger a new build
- Detect changes and load Local XLIFF 

## Install

```sh
  bower install --save live-localizer
```

## Import

```html
  <link rel="import" href="path/to/bower_components/live-localizer.html">
```

## Apply

Attach at the end of the main body element.

### With Firebase Cloud Storage:

```html
<body>
  ...
  <live-localizer>
    <!--
      Firebase Cloud Storage configuration with the example parameters for Live Localizer demo app:
      For each target app, a dedicated Firebase project has to be configured.
    -->
    <live-localizer-firebase-storage id="firebase-storage" class="storage cloud"
      auth-provider="google"
      auth-domain="live-localizer-demo.firebaseapp.com"
      database-url="https://live-localizer-demo.firebaseio.com"
      api-key="AIzaSyCjrlPhl0cLSZVRsDvuajq16vkerhcu_UM">
    </live-localizer-firebase-storage>
  </live-localizer>
</body>
```

### Without Firebase Cloud Storage:

```html
<body>
  ...
  <live-localizer></live-localizer>
</body>
```

## Firebase Setup

Firebase project has to be set up for storing XLIFF.

### Supported Authentication Methods

- Anonymous
- Google (OAuth)
- GitHub (OAuth)
- Twitter (OAuth)

### Firebase Security Rules

```javascript
{
  "rules": {
    "users": {
      ".read": "auth.uid === 'xliff-watcher'",
      "$uid": {
        ".read": "$uid === auth.uid",
        ".write": "$uid === auth.uid"
      }
    }
  }
}
```

### XLIFF Watcher (Optional)

See `gulp fetch-xliff` and `gulp watch-xliff` tasks in `demo/gulpfile.js`

#### Notes on XLIFF Watcher:

- `firebase login:ci` token from `firebase-tools` is required for `gulp fetch-xliff` task
- Service account credentials' JSON is required for `gulp watch-xliff` task

## Local XLIFF Watcher (Optional)

Local XLIFF changes can be watched via local HTTP server at http://localhost:8887/UPLOADED_XLIFF_FILE

Check "Watch and Load XLIFF" in the local file storage control panel to start watching the local XLIFF file

### HTTP Server for Local XLIFF Watcher Setup

- Install [NodeJS](https://nodejs.org/) on the local host of the translator
- Install [the local HTTP server npm package](https://www.npmjs.com/package/http-server)
```
npm install -g http-server
```
- For HTTP sites, start local HTTP server at http://localhost:8887 with the root folder containing the XLIFF file
```
http-server FOLDER_TO_CONTAIN_XLIFF -d false -c-1 -r -a localhost -p 8887 --cors=If-Modified-Since
```
- For HTTPS sites, start local HTTPS server at https://localhost:8887 with the root folder containing the XLIFF file

Batch File to Launch local HTTPS server on Windows: [`https-server.bat`](https://github.com/t2ym/live-localizer/blob/master/http-local-server.zip?raw=true) FOLDER_TO_CONTAIN_XLIFF
```
@echo off
if "%1"=="" echo Please specify the path to the root folder containing the target XLIFF file
if "%1"=="" echo %0 C:\Path\To\XLIFF\Folder
if "%1"=="" goto :end
if exist C:\OpenSSL-Win64 set OPENSSL_CONF=C:\OpenSSL-Win64\bin\openssl.cfg
if exist C:\OpenSSL-Win64 set PATH=%PATH%;C:\OpenSSL-Win64\bin
if exist C:\OpenSSL-Win32 set OPENSSL_CONF=C:\OpenSSL-Win32\bin\openssl.cfg
if exist C:\OpenSSL-Win32 set PATH=%PATH%;C:\OpenSSL-Win32\bin
if "%OPENSSL_CONF%"=="" echo Please install OpenSSL package for Windows from https://slproweb.com/products/Win32OpenSSL.html linked from https://wiki.openssl.org/index.php/Binaries
if "%OPENSSL_CONF%"=="" goto :end
if not exist demoCA mkdir demoCA
cd demoCA
if not exist newcerts mkdir newcerts
type nul >index.txt
echo 01 >serial
echo y >yy
echo y >>yy
if not exist localhostCA.key openssl genrsa 2048 >localhostCA.key
if not exist localhostCA.csr openssl req -new -key localhostCA.key -subj "/C=JP/ST=Tokyo/O=i18n-behavior/OU=Live Localizer/CN=Live Localizer Localhost CA" -out localhostCA.csr
del /Q CAcreation
if not exist localhostCA.crt type nul >CAcreation
if not exist localhostCA.crt openssl x509 -days 3650 -sha256 -req -signkey localhostCA.key -in localhostCA.csr -out localhostCA.crt
if not exist localhost.key openssl genrsa 2048 >localhost.key
if not exist localhost.csr openssl req -new -key localhost.key -subj "/C=JP/ST=Tokyo/O=i18n-behavior/OU=Live Localizer/CN=localhost" -out localhost.csr
cd ..
if not exist demoCA\localhost.crt openssl ca -md sha256 -days 3650 -cert demoCA\localhostCA.crt -keyfile demoCA\localhostCA.key -in demoCA\localhost.csr -out demoCA\localhost.crt <demoCA\yy
if exist demoCA\CAcreation echo Please install the generated Localhost CA certificate as "Trusted Root Certification Authorities"
if exist demoCA\CAcreation demoCA\localhostCA.crt
http-server "%1" -d false -c-1 -r -a localhost -p 8887 --cors=If-Modified-Since --ssl --cert demoCA\localhost.crt --key demoCA\localhost.key
:end
```

#### Notes:
- The XLIFF folder should contain only the target XLIFF file(s) for the project for security.
- The HTTP server is accessible only from the localhost and disallows directory listing.
- If the XLIFF file name is prefixed with an unpredictable string, it can serve as a kind of "password" to block malicious access from other local HTTP clients.

## TODOs

- Code cleanup/optimization/refactoring
- Automated tests
- Documentation
- Fix issues

## Plans

TBD

## License

[BSD-2-Clause](https://github.com/t2ym/live-localizer/blob/master/LICENSE.md)
