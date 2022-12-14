# Firebase app distribution 

Firebase App Distribution makes distributing your apps to trusted testers painless. By getting your apps onto testers' devices quickly, you can get feedback early and often. And if you use Crashlytics in your apps, you’ll automatically get stability metrics for all your builds, so you know when you’re ready to ship.


# Github Action
GitHub Actions is a continuous integration and continuous delivery (CI/CD) platform that allows you to automate your build, test, and deployment pipeline. You can create workflows that build and test every pull request to your repository, or deploy merged pull requests to production.


# Setup Project 

- Create a new flutter projects.
- Add firebase to your projects(do not forgot to add firebase keys files of android,ios,web in .gitignore ).
- goto firebase app distribution and create a new group and add the testers in the created group.
- Push your code to the github.
- Create following **Secrets** 
    - ***APPID*** : Add your firebase app id(Android)
    - ***FIREBASE_OPTION*** : Copy paste content of **/lib/firebase_options.dart**.
    - ***TOKEN*** : Add your firebase tokken, get from following command
            ```
            firebase login:cli
            ```
# Create Action

Create a new action and following content in the file:

```
  name: Build & upload to Firebase App Distribution

on:
  push:
    branches:
      - main

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-java@v2
        with:
          distribution: "zulu"
          java-version: "11"
      - name: Decode google-services.json
        env: 
            GOOGLE_SERVICES_JSON: ${{ secrets.GOOGLE_SERVICES_JSON }}
        run: echo "$GOOGLE_SERVICES_JSON" > android/app/google-services.json
      - name: Decode firebase_option
        env:
          firebase_options_dart: ${{secrets.FIREBASE_OPTION}}
        run : echo "$firebase_options_dart" > lib/firebase_options.dart

      - uses: subosito/flutter-action@v2
        with:
          channel: "stable"
      - run: flutter pub get
      - run: flutter build apk
      - uses: actions/upload-artifact@v1
        with:
          name: release-apk
          path: build/app/outputs/apk/release/app-release.apk
      - name: upload artifact to Firebase App Distribution
        uses: wzieba/Firebase-Distribution-Github-Action@v1
        with:
            appId: ${{secrets.APPID}}
            token: ${{secrets.TOKEN}}
            groups: pre-tester
            file: build/app/outputs/apk/release/app-release.apk

```


# Note do not forgot to change ***groups*** name in above code with your group name.


