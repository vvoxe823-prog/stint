# Business Document Backup - Final Android Build

## Exact behavior

This build is designed around a one-time setup.

### First installation
1. Open the app.
2. Enter the business/user phone number.
3. Tap **Select WhatsApp Document Folder** and choose the folder where WhatsApp downloads/saves the documents you want backed up.
4. Tap **CONNECT** once.
5. The phone number, selected folder URI/name, device ID, and setup-completed flag are stored in the app's private local storage.
6. The automatic foreground scanning service starts.

### After CONNECT
The setup form is hidden. The user does **not** need to:
- enter the phone number again;
- select the WhatsApp folder again;
- connect again for every WhatsApp download;
- press upload for every document.

The service scans the selected folder approximately every 5 seconds. It recursively checks supported files, waits for a new file to remain stable across scans, calculates SHA-256, and uploads the new file to:

`https://stint.gt.tc/upload_document.php`

Every upload sends:
- `phone_number`
- `device_id`
- `original_file_name`
- `file_size`
- `mime_type`
- `file_hash`
- `local_modified_time`
- `file`

Existing files present during the first baseline scan are not uploaded. Only files detected after setup/baseline are candidates for automatic upload.

## Important scope

The app does not read WhatsApp chats, messages, databases, or notifications. It only reads the folder selected by the user through Android's Storage Access Framework.

The folder access grant is persisted by Android so the app can continue accessing the selected directory across app/device restarts, subject to Android/provider rules. If the folder is moved/deleted or its provider revokes access, the app may require the folder to be selected again.

The phone number and folder URI remain in the app's private storage until the app is uninstalled or its app data is explicitly cleared.

The foreground service is approximately 5 seconds, not a hard real-time guarantee. Android can stop/restrict apps, and force-stopping the app stops the service. A boot receiver restarts the service after a normal device reboot when setup is already complete.

## Build APK in Android Studio

Requirements:
- Android Studio
- JDK 17
- Android SDK Platform 35

1. Extract this ZIP.
2. Open the `BusinessDocumentBackup` folder in Android Studio.
3. Allow Gradle to sync and download required Android/Gradle dependencies.
4. Connect an Android phone with USB debugging enabled, or use an emulator.
5. For a debug APK: **Build > Generate App Bundles or APKs > Generate APKs**.
6. The debug APK will normally be under:
   `app/build/outputs/apk/debug/app-debug.apk`

For a distributable signed release APK, use **Build > Generate Signed Bundle / APK** and create/select a keystore.

## Test checklist

1. Install the app on a fresh device.
2. Confirm the setup screen appears.
3. Enter phone number and select the WhatsApp document folder.
4. Tap CONNECT.
5. Close the app UI; the foreground service notification should remain.
6. Download a new PDF/JPG/PNG/DOC/DOCX/XLS/XLSX file into the selected folder.
7. Wait for the automatic scan/upload.
8. Verify the PHP endpoint receives the phone number and device ID with the file.
9. Restart the phone and verify the configured service starts again.
10. Uninstall/reinstall the app and confirm first-time setup appears again.

## Build online without Android Studio (Codemagic)

This project includes `codemagic.yaml`. It is configured to build the debug APK in Codemagic without requiring Android Studio on your computer. The workflow installs Gradle 8.13 on the build machine, sets the Android SDK location, and runs `assembleDebug`.

1. Create/sign in to a Codemagic account.
2. Create a GitHub repository and upload the contents of this `BusinessDocumentBackup` folder to the repository root. Do not upload the outer ZIP itself as the repository contents.
3. In Codemagic choose **Add application**, connect GitHub, select the repository, and select the native Android project.
4. Make sure `codemagic.yaml` is detected at the repository root.
5. Start the `android-debug-apk` workflow.
6. When the build succeeds, open the build's artifacts and download the APK.

The APK artifact is expected at:
`app/build/outputs/apk/debug/app-debug.apk`

This is a debug APK and can be installed directly on an Android phone after allowing installation of apps from the relevant source. For Google Play distribution, create a signed release build and signing key separately.
