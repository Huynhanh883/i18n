# crashReporter

> Soumet un rapport de plantage à un serveur distant.

Processus : [Main](../glossary.md#main-process), [Renderer](../glossary.md#renderer-process)

Voici un exemple d'envoi automatique d'un rapport de plantage à un serveur distant :

```javascript
const { crashReporter } = require('electron')

crashReporter.start({
  productName: 'YourName',
  companyName: 'YourCompany',
  submitURL: 'https://your-domain.com/url-to-submit',
  uploadToServer: true
})
```

For setting up a server to accept and process crash reports, you can use following projects:

* [socorro](https://github.com/mozilla/socorro)
* [mini-breakpad-server](https://github.com/electron/mini-breakpad-server)

Or use a 3rd party hosted solution:

* [Backtrace](https://backtrace.io/electron/)
* [Sentry](https://docs.sentry.io/clients/electron)
* [BugSplat](https://www.bugsplat.com/docs/platforms/electron)

Crash reports are saved locally in an application-specific temp directory folder. For a `productName` of `YourName`, crash reports will be stored in a folder named `YourName Crashes` inside the temp directory. You can customize this temp directory location for your app by calling the `app.setPath('temp', '/my/custom/temp')` API before starting the crash reporter.

## Méthodes

Le module `crashReporter` dispose des méthodes suivantes :

### `crashReporter.start(options)`

* `options` Objet 
  * `companyName` String
  * `submitURL` String - URL that crash reports will be sent to as POST.
  * `productName` String (optional) - Defaults to `app.name`.
  * `uploadToServer` Boolean (optional) - Whether crash reports should be sent to the server. Default is `true`.
  * `ignoreSystemCrashHandler` Boolean (optional) - Default is `false`.
  * `extra` Record<String, String> (optional) - An object you can define that will be sent along with the report. Only string properties are sent correctly. Nested objects are not supported. When using Windows, the property names and values must be fewer than 64 characters.
  * `crashesDirectory` String (optional) - Directory to store the crash reports temporarily (only used when the crash reporter is started via `process.crashReporter.start`).

You are required to call this method before using any other `crashReporter` APIs and in each process (main/renderer) from which you want to collect crash reports. You can pass different options to `crashReporter.start` when calling from different processes.

**Note** Child processes created via the `child_process` module will not have access to the Electron modules. Therefore, to collect crash reports from them, use `process.crashReporter.start` instead. Pass the same options as above along with an additional one called `crashesDirectory` that should point to a directory to store the crash reports temporarily. You can test this out by calling `process.crash()` to crash the child process.

**Note:** If you need send additional/updated `extra` parameters after your first call `start` you can call `addExtraParameter` on macOS or call `start` again with the new/updated `extra` parameters on Linux and Windows.

**Note:** On macOS and windows, Electron uses a new `crashpad` client for crash collection and reporting. If you want to enable crash reporting, initializing `crashpad` from the main process using `crashReporter.start` is required regardless of which process you want to collect crashes from. Once initialized this way, the crashpad handler collects crashes from all processes. You still have to call `crashReporter.start` from the renderer or child process, otherwise crashes from them will get reported without `companyName`, `productName` or any of the `extra` information.

### `crashReporter.getLastCrashReport()`

Retourne [`CrashReport`](structures/crash-report.md) :

Renvoi la date et l'identifiant du dernier crash. Seuls les rapports de plantages qui ont étés téléchargés seront renvoyés ; même si un rapport de plantages est présent sur le disque, il ne sera pas renvoyé avant qu'il soit téléchargé. Dans le cas où il n'y a pas de rapports téléchargés, `null` est retourné.

### `crashReporter.getUploadedReports()`

Retourne [`CrashReport[]`](structures/crash-report.md) :

Returns all uploaded crash reports. Each report contains the date and uploaded ID.

### `crashReporter.getUploadToServer()`

Returns `Boolean` - Whether reports should be submitted to the server. Set through the `start` method or `setUploadToServer`.

**Note:** This API can only be called from the main process.

### `crashReporter.setUploadToServer(uploadToServer)`

* `uploadToServer` Boolean *macOS* - Whether reports should be submitted to the server.

This would normally be controlled by user preferences. This has no effect if called before `start` is called.

**Note:** This API can only be called from the main process.

### `crashReporter.addExtraParameter(key, value)` *macOS* *Windows*

* `key` String - Parameter key, must be less than 64 characters long.
* `value` String - Parameter value, must be less than 64 characters long.

Set an extra parameter to be sent with the crash report. The values specified here will be sent in addition to any values set via the `extra` option when `start` was called. This API is only available on macOS and windows, if you need to add/update extra parameters on Linux after your first call to `start` you can call `start` again with the updated `extra` options.

### `crashReporter.removeExtraParameter(key)` *macOS* *Windows*

* `key` String - Parameter key, must be less than 64 characters long.

Remove a extra parameter from the current set of parameters so that it will not be sent with the crash report.

### `crashReporter.getParameters()`

See all of the current parameters being passed to the crash reporter.

## Payload du Crash Report

Le rapporteur de plantage enverra les données suivantes à `submitURL` comme un `POST` en `multipart/form-data` :

* `ver` String - La version d'Electron.
* `platform` String - Par exemple 'win32'.
* `process_type` String - Par exemple 'renderer'.
* `guid` String - Par exemple '5e1286fc-da97-479e-918b-6bfb0c3d1c72'.
* `_version` String - La version dans `package.json`.
* `_productName` String - Le nom du produit dans l'objet `options` de `crashReporter`.
* `prod` String - Nom du produit sous-jacent. Dans ce cas Electron.
* `_companyName` String - Le nom de l'entreprise dans l'objet `options` de `crashReporter`.
* `upload_file_minidump` File - Le rapport d'incident dans le format `minidump`.
* Toutes les propriétés de niveau 1 de l'objet `extra` dans l'objet `options` de `crashReporter`.