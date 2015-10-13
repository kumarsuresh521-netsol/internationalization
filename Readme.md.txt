Download the latest stable release of Angular Translate and copy the angular-translate.min.js file into your Ionic project at the following path:

Add Library To ProjectShell
1
/ExampleProject/www/js/angular-translate.min.js
When this has been completed, the library must referenced in your index.html file.  Your file should look something like this at the top:

index.htmlXHTML
1
2
3
4
5
6
7
8
9
10
11
12
13
14
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="initial-scale=1, maximum-scale=1, user-scalable=no, width=device-width">
        <title></title>
        <link href="lib/ionic/css/ionic.css" rel="stylesheet">
        <link href="css/style.css" rel="stylesheet">
        <script src="lib/ionic/js/ionic.bundle.js"></script>
        <script src="js/angular-translate.min.js"></script>
        <script src="js/app.js"></script>
        <script src="cordova.js"></script>
    </head>
</html>
Now things are going to get a little more interesting.  We need to add Angular Translate into our AngularJS application module and config.  This will not only make it usable, but easy to control as well.

app.jsJavaScript
1
2
3
4
5
6
7
8
9
10
11
12
13
var exampleApp = angular.module('example', ['ionic', 'pascalprecht.translate'])
    .config(function($stateProvider, $urlRouterProvider, $translateProvider) {
        $translateProvider.translations('en', {
            hello_message: "Howdy",
            goodbye_message: "Goodbye"
        });
        $translateProvider.translations('es', {
            hello_message: "Hola",
            goodbye_message: "Adios"
        });
        $translateProvider.preferredLanguage("en");
        $translateProvider.fallbackLanguage("en");
    });
You’ll see above that I created two language tables.  As simple as they may be, one is for English and one is for Spanish.  The preferred language, which will be chosen on application start is English and the fallback in case a translation does not exist, will be English as well.

If at any time you want to change languages during run-time, it can be done in your controller via the $translate.use() method.  If you want to change to Spanish you can use $translate.use(“es”) and all text linking to your translation tables will be changed.

So how do we link to the translation tables in our views?  This is pretty easy.  See the following example code:

example.htmlXHTML
1
2
3
4
5
6
<ion-view title="Example">
    <ion-content>
        <h1>{{"hello_message" | translate}}</h1>
        <h2>{{"goodbye_message" | translate}}</h2>
    </ion-content>
</ion-view>
Every time the language changes, so will anything formatted like above.  You might be asking how your Ionic app knows what language the device is set for.  The short answer is, your app by default has no idea what language to use.  To figure out which language the device is using, you’ll need to download and install the Apache Globalization Plugin for Cordova.

In the root of your Ionic project, with the Android and iOS platforms already installed, run the following command:

Install GlobalizationShell
1
cordova plugin add https://git-wip-us.apache.org/repos/asf/cordova-plugin-globalization.git
This plugin offers all kinds of useful internationalization tools, but we are only interested in the getPreferredLanguage and getLocaleName functions.

Adding the following to your module’s run method will result in the preferred language being found and then an application language being chosen:

app.jsJavaScript
1
2
3
4
5
6
7
8
9
if(typeof navigator.globalization !== "undefined") {
    navigator.globalization.getPreferredLanguage(function(language) {
        $translate.use((language.value).split("-")[0]).then(function(data) {
            console.log("SUCCESS -> " + data);
        }, function(error) {
            console.log("ERROR -> " + error);
        });
    }, null);
}
We added some split logic to the above example because getPreferredLanguage will return values such as en-US when we only care about the first two characters.

Here is a fuller example to give you a better idea:

app.jsJavaScript
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
var exampleApp = angular.module('example', ['ionic', 'pascalprecht.translate'])
    .config(function($stateProvider, $urlRouterProvider, $translateProvider) {
        $translateProvider.translations('en', {
            hello_message: "Howdy",
            goodbye_message: "Goodbye"
        });
        $translateProvider.translations('es', {
            hello_message: "Hola",
            goodbye_message: "Adios"
        });
        $translateProvider.preferredLanguage("en");
        $translateProvider.fallbackLanguage("en");
    })
    .run(function($ionicPlatform, $translate) {
        $ionicPlatform.ready(function() {
            if(typeof navigator.globalization !== "undefined") {
                navigator.globalization.getPreferredLanguage(function(language) {
                    $translate.use((language.value).split("-")[0]).then(function(data) {
                        console.log("SUCCESS -> " + data);
                    }, function(error) {
                        console.log("ERROR -> " + error);
                    });
                }, null);
            }
        });
    });
It is important to note that Cordova plugins do not work in PC web browsers.  To have the language automatically determined with the globalization plugin, the Ionic project must be run on an Android or iOS device.  However, Angular Translate will work fine on anything.