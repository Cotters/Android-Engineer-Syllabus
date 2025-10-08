
Android provides a disk-based file system, like any OS such as Mac, Windows, or in this case, Linux. There are different options for different reasons - let's dig in.

## App / User Preferences

If you need to store app or user settings, such as light/dark mode preference, language, colour schemes, or developer features, you want to use Preferences.

Preferences uses a private (to your app) SharedPreferences and it's great for non-sensitive data. In Android Views, you can use `PreferenceFragment` along with Preference views such as `SwitchPreferenceCompat` or `EditTextPreference`.
It Compose, it looks like you'll need to create your own version of this.

## DataStore

Ideal for small and simple datasets.

DataStore is part of the *Android Jetpack*, and aims at replacing SharedPreferences. DataStore uses coroutines and Flow for asynchronous reading and writing.

One reason for migrating from SharedPreferences to DataStore is for asynchronous and thread-safe functionality. From Android docs:

> SharedPreferences has a synchronous API that can appear safe to call on the UI thread, but which actually does disk I/O operations. Furthermore, `apply()` blocks the UI thread on `fsync()`. Pending `fsync()` calls are triggered every time any service starts or stops, and every time an activity starts or stops anywhere in your application. The UI thread is blocked on pending `fsync()` calls scheduled by `apply()`, often becoming a source of [ANRs](https://developer.android.com/topic/performance/vitals/anr).

DataStore, on the other hand moves to Dispatchers.IO under the hood and therefore won't block the main thread.

**Question**: Why is Proto DataStore type safe and the default Preferences DataStore isn't?

