
Baseline Profiles can be used to drastically improve start up time of an app or loading times of popular routes.

Play Store uses 'Cloud Profiles' to improve runtime speeds by aggregating data of your app while it's being used. Cloud Profiles are simply a list of the pieces of code your app uses during start up. This list is shipped with subsequent downloads of the app to improve it's performance. At install time, Android will use this file to pre-compile the listed classes and methods - as apposed to the 'just-in-time' compilation that would otherwise happen.

However, when you release a new version of your app (a new release every 1 or 2 weeks is very common in the industry), Play Store needs to construct a new Cloud Profile and we're back to square one. To address this, we can use Baseline Profiles, where we measure the app ourselves, and ship a file to optimise the load times certain paths.

> Baseline Profiles require a profile build, as the standard debugging build is not profile-able.

Use the *Macrobenchmark* library provided by Android to test and measure performance in the Android app.