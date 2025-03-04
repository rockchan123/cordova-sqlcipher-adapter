# Cordova/PhoneGap SQLCipher adapter plugin - maintenance only

Native interface to **SQLCipher version 4** in a Cordova/PhoneGap plugin for Android, iOS, macOS, ~~and TBD Windows,~~ with API similar to HTML5/[Web SQL API](http://www.w3.org/TR/webdatabase/). Plugin version `0.2.x` is required for SQLCipher 3 support. FUTURE TODO: support migration between SQLCipher 3 and SQLCipher 4 ([brodybits/cordova-sqlcipher-adapter#83](https://github.com/brodybits/cordova-sqlcipher-adapter/issues/83)). Note that this project is currently not under active development ([brodybits/cordova-sqlcipher-adapter#81](https://github.com/brodybits/cordova-sqlcipher-adapter/issues/81)).

License terms for Android and DISABLED (unsupported) Windows platform versions: MIT or Apache 2.0

License terms for iOS/macOS platform version: MIT only

<!-- END About this plugin version branch -->

## IMPORTANT WARNING NOTICES

**WARNING:** Without the database password there is no proven way to recover the data.

**IMPORTANT EXPORT REQUIREMENTS** described at: <https://discuss.zetetic.net/t/export-requirements-for-applications-using-sqlcipher/47>

This project is under maintenance for security, data loss risk, and other critical issues at this point ([brodybits/cordova-sqlcipher-adapter#81](https://github.com/brodybits/cordova-sqlcipher-adapter/issues/81)). Active development may be resumed someday in the future, in case of sufficient interest from the user community. For priority feature requirements please contact <sales@litehelpers.net> for estimation and discussion.

### Multiple database problem on Android

__Multiple database access problem on Android__

This plugin uses SQLCipher for Android which is a non-standard SQLite implementation on Android. In case an application access the SAME database using multiple plugins (with or without encryption) there is a risk of data corruption ref: [litehelpers/Cordova-sqlite-storage#626](https://github.com/litehelpers/Cordova-sqlite-storage/issues/626)) as described in <http://ericsink.com/entries/multiple_sqlite_problem.html> and <https://www.sqlite.org/howtocorrupt.html>.

### Multiple database access problem on other platforms

This plugin version also uses SQLCipher which is based on a particular version of sqlite3 on iOS, macOS, and Windows. In case the application accesses the SAME database using multiple plugins there is a risk of data corruption as described in <https://www.sqlite.org/howtocorrupt.html> (similar to the multiple sqlite problem for Android as described in <http://ericsink.com/entries/multiple_sqlite_problem.html>).

### Additional notice

Windows platform support is now disabled in this plugin version, with CRYPTO provider (libTomCrypt) completely removed. This plugin version is no longer tested on Windows. For future consideration: enable Windows build again with encryption using a recent build of the OpenSSL crypto library

<!-- END WARNING NOTICES -->

## Services available

The primary author and maintainer [@brodybits (Christopher J. Brody aka Chris Brody)](https://github.com/brodybits) is available for part-time contract assignments. Services available for this project include:

- Priority issue support
- Help with application code such as debugging, optimization, etc.
- Warranty and support retainers
- Priority fixes and enhancements
- Custom feature development

Other services available include:

- Front-end/back-end development
- Mentoring and training services

For more information:
- <http://litehelpers.net/>
- <sales@litehelpers.net>

<!-- END Services available -->

## A quick tour

To open a database:

```Javascript
var db = null;

document.addEventListener('deviceready', function() {
  db = window.sqlitePlugin.openDatabase({
    name: 'my-encrypted.db',
    key: 'user-password-here',
    location: 'default'
  });
});
```

**IMPORTANT:** Like with the other Cordova plugins your application must wait for the `deviceready` event. This is especially tricky in Angular/ngCordova/Ionic controller/factory/service callbacks which may be triggered before the `deviceready` event is fired.

### Using DRAFT standard transaction API

To populate a database using the DRAFT standard transaction API:

```Javascript
  db.transaction(function(tx) {
    tx.executeSql('CREATE TABLE IF NOT EXISTS DemoTable (name, score)');
    tx.executeSql('INSERT INTO DemoTable VALUES (?,?)', ['Alice', 101]);
    tx.executeSql('INSERT INTO DemoTable VALUES (?,?)', ['Betty', 202]);
  }, function(error) {
    console.log('Transaction ERROR: ' + error.message);
  }, function() {
    console.log('Populated database OK');
  });
```

To check the data using the "standard" (deprecated) transaction API:

```Javascript
  db.transaction(function(tx) {
    tx.executeSql('SELECT count(*) AS mycount FROM DemoTable', [], function(tx, rs) {
      console.log('Record count (expected to be 2): ' + rs.rows.item(0).mycount);
    }, function(tx, error) {
      console.log('SELECT error: ' + error.message);
    });
  });
```

### Using recommended API calls

To populate a database using the SQL batch API:

```Javascript
  db.sqlBatch([
    'CREATE TABLE IF NOT EXISTS DemoTable (name, score)',
    [ 'INSERT INTO DemoTable VALUES (?,?)', ['Alice', 101] ],
    [ 'INSERT INTO DemoTable VALUES (?,?)', ['Betty', 202] ],
  ], function() {
    console.log('Populated database OK');
  }, function(error) {
    console.log('SQL batch ERROR: ' + error.message);
  });
```

To check the data using the single SQL statement API:

```Javascript
  db.executeSql('SELECT count(*) AS mycount FROM DemoTable', [], function(rs) {
    console.log('Record count (expected to be 2): ' + rs.rows.item(0).mycount);
  }, function(error) {
    console.log('SELECT SQL statement ERROR: ' + error.message);
  });
```

See the [Sample section](#sample) for a sample with a more detailed explanation (using the DRAFT standard transaction API).

<!-- END quick tour -->

## Status

- Windows platform support is now disabled in this plugin version, with CRYPTO provider (libTomCrypt) completely removed (ref: [litehelpers / Cordova-sqlcipher-adapter#63](https://github.com/litehelpers/Cordova-sqlcipher-adapter/issues/63)). For future consideration: enable Windows build again with encryption using a recent build of the OpenSSL crypto library ref: [litehelpers/Cordova-sqlcipher-adapter#30](https://github.com/litehelpers/Cordova-sqlcipher-adapter/issues/30)
- Alpha version:
  - SQLCipher `4.0.1` (major update) for iOS/macOS
  - SQLCipher `4.0.1` for Android, in custom build from <https://github.com/brodybits/android-database-sqlcipher/tree/4.0.x%2Bcustom-jar-build> (`4.0.x+custom-jar-build` branch)
  - with OpenSSL libcrypto for Android
  - using CommonCrypto framework for iOS/macOS
  - NO ENCRYPTION ENABLED (completely removed) for Windows
  - for future consideration: embed OpenSSL libcrypto for all target platforms
- This plugin is NOT supported by PhoneGap Developer App or PhoneGap Desktop App.
- A recent version of the Cordova CLI (such as `6.5.0` / `7.1.0` / `8.1.x`) is recommended. Cordova versions older than `6.0.0` are missing the `cordova-ios@4.0.0` security fixes. In addition it may be needed to use `cordova prepare` in case of cordova-ios older than `4.3.0` (Cordova CLI `6.4.0`).
- SQLCipher build settings used:
  - `SQLITE_HAS_CODEC` (no longer enabled in Windows SQLite3 library build)
  - `SQLITE_SOUNDEX` (Android only)
  - `SQLITE_MAX_VARIABLE_NUMBER=99999` (Android only)
  - `SQLITE_DEFAULT_JOURNAL_SIZE_LIMIT=1048576` (Android only)
  - `HAVE_USLEEP=1`
  - `SQLITE_TEMP_STORE=3`
  - `SQLCIPHER_CRYPTO_CC` (iOS/macOS only)
  - `SQLITE_LOCKING_STYLE=1` (iOS/macOS only)
  - `DSQLITE_DEFAULT_JOURNAL_SIZE_LIMIT=1048576` (Android only)
  - `NDEBUG` (`NDEBUG=1` on Android)
  - `SQLITE_THREADSAFE=1`
  - `SQLITE_DEFAULT_SYNCHRONOUS=3` (Android/iOS/macOS)
  - `SQLITE_ENABLE_MEMORY_MANAGEMENT=1` (Android only)
  - `SQLITE_DEFAULT_MEMSTATUS=0`
  - `SQLITE_OMIT_DECLTYPE` (iOS/macOS/Windows)
  - `SQLITE_OMIT_DEPRECATED` - iOS/macOS (FUTURE TBD: Android ref: [brodybits/cordova-sqlcipher-adapter#82](https://github.com/brodybits/cordova-sqlcipher-adapter/issues/82))
  - `SQLITE_OMIT_PROGRESS_CALLBACK` (iOS/macOS/Windows)
  - `SQLITE_OMIT_SHARED_CACHE`
  - `SQLITE_ENABLE_LOAD_EXTENSION` (Android only)
  - `SQLITE_OMIT_LOAD_EXTENSION` (iOS/macOS/Windows)
  - `SQLITE_ENABLE_COLUMN_METADATA` (Android only)
  - `SQLITE_ENABLE_UNLOCK_NOTIFY` (Android only)
  - `SQLITE_ENABLE_FTS3` (iOS/macOS/Windows)
  - `SQLITE_ENABLE_FTS3_PARENTHESIS`
  - `SQLITE_ENABLE_FTS4`
  - `SQLITE_ENABLE_RTREE`
  - `SQLITE_ENABLE_STAT3` for Android only
  - `SQLITE_ENABLE_STAT4` for Android only
  - `SQLITE_ENABLE_FTS5`
  - `SQLITE_ENABLE_JSON1`
  - `SQLITE_OS_WINRT` (Windows only)
  - `SQLCIPHER_CRYPTO_OPENSSL` (Android only)
- The iOS database location is now mandatory, as documented below.
- Amazon Fire-OS is dropped due to lack of support by Cordova. Android version should be used to deploy to Fire-OS 5.0(+) devices. For reference: [cordova/cordova-discuss#32 (comment)](https://github.com/cordova/cordova-discuss/issues/32#issuecomment-167021676)
- Windows platform version (using a customized version of the performant [doo / SQLite3-WinRT](https://github.com/doo/SQLite3-WinRT) C++ component) is now disabled in this plugin version (with CRYPTO provider completely removed) ref: [litehelpers / Cordova-sqlcipher-adapter#63](https://github.com/litehelpers/Cordova-sqlcipher-adapter/issues/63), has the following known limitations:
  - This plugin version branch has dependency on `v140` toolset libraries included by Visual Studio 2015 ref: [litehelpers/Cordova-sqlite-storage#580](https://github.com/litehelpers/Cordova-sqlite-storage/issues/580) (UNTESTED and UNSUPPORTED WORKAROUND for Visual Studio 2017 is described at: <https://developercommunity.visualstudio.com/content/problem/48806/cant-find-v140-in-visual-studio-2017.html>)
  - Encryption no longer enabled in Windows SQLite3 library build. For future consideration: enable Windows build again with encryption using a recent build of the OpenSSL crypto library ref: [litehelpers/Cordova-sqlcipher-adapter#30](https://github.com/litehelpers/Cordova-sqlcipher-adapter/issues/30)
  - It is **not** possible to use this plugin with the default "Any CPU" target. A specific target CPU type **must** be specified when building an app with this plugin.
  - Truncation issue with UNICODE `\u0000` character (same as `\0`)
  - No background processing
  - INCORRECT error code (0) and INCONSISTENT error message (missing actual error info) in error callbacks ref: [litehelpers/Cordova-sqlite-storage#539](https://github.com/litehelpers/Cordova-sqlite-storage/issues/539)
  - It is NOT possible to SELECT BLOB column values directly. It is recommended to use built-in HEX function to retrieve BLOB column values, which should work consistently across all platform implementations as well as (WebKit) Web SQL. Non-standard BASE64 function to SELECT BLOB column values in Base64 format is supported by [litehelpers / cordova-sqlite-ext](https://github.com/litehelpers/cordova-sqlite-ext) (permissive license terms) and [litehelpers / Cordova-sqlite-evcore-extbuild-free](https://github.com/litehelpers/Cordova-sqlite-evcore-extbuild-free) (GPL or commercial license terms), may be supported with SQLCipher in case of sufficient demand.
  - Windows platform version uses `UTF-16le` internal database encoding while the other platform versions use `UTF-8` internal encoding. (`UTF-8` internal encoding is preferred ref: [litehelpers/Cordova-sqlite-storage#652](https://github.com/litehelpers/Cordova-sqlite-storage/issues/652))
- The macOS platform version ("osx" platform) is not tested in a release build and should be considered pre-alpha.
- Android platform version:
  - Android versions supported: 2.3.3 - 7.1.1 (API level 10 - 25), depending on Cordova version ref: <https://cordova.apache.org/docs/en/latest/guide/platforms/android/>
  - SQLCipher for Android build uses the OpenSSL crypto library for encryption
  - ICU case-insensitive matching and other Unicode string manipulations is no longer supported for Android.
- iOS platform version:
  - iOS versions supported: 8.x / 9.x / 10.x / 11.x see [deviations section](#deviations) below for differences in case of WKWebView)
  - REGEXP is no longer supported for iOS.
- The macOS platform version ("osx" platform) is not tested in a release build and should be considered pre-alpha.
- Android versions supported: 2.3.3 - 7.1.1 (API level 10 - 25), depending on Cordova version ref: <https://cordova.apache.org/docs/en/latest/guide/platforms/android/>
- iOS versions supported: 8.x / 9.x / 10.x / 11.x (see [deviations section](#deviations) below for differences in case of WKWebView)
- FTS3, FTS4, and R-Tree are fully tested and supported for all target platforms in this version branch.
- Default `PRAGMA journal_mode` setting (*tested*):
  - Android with builtin android.database implementation (as selected using the `androidDatabaseImplementation` option in `window.sqlitePlugin.openDatabase`): `persist` (pre-8.x) / `truncate` (Android 8+)
  - otherwise: `delete`
- AUTO-VACUUM is not enabled by default. If no form of `VACUUM` or `PRAGMA auto_vacuum` is used then sqlite will automatically reuse deleted data space for new data but the database file will never shrink. For reference: <http://www.sqlite.org/pragma.html#pragma_auto_vacuum> and [litehelpers/Cordova-sqlite-storage#646](https://github.com/litehelpers/Cordova-sqlite-storage/issues/646)
- In case of memory issues please use smaller transactions (evcore enhancements may be supported with SQLCipher in case of sufficient demand).
- Pre-populatd DB is NOT supported by this version.
- Lawnchair adapter has NOT been validated with this plugin version and is NOT guaranteed to work (see below).

<!-- END Status -->

## Announcements

- Nice overview of cordova-sqlite-storage and other alternatives for storing local data at: <https://www.sitepoint.com/storing-local-data-in-a-cordova-app/>
- New alternative solution for small data storage (without SQLCipher): [TheCocoaProject/ cordova-plugin-nativestorage](https://github.com/TheCocoaProject/cordova-plugin-nativestorage) - simpler "native storage of variables" for Android/iOS/Windows
- Resolved Java 6/7/8 concurrent map compatibility issue reported in [litehelpers/Cordova-sqlite-storage#726](https://github.com/litehelpers/Cordova-sqlite-storage/issues/726), THANKS to pointer by [@NeoLSN (Jason Yang/楊朝傑)](https://github.com/NeoLSN) in [litehelpers/Cordova-sqlite-storage#727](https://github.com/litehelpers/Cordova-sqlite-storage/issues/727).
- Updated workaround solution to [BUG 666 (litehelpers/Cordova-sqlite-storage#666)](https://github.com/litehelpers/Cordova-sqlite-storage/issues/666) (possible transaction issue after window.location change with possible data loss): close database if already open before opening again
- Windows 10 (UWP) build with /SAFESEH flag on Win32 (x86) target to specify "Image has Safe Exception Handlers" as described in <https://docs.microsoft.com/en-us/cpp/build/reference/safeseh-image-has-safe-exception-handlers>
- Fixed iOS/macOS platform version to use [PSPDFThreadSafeMutableDictionary.m](https://gist.github.com/steipete/5928916) to avoid threading issue ref: [litehelpers/Cordova-sqlite-storage#716](https://github.com/litehelpers/Cordova-sqlite-storage/issues/716)
- This plugin version references Windows platform toolset v141 to support Visual Studio 2017 ref: [litehelpers/Cordova-sqlite-storage#580](https://github.com/litehelpers/Cordova-sqlite-storage/issues/580). (Visual Studio 2015 is now supported by [litehelpers / cordova-sqlite-legacy](https://github.com/litehelpers/cordova-sqlite-legacy) (permissive license terms, no performance enhancements for Android) and [litehelpers / Cordova-sqlite-evcore-common-free](https://github.com/litehelpers/Cordova-sqlite-evcore-common-free) (GPL or commercial license terms, with performance enhancements for Android). UNTESTED workaround for Visual Studio 2015: it *may* be possible to support this plugin version on Visual Studio 2015 Update 3 by installing platform toolset v141.)
- The Lawnchair adapter is now moved to [litehelpers / cordova-sqlite-lawnchair-adapter](https://github.com/litehelpers/cordova-sqlite-lawnchair-adapter).
- [litehelpers / cordova-sqlite-ext](https://github.com/litehelpers/cordova-sqlite-ext) (WITHOUT SQLCipher support) now supports SELECT BLOB data in Base64 format on all platforms in addition to REGEXP (Android/iOS/macOS) and pre-populated database (all platforms). FOR FUTURE CONSIDERATION: SELECT BLOB in BASE64 format and REGEXP support for SQLCipher may be added here upon sufficient demand.
- [brodybits / sql-promise-helper](https://github.com/brodybits/sql-promise-helper) provides a Promise-based API wrapper.
- [nolanlawson / pouchdb-adapter-cordova-sqlite](https://github.com/nolanlawson/pouchdb-adapter-cordova-sqlite) supports this plugin along with other implementations such as [nolanlawson / sqlite-plugin-2](https://github.com/nolanlawson/sqlite-plugin-2) and [Microsoft / cordova-plugin-websql](https://github.com/Microsoft/cordova-plugin-websql).
- macOS ("osx" platform) is now supported
- New [litehelpers / Cordova-sqlite-evcore-extbuild-free](https://github.com/litehelpers/Cordova-sqlite-evcore-extbuild-free) plugin version (WITHOUT SQLCipher support) with Android JSON and SQL statement handling implemented in C, as well as support for PhoneGap Build, Intel XDK, etc. (GPL or commercial license terms). Handles large SQL batches in less than half the time as this plugin version. Also supports arbitrary database location on Android. FOR FUTURE CONSIDERATION: evcore version with SQLCipher supported may be added upon sufficient demand.
- Published [brodybits / Cordova-quick-start-checklist](https://github.com/brodybits/Cordova-quick-start-checklist) and [brodybits / Avoiding-some-Cordova-pitfalls](https://github.com/brodybits/Avoiding-some-Cordova-pitfalls).
- Self-test functions to verify proper installation and operation of this plugin
- More explicit `openDatabase` and `deleteDatabase` `iosDatabaseLocation` option
- Added straightforward sql batch function
- PhoneGap Build is now supported through the npm package: http://phonegap.com/blog/2015/05/26/npm-plugins-available/
- [MetaMemoryT / websql-promise](https://github.com/MetaMemoryT/websql-promise) now provides a Promises-based interface to both Web SQL and this plugin

## Highlights

- This plugin version is built with [SQLCipher](https://www.zetetic.net/sqlcipher/) included.
- Drop-in replacement for HTML5/[Web SQL (DRAFT) API](http://www.w3.org/TR/webdatabase/): the only change should be to replace the static `window.openDatabase()` factory call with `window.sqlitePlugin.openDatabase()`, with parameters as documented below. Known deviations are documented in the [deviations section](#deviations) below.
- Failure-safe nested transactions with batch processing optimizations (according to HTML5/[Web SQL (DRAFT) API](http://www.w3.org/TR/webdatabase/))
- Transaction API (based on HTML5/[Web SQL (DRAFT) API](http://www.w3.org/TR/webdatabase/)) is designed for maximum flexiblibility, does not allow any transactions to be left hanging open.
- As described in [this posting](http://brodyspark.blogspot.com/2012/12/cordovaphonegap-sqlite-plugins-offer.html):
  - Keeps sqlite database in known, platform specific user data location on all supported platforms (Android/iOS/macOS/...), which can be reconfigured on iOS/macOS. Whether or not the database on the iOS platform is synchronized to iCloud depends on the selected database location.
  - No arbitrary size limit. SQLite limits described at: <http://www.sqlite.org/limits.html>
- Also validated for multi-page applications by internal test selfTest function.
- This project is self-contained. There are no dependencies on other plugins such as cordova-plugin-file.
- Windows platform version _(NOW DISABLED IN THIS PLUGIN VERSION)_ uses a customized version of the performant [doo / SQLite3-WinRT](https://github.com/doo/SQLite3-WinRT) C++ component.
- [SQLCipher](https://www.zetetic.net/sqlcipher/) support for Android/iOS/macOS/Windows is available in: [litehelpers / Cordova-sqlcipher-adapter](https://github.com/litehelpers/Cordova-sqlcipher-adapter)
- Intellectual property:
  - All source code is tracked to the original author in git
  - Major authors are tracked in AUTHORS.md
  - License of each component is tracked in LICENSE.md
  - History of this project is also described in HISTORY.md

**TIP:** It is possible to migrate from Cordova to a pure native solution and continue using the data stored by this plugin.

<!-- END Highlights -->

## Getting started

### Recommended prerequisites

- Install a recent version of Cordova CLI, create a simple app with no plugins, and run it on the desired target platforms.
- Add a very simple plugin such as `cordova-plugin-dialogs` or an echo plugin and get it working. Ideally you should be able to handle a callback with some data coming from a prompt.

These prereqisites are very well documented in a number of excellent resources including:
- <http://cordova.apache.org/> (redirected from <http://cordova.io>)
- <http://www.tutorialspoint.com/cordova/>
- <https://ccoenraets.github.io/cordova-tutorial/>
- <https://www.toptal.com/mobile/developing-mobile-applications-with-apache-cordova>
- <http://www.tutorialspoint.com/cordova/index.htm>

More resources can be found by <https://www.google.com/search?q=cordova+tutorial>. There are even some tutorials available on YouTube as well.

In addition, this guide assumes a basic knowledge of some key JavaScript concepts such as variables, function calls, and callback functions. There is an excellent explanation of JavaScript callbacks at <http://cwbuecheler.com/web/tutorials/2013/javascript-callbacks/>.

**MAJOR TIPS:** As described in the [Installing](#installing) section:
- It is recommended to use the `--save` flag when installing plugins to track them in `config.xml` _in case of Cordova CLI pre-7.x_. If all plugins are tracked in `config.xml` then there is no need to commit the `plugins` subdirectory tree into the source repository.
- In general it is *not* recommended to commit the `platforms` subdirectory tree into the source repository.

**NOTICE:** This plugin is only supported with the Cordova CLI. This plugin is *not* supported with other Cordova/PhoneGap systems such as PhoneGap CLI, PhoneGap Build, Plugman, Intel XDK, Webstorm, etc.

<!-- TBD FOR FUTURE CONSIDERATION: SUPPORT Windows build with OpenSSL CRYPTO PROVIDER ...
### Windows platform notes

The Windows platform can present a number of challenges which increase when using this plugin. The following tips are recommended for getting started with Windows:

- First start to build and run an app on another platform such as Android or iOS with this plugin.
- Try working with a very simple app using simpler plugins such as cordova-plugin-dialogs and possibly cordova-plugin-file on the Windows platform.
- Read through the **Windows platform usage** of the [Installing](#installing) section.
- Then try adding this plugin to a very simple app such as [brodybits / cordova-sqlite-test-app](https://github.com/brodybits/cordova-sqlite-test-app) and running the Windows project in the Visual Studio GUI with a specific target CPU selected. **WARNING:** It is not possible to use this plugin with the "Any CPU" target.
 ... -->

### Quick installation

Use the following command to install this plugin from the Cordova CLI:

```shell
cordova plugin add cordova-sqlcipher-adapter # --save RECOMMENDED for Cordova CLI pre-7.0
```

Add any desired platform(s) if not already present, for example:

```shell
cordova platform add android
```

**OPTIONAL:** prepare before building (**MANDATORY** for cordova-ios older than `4.3.0` (Cordova CLI `6.4.0`))

```shell
cordova prepare
```

or to prepare for a single platform, Android for example:

```shell
cordova prepare android
```

Please see the [Installing](#installing) section for more details.

**NOTE:** The new [brodybits / cordova-sqlite-test-app](https://github.com/brodybits/cordova-sqlite-test-app) project includes the echo test, self test, and string test described below along with some more sample functions.

<!-- END Quick installation -->

### Self test

Try the following programs to verify successful installation and operation:

**Echo test** - verify successful installation and build:

```js
document.addEventListener('deviceready', function() {
  window.sqlitePlugin.echoTest(function() {
    console.log('ECHO test OK');
  });
});
```

**Self test** - automatically verify basic database access operations including opening a database; basic CRUD operations (create data in a table, read the data from the table, update the data, and delete the data); close and delete the database:

```js
document.addEventListener('deviceready', function() {
  window.sqlitePlugin.selfTest(function() {
    console.log('SELF test OK');
  });
});
```

**NOTE:** It may be easier to use a JavaScript or native `alert` function call along with (or instead of) `console.log`  to verify that the installation passes both tests. Same for the SQL string test variations below. (Note that the Windows platform does not support the standard `alert` function, please use `cordova-plugin-dialogs` instead.)

### SQL string test

This test verifies that you can open a database, execute a basic SQL statement, and get the results (should be `TEST STRING`):

```js
document.addEventListener('deviceready', function() {
  var db = window.sqlitePlugin.openDatabase({name: 'test.db', key: 'user-password', location: 'default'});
  db.transaction(function(tr) {
    tr.executeSql("SELECT upper('Test string') AS upperString", [], function(tr, rs) {
      console.log('Got upperString result: ' + rs.rows.item(0).upperString);
    });
  });
});
```

Here is a variation that uses a SQL parameter instead of a string literal:

```js
document.addEventListener('deviceready', function() {
  var db = window.sqlitePlugin.openDatabase({name: 'test.db', key: 'user-password', location: 'default'});
  db.transaction(function(tr) {
    tr.executeSql('SELECT upper(?) AS upperString', ['Test string'], function(tr, rs) {
      console.log('Got upperString result: ' + rs.rows.item(0).upperString);
    });
  });
});
```

### Moving forward

It is recommended to read through the [usage](#usage) and [sample](#sample) sections before building more complex applications. In general it is recommended to start by doing things one step at a time, especially when an application does not work as expected.

The new [brodybits / cordova-sqlite-test-app](https://github.com/brodybits/cordova-sqlite-test-app) sample is intended to be a boilerplate to reproduce and demonstrate any issues you may have with this plugin. You may also use it as a starting point to build a new app.

In case you get stuck with something please read through the [support](#support) section and follow the instructions before raising an issue. Professional support is also available by contacting: <sales@litehelpers.net>

### Plugin usage examples

FUTURE TODO (WANTED): samples using this plugin version (with encryption)

**WITHOUT SQLCIPHER:**

- [brodybits / cordova-sqlite-storage-starter-app](https://github.com/brodybits/cordova-sqlite-storage-starter-app) (using `cordova-sqlite-storage` plugin version)
- <http://stackoverflow.com/questions/26604952/a-simple-cordova-android-example-including-sqlite-read-write-and-search> (also using `cordova-sqlite-storage` plugin version)
- <http://stackoverflow.com/questions/26604952/a-simple-cordova-android-example-including-sqlite-read-write-and-search>

### Plugin tutorials

FUTURE TODO (WANTED): tutorials using this plugin version (with encryption)

**WITHOUT SQLCIPHER:**

- <https://phonegappro.com/tutorials/phonegap-sqlite-tutorial-with-example-apache-cordova/> (using `cordova-sqlite-storage` plugin version)

**NOTICE:** The above tutorial shows `cordova plugin add cordova-sqlite-storage` with the `--save` flag missing. Please be sure to use the `--save` flag in case of Cordova CLI pre-7.0 to keep the plugins in `config.xml` (automatic starting with Cordova CLI 7.0).

Other plugin tutorials wanted ref: [litehelpers/Cordova-sqlite-storage#609](https://github.com/litehelpers/Cordova-sqlite-storage/issues/609)

### SQLite resources

- <http://www.tutorialspoint.com/sqlite/index.htm> with a number of helpful articles

### Some other Cordova resources

- <http://www.tutorialspoint.com/cordova/cordova_file_system.htm>

<!-- END Getting started -->

## Some apps using Cordova SQLCipher adapter plugin version

TBD *YOUR APP HERE*

<!-- END Some apps using this plugin version -->

## Security

### Security of sensitive data

According to [Web SQL Database API 7.2 Sensitivity of data](https://www.w3.org/TR/webdatabase/#sensitivity-of-data):
>User agents should treat persistently stored data as potentially sensitive; it's quite possible for e-mails, calendar appointments, health records, or other confidential documents to be stored in this mechanism.
>
>To this end, user agents should ensure that when deleting data, it is promptly deleted from the underlying storage.

Unfortunately this plugin will not actually overwrite the deleted content unless the [secure_delete PRAGMA](https://www.sqlite.org/pragma.html#pragma_secure_delete) is used.

### SQL injection

As "strongly recommended" by [Web SQL Database API 8.5 SQL injection](https://www.w3.org/TR/webdatabase/#sql-injection):
>Authors are strongly recommended to make use of the `?` placeholder feature of the `executeSql()` method, and to never construct SQL statements on the fly.

<!-- END Security -->

# Avoiding data loss

- Double-check that the application code follows the documented API for SQL statements, parameter values, success callbacks, and error callbacks.
- For standard Web SQL transactions include a transaction error callback with the proper logic that indicates to the user if data cannot be stored for any reason. In case of individual SQL error handlers be sure to indicate to the user if there is any issue with storing data.
- For single statement and batch transactions include an error callback with logic that indicates to the user if data cannot be stored for any reason.

<!-- Avoiding data loss -->

## Deviations

### Some known deviations from the Web SQL database standard

- The `window.sqlitePlugin.openDatabase` static factory call takes a different set of parameters than the standard Web SQL `window.openDatabase` static factory call. In case you have to use existing Web SQL code with no modifications please see the **Web SQL replacement tip** below.
- This plugin does *not* support the database creation callback or standard database versions. Please read the **Database schema versions** section below for tips on how to support database schema versioning.
- This plugin does *not* support the synchronous Web SQL interfaces.
- Known issues with handling of certain ASCII/UNICODE characters as described below.
- It is possible to request a SQL statement list such as "SELECT 1; SELECT 2" within a single SQL statement string, however the plugin will only execute the first statement and silently ignore the others ref: [litehelpers/Cordova-sqlite-storage#551](https://github.com/litehelpers/Cordova-sqlite-storage/issues/551)
- It is possible to insert multiple rows like: `transaction.executeSql('INSERT INTO MyTable VALUES (?,?),(?,?)', ['Alice', 101, 'Betty', 102]);` which was not supported by SQLite 3.6.19 as referenced by [Web SQL (DRAFT) API section 5](https://www.w3.org/TR/webdatabase/#web-sql). The iOS WebKit Web SQL implementation seems to support this as well.
- Unlike the HTML5/[Web SQL (DRAFT) API](http://www.w3.org/TR/webdatabase/) this plugin handles executeSql calls with too few parameters without error reporting. In case of too many parameters this plugin reports error code 0 (SQLError.UNKNOWN_ERR) while Android/iOS (WebKit) Web SQL correctly reports error code 5 (SQLError.SYNTAX_ERR) ref: https://www.w3.org/TR/webdatabase/#dom-sqlexception-code-syntax
- Positive and negative `Infinity` SQL parameter argument values are treated like `null` by this plugin on Android and iOS ref: [litehelpers/Cordova-sqlite-storage#405](https://github.com/litehelpers/Cordova-sqlite-storage/issues/405)
- Positive and negative `Infinity` result values cause a crash on iOS/macOS cases ref: [litehelpers/Cordova-sqlite-storage#405](https://github.com/litehelpers/Cordova-sqlite-storage/issues/405)
- Known issue(s) with of certain ASCII/UNICODE characters as described below.
- Boolean `true` and `false` values are handled by converting them to the "true" and "false" TEXT string values, same as WebKit Web SQL on Android and iOS. This does not seem to be 100% correct as discussed in: [litehelpers/Cordova-sqlite-storage#545](https://github.com/litehelpers/Cordova-sqlite-storage/issues/545)
- A number of uncategorized errors such as CREATE VIRTUAL TABLE USING bogus module are reported with error code 5 (SQLError.SYNTAX_ERR) on Android/iOS/macOS by both (WebKit) Web SQL and this plugin.
- Issues with error code on Windows as well as Android described below.
- In case of an issue that causes an API function to throw an exception (Android/iOS WebKit) Web SQL includes includes a code member with value of 0 (SQLError.UNKNOWN_ERR) in the exception while the plugin includes no such code member.
- This plugin supports some non-standard features as documented below.
- Results of SELECT with BLOB data such as `SELECT LOWER(X'40414243') AS myresult`, `SELECT X'40414243' AS myresult`, or reading data stored by `INSERT INTO MyTable VALUES (X'40414243')` are not consistent on Android or Windows. (These work with Android/iOS WebKit Web SQL and have been supported by SQLite for a number of years.)
- Whole number parameter argument values such as `42`, `-101`, or `1234567890123` are handled as INTEGER values by this plugin on Android, iOS (default UIWebView), and Windows while they are handled as REAL values by (WebKit) Web SQL and this plugin on macOS or iOS with WKWebView. This is evident in certain test operations such as `SELECT ? as myresult` or `SELECT TYPEOF(?) as myresult` and storage in a field with TEXT affinity.
- INTEGER, REAL, +/- `Infinity`, `NaN`, `null`, `undefined` parameter argument values are handled as TEXT string values on Android. (This is evident in certain test operations such as `SELECT ? as myresult` or `SELECT TYPEOF(?) as myresult` and storage in a field with TEXT affinity.)
- In case of invalid transaction callback arguments such as string values the plugin attempts to execute the transaction while (WebKit) Web SQL would throw an exception.
- The plugin handles invalid SQL arguments array values such as `false`, `true`, or a string as if there were no arguments while (WebKit) Web SQL would throw an exception. NOTE: In case of a function in place of the SQL arguments array WebKit Web SQL would report a transaction error while the plugin would simply ignore the function.
- In case of invalid SQL callback arguments such as string values the plugin may execute the SQL and signal transaction success or failure while (WebKit) Web SQL would throw an exception.
- In certain cases such as `transaction.executeSql(null)` or `transaction.executeSql(undefined)` the plugin throws an exception while (WebKit) Web SQL indicates a transaction failure.
- In certain cases such as `transaction.executeSql()` with no arguments (Android/iOS WebKit) Web SQL includes includes a code member with value of 0 (SQLError.UNKNOWN_ERR) in the exception while the plugin includes no such code member.
- If the SQL arguments are passed in an `Array` subclass object where the `constructor` does not point to `Array` then the SQL arguments are ignored by the plugin.
- The results data objects are not immutable as specified/implied by [Web SQL (DRAFT) API section 4.5](https://www.w3.org/TR/webdatabase/#database-query-results).
- This plugin version provides encryption which is NOT covered by the HTML5/[Web SQL API](http://www.w3.org/TR/webdatabase/).

### Security of deleted data

See **Security of sensitive data** in the [Security](#security) section above.

### Other differences with WebKit Web SQL implementations

- FTS3, FTS4, and R-Tree are not consistently supported by (WebKit) Web SQL on Android/iOS.
- In case of ignored INSERT OR IGNORE statement WebKit Web SQL (Android/iOS) reports insertId with an old INSERT row id value while the plugin reports insertId: undefined.
- In case of a SQL error handler that does not recover the transaction, WebKit Web SQL (Android/iOS) would incorrectly report error code 0 while the plugin would report the same error code as in the SQL error handler. (In case of an error with no SQL error handler then Android/iOS WebKit Web SQL would report the same error code that would have been reported in the SQL error hander.)
- In case a transaction function throws an exception, the message and code if present are reported by the plugin but *not* by (WebKit) Web SQL.
- SQL error messages are inconsistent on Windows.
- There are some other differences in the SQL error messages reported by WebKit Web SQL and this plugin. NOTE that unlike the android.database.sqlite implementation on Android 4.x(+) SQLCipher for Android does not seem to include the error code in most of the error message.

<!-- END Deviations -->

## Known issues

- The iOS/macOS platform versions do not support certain rapidly repeated open-and-close or open-and-delete test scenarios due to how the implementation handles background processing
- ~~INCORRECT HANDLING and POSSIBLE CRASH on some Android system versions in case of emojis and other 4-byte UTF-8 characters on default Android sqlite access implementation ref: [litehelpers/Cordova-sqlite-storage#564](https://github.com/litehelpers/Cordova-sqlite-storage/issues/564). It is sometimes possible to store and retrieve such characters but certain operations such as HEX conversions do not work properly when using the default [Android-sqlite-connector](https://github.com/liteglue/Android-sqlite-connector) (NDK) implementation. It is suspected that such characters would be stored incorrectly by the default Android platform version. Note that this is not an issue in case the built-in Android database is used (using the `androidDatabaseImplementation: 2` setting in `window.sqlitePlugin.openDatabase`)~~ (NOT an issue in this plugin version)
- Cannot read encrypted database with CORRECT password directly after attempt to open with INCORRECT password ref: [litehelpers/Cordova-sqlcipher-adapter#43](https://github.com/litehelpers/Cordova-sqlcipher-adapter/issues/43)
- It is possible to request a SQL statement list such as "SELECT 1; SELECT 2" within a single SQL statement string, however the plugin will only execute the first statement and silently ignore the others ref: [litehelpers/Cordova-sqlite-storage#551](https://github.com/litehelpers/Cordova-sqlite-storage/issues/551)
- Execution of INSERT statement that affects multiple rows (due to SELECT cause or using TRIGGER(s), for example) reports incorrect rowsAffected on Android in case the built-in Android database used (using the `androidDatabaseImplementation` option in `window.sqlitePlugin.openDatabase`)
- Memory issue observed when adding a large number of records due to the JSON implementation which is improved in [litehelpers / Cordova-sqlite-evcore-extbuild-free](https://github.com/litehelpers/Cordova-sqlite-evcore-extbuild-free) (GPL or commercial license terms)
- Infinity (positive or negative) values are not supported on Android/iOS/macOS due to issues described above including a possible crash on iOS/macOS ref: [litehelpers/Cordova-sqlite-storage#405](https://github.com/litehelpers/Cordova-sqlite-storage/issues/405)
- A stability issue was reported on the iOS platform version when in use together with [SockJS](http://sockjs.org/) client such as [pusher-js](https://github.com/pusher/pusher-js) at the same time (see [litehelpers/Cordova-sqlite-storage#196](https://github.com/litehelpers/Cordova-sqlite-storage/issues/196)). The workaround is to call sqlite functions and [SockJS](http://sockjs.org/) client functions in separate ticks (using setTimeout with 0 timeout).
- SQL errors are reported with an INCORRECT error code (0) on Android and Windows.
- ~~Possible crash on certain Android versions when using Unicode emoji and other 4-byte UTF-8 characters due to [Android bug 81341](https://code.google.com/p/android/issues/detail?id=81341)~~ (NOT an issue in this plugin version)
- Close/delete database bugs described below.
- When a database is opened and deleted without closing, the iOS/macOS platform version is known to leak resources.
- It is NOT possible to open multiple databases with the same name but in different locations (iOS/macOS platform version).

Some additional issues are tracked in [open Cordova-sqlite-storage bug-general issues](https://github.com/litehelpers/Cordova-sqlite-storage/issues?q=is%3Aissue+is%3Aopen+label%3Abug-general) and [open Cordova-sqlcipher-adapter bug-general issues](https://github.com/litehelpers/Cordova-sqlcipher-adapter/issues?q=is%3Aissue+is%3Aopen+label%3Abug-general).

<!-- END Known issues -->

## Other limitations

- ~~The db version, display name, and size parameter values are not supported and will be ignored.~~ (No longer supported by the API)
- Absolute and relative subdirectory path(s) are not tested or supported.
- This plugin will not work before the callback for the 'deviceready' event has been fired, as described in **Usage**. (This is consistent with the other Cordova plugins.)
- Extremely large records are not supported by this plugin. It is recommended to store images and similar binary data in separate files. TBD: specify maximum record. For future consideration: support in a plugin version such as [litehelpers / Cordova-sqlite-evcore-extbuild-free](https://github.com/litehelpers/Cordova-sqlite-evcore-extbuild-free) (GPL or commercial license terms).
- This plugin version will not work within a web worker (not properly supported by the Cordova framework). Use within a web worker is supported for Android/iOS/macOS (WITHOUT SQLCipher) in [litehelpers / cordova-sqlite-evmax-ext-workers-legacy-build-free](https://github.com/litehelpers/cordova-sqlite-evmax-ext-workers-legacy-build-free) (GPL or premium commercial license terms).
- In-memory database `db=window.sqlitePlugin.openDatabase({name: ':memory:', ...})` is currently not supported.
- The Android platform version cannot properly support more than 100 open database files due to the threading model used.
- SQL error messages reported by Windows platform version are not consistent with Android/iOS/macOS platform versions.
- UNICODE `\u2028` (line separator) and `\u2029` (paragraph separator) characters are currently not supported and known to be broken on iOS, macOS, and Android platform versions due to JSON issues reported in [Cordova bug CB-9435](https://issues.apache.org/jira/browse/CB-9435) and [cordova/cordova-discuss#57](https://github.com/cordova/cordova-discuss/issues/57). This is fixed with a workaround for iOS/macOS (WITHOUT SQLCipher) in: [litehelpers / Cordova-sqlite-evplus-legacy-free](https://github.com/litehelpers/Cordova-sqlite-evplus-legacy-free) and [litehelpers / Cordova-sqlite-evplus-legacy-attach-detach-free](https://github.com/litehelpers/Cordova-sqlite-evplus-legacy-attach-detach-free) (GPL or special commercial license terms) as well as [litehelpers / cordova-sqlite-evmax-ext-workers-legacy-build-free](https://github.com/litehelpers/cordova-sqlite-evmax-ext-workers-legacy-build-free) (GPL or premium commercial license terms).
- SELECT BLOB column value type is not supported consistently across all platforms (not supported on Windows). It is recommended to use the built-in HEX function to SELECT BLOB column data in hexadecimal format, working consistently across all platforms. As an alternative (WITHOUT SQLCipher): SELECT BLOB in Base64 format is supported by [litehelpers / cordova-sqlite-ext](https://github.com/litehelpers/cordova-sqlite-ext) (permissive license terms) and [litehelpers / Cordova-sqlite-evcore-extbuild-free](https://github.com/litehelpers/Cordova-sqlite-evcore-extbuild-free) (GPL or commercial license options).
- Database files with certain multi-byte UTF-8 characters are not tested and not expected to work consistently across all platform implementations.
- Truncation in case of UNICODE `\u0000` (same as `\0`) character on Android (default Android-sqlite-connector database implementation) and Windows.
- Case-insensitive matching and other string manipulations on Unicode characters, which is provided by optional ICU integration in the sqlite source and working with recent versions of Android, is not supported for any target platforms.
- The iOS/macOS platform version uses a thread pool but with only one thread working at a time due to "synchronized" database access.
- Some large query results may be slow, also due to the JSON implementation.
- ATTACH to another database file is not supported by this version branch. ATTACH/DETACH is supported (along with the memory and iOS UNICODE `\u2028` line separator / `\u2029` paragraph separator fixes, WITHOUT SQLCipher) in [litehelpers / Cordova-sqlite-evplus-legacy-attach-detach-free](https://github.com/litehelpers/Cordova-sqlite-evplus-legacy-attach-detach-free) (GPL or special commercial license terms).
- UPDATE/DELETE with LIMIT or ORDER BY is not supported.
- User-defined savepoints are not supported and not expected to be compatible with the transaction locking mechanism used by this plugin. In addition, the use of BEGIN/COMMIT/ROLLBACK statements is not supported.
- Issues have been reported with using this plugin together with Crosswalk for Android, especially on `x86_64` CPU ([litehelpers/Cordova-sqlite-storage#336](https://github.com/litehelpers/Cordova-sqlite-storage/issues/336)). Please see [litehelpers/Cordova-sqlite-storage#336 (comment)](https://github.com/litehelpers/Cordova-sqlite-storage/issues/336#issuecomment-364752652) for workaround on x64 CPU. In addition it may be helpful to install Crosswalk as a plugin instead of using Crosswalk to create a project that will use this plugin.
- Does not work with [axemclion / react-native-cordova-plugin](https://github.com/axemclion/react-native-cordova-plugin) since the `window.sqlitePlugin` object is NOT properly exported (ES5 feature). It is recommended to use [andpor / react-native-sqlite-storage](https://github.com/andpor/react-native-sqlite-storage) for SQLite database access with React Native Android/iOS instead.

Additional limitations are tracked in [cordova-sqlite-help doc-todo issues](https://github.com/litehelpers/Cordova-sqlite-help/issues?q=is%3Aissue%20label%3Adoc-todo), [cordova-sqlite-storage doc-todo issues](https://github.com/litehelpers/Cordova-sqlite-storage/issues?q=is%3Aissue+is%3Aopen+label%3Adoc-todo), and [Cordova-sqlcipher-adapter doc-todo issues](https://github.com/litehelpers/Cordova-sqlcipher-adapter/issues?q=is%3Aissue+label%3Adoc-todo).

<!-- END Other limitations -->

## Further testing needed

- Integration with PhoneGap developer app
- Use within [InAppBrowser](http://docs.phonegap.com/en/edge/cordova_inappbrowser_inappbrowser.md.html)
- Use within an iframe (see [litehelpers/Cordova-sqlite-storage#368 (comment)](https://github.com/litehelpers/Cordova-sqlite-storage/issues/368#issuecomment-154046367))
- Date/time handling
- Maximum record size supported
- Actual behavior when using SAVEPOINT(s)
- R-Tree is not fully tested with Android
- UNICODE characters not fully tested
- ORDER BY RANDOM() (ref: [litehelpers/Cordova-sqlite-storage#334](https://github.com/litehelpers/Cordova-sqlite-storage/issues/334))
- UPDATE/DELETE with LIMIT or ORDER BY (newer Android/iOS versions)
- Integration with JXCore for Cordova (must be built without sqlite(3) built-in)
- Delete an open database inside a statement or transaction callback.
- WITH clause (not supported by some older sqlite3 versions)
- Handling of invalid transaction and transaction.executeSql arguments
- Use of database locations on macOS
- Extremely large and small INTEGER and REAL values ref: [litehelpers/Cordova-sqlite-storage#627](https://github.com/litehelpers/Cordova-sqlite-storage/issues/627)
- More emojis and other 4-octet UTF-8 characters
- More database file names with multi-byte UTF-8 characters including emojis and other 4-byte UTF-8 characters
- Use of `?NNN`/`:AAA`/`@AAAA`/`$AAAA` parameter placeholders as documented in <https://www.sqlite.org/lang_expr.html#varparam> and <https://www.sqlite.org/c3ref/bind_blob.html>) (currently NOT supported by this plugin) ref: [litehelpers/Cordova-sqlite-storage#717](https://github.com/litehelpers/Cordova-sqlite-storage/issues/717)
- Single-statement and SQL batch transaction calls with invalid arguments (TBD behavior subject to change)
- Plugin vs (WebKit) Web SQL transaction behavior in case of an error handler which returns various falsy vs truthy values
- Other [open Cordova-sqlite-storage testing issues](https://github.com/litehelpers/Cordova-sqlite-storage/issues?q=is%3Aissue+is%3Aopen+label%3Atesting) and [open Cordova-sqlcipher-adapter testing issues](https://github.com/litehelpers/Cordova-sqlcipher-adapter/issues?q=is%3Aissue+is%3Aopen+label%3Atesting)

<!-- END Further testing needed -->

## Some tips and tricks

- If you run into problems and your code follows the asynchronous HTML5/[Web SQL](http://www.w3.org/TR/webdatabase/) transaction API, you can try opening a test database using `window.openDatabase` and see if you get the same problems.
- In case your database schema may change, it is recommended to keep a table with one row and one column to keep track of your own schema version number. It is possible to add it later. The recommended schema update procedure is described below.

<!-- END Some tips and tricks -->

## Pitfalls

### Some common pitfall(s)

- If a database is opened using the standard `window.openDatabase` call it will not have any of the benefits of this plugin and features such as the `sqlBatch` call would not be available.
- It is NOT allowed to execute sql statements on a transaction that has already finished, as described below. This is consistent with the HTML5/[Web SQL (DRAFT) API](http://www.w3.org/TR/webdatabase/).
- The plugin class name starts with "SQL" in capital letters, but in Javascript the `sqlitePlugin` object name starts with "sql" in small letters.
- Attempting to open a database before receiving the 'deviceready' event callback.
- Inserting STRING into ID field
- Auto-vacuum is NOT enabled by default. It is recommended to periodically VACUUM the database. If no form of `VACUUM` or `PRAGMA auto_vacuum` is used then sqlite will automatically reuse deleted data space for new data but the database file will never shrink. For reference: <http://www.sqlite.org/pragma.html#pragma_auto_vacuum> and [litehelpers/Cordova-sqlite-storage#646](https://github.com/litehelpers/Cordova-sqlite-storage/issues/646)
- Transactions on a database are run sequentially. A large transaction could block smaller transactions requested afterwards.

### Some weird pitfall(s)

- intent whitelist: blocked intent such as external URL intent *may* cause this and perhaps certain Cordova plugin(s) to misbehave (see [litehelpers/Cordova-sqlite-storage#396](https://github.com/litehelpers/Cordova-sqlite-storage/issues/396))

### Angular/ngCordova/Ionic-related pitfalls

- Angular/ngCordova/Ionic controller/factory/service callbacks may be triggered before the 'deviceready' event is fired
- As discussed in [litehelpers/Cordova-sqlite-storage#355](https://github.com/litehelpers/Cordova-sqlite-storage/issues/355), it may be necessary to install ionic-plugin-keyboard
- Navigation items such as root page can be tricky on Ionic 2 ref: [litehelpers/Cordova-sqlite-storage#613](https://github.com/litehelpers/Cordova-sqlite-storage/issues/613)

<!-- FUTURE TBD SUPPORT Windows build with OpenSSL CRYPTO PROVIDER ...
### Windows platform pitfalls

- Visual Studio 2015 is no longer supported by this version. Visual Studio 2015 is now supported by [litehelpers / Cordova-sqlite-legacy-build-support](https://github.com/litehelpers/Cordova-sqlite-legacy-build-support).
- This plugin does **not** work with the default "Any CPU" target. A specific, valid CPU target platform **must** be specified.
- It is **not** allowed to change the app ID in the Windows platform project. As described in the **Windows platform usage** of the [Installing](#installing) section a Windows-specific app ID may be declared using the `windows-identity-name` attribute or "WindowsStoreIdentityName" setting.
- A problem locating `SQLite3.md` generally means that there was a problem building the C++ library.
 ... -->

### General Cordova pitfalls

Documented in: [brodybits / Avoiding-some-Cordova-pitfalls](https://github.com/brodybits/Avoiding-some-Cordova-pitfalls)

### General SQLite pitfalls

From <https://www.sqlite.org/datatype3.html#section_1>:
> SQLite uses a more general dynamic type system.

This is generally nice to have, especially in conjunction with a dynamically typed language such as JavaScript. Here are some major SQLite data typing principles:
- From <https://www.sqlite.org/datatype3.html#section_3>: the CREATE TABLE SQL statement declares each column with one of the following type affinities: TEXT, NUMERIC, INTEGER, REAL, or BLOB.
- From <https://www.sqlite.org/datatype3.html#section_3_1> with column type affinity determination rules: it should be possible to do CREATE TABLE with columns of almost any type name (for example: `CREATE TABLE MyTable (data ABC);`) and each column type affinity is determined according to pattern matching. If a declared column type name does not match any of the patterns the column has NUMERIC affinity.
- From <https://www.sqlite.org/datatype3.html#section_3_2>: a column with no data type name specified actually gets the BLOB affinity.

However there are some possible gotchas:

1. From <https://www.sqlite.org/datatype3.html#section_3_2>:
> Note that a declared type of "FLOATING POINT" would give INTEGER affinity, not REAL affinity, due to the "INT" at the end of "POINT". And the declared type of "STRING" has an affinity of NUMERIC, not TEXT.

2. From ibid: a column declared as "DATETIME" has NUMERIC affinity, which gives no hint whether an INTEGER Unix time value, a REAL Julian time value, or possibly even a TEXT ISO8601 date/time string may be stored (further refs:  <https://www.sqlite.org/datatype3.html#section_2_2>, <https://www.sqlite.org/datatype3.html#section_3>)

From <https://groups.google.com/forum/#!topic/phonegap/za7z51_fKRw>, as discussed in [litehelpers/Cordova-sqlite-storage#546](https://github.com/litehelpers/Cordova-sqlite-storage/issues/546): it was discovered that are some more points of possible confusion with date/time. For example, there is also a `datetime` function that returns date/time in TEXT string format. This should be considered a case of "DATETIME" overloading since SQLite is *not* case sensitive. This could really become confusing if different programmers or functions consider date/time to be stored in different ways.

FUTURE TBD: Proper date/time handling will be further tested and documented at some point.

<!-- END pitfalls -->

## Major TODOs

- More formal documentation of API, especially for non-standard functions
- Browser platform (likely without actual encryption if SQL.js is used as discussed in [litehelpers/Cordova-sqlite-storage#576](https://github.com/litehelpers/Cordova-sqlite-storage/pull/576))
- IndexedDBShim adapter (possibly based on IndexedDBShim)
- Further cleanup of [support](#support) section
- Resolve or document remaining [open Cordova-sqlite-storage bugs](https://github.com/litehelpers/Cordova-sqlite-storage/issues?q=is%3Aissue+is%3Aopen+label%3Abug-general)
- Resolve [cordova-sqlite-help doc-todo issues](https://github.com/litehelpers/Cordova-sqlite-help/issues?q=is%3Aissue%20label%3Adoc-todo) and [marked Cordova-sqlite-storage doc-todo issues](https://github.com/litehelpers/Cordova-sqlite-storage/issues?q=is%3Aissue+label%3Adoc-todo)

<!-- END Major TODOs -->

## For future considertion

- Explicit auto-vacuum option ref: [litehelpers/Cordova-sqlite-storage#646](https://github.com/litehelpers/Cordova-sqlite-storage/issues/646)
- Support for extremely large records in a plugin version such as [litehelpers / Cordova-sqlite-evcore-extbuild-free](https://github.com/litehelpers/Cordova-sqlite-evcore-extbuild-free) (available with GPL or commercial license options)
- Integrate with some other libraries such as Sequelize, Squel.js, WebSqlSync, Persistence.js, Knex, etc.

<!-- END For future considertion -->

## Alternatives

**NOTE:** None of the other alternatives currently support SQLCipher.

### Comparison of sqlite plugin versions

- [litehelpers / Cordova-sqlite-storage](https://github.com/litehelpers/Cordova-sqlite-storage) - core plugin version for Android/iOS/macOS/Windows (permissive license terms)
- [litehelpers / cordova-sqlite-ext](https://github.com/litehelpers/cordova-sqlite-ext) - plugin version with REGEXP (Android/iOS/macOS), SELECT BLOB in Base64 format (all platforms Android/iOS/macOS/Windows), and pre-populated databases (all platforms Android/iOS/macOS/Windows). Permissive license terms.
- [litehelpers / cordova-sqlite-legacy](https://github.com/litehelpers/cordova-sqlite-legacy) - support for Windows 8.1/Windows Phone 8.1 along with Android/iOS/macOS/Windows 10, with support for REGEXP (Android/iOS/macOS), SELECT BLOB in Base64 format (all platforms Android/iOS/macOS/Windows), and pre-populated databases (all platforms Android/iOS/macOS/Windows). Limited updates. Permissive license terms.
- [brodybits / cordova-sqlite-legacy-build-support](https://github.com/brodybits/cordova-sqlite-legacy-build-support) - maintenance of WP8 platform version along with Windows 8.1/Windows Phone 8.1 and the other supported platforms Android/iOS/macOS/Windows 10; limited support for PhoneGap CLI/PhoneGap Build/plugman/Intel XDK; limited testing; limited updates. Permissive license terms.
- [litehelpers / Cordova-sqlcipher-adapter](https://github.com/litehelpers/Cordova-sqlcipher-adapter) - supports [SQLCipher](https://www.zetetic.net/sqlcipher/) for Android/iOS/macOS/Windows
- [litehelpers / Cordova-sqlite-evcore-extbuild-free](https://github.com/litehelpers/Cordova-sqlite-evcore-extbuild-free) - Enhancements for Android: JSON and SQL statement handling implemented in C, supports larger transactions and handles large SQL batches in less than half the time as this plugin version. Supports arbitrary database location on Android. Support for build environments such as PhoneGap Build and Intel XDK. Also includes REGEXP (Android/iOS/macOS) and SELECT BLOB in Base64 format (all platforms Android/iOS/macOS/Windows). GPL or commercial license terms.
- [litehelpers / cordova-sqlite-evplus-ext-legacy-build-free](https://github.com/litehelpers/cordova-sqlite-evplus-ext-legacy-build-free) - internal memory improvements to support larger transactions (Android/iOS) and fix to support all Unicode characters (iOS). (GPL or special commercial license terms).
- [litehelpers / Cordova-sqlite-evplus-legacy-attach-detach-free](https://github.com/litehelpers/Cordova-sqlite-evplus-legacy-attach-detach-free) - plugin version with support for ATTACH, includes internal memory improvements to support larger transactions (Android/iOS) and fix to support all Unicode characters (GPL or special commercial license terms).
- [litehelpers / cordova-sqlite-evmax-ext-workers-legacy-build-free](https://github.com/litehelpers/cordova-sqlite-evmax-ext-workers-legacy-build-free) - plugin version with support for web workers, includes internal memory improvements to support larger transactions (Android/iOS) and fix to support all Unicode characters (iOS). (GPL or special premium commercial license terms).
- Adaptation for React Native Android and iOS: [andpor / react-native-sqlite-storage](https://github.com/andpor/react-native-sqlite-storage) (permissive license terms)
- Original plugin version for iOS (with a non-standard, outdated transaction API): [davibe / Phonegap-SQLitePlugin](https://github.com/davibe/Phonegap-SQLitePlugin) (permissive license terms)

<!-- END Comparison of sqlite plugin versions -->

### Other SQLite access projects

- [an-rahulpandey / cordova-plugin-dbcopy](https://github.com/an-rahulpandey/cordova-plugin-dbcopy) - Supports pre-populated database (*NOT* expected with work SQLCipher for Android)
- [object-layer / AnySQL](https://github.com/object-layer/anysql) - Unified SQL API over multiple database engines
- [samikrc / CordovaSQLite](https://github.com/samikrc/CordovaSQLite) - Simpler sqlite plugin with a simpler API and browser platform
- [nolanlawson / sqlite-plugin-2](https://github.com/nolanlawson/sqlite-plugin-2) - Simpler fork/rewrite
- [nolanlawson / node-websql](https://github.com/nolanlawson/node-websql) - Web SQL API (DRAFT API) implementation for Node.js
- [an-rahulpandey / cordova-plugin-dbcopy](https://github.com/an-rahulpandey/cordova-plugin-dbcopy) - Alternative way to copy pre-populated database
- [EionRobb / phonegap-win8-sqlite](https://github.com/EionRobb/phonegap-win8-sqlite) - WebSQL add-on for Win8/Metro apps (perhaps with a different API), using an old version of the C++ library from [SQLite3-WinRT Component](https://github.com/doo/SQLite3-WinRT) (as referenced by [01org / cordova-win8](https://github.com/01org/cordova-win8))
- [SQLite3-WinRT Component](https://github.com/doo/SQLite3-WinRT) - C++ component that provides a nice SQLite API with promises for WinJS
- [01org / cordova-win8](https://github.com/01org/cordova-win8) - old, unofficial version of Cordova API support for Windows 8 Metro that includes an old version of the C++ [SQLite3-WinRT Component](https://github.com/doo/SQLite3-WinRT)
- [Microsoft / cordova-plugin-websql](https://github.com/Microsoft/cordova-plugin-websql) - Windows 8(+) and Windows Phone 8(+) WebSQL plugin versions in C#
- [Thinkwise / cordova-plugin-websql](https://github.com/Thinkwise/cordova-plugin-websql) - fork of [Microsoft / cordova-plugin-websql](https://github.com/Microsoft/cordova-plugin-websql) that supports asynchronous execution
- [MetaMemoryT / websql-client](https://github.com/MetaMemoryT/websql-client) - provides the same API and connects to [websql-server](https://github.com/MetaMemoryT/websql-server) through WebSockets.

<!-- END Other SQLite access projects -->

### Alternative storage solutions

- <https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API> available on newer Android/iOS/Windows versions, please see best practices at <https://developers.google.com/web/fundamentals/instant-and-offline/web-storage/indexeddb-best-practices>
- [TheCocoaProject/ cordova-plugin-nativestorage](https://github.com/TheCocoaProject/cordova-plugin-nativestorage) - simpler "native storage of variables" (small data storage) for Android/iOS/macOS/Windows (TBD browser support?)
- Use [phearme / cordova-ContentProviderPlugin](https://github.com/phearme/cordova-ContentProviderPlugin) to query content providers on Android devices
- [ABB-Austin / cordova-plugin-indexeddb-async](https://github.com/ABB-Austin/cordova-plugin-indexeddb-async) - Asynchronous IndexedDB plugin for Cordova that uses [axemclion / IndexedDBShim](https://github.com/axemclion/IndexedDBShim) (Browser/iOS/Android/Windows) and [Thinkwise / cordova-plugin-websql](https://github.com/Thinkwise/cordova-plugin-websql) - (Windows)
- Use [NativeScript](https://www.nativescript.org) with its web view and [NathanaelA / nativescript-sqlite](https://github.com/NathanaelA/nativescript-sqlite) (Android and/or iOS)
- Standard HTML5 [local storage](https://en.wikipedia.org/wiki/Web_storage#localStorage)
- [Realm.io](https://realm.io/)
- Other Cordova storage alternatives described at:
  - <https://www.sitepoint.com/storing-local-data-in-a-cordova-app/>
  - <https://www.joshmorony.com/a-summary-of-local-storage-options-for-phonegap-applications/>
  - <http://cordova.apache.org/docs/en/latest/cordova/storage/storage.html>

<!-- END Alternative storage solutions -->

# Usage

## Self-test functions

To verify that both the Javascript and native part of this plugin are installed in your application:

```js
window.sqlitePlugin.echoTest(successCallback, errorCallback);
```

To verify that this plugin is able to open a database (named `___$$$___litehelpers___$$$___test___$$$___.db`), execute the CRUD (create, read, update, and delete) operations, and clean it up properly:

```js
window.sqlitePlugin.selfTest(successCallback, errorCallback);
```

**IMPORTANT:** Please wait for the 'deviceready' event (see below for an example).

## General

- Drop-in replacement for HTML5/[Web SQL (DRAFT) API](http://www.w3.org/TR/webdatabase/): the only change should be to replace the static `window.openDatabase()` factory call with `window.sqlitePlugin.openDatabase()`, with parameters as documented below. Some other known deviations are described throughout this document. Reports of any other deviations would be appreciated.
- Single-page application design is recommended.
- In case of a multi-page application the JavaScript used by each page must use `sqlitePlugin.openDatabase` to open the database access handle object before it can access the data.

**NOTE:** If a sqlite statement in a transaction fails with an error, the error handler *must* return `false` in order to recover the transaction. This is correct according to the HTML5/[Web SQL (DRAFT) API](http://www.w3.org/TR/webdatabase/) standard. This is different from the WebKit implementation of Web SQL in Android and iOS which recovers the transaction if a sql error hander returns a truthy value.

See the [Sample section](#sample) for a sample with detailed explanations.

## Opening a database

To open a database access handle object (in the **new** default location):

```js
var db = window.sqlitePlugin.openDatabase({name: 'my.db',
  key: 'user-password-here',
  location: 'default'
}, successcb, errorcb);
```

**WARNING:** The new "default" location value is *NOT* the same as the old default location and would break an upgrade for an app that was using the old default value (0) on iOS.

**WARNING 2:** As described above: by default this plugin uses a non-standard SQLCipher database implementation on Android. In case an application access the **same** database using multiple plugins there is a risk of data corruption ref: [litehelpers/Cordova-sqlite-storage#626](https://github.com/litehelpers/Cordova-sqlite-storage/issues/626) as described in <http://ericsink.com/entries/multiple_sqlite_problem.html> and <https://www.sqlite.org/howtocorrupt.html>. ~~The workaround is to use the `androidDatabaseImplementation: 2` setting as described in the **Android sqlite implementation** section below.~~ _(There is no workaround in this plugin version.)_

To specify a different location (affects iOS/macOS *only*):

```js
var db = window.sqlitePlugin.openDatabase({name: 'my.db', key: 'your-password-here', iosDatabaseLocation: 'Library'}, successcb, errorcb);
```

where the `iosDatabaseLocation` option may be set to one of the following choices:
- `default`: `Library/LocalDatabase` subdirectory - *NOT* visible to iTunes and *NOT* backed up by iCloud
- `Library`: `Library` subdirectory - backed up by iCloud, *NOT* visible to iTunes
- `Documents`: `Documents` subdirectory - visible to iTunes and backed up by iCloud

**WARNING:** Again, the new "default" iosDatabaseLocation value is *NOT* the same as the old default location and would break an upgrade for an app using the old default value (0) on iOS.

*ALTERNATIVE (deprecated):*
- `var db = window.sqlitePlugin.openDatabase({name: 'my.db', key: 'user-password-here', location: 1}, successcb, errorcb);`

with the `location` option set to one the following choices (affects iOS *only*):
- `0` ~~(default)~~: `Documents` - visible to iTunes and backed up by iCloud
- `1`: `Library` - backed up by iCloud, *NOT* visible to iTunes
- `2`: `Library/LocalDatabase` - *NOT* visible to iTunes and *NOT* backed up by iCloud (same as using "default")

No longer supported (see tip below to overwrite `window.openDatabase`): ~~`var db = window.sqlitePlugin.openDatabase("myDatabase.db", "1.0", "Demo", -1);`~~

**IMPORTANT:** Please wait for the 'deviceready' event, as in the following example:

```js
// Wait for Cordova to load
document.addEventListener('deviceready', onDeviceReady, false);

// Cordova is ready
function onDeviceReady() {
  var db = window.sqlitePlugin.openDatabase({name: 'my.db',
    key: 'user-password-here',
    location: 'default'
  });
  // ...
}
```

The successcb and errorcb callback parameters are optional but can be extremely helpful in case anything goes wrong. For example:

```js
window.sqlitePlugin.openDatabase({name: 'my.db',
  key: 'user-password-here',
  location: 'default'
}, function(db) {
  db.transaction(function(tx) {
    // ...
  }, function(err) {
    console.log('Open database ERROR: ' + JSON.stringify(err));
  });
});
```

If any sql statements or transactions are attempted on a database object before the openDatabase result is known, they will be queued and will be aborted in case the database cannot be opened.

**DATABASE NAME NOTES:**

- Database file names with slash (`/`) character(s) are not supported and not expected to work on any platform.
- Database file names with ASCII control characters such as tab, vertical tab, carriage return, line feed, form feed, and backspace are NOT RECOMMENDED and do not work on Windows.
- Some other ASCII characters NOT RECOMMENDED and do not work on Windows: `*` `<` `>` `?` `\` `"` `|`
- Database file names with multi-byte UTF-8 characters are NOT RECOMMENDED (very limited testing, not working on all plugin versions).
- Database file names with emojis and other 4-byte UTF-8 characters are NOT RECOMMENDED (limited testing, not working properly on all plugin versions).

**OTHER NOTES:**
- The database file name should include the extension, if desired.
- It is possible to open multiple database access handle objects for the same database.
- The database handle access object can be closed as described below.

<!-- TBD NOT WANTED for SQLCipher (how to deal with password?):
**Web SQL replacement tip:**

To overwrite `window.openDatabase`:

```Javascript
window.openDatabase = function(dbname, ignored1, ignored2, ignored3) {
  return window.sqlitePlugin.openDatabase({name: dbname, location: 'default'});
};
```
 ... -->

### iCloud backup notes

As documented in the "**A User’s iCloud Storage Is Limited**" section of [iCloudFundamentals in Mac Developer Library iCloud Design Guide](https://developer.apple.com/library/mac/documentation/General/Conceptual/iCloudDesignGuide/Chapters/iCloudFundametals.html) (near the beginning):

<blockquote>
<ul>
<li><b>DO</b> store the following in iCloud:
  <ul>
   <li>[<i>other items omitted</i>]</li>
   <li>Change log files for a SQLite database (a SQLite database’s store file must never be stored in iCloud)</li>
  </ul>
</li>
<li><b>DO NOT</b> store the following in iCloud:
  <ul>
   <li>[<i>items omitted</i>]</li>
  </ul>
</li>
</ul>
- <cite><a href="https://developer.apple.com/library/mac/documentation/General/Conceptual/iCloudDesignGuide/Chapters/iCloudFundametals.html">iCloudFundamentals in Mac Developer Library iCloud Design Guide</a>
</blockquote>

#### How to disable iCloud backup

Use the `location` or `iosDatabaseLocation` option in `sqlitePlugin.openDatabase()` to store the database in a subdirectory that is *NOT* backed up to iCloud, as described in the section below.

**NOTE:** Changing `BackupWebStorage` in `config.xml` has no effect on a database created by this plugin. `BackupWebStorage` applies only to local storage and/or Web SQL storage created in the WebView (*not* using this plugin). For reference: [phonegap/build#338 (comment)](https://github.com/phonegap/build/issues/338#issuecomment-113328140)

<!-- END iCloud backup notes -->

### Android sqlite database implementation

As described above this plugin uses SQLCipher for Android which is a non-standard SQLite implementation on Android.

**IMPORANT WARNING:** As described above: in case an application access the **same** database using multiple plugins (with or without encryption) there is a risk of data corruption ref: [litehelpers/Cordova-sqlite-storage#626](https://github.com/litehelpers/Cordova-sqlite-storage/issues/626)) as described in <http://ericsink.com/entries/multiple_sqlite_problem.html> and <https://www.sqlite.org/howtocorrupt.html>.

There is no workaround in this plugin version.

<!-- END Android sqlite database implementation -->

## SQL transactions

The following types of SQL transactions are supported by this plugin version:
- Single-statement transactions
- SQL batch transactions
- DRAFT Standard asynchronous transactions

**NOTE:** Transaction requests are kept in one queue per database and executed in sequential order, according to the HTML5/[Web SQL (DRAFT) API](http://www.w3.org/TR/webdatabase/).

**WARNING:** It is possible to request a SQL statement list such as "SELECT 1; SELECT 2" within a single SQL statement string, however the plugin will only execute the first statement and silently ignore the others. This could result in data loss if such a SQL statement list with any INSERT or UPDATE statement(s) are included. For reference: [litehelpers/Cordova-sqlite-storage#551](https://github.com/litehelpers/Cordova-sqlite-storage/issues/551)

### Single-statement transactions

Sample with INSERT:

```Javascript
db.executeSql('INSERT INTO MyTable VALUES (?)', ['test-value'], function (resultSet) {
  console.log('resultSet.insertId: ' + resultSet.insertId);
  console.log('resultSet.rowsAffected: ' + resultSet.rowsAffected);
}, function(error) {
  console.log('SELECT error: ' + error.message);
});
```

Sample with SELECT:

```Javascript
db.executeSql("SELECT LENGTH('tenletters') AS stringlength", [], function (resultSet) {
  console.log('got stringlength: ' + resultSet.rows.item(0).stringlength);
}, function(error) {
  console.log('SELECT error: ' + error.message);
});
```

**NOTE/minor bug:** The object returned by `resultSet.rows.item(rowNumber)` is **not** immutable. In addition, multiple calls to `resultSet.rows.item(rowNumber)` with the same `rowNumber` on the same `resultSet` object return the same object. For example, the following code will show `Second uppertext result: ANOTHER`:

```Javascript
db.executeSql("SELECT UPPER('First') AS uppertext", [], function (resultSet) {
  var obj1 = resultSet.rows.item(0);
  obj1.uppertext = 'ANOTHER';
  console.log('Second uppertext result: ' + resultSet.rows.item(0).uppertext);
  console.log('SELECT error: ' + error.message);
});
```

<!-- END Single-statement transactions -->

### SQL batch transactions

Sample:

```Javascript
db.sqlBatch([
  'DROP TABLE IF EXISTS MyTable',
  'CREATE TABLE MyTable (SampleColumn)',
  [ 'INSERT INTO MyTable VALUES (?)', ['test-value'] ],
], function() {
  db.executeSql('SELECT * FROM MyTable', [], function (resultSet) {
    console.log('Sample column value: ' + resultSet.rows.item(0).SampleColumn);
  });
}, function(error) {
  console.log('Populate table error: ' + error.message);
});
```

In case of an error, all changes in a sql batch are automatically discarded using ROLLBACK.

<!-- END SQL batch transactions -->

### Standard asynchronous transactions

DRAFT standard asynchronous transactions follow the HTML5/[Web SQL (DRAFT) API](http://www.w3.org/TR/webdatabase/) which is very well documented and uses BEGIN and COMMIT or ROLLBACK to keep the transactions failure-safe. Here is a simple example:

```Javascript
db.transaction(function(tx) {
  tx.executeSql('DROP TABLE IF EXISTS MyTable');
  tx.executeSql('CREATE TABLE MyTable (SampleColumn)');
  tx.executeSql('INSERT INTO MyTable VALUES (?)', ['test-value'], function(tx, resultSet) {
    console.log('resultSet.insertId: ' + resultSet.insertId);
    console.log('resultSet.rowsAffected: ' + resultSet.rowsAffected);
  }, function(tx, error) {
    console.log('INSERT error: ' + error.message);
  });
}, function(error) {
  console.log('transaction error: ' + error.message);
}, function() {
  console.log('transaction ok');
});
```

In case of a read-only transaction, it is possible to use `readTransaction` which will not use BEGIN, COMMIT, or ROLLBACK:

```Javascript
db.readTransaction(function(tx) {
  tx.executeSql("SELECT UPPER('Some US-ASCII text') AS uppertext", [], function(tx, resultSet) {
    console.log("resultSet.rows.item(0).uppertext: " + resultSet.rows.item(0).uppertext);
  }, function(tx, error) {
    console.log('SELECT error: ' + error.message);
  });
}, function(error) {
  console.log('transaction error: ' + error.message);
}, function() {
  console.log('transaction ok');
});
```

**WARNING:** It is NOT allowed to execute sql statements on a transaction after it has finished. Here is an example from the **Populating Cordova SQLite storage with the JQuery API post** at <http://www.brodybits.com/cordova/sqlite/api/jquery/2015/10/26/populating-cordova-sqlite-storage-with-the-jquery-api.html>:

```Javascript
  // BROKEN SAMPLE:
  var db = window.sqlitePlugin.openDatabase({
    name: 'my-encrypted.db',
    key: 'user-password-here',
    location: 'default'
  });
  db.executeSql("DROP TABLE IF EXISTS tt");
  db.executeSql("CREATE TABLE tt (data)");

  db.transaction(function(tx) {
    $.ajax({
      url: 'https://api.github.com/users/litehelpers/repos',
      dataType: 'json',
      success: function(res) {
        console.log('Got AJAX response: ' + JSON.stringify(res));
        $.each(res, function(i, item) {
          console.log('REPO NAME: ' + item.name);
          tx.executeSql("INSERT INTO tt values (?)", JSON.stringify(item.name));
        });
      }
    });
  }, function(e) {
    console.log('Transaction error: ' + e.message);
  }, function() {
    // Check results:
    db.executeSql('SELECT COUNT(*) FROM tt', [], function(res) {
      console.log('Check SELECT result: ' + JSON.stringify(res.rows.item(0)));
    });
  });
```

You can find more details and a step-by-step description how to do this right in the **Populating Cordova SQLite storage with the JQuery API** post at: <http://www.brodybits.com/cordova/sqlite/api/jquery/2015/10/26/populating-cordova-sqlite-storage-with-the-jquery-api.html>

**NOTE/minor bug:** Just like the single-statement transaction described above, the object returned by `resultSet.rows.item(rowNumber)` is **not** immutable. In addition, multiple calls to `resultSet.rows.item(rowNumber)` with the same `rowNumber` on the same `resultSet` object return the same object. For example, the following code will show `Second uppertext result: ANOTHER`:

```Javascript
db.readTransaction(function(tx) {
  tx.executeSql("SELECT UPPER('First') AS uppertext", [], function(tx, resultSet) {
    var obj1 = resultSet.rows.item(0);
    obj1.uppertext = 'ANOTHER';
    console.log('Second uppertext result: ' + resultSet.rows.item(0).uppertext);
    console.log('SELECT error: ' + error.message);
  });
});
```

**FUTURE TBD:** It should be possible to get a row result object using `resultSet.rows[rowNumber]`, also in case of a single-statement transaction. This is non-standard but is supported by the Chrome desktop browser.

<!-- END Standard asynchronous transactions -->

## Background processing

The threading model depends on which platform version is used:
- For Android, one background thread per db;
- for iOS/macOS, background processing using a very limited thread pool (only one thread working at a time);
- for Windows _(disabled DUE TO MISSING CRYPTO PROVIDER in this plugin version)_, no background processing.

<!-- END Background processing -->

## Sample with PRAGMA feature

Creates a table, adds a single entry, then queries the count to check if the item was inserted as expected. Note that a new transaction is created in the middle of the first callback.

```js
// Wait for Cordova to load
document.addEventListener('deviceready', onDeviceReady, false);

// Cordova is ready
function onDeviceReady() {
  var db = window.sqlitePlugin.openDatabase({
    name: 'my-encrypted.db',
    key: 'user-password-here',
    location: 'default'
  });

  db.transaction(function(tx) {
    tx.executeSql('DROP TABLE IF EXISTS test_table');
    tx.executeSql('CREATE TABLE IF NOT EXISTS test_table (id integer primary key, data text, data_num integer)');

    // demonstrate PRAGMA:
    db.executeSql("pragma table_info (test_table);", [], function(res) {
      console.log("PRAGMA res: " + JSON.stringify(res));
    });

    tx.executeSql("INSERT INTO test_table (data, data_num) VALUES (?,?)", ["test", 100], function(tx, res) {
      console.log("insertId: " + res.insertId + " -- probably 1");
      console.log("rowsAffected: " + res.rowsAffected + " -- should be 1");

      db.transaction(function(tx) {
        tx.executeSql("select count(id) as cnt from test_table;", [], function(tx, res) {
          console.log("res.rows.length: " + res.rows.length + " -- should be 1");
          console.log("res.rows.item(0).cnt: " + res.rows.item(0).cnt + " -- should be 1");
        });
      });

    }, function(e) {
      console.log("ERROR: " + e.message);
    });
  });
}
```

**NOTE:** PRAGMA statements must be executed in `executeSql()` on the database object (i.e. `db.executeSql()`) and NOT within a transaction.

<!-- END Sample with PRAGMA feature -->

## Sample with transaction-level nesting

In this case, the same transaction in the first executeSql() callback is being reused to run executeSql() again.

```js
// Wait for Cordova to load
document.addEventListener('deviceready', onDeviceReady, false);

// Cordova is ready
function onDeviceReady() {
  var db = window.sqlitePlugin.openDatabase({
    name: 'my-encrypted.db',
    key: 'user-password-here',
    location: 'default'
  });

  db.transaction(function(tx) {
    tx.executeSql('DROP TABLE IF EXISTS test_table');
    tx.executeSql('CREATE TABLE IF NOT EXISTS test_table (id integer primary key, data text, data_num integer)');

    tx.executeSql("INSERT INTO test_table (data, data_num) VALUES (?,?)", ["test", 100], function(tx, res) {
      console.log("insertId: " + res.insertId + " -- probably 1");
      console.log("rowsAffected: " + res.rowsAffected + " -- should be 1");

      tx.executeSql("select count(id) as cnt from test_table;", [], function(tx, res) {
        console.log("res.rows.length: " + res.rows.length + " -- should be 1");
        console.log("res.rows.item(0).cnt: " + res.rows.item(0).cnt + " -- should be 1");
      });

    }, function(tx, e) {
      console.log("ERROR: " + e.message);
    });
  });
}
```

This case will also works with Safari (WebKit) (with no encryption), assuming you replace `window.sqlitePlugin.openDatabase` with `window.openDatabase`.

<!-- END Sample with transaction-level nesting -->

<!-- END Sample with transaction-level nesting -->

## Close a database object

This will invalidate **all** handle access handle objects for the database that is closed:

```js
db.close(successcb, errorcb);
```

It is OK to close the database within a transaction callback but *NOT* within a statement callback. The following example is OK:

```Javascript
db.transaction(function(tx) {
  tx.executeSql("SELECT LENGTH('tenletters') AS stringlength", [], function(tx, res) {
    console.log('got stringlength: ' + res.rows.item(0).stringlength);
  });
}, function(error) {
  // OK to close here:
  console.log('transaction error: ' + error.message);
  db.close();
}, function() {
  // OK to close here:
  console.log('transaction ok');
  db.close(function() {
    console.log('database is closed ok');
  });
});
```

The following example is NOT OK:

```Javascript
// BROKEN:
db.transaction(function(tx) {
  tx.executeSql("SELECT LENGTH('tenletters') AS stringlength", [], function(tx, res) {
    console.log('got stringlength: ' + res.rows.item(0).stringlength);
    // BROKEN - this will trigger the error callback:
    db.close(function() {
      console.log('database is closed ok');
    }, function(error) {
      console.log('ERROR closing database');
    });
  });
});
```

**BUG:** It is currently NOT possible to close a database in a `db.executeSql` callback. For example:

```Javascript
// BROKEN DUE TO BUG:
db.executeSql("SELECT LENGTH('tenletters') AS stringlength", [], function (res) {
  var stringlength = res.rows.item(0).stringlength;
  console.log('got stringlength: ' + res.rows.item(0).stringlength);

  // BROKEN - this will trigger the error callback DUE TO BUG:
  db.close(function() {
    console.log('database is closed ok');
  }, function(error) {
    console.log('ERROR closing database');
  });
});
```

**SECOND BUG:** When a database connection is closed, any queued transactions are left hanging. TODO: All pending transactions should be errored whenever a database connection is closed.

**NOTE:** As described above, if multiple database access handle objects are opened for the same database and one database handle access object is closed, the database is no longer available for the other database handle objects. Possible workarounds:
- It is still possible to open one or more new database handle objects on a database that has been closed.
- It *should* be OK not to explicitly close a database handle since database transactions are [ACID](https://en.wikipedia.org/wiki/ACID) compliant and the app's memory resources are cleaned up by the system upon termination.

**FUTURE TBD:** `dispose` method on the database access handle object, such that a database is closed once **all** access handle objects are disposed.

<!-- END Close a database object -->

## Delete a database

```js
window.sqlitePlugin.deleteDatabase({name: 'my.db', location: 'default'}, successcb, errorcb);
```

with `location` or `iosDatabaseLocation` parameter *required* as described above for `openDatabase` (affects iOS/macOS *only*)

**BUG:** When a database is deleted, any queued transactions for that database are left hanging. TODO: All pending transactions should be errored when a database is deleted.

<!-- END Delete a database -->

## Database schema versions

The transactional nature of the API makes it relatively straightforward to manage a database schema that may be upgraded over time (adding new columns or new tables, for example). Here is the recommended procedure to follow upon app startup:
- Check your database schema version number (you can use `db.executeSql` since it should be a very simple query)
- If your database needs to be upgraded, do the following *within a single transaction* to be failure-safe:
  - Create your database schema version table (single row single column) if it does not exist (you can check the `sqlite_master` table as described at: http://stackoverflow.com/questions/1601151/how-do-i-check-in-sqlite-whether-a-table-exists)
  - Add any missing columns and tables, and apply any other changes necessary

**IMPORTANT:** Since we cannot be certain when the users will actually update their apps, old schema versions will have to be supported for a very long time.

<!-- END Database schema versions -->

## Use with Ionic/ngCordova/Angular

### Ionic 2

Tutorials with Ionic 2:
- <https://www.thepolyglotdeveloper.com/2016/08/using-sqlstorage-instead-sqlite-ionic-2-app/> (title is somewhat misleading, "SQL storage" *does* use this sqlite plugin)
- <https://www.thepolyglotdeveloper.com/2015/12/use-sqlite-in-ionic-2-instead-of-local-storage/> (older tutorial)

Sample for Ionic 2 wanted ref: [litehelpers/Cordova-sqlite-storage#585](https://github.com/litehelpers/Cordova-sqlite-storage/issues/585)

### Ionic 1

Tutorial with Ionic 1: <https://blog.nraboy.com/2014/11/use-sqlite-instead-local-storage-ionic-framework/>

A sample for Ionic 1 is provided at: [litehelpers / Ionic-sqlite-database-example](https://github.com/litehelpers/Ionic-sqlite-database-example)

Documentation at: <http://ngcordova.com/docs/plugins/sqlite/>

Other resource (apparently for Ionic 1): <https://www.packtpub.com/books/content/how-use-sqlite-ionic-store-data>

**NOTE:** Some Ionic and other Angular pitfalls are described above.

<!-- END Use with Ionic/ngCordova/Angular -->

# Installing

## Easy installation with Cordova CLI tool

```shell
npm install -g cordova # (in case you don't have cordova)
cordova create MyProjectFolder com.my.project MyProject && cd MyProjectFolder # if you are just starting
cordova plugin add cordova-sqlcipher-adapter # --save RECOMMENDED for Cordova CLI pre-7.0
cordova platform add <desired platform> # repeat for all desired platform(s)
cordova prepare # OPTIONAL (MAY BE NEEDED cordova-ios pre-4.3.0 (Cordova CLI pre-6.4.0))
```

**Additional Cordova CLI NOTES:**

- In case of Cordova CLI pre-7.0 it is recommended to add plugins including standard plugins such as `cordova-plugin-whitelist` with the `--save` flag to track these in `config.xml` (automatically saved in `config.xml` starting with Cordova CLI 7.0).
- In general there is no need to keep the Cordova `platforms` subdirectory tree in source code control (such as git). In case ALL plugins are tracked in `config.xml` (automatic starting with Cordova CLI 7.0, `--save` flag needed for Cordova CLI pre-7.0) then there is no need to keep the `plugins` subdirectory tree in source code control either.
- It may be necessary to use `cordova prepare` in case of cordova-ios older than `4.3.0` (Cordova CLI `6.4.0`).
- In case of problems with building and running it is recommended to try again after `cordova prepare`.
- If you cannot build for a platform after `cordova prepare`, you may have to remove the platform and add it again, such as:

```shell
cordova platform rm ios
cordova platform add ios
```

or more drastically:

```shell
rm -rf platforms
cordova platform add ios
```

<!-- END Easy installation with Cordova CLI tool -->

## Plugin installation sources

- `cordova-sqlcipher-adapter` - stable npm package version
- https://github.com/litehelpers/Cordova-sqlcipher-adapter - latest version

<!-- END Plugin installation sources -->

<!-- FUTURE TBD SUPPORT Windows build with OpenSSL CRYPTO PROVIDER ...
## Windows platform usage

This plugin can be challenging to use on Windows since it includes a native SQLite3 library that is built as a part of the Cordova app. Here are some requirements:

- It is **not** possible to use this plugin with the default "Any CPU" target. A specific CPU target must be selected.
- It is **not** allowed to manually change the app id in the Windows platform. The Windows app ID may be declared:
  - using a `windows-identity-name` attribute (ref: <http://phonegap.com/blog/2016/04/25/windows-10-and-phonegap-cli-6_1-now-on-build/>);
  - "WindowsStoreIdentityName" setting (ref: <https://cordova.apache.org/docs/en/latest/config_ref/>).
 ... -->

<!-- END Windows platform usage -->

## Installation test

### Easy installation test

Use `window.sqlitePlugin.echoTest` and/or `window.sqlitePlugin.selfTest` as described above (please wait for the `deviceready` event).

### Quick installation test

Assuming your app has a recent template as used by the Cordova create script, add the following code to the `onDeviceReady` function, after `app.receivedEvent('deviceready');`:

```Javascript
  window.sqlitePlugin.openDatabase({ name: 'hello-world.db', location: 'default' }, function (db) {
    db.executeSql("select length('tenletters') as stringlength", [], function (res) {
      var stringlength = res.rows.item(0).stringlength;
      console.log('got stringlength: ' + stringlength);
      document.getElementById('deviceready').querySelector('.received').innerHTML = 'stringlength: ' + stringlength;
   });
  });
```

<!-- END Installation test -->

# Support

## Free support policy

Free support is provided on a best-effort basis and is only available in public forums. Please follow the steps below to be sure you have done your best before requesting help.

## Professional support

Professional support is available by contacting: <sales@litehelpers.net>

For more information: <http://litehelpers.net/>

## Before seeking help

First steps:
- Verify that you have followed the steps in [brodybits / Cordova-quick-start-checklist](https://github.com/brodybits/Cordova-quick-start-checklist)
- Try the self-test functions as described above
- Check the pitfalls and troubleshooting steps in the [pitfalls](#pitfalls) section and [brodybits / Avoiding-some-Cordova-pitfalls](https://github.com/brodybits/Avoiding-some-Cordova-pitfalls) for possible troubleshooting

and check the following:
- You are using the latest version of the Plugin (Javascript and platform-specific part) from this repository.
- The plugin is installed correctly.
- You have included the correct version of `cordova.js`.
- You have registered the plugin properly in `config.xml`.

If you still cannot get something to work:
- create a fresh, clean Cordova project, ideally based on [brodybits / cordova-sqlite-test-app](https://github.com/brodybits/cordova-sqlite-test-app);
- add this plugin according to the instructions above;
- double-check that you follwed the steps in [brodybits / Cordova-quick-start-checklist](https://github.com/brodybits/Cordova-quick-start-checklist);
- try a simple test program;
- double-check the pitfalls in [brodybits / Avoiding-some-Cordova-pitfalls](https://github.com/brodybits/Avoiding-some-Cordova-pitfalls)

## Issues with AJAX

General: As documented above with a negative example the application must wait for the AJAX query to finish before starting a transaction and adding the data elements.

In case of issues *it is recommended to* rework the reproduction program insert the data from a JavaScript object after a delay. There is already a test function for this in [brodybits / cordova-sqlite-test-app](https://github.com/brodybits/cordova-sqlite-test-app).

FUTURE TBD examples

## Test program to seek help

If you continue to see the issue: please make the simplest test program possible based on [brodybits / cordova-sqlite-test-app](https://github.com/brodybits/cordova-sqlite-test-app) to demonstrate the issue with the following characteristics:
  - it completely self-contained, i.e. it is using no extra libraries beyond cordova & SQLitePlugin.js;
  - if the issue is with *adding* data to a table, that the test program includes the statements you used to open the database and create the table;
  - if the issue is with *retrieving* data from a table, that the test program includes the statements you used to open the database, create the table, and enter the data you are trying to retrieve.

## What will be supported for free

It is *recommended* to make a small, self-contained test program based on [brodybits / cordova-sqlite-test-app](https://github.com/brodybits/cordova-sqlite-test-app) that can demonstrate your problem and post it. Please do not use any other plugins or frameworks than are absolutely necessary to demonstrate your problem.

In case of a problem with a pre-populated database, please post your entire project.

## What is NOT supported for free

- Debugging, optimization, and other help with application code.

## What information is needed for help

Please include the following:
- Which platform(s) (Android/iOS/macOS, ~~Windows~~)
- Clear description of the issue
- A small, complete, self-contained program that demonstrates the problem, preferably as a Github project, based on [brodybits / cordova-sqlite-test-app](https://github.com/brodybits/cordova-sqlite-test-app). ZIP/TGZ/BZ2 archive available from a public link is OK. No RAR or other such formats please.
- In case of a Windows build problem please capture the entire compiler output.

## Please do NOT use any of these formats

- screen casts or videos
- RAR or similar archive formats
- Intel, MS IDE, or similar project formats unless absolutely necessary

## Where to request help

- [brodybits / cordova-sqlcipher-adapter / issues](https://github.com/brodybits/cordova-sqlcipher-adapter/issues)
- [litehelpers / Cordova-sqlite-help / issues](https://github.com/litehelpers/Cordova-sqlite-help/issues)

Please include the information described above otherwise.

<!-- END Support -->

# Unit tests

Unit testing is done in `spec`.

## running tests from shell

**TBD** `test.sh` testing limited with sqlcipher version of this plugin, *does not auto-remove correct plugin id*

To run the tests from \*nix shell, simply do either:

    ./bin/test.sh ios

or for Android:

    ./bin/test.sh android

To run from a windows powershell (here is a sample for android target):

    .\bin\test.ps1 android

<!-- END Unit tests -->

# Adapters

**GENERAL:** The adapters described here are community maintained.

## Lawnchair Adapter

- [litehelpers / cordova-sqlite-lawnchair-adapter](https://github.com/litehelpers/cordova-sqlite-lawnchair-adapter)

**POSSIBLY BROKEN:** The Lawnchair adapter ~~does~~ *may* not support *all* `openDatabase` options such as `key`, `location` or `iosDatabaseLocation` options and is therefore *not* ~~expected~~ *guaranteed* to work with this plugin.

## PouchDB

- [nolanlawson / pouchdb-adapter-cordova-sqlite](https://github.com/nolanlawson/pouchdb-adapter-cordova-sqlite)

## Adapters wanted

- IndexedDBShim adapter (possibly based on IndexedDBShim)

<!-- END Adapters -->

# Sample

Contributed by [@Mikejo5000 (Mike Jones)](https://github.com/Mikejo5000) from Microsoft.

## Interact with the SQLite database

The SQLite storage plugin sample allows you to execute SQL statements to interact with the database. The code snippets in this section demonstrate simple plugin tasks including:

* [Open the database and create a table](#openDb)
* [Add a row (record) to the database](#addRow)
* [Read rows from the database](#readRow) that match a column value
* [Remove a row from the database](#removeRow) that matches a column value
* [Update rows in the database](#updateRow) that match a column value
* [Close the database](#closeDb)

## Open the database and create a table <a name="openDb"></a>

Call the `openDatabase()` function to get started, passing in the name and location for the database.

```Javascript
var db = window.sqlitePlugin.openDatabase({ name: 'my.db', key: 'user-password-here', location: 'default' }, function (db) {

    // Here, you might create or open the table.

}, function (error) {
    console.log('Open database ERROR: ' + JSON.stringify(error));
});
```

Create a table with three columns for first name, last name, and a customer account number. If the table already exists, this SQL statement opens the table.

```Javascript
db.transaction(function (tx) {
    // ...
    tx.executeSql('CREATE TABLE customerAccounts (firstname, lastname, acctNo)');
}, function (error) {
    console.log('transaction error: ' + error.message);
}, function () {
    console.log('transaction ok');
});
```

By wrapping the previous `executeSql()` function call in `db.transaction()`, we will make these tasks asynchronous. If you want to, you can use multiple `executeSql()` statements within a single transaction (not shown).

## Add a row to the database <a name="addRow"></a>

Add a row to the database using the INSERT INTO SQL statement.

```Javascript
function addItem(first, last, acctNum) {

    db.transaction(function (tx) {

        var query = "INSERT INTO customerAccounts (firstname, lastname, acctNo) VALUES (?,?,?)";

        tx.executeSql(query, [first, last, acctNum], function(tx, res) {
            console.log("insertId: " + res.insertId + " -- probably 1");
            console.log("rowsAffected: " + res.rowsAffected + " -- should be 1");
        },
        function(tx, error) {
            console.log('INSERT error: ' + error.message);
        });
    }, function(error) {
        console.log('transaction error: ' + error.message);
    }, function() {
        console.log('transaction ok');
    });
}
```

To add some actual rows in your app, call the `addItem` function several times.

```Javascript
addItem("Fred", "Smith", 100);
addItem("Bob", "Yerunkle", 101);
addItem("Joe", "Auzomme", 102);
addItem("Pete", "Smith", 103);
```

## Read data from the database <a name="readRow"></a>

Add code to read from the database using a SELECT statement. Include a WHERE condition to match the resultSet to the passed in last name.

```Javascript
function getData(last) {

    db.transaction(function (tx) {

        var query = "SELECT firstname, lastname, acctNo FROM customerAccounts WHERE lastname = ?";

        tx.executeSql(query, [last], function (tx, resultSet) {

            for(var x = 0; x < resultSet.rows.length; x++) {
                console.log("First name: " + resultSet.rows.item(x).firstname +
                    ", Acct: " + resultSet.rows.item(x).acctNo);
            }
        },
        function (tx, error) {
            console.log('SELECT error: ' + error.message);
        });
    }, function (error) {
        console.log('transaction error: ' + error.message);
    }, function () {
        console.log('transaction ok');
    });
}
```

## Remove a row from the database <a name="removeRow"></a>

Add a function to remove a row from the database that matches the passed in customer account number.

```Javascript
function removeItem(acctNum) {

    db.transaction(function (tx) {

        var query = "DELETE FROM customerAccounts WHERE acctNo = ?";

        tx.executeSql(query, [acctNum], function (tx, res) {
            console.log("removeId: " + res.insertId);
            console.log("rowsAffected: " + res.rowsAffected);
        },
        function (tx, error) {
            console.log('DELETE error: ' + error.message);
        });
    }, function (error) {
        console.log('transaction error: ' + error.message);
    }, function () {
        console.log('transaction ok');
    });
}
```

## Update rows in the database <a name="updateRow"></a>

Add a function to update rows in the database for records that match the passed in customer account number. In this form, the statement will update multiple rows if the account numbers are not unique.

```Javascript
function updateItem(first, id) {
    // UPDATE Cars SET Name='Skoda Octavia' WHERE Id=3;
    db.transaction(function (tx) {

        var query = "UPDATE customerAccounts SET firstname = ? WHERE acctNo = ?";

        tx.executeSql(query, [first, id], function(tx, res) {
            console.log("insertId: " + res.insertId);
            console.log("rowsAffected: " + res.rowsAffected);
        },
        function(tx, error) {
            console.log('UPDATE error: ' + error.message);
        });
    }, function(error) {
        console.log('transaction error: ' + error.message);
    }, function() {
        console.log('transaction ok');
    });
}
```

To call the preceding function, add code like this in your app.

```Javascript
updateItem("Yme", 102);
```

## Close the database <a name="closeDb"></a>

When you are finished with your transactions, close the database. Call `closeDB` within the transaction success or failure callbacks (rather than the callbacks for `executeSql()`).

```Javascript
function closeDB() {
    db.close(function () {
        console.log("DB closed!");
    }, function (error) {
        console.log("Error closing DB:" + error.message);
    });
}
```

<!-- END Sample -->

## Source tree

- `SQLitePlugin.coffee.md`: platform-independent (Literate CoffeeScript, can be compiled with a recent CoffeeScript (1.x) compiler)
- `www`: platform-independent Javascript as generated from `SQLitePlugin.coffee.md` using `coffeescript@1` (and committed!)
- `src`: platform-specific source code
- `node_modules`: placeholder for external dependencies
- `scripts`: installation hook script to fetch the external dependencies via `npm`
- `spec`: test suite using Jasmine (`2.4.1`)
- `tests`: very simple Jasmine test suite that is run on Circle CI (Android platform) and Travis CI (iOS platform) (used as a placeholder)

<!-- END Source tree -->

# Contributing

## Community

- Testimonials of apps that are using this plugin would be especially helpful.
- Reporting issues in [litehelpers / Cordova-sqlcipher-adapter / issues](https://github.com/litehelpers/Cordova-sqlcipher-adapter/issues) can help improve the quality of this plugin.

## Code

**WARNING:** Please do NOT propose changes from your default branch. Contributions may be rebased using `git rebase` or `git cherry-pick` and not merged.

- Patches with bug fixes are helpful, especially when submitted with test code.
- Other enhancements welcome for consideration, when submitted with test code and are working for all supported platforms. Increase of complexity should be avoided.
- All contributions may be reused by [@brodybits](https://github.com/brodybits) under another license in the future. Efforts will be taken to give credit for major contributions but it will not be guaranteed.
- Project restructuring, i.e. moving files and/or directories around, should be avoided if possible.
- If you see a need for restructuring, it is better to first discuss it in new issue where alternatives can be discussed before reaching a conclusion. If you want to propose a change to the project structure:
  - Remember to make (and use) a special branch within your fork from which you can send the proposed restructuring;
  - Always use `git mv` to move files & directories;
  - Never mix a move/rename operation with any other changes in the same commit.

<!-- END Contributing -->

## Contact

<sales@litehelpers.net>
