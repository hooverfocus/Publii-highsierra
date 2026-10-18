# Publii 0.46.5 for macOS High Sierra (unofficial build)

An unofficial build of [Publii](https://getpublii.com) 0.46.5 that runs on
**macOS 10.13.6 (High Sierra), Intel Mac**. It is based on release 0.46.5.
Newer Publii versions exist and may have fixed issues. It also uses older 
components that may lack security fixes. Install and use at your own risk, 
and keep backups of your sites. Publii is made by TidyCustoms. This
project is not affiliated with them. Licence: GPL-3.0, same as Publii.

The original project and its README for this version:
<https://github.com/GetPublii/Publii/blob/v.0.46.5-build-17089/README.md>

The app shows "High Sierra" after the version number on the opening screen and
in the About section, so you can tell it apart from the official build.

## Download

Get `Publii-0.46-HS.zip` from the **Releases** page and unzip it. A disk image
(.dmg) is also available.
The app is not signed. The first time, right-click it and choose **Open**.

## What was tested

macOS 10.13.6, Node 22.2.0 (custom build, see "Required software" below),
Electron 26.3.0. 

Tested: Block and WYSIWYG editors, saving, featured images, images (jpg),
galleries, preview, backups, SFTP connection test and a real sync with a
password login.

## What was changed from the official 0.46.5

The full list of changed lines is on the
[compare page](https://github.com/hooverfocus/Publii-highsierra/compare/v.0.46.5-build-17089...highsierra-0.46.5).

1. **`ssh2` native speed-up file removed.** It crashed SFTP on High Sierra.
   SFTP uses the built-in slower fallback. The copy shipped with Publii 0.44
   (`ssh2` 1.14.0) did not crash, so this may depend on the `ssh2` version.
   Untested: SFTP with an SSH key instead of a password.
2. **`node-sass` 9.0.0 replaced by `sass` 1.77.8.** It only builds the editor
   stylesheets while building the app, so it should not change how the app
   behaves.
3. **`sharp` (images) set to 0.32.6**, the version used by Publii 0.44. The
   official 0.34.1 does not load on High Sierra. Publii only uses basic
   resizing and JPEG/WebP conversion, so little should be lost, but the two
   versions were not compared in detail.
4. **SFTP connection test (`sftp.js`)** lists your own starting folder
   instead of `/`. Some hosts refuse to list `/`. The same problem exists in
   Publii 0.44.
5. **Electron 26.3.0**, the version that was tested. Later 26.x releases may
   also work.
6. **The packager reuses the existing native modules** instead of recompiling
   them (`npmRebuild: false`). This is why the database module has to be
   fetched for Electron by hand in the build process below.
7. **Typo fixed** in the Settings sort option for post title
   (`settings.postTile` was a typo in the official code).
8. **"High Sierra" label** added to the version text on the opening screen
   and in the About section.

## Required software

For the app build you will need:

* Node.js 22.2.0 for High Sierra (the official wiki asks for 20.*, but this
  was tested with 22.2.0). Install it with this guide:
  <https://tech.amikelive.com/node-1687/how-to-install-node-js-22-on-macos-high-sierra/>
* npm (>= 10.*), the one that comes with that Node
* Python (>= 2.5.0 && < 3.12.0). Tested with 3.11.7
* Xcode command line tools

No global node.js modules were installed for this build. Everything it uses is
installed by `npm install`.

## Build process

1. In the root project directory run:

```
npm install
cd app
npm install
cd ..
```

2. Fetch the database module built for Electron 26. This is the same fix the
   official wiki gives for
   [improper node module versions](https://github.com/GetPublii/Publii/wiki/App-build-process#keytar-sharp-and-node-expat-node-modules-compilation),
   but it is needed every time here: `npm install` fetches the module for your
   Node 22, and the app runs it inside Electron 26, which refuses it (the default
   download is built for your Node version and fails inside the app). Run:

```
cd app/node_modules/better-sqlite3
../.bin/prebuild-install --runtime=electron --target=26.3.0 --arch=x64 --platform=darwin
cd ../../..
```

3. Remove the `ssh2` speed-up file, if npm built one:

```
rm -rf app/node_modules/ssh2/lib/protocol/crypto/build
```

4. Compile the front-end files once (the official wiki uses `npm run dev`,
   which keeps running; `npm run prod` compiles and stops):

```
npm run prod
```

5. When the files are compiled run:

```
npm run prepare-editor
```

6. Now you can run the Publii app from the folder to test it:

```
npm run start
```

If the database module download in step 2 is not available, the official wiki
suggests compiling it. Please refer to [improper node module versions](https://github.com/GetPublii/Publii/wiki/App-build-process#keytar-sharp-and-node-expat-node-modules-compilation). This was not tried on High Sierra.

## Creating the app

Before packaging, make sure the front-end files are up to date by running
`npm run prod` (step 4 above). Then, in the root directory:

```
CSC_IDENTITY_AUTO_DISCOVERY=false npm run packager:mac-intel -- -c.electronDist=node_modules/electron/dist
```

This is the project's own packaging command. It skips code signing and uses
the Electron you already installed. It creates an unsigned `Publii.app` in
`dist/mac`, plus `Publii-0.46.5.dmg` and `Publii-0.46.5-mac.zip` in `dist`.

A lighter alternative that only makes the `.app` folder:

```
CSC_IDENTITY_AUTO_DISCOVERY=false ./node_modules/.bin/electron-builder --mac --dir -c.electronDist=node_modules/electron/dist
```

In this case the app appears in `dist/mac/`.

## Not tested

- SFTP login with an SSH key
- Image features that may exist only in `sharp` 0.34
- Electron releases after 26.3.0

## Known issues

- None known beyond the list above.

## Screenshot

Launch screen shows Publii 0.46 patched to run on OSX 10.12.6 High Sierra
<img width="1000" height="563" alt="Publii-0 46-HS_screenshot" src="https://github.com/user-attachments/assets/1b872d8d-5020-4f0a-abc1-7b3d2216a39d" />
