
### AndroidManifest.xml

Defines all the components (activities) used in the app, as well as permissions, intents, API levels, and hardware features.

### ADB

Android Debug Bridge; used to communicate with Android devices and emulators. Can be used to search files, simulate hardware events such as touching the screen or rotating the device.

### Gradle

Module level `build.gradle` defines the build configurations. You can also define tasks to run from the ‘gradle’ pane.
Module/App level `build.gradle`  defines all the dependencies and SDK versions - think `minSdkVersion` or `compileSdkVersion`.

### APK & AAB

Android Application Package (.apk). Used to distribute Android apps.
As of recent, you should now submit Android App Bundles (.aab) to the Play Store, not APKs. The Play Store servers are responsible for generating smaller, more device-specific APKs when the app is downloaded.
~Android Asset Packaging Tool~ (AAPT) builds the ‘.apk’ distributable Android package file.

### Android Runtime (ART)

The runtime environment. It is an application used by the OS.
Replaced Dalvik. ART uses AoT[^1] compilation.
ART translates the bytecode of the application into instructions.

> **Ahead-of-time compilation** (**AOT compilation**) is the act of compiling code into a lower-level language before execution, usually at build-time, to reduce the amount of work needed to be performed at run time.

[^1]: Ahead of time compiler. Android runtime converts the app's bytecode into machine code during the app installation phase. This results in faster start-up time and overall app performance.
### ANR

Application Not Responding. After 5 seconds of being unresponsive, the app shows an ANR which is also logged and displayed in the Play Store Console.

### Task

A task manages activities, like a bucket: Stack (LIFO) structure of activities. 
An application can have multiple tasks.
Different apps’ activities can be managed in one stack. Opening Mail app from your app to send an email is an example of this. It opens the ‘Compose’ activity in Mail, but all within one task. Alternatively, you may launch it as `singleTask` where a new task is made for that activity, and it appears to launch another (Mail) application rather than showing that screen within your app.
Tasks remain in memory even if the app is not in foreground but may be cleaned up by the OS.

#### Launch Modes
> In `singleTask`, a new Task is launched and that new task can contain other activities as well. 
> In `singleInstance`, a new Task is launched and that cannot contain other activities.

**SingleTop** reuses already launched activities if it exists, or default.
**Default** always adds an activity to the task stack, regardless.

### Process

When an app starts, it create a new Linux process, with a single thread of execution - this is your application. 
It may already exist, and therefore when the app launches, it may continue to use the existent thread. 
You can arrange different components to use different threads - however, this is not necessary for most applications. Creating too many threads can lead to priority and scheduling issues which the CPU might struggle to handle.

### Threads

Main thread (UI thread) dispatches events to the appropriate UI widgets.
All components in the same process run on the same thread.

### Intents

Messaging objects. Used to request an action from another app component, start an activity, start a service, or broadcast.
Pass intent to `startActivity`. View: show data, Send: share data with another app.
You can: start the device, start a specific activity, and start a broadcast.
**Explicit Intent**: Directly identifies the target component.
**Implicit Intent**: No defined target component, instead the user can decide which App to use.

### Pending Intents

When one app wants another app to perform a certain action in the future on it's behalf.
My use case is for NFC reading: the app listens for tags being read by the device, and sets a pending intent for any 'discovered' tags. The pending intent is handled once a tag is discovered, and in my case, it started a certain activity. 
### Content Providers

Store data and make this data available for other applications.
*Example*: Calendar ContentProvider, allows you to edit the calendar.
*Example*: Read your Phone’s contacts and add them to WhatsApp.

### Context

Provides information about the application, access to resources and classes, as well as application-level operations such as launching activities.
*Example*: Use context to get INPUT_METHOD_SERVICE to hide the keyboard.

### Activity

An Activity is the entry point for the user. An Activity contains and displays the application’s UI. Each Activity might be responsible for a single screen or a user-journey as an Activity can display one or many Fragments. You can navigate to other Activities using Intents.

### Activity Lifecycle

TODO
### Fragment

A Fragment also contains and displays the application’s UI. Fragments are part or a portion of an Activity, and can be reused. You display fragments using transactions and these can be part of the back stack, or they can be ignored. For example, if you have a search screen, you wouldn't want a new fragment for each search, you'd just replace the existing one. Then, after several searches when you tap back, you go to the screen before the search screen, and not back to other search results.

### Broadcast receivers

Broadcast receivers listen to events that are broadcasted to any app that is listening. These events could be from other apps or from the system.
An example; A VPN app might listen for the ‘BOOTED’ event, to the connect to a server. This ensures that the device is always connected to a VPN.

### Background Worker

Guarantees that tasks are completed in an efficient way.
*Example*:  Note taking app that should *sync* your notes with the server to ensure you have access to them anywhere.
Tasks still execute even when the app is in background, or even closed.

### Storage

**Shared Preferences** : Key-Value storage.
**Internal Storage** : Device’s inner memory.
**SQLite** : Free open-sauce. The *Room* library wraps SQLite nicely.
Otherwise a remote server, such as Firebase Firestore, or your own service.

### How can you mitigate memory leaks?

* You can use profiler on the app to understand how well memory is utilised.
* Use Application Context instead of Activity Context as Activities tend to disappear.
* Never use AsyncTask - this holds a reference to the activity for a long time!
	* Use other, better, frameworks such as *RxJava* or *Kotlin Coroutines*.

### Locale

Use formatters instead of hard-coding.
Use `resConfigs` in the module `build.gradle` to specify the languages the app supports.
You can query using `LocaleList` to provide more complex behaviour around providing locale support.
Android has a ‘resource resolution’ which basically looks for resources in all the languages the user can understand, so if their preferred language isn’t supported, but another language they understand is, Android will select the other language to ensure they can understand.

### Security

- Keeping things (like dependencies) updated.
- Use of biometric identification for login and other information-sensitive areas of the app.
- Hiding sensitive screens when in app switcher.
- Opt out of clear-text traffic using network config files.
- Use SSL traffic (HTTPS URLs) in the app.
	- Should be impossible to do otherwise with the above.
- Allow the user to choose the app when using intents.
- Do not use JavaScript in WebViews unless strictly necessary.
	- I had to migrate from JavaScript interfaces to HTML message channels as these are the recommended alternative.
- Make sure you’re using the most up-to-date encryption methods.
	- Believe this is used for notifications handling.
- Store private data within internal storage, as this is sandboxed and other apps cannot access it. Additionally, it is deleted when the app is uninstalled.
	- This ties into using up-to-date encryption methods.
- Non-exported content providers.
- Use `DataStore` for a modern data storage solution.
	- No reason to use `SharedPreferences`, but it did have a private mode using `MODE_PRIVATE` that is worth knowing about.

#jobs #android