We publish the wTest tool and the evaluation results in this page.

# Tool
Our tool is composed of 2 parts. The first part instruments an Android app. The second part generates tests for the instrumented app. But before testing an app, a customized Android OS needs to be loaded by Android emulators or real devices.

## Customized Android OS
The instrumented app needs to run on a customized Android system because we add one more ID field in `java.lang.Object` in order to ease the variable tracking. It is very simple to use our customized Android system image, you just need to follow the following 2 steps
* Download our customized [system.img](), [ramdisk.img](), [VerifiedBootParams.textproto]() (They are for Android 10)
* Replace the original `system.img`, `ramdisk.img`, `VerifiedBootParams.textproto` in your Android SDK with the downloaded files

Usually, these files are placed under `<Android_SDK_Root>/system-images/android-29/google_apis/x86`

Once you have finished the above two steps, you can try to start the Android emulator as usual.

## Instrumentation tool ([download]())
### Requirements
* Java 1.8 or above
* Python 2 or 3 (We have tested on Python 2.7.16 and Python 3.6.8)
* Node.js (We have tested on v16.13.2, v12.16.2, and v10.16.0. You may find these versions [here](https://nodejs.org/en/download/releases/). If you are using a Linux-based OS, you may install node by following the steps [here](https://www.digizol.com/2017/08/nodejs-install-no-root-sudo-permission-linux-centos.html))

### Steps to use
* Create a folder for an app under test. The folder name should be the app's package name (We have created a folder for Wikipedia app under `<path_to_Instrumentation>/apps`).
* Run `sh instrument.sh APP_FOLDER APP_PACKAGE_NAME PORT_NUM APP_TYPE` to instrument an app. `APP_TYPE` can be `OPEN_SOURCE` or `CLOSE_SOURCE`. `PORT_NUM` is a port used by the instrumented app to communicate with a nodejs server to instrument dynamically-loaded JavaScript code. It is recommended to be set as 3016, 3018, 3020, ..., etc. An example command can be `sh instrument.sh apps/org.wikipedia org.wikipedia 3016 OPEN_SOURCE`).
* When instrumentation finishes, you can see an `output` folder under the `APP_FOLDER`. The apk file whose name ends with `-aligned-debugSigned.apk` is the instrumented apk.

## Test generation (wTest) ([download]())
### requirements
* `export JAVA_HOME=YOUR_JAVA_HOME` (example: `export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_121.jdk/Contents/Home/`)
* `export ANDROID_HOME=YOUR_ANDROID_HOME` (example: `export ANDROID_HOME=/Library/Android/sdk`)
* `export PATH=$PATH:${ANDROID_HOME}`
* `export PATH=$PATH:<path_to_wTest>/node_modules/.bin` (make sure you have downloaded `wTest`)
* `export PATH=$PATH:${ANDROID_HOME}/emulator`
* `export PATH=$PATH:${ANDROID_HOME}/platform-tools`

### Steps to use
* Enter `<path_to_Instrumentation>/js` and run `sh launchServer.sh PORT_NUM`. This nodejs server is registered on a `PORT_NUM` in order to recevice JavaScript code that is dynamically constructed in the app under test. The server is responsible for instrumenting the js code and sending the instrumented code back to the app. `PORT_NUM` should be the same as the one used when instrumenting the app. An example command can be `sh launchServer.sh 3016`
* Launch Appium by `appium -p APPIUM_PORT` (example: `appium -p 4723`)
* Launch an Android emulator
* Enter `<path_to_wTest>` and run `sh wTest.sh STRATEGY PATH_TO_APP_FOLDER TIME_LIMIT ANDROID_SDK_PATH EMULATOR_ID APPIUM_PORT APP_TYPE`. `STRATEGY` can be `wVar` (stands for wTest), or `wDroid`, or `api` (stands for wAPI), or `QTesting`, or `ComboDroid`, or `Fastbot` (stands for Fastbot2). An example command can be 
`sh test.sh wVar <path_to_Instrumentation>/apps/org.wikipedia 3600 ${ANDROID_HOME} emulator-5554 4723 OPEN_SOURCE`
* If you want to run Q-Testing, you need to run `sh wTest.sh STRATEGY PATH_TO_APP_FOLDER TIME_LIMIT ANDROID_SDK_PATH EMULATOR_ID APPIUM_PORT APP_TYPE Q_Testing_root Q_Testing_config_path AVD_NAME`. The last 3 arguments stands for the path to the Q-Testing folder, the path to the config file required by Q-Testing, and the Android emulator name, respectively. For instance, you can change the example command to be `sh wTest.sh wVar <path_to_Instrumentation-ICSE22-Submit>/apps/org.wikipedia 3600 ${ANDROID_HOME} emulator-5554 4723 OPEN_SOURCE <path_to_QTesting_root> <path_to_QTesting_config_file> emulator0`
* If you want to run ComboDroid, you need to run `sh wTest.sh STRATEGY PATH_TO_APP_FOLDER TIME_LIMIT ANDROID_SDK_PATH EMULATOR_ID APPIUM_PORT APP_TYPE NULL NULL NULL ComboDroid_config_path`. The last argument stands for the path to the config file required by ComboDroid. For instance, you can change the example command to be `sh wTest.sh wVar <path_to_Instrumentation-ICSE22-Submit>/apps/org.wikipedia 3600 ${ANDROID_HOME} emulator-5554 4723 OPEN_SOURCE NULL NULL NULL <path_to_ComboDroid_config_file>`
* When reaching the time limit, you can see the output under `<path_to_wTest>/output/<STRATEGY>/<APP_PACKAGE_NAME>` (e.g., `<path_to_wTest>/output/wVar/org.wikipedia`). Under this folder, `coverage.txt` contains WebView-specific property coverage and code/method coverage results. In `coverage.txt`, you may see lines starting with `time0` (stands for 10mins), `time1` (stands for 20mins), ..., `time5` (stands for 60mins).
