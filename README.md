# React Native 0.62.2 Typescript Workspaces Starter

React Native is full of hardcoded dependencies on the location of `node_modules` making it more cumbersome to set-up a package that uses shared code in a yarn workspaces mono repo. This 0.62.2 starter repo eliminates many of the error prone [manual steps](#References) required to hoist a React Native package.

_Caveat: I created this repo for myself after prepping react native packages for a mono repo a couple of times. I decided to share the code along with enough documentation to help update it in the future or to help others roll their own, e.g. perhaps you want to use JavaScript instead of TypeScript. Don't expect support for this project, updates will be dependent on my future React Native needs._

## Usage

These details are for a Mac (or another bash-like shell):

1. Clone the `rnwsstarter` repo to the same directory where you have your yarn workspaces repo. Open a terminal window to that directory and copy `rnwsstarter` to your packages folder:

   ```bash
   cp -r rnwsstarter mymono/packages/myrnproject
   ```

   Be sure you have no hypens in the name. This is a React Native limitation that you can work around with some searching and replacing later but these steps assume a React Native compatible name.

1. Open `myrnproject` (your whole mono repo is fine too) in your code editor and do a global replace of `rnwsstarter` with `myrnproject`.

1. Rename `rnwsstarter` in filenames in the ios folder with `myrnproject`. These are in the ios `rnwsstarter.xcodeproj/xcshareddata/xcschemes` subfolder, the `rnwsstarterTests` subfolder and in the ios root.

1. Make sure all packages in your mono repo are using the same version of react as this repo. i.e. All of your `package.json` files that use `react` use `16.13.1`:

   ```json
   "react": "16.13.1",
   ```

   Just like adding other packages to your repo, you'll likely avoid other headaches if you match the versions of other packages as well. Unlike the `react` situation, it's often not as critical and you may be able to modify the version these versions in `myrnproject` instead. Here's the major packages in your `myrnproject` for reference:

   ```json
    "eslint": "^6.8.0",
    "jest": "^25.1.0",
    "typescript": "^4.0.3",
   ```

   _Note: See [React Native — Monorepos & Code Sharing](https://engineering.brigad.co/react-native-monorepos-code-sharing-f6c08172b417) if you need multiple versions of React Native or other packages._

1. Add a `react-native` dependency to the root `package.json` of your mono repo to make sure the `react-native` package is hoisted to the top, e.g. add this if you have no other dependencies:

   ```json
   "dependencies": {
      "react-native": "*"
   }
   ```

1. Delete `.git`, install node modules, pods and run:

   ```bash
   cd mymono/packages/myrnproject
   rm -rf .git
   yarn
   cd ios && pod install && cd ..
   yarn ios # and/or yarn android
   ```

## Yarn 2

This project is for only intended for Yarn 1. Yarn 2 provides a [node-modules plugin](https://github.com/yarnpkg/berry/tree/master/packages/plugin-node-modules) that can be used to get React Native working with Yarn 2 (Berry) but, at this point in time, I don't know if you can work around any issues that arise or you'll end up stuck with the [nohoist](https://classic.yarnpkg.com/blog/2018/02/15/nohoist/) approach for React Native.

## Recreating this repo

This repo was created by largely following the [Tutorial: How to share code between iOS, Android & Web using React Native, react-native-web and monorepo](https://dev.to/brunolemos/tutorial-100-code-sharing-between-ios-android--web-using-react-native-web-andmonorepo-4pej) post.
After doiing this a couple of times the process deviated in some minor ways:

- The search and replace for `node_modules/react-native/` with `../../node_modules/react-native/` was extended to replace every `node_modules` with `../../node_modules/`. This picks up coverage of the `Podfile`, `node_modules/hermes-engine` and the `node_modules/@react-native-community` paths.
- Pods now need to be installed.
- I used a slightly different order to eliminate a couple of steps.

This process is generally detailed here to make it easier to recreate with new versions of React Native.

1. Get a `react-native` project built outside of the mono repo:

   ```bash
   cwd _someplace out of the mono repo_
   npx react-native init rnwsstarter --template react-native-template-typescript
   cd rnwsstarter
   rm -rf node_modules
   rm yarn.lock
   code .
   ```

1. Do a global search and replace of `../node_modules` with `../../../node_modules`. It's a good idea to look through what's being changed but it was a global replace for this run.

1. Copy the project to the packages folder of the mono repo.

1. Open packages/rnwsstarter/metro.config.js and set the projectRoot field on it as well so it looks like this:

   ```javascript
   const path = require('path'); // <- add this line

   module.exports = {
     projectRoot: path.resolve(__dirname, '../../'), // <- add this line
     transformer: {
       getTransformOptions: async () => ({
         transform: {
           experimentalImportSupport: false,
           inlineRequires: false,
         },
       }),
     },
   };
   ```

1. Add a `react-native` dependency to the root `package.json` of your mono repo to make sure the `react-native` package is hoisted to the top, e.g. add this if you have no other dependencies:

   ```json
   "dependencies": {
      "react-native": "*"
   }
   ```

1. Go to the mono root dir and run `yarn`

1. Open the project in XCode.

1. In `AppDelegate.m`, find `jsBundleURLForBundleRoot:@"index"` and replace `index` with `packages/rnwsstarter/index`

1. In Xcode, click on your project name on the left, and then go to `Build Phases > Bundle React Native code and Images`. Add this to the shell script line:

   ```bash
   export EXTRA_PACKAGER_ARGS="--entry-file packages/rnwsstarter/index.js"
   ```

   So the entry looks like:

   ```bash
   export NODE_BINARY=node
   export EXTRA_PACKAGER_ARGS="--entry-file packages/rnwsstarter/index.js"
   ../../../node_modules/react-native/scripts/react-native-xcode.sh
   ```

1. Reinstall the Pods from the `rnwsstarter` package folder:

   ```bash
   cd ios
   rm -rf Pods
   pod install
   ```

1. Give it a whirl and if it works IOS is done:

   ```bash
   cd ..
   yarn ios
   ```

1. In Android Studio, open the `android` folder and in the `Gradle Scripts` module app `build.gradle` Search for `project.ext.react = [...]` and add this to its contents:

   ```javascript
   entryFile: "packages/rnwsstarter/index.js",
   root: "../../../../"
   ```

   Be sure to do a `Sync Now` from the refactor lightbulb indicator after the change is saved.

1. In `android/app/src/main/java/com/rnwsstarter/MainApplication.java`. Search for the `getJSMainModuleName` method. Replace the return of `index` with `packages/rnwsstarter/index`.

1. Run in the `rnwsstarter` package with `yarn android`

1. Add this README.MD file.

1. Move the `rnwsstarter` package out of the mono repo, delete all of the `.gitignore` folders and do a `git init` in `rnwsstarter`.

## References

- [Tutorial: How to share code between iOS, Android & Web using React Native, react-native-web and monorepo](https://dev.to/brunolemos/tutorial-100-code-sharing-between-ios-android--web-using-react-native-web-andmonorepo-4pej) by Bruno Lemos.

- [React Native — Monorepos & Code Sharing](https://engineering.brigad.co/react-native-monorepos-code-sharing-f6c08172b417) by Thibault Malbranche.
