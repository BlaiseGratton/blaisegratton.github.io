---
layout: post
title: "TFS Builds with Gulp, Chutzpah, & Jasmine"
date: 2016-07-01
---
# Integrating Gulp Processes into TFS Builds (and making them fail when they should)

Though the new TFS 2015 build process is out, you may still find yourself needing (or wanting?) for whichever reason to use the XAML build definitions for Team Foundation Server. "But how can we take advantage of modern front-end build tools to provide better organisation and ensure code quality while still using the XAML definitions?" you might ask. Hopefully this gist will get you there. As an exploratory process, a small task force of us at TDOT have configured a build definition such that a gulp process transpiles either TypeScript or ES6 (why not both?) into a separate output folder, runs Jasmine unit tests on the output script, and fails the build when the Jasmine tests fail. 
<br/>
<br/>
* [Running Gulp](#running-gulp)
  > [A Quick Note on Node Modules](#node-modules)

* [Configuring Gulp Tasks](#gulp-tasks)
  > [ES6](#es6)
  >
  > [TypeScript](#typescript)

* [Installing Chutzpah & Jasmine](#chutzpah-jasmine)

* [Configuring Your Build](#build)

<br/>
<br/>
<a name="running-gulp"></a>
## Running Gulp
<br/>

Gulp good, Grunt bad `</rant>` (But seriously, who wants to edit 300 lines of JSON?). 
We have Node.js installed globally on our local TFS server. Gulp, however, is checked in as a local node module. If you would like to use a global installation of gulp on your build server, you will need to alter the path to the gulp.js file. 
Open your .csproj file, and add the following tags and script inside the `<Property></Property` tag (ours is near the bottom of the file):
<br/>
<br/>
```
<PropertyGroup>
  <PreBuildEvent>
    set NODE_PATH=$(MSBuildProjectDirectory)\node_modules;%25NODE_PATH%25
    "C:\Program Files\nodejs\node.exe" "$(MSBuildProjectDirectory)\node_modules\gulp\bin\gulp.js"
    --gulpfile "$(MSBuildProjectDirectory)\gulpfile.js" "build"
  </PreBuildEvent>
</PropertyGroup>
```
<br/>
<br/>
This is assuming that your gulpfile is in the root of your project and that the gulp task you want to run is "build". If any of these are different, change the script accordingly.
<br/>
<br/>
<br/>

<a name="node-modules"></a>
> ## A Quick Note on Node Modules
> In case you, like me, hadn't heard, the current best practice in the Node.js community is to check in your Node modules into your source control (\*gasps!\*) - or at a minimum just the executables you need, but with anything other than front-end assets, that's really tough to do. This will save you from those subtle-version-changes-which-make-you-feel-like-you-took-crazy-pills bugs from running `npm install` in a different environment at a later time.
>
> In Visual Studio, we check in our Node modules via Source Control Explorer. Navigate to the parent directory of `node_modules`, right click `node_modules`, and click `Add Items to Folder`: 
>
> <img src="https://raw.githubusercontent.com/BlaiseGratton/tfs_photos/master/node_modules_source_control.png" width="250" />
>
>Select all items in the window that pops up, click `Next >`, and make sure all items are in the `Items to add` tab. If some are in the `Excluded items` tab, select them all and click `Include item(s)`, then click `Finish`. 

<br/>
<br/>
<br/>
<a name="gulp-tasks"></a>
## Configuring Gulp Tasks
<br/>
Regardless of which type of transcomformdeconstrapiling you are doing with your JavaScript (that is between you and your code, please), it's a good idea to bundle and minify your js code to reduce load time and minimize HTTP requests. A basic gulp task for that would look like this:
<br/>
<br/>

```
var gulp = require('gulp'),
jsConcat = require('gulp-concat'),
    maps = require('gulp-sourcemaps'),
  rename = require('gulp-rename'),
  uglify = require('gulp-uglify');

gulp.task('build', function() {
  return gulp.src('Scripts/src/**/*.js')
    .pipe(maps.init())
    .pipe(jsConcat('app.js'))
    .pipe(uglify())
    .pipe(rename('app.min.js'))
    .pipe(maps.write('./'))
    .pipe(gulp.dest('Scripts/dist/'));
});
```

<br/>
<br/>
This takes all of our source scripts, concatenates them together, minifies the concatenated script, and outputs it into a `Scripts/dist/` directory. Fancier tasks and configuration could have simply an unminified `app.js` bundle for development purposes with source maps, and an unmapped minified `app.min.js` file for deployment.
<br/>
<br/>
<br/>
<a name="es6"></a>
### &nbsp;&nbsp;&nbsp;&nbsp;ES6
For transpiling ES6, we'll use the gulp wrapper for babeljs. You will need to install the npm package `gulp-babel` and at least one preset package for babel, such as `babel-preset-es2015` (and [check them in](#node-modules)). Once these are installed, your gulp build task can look something like this:

```
var gulp = require("gulp");
var sourcemaps = require("gulp-sourcemaps");
var babel = require("gulp-babel");
var concat = require("gulp-concat");


gulp.task('build', function() {
  return gulp.src("scripts/src/**/*.js")
    .pipe(sourcemaps.init())
    .pipe(babel({
        presets: ['es2015']
    }))
    .pipe(concat("app.js"))
    .pipe(sourcemaps.write("."))
    .pipe(gulp.dest("scripts/dist/"));
});  
```

This task bundles everything into an `app.js` file and places it into an output directory of `scripts/dist/` with accompanying sourcemaps.
<br/>
<br/>
<br/>
<a name="typescript"></a>
### &nbsp;&nbsp;&nbsp;&nbsp;TypeScript
For transpiling TypeScript, you will need to install the npm package `gulp-typescript` (and [check it in](#node-modules)). Configure your build task to look like so:

```
var gulp = require("gulp");
var sourcemaps = require("gulp-sourcemaps");
var concat = require("gulp-concat");
var ts = require('gulp-typescript');

gulp.task('build', function() {
  return gulp.src("scripts/src/**/*.ts")
    .pipe(sourcemaps.init())
    .pipe(ts({
      noImplicitAny: false,
      noEmitOnError: true,
      removeComments: false,
      sourceMap: true,
      out: "appBundle.js",
      target: "es5"
   }))
  .pipe(concat("app.js"))
  .pipe(sourcemaps.write("."))
  .pipe(gulp.dest("scripts/dist/"));
});  
```

You can configure gulp tasks to hook into a number of different Visual Studio controls, but for our purposes here, this gulp task will only run on a solution build or when manually invoked. By default, when Visual Studio recognizes the presence of TypeScript files, it will automatically transpile them into ES5 into the same directory where they exist when a save or build occurs. Since we're letting gulp handle the transpiling of TypeScript, this is at least unnecessary, if not downright annoying, and wastes processing time and power on every save or build on creating extra file output. 

This can be handled in a number of different ways, from redirecting the output to a different directory, but turning it off entirely is a straightforward approach. (Never, ever, ever, ever try to just delete the `<TypeScriptCompile Include="Scripts/**"/>` tag which gets added, since it will keep coming back into the .csproj file like a determined mosquito.) Leave that tag alone, but right after it, add this: 
```
<ItemGroup>
  <TypeScriptCompile Include="Scripts\src\main.ts" />              <!-- dear god leave this alone -->
</ItemGroup>
<PropertyGroup>
  <TypeScriptCompileOnSaveEnabled>False</TypeScriptCompileOnSaveEnabled>
  <TypeScriptCompileBlocked>True</TypeScriptCompileBlocked>
</PropertyGroup>
```

This should effectively burn any TypeScript compile functionality on behalf of Visual Studio to the ground, if that's what you want.

<br/>
<br/>
<br/>
<a name="chutzpah-jasmine"></a>
## Installing Chutzpah & Jasmine
<br/>
Now that you have a front-end build process up and running, you probably want to integrate tests to run on your output scripts (you do want that, right?). We'll first configure Chutzpah and Jasmine locally, then integrate them into the remote build in the next section.

Via NuGet Package Manager, install both the packages for Chutzpah and Jasmine. If you have a separate test project, you only need to install them to that project. Next, under Tools > Extensions And Updates, install the Chutzpah Test Adapter for the Test Explorer. This allows you to see and run your Jasmine tests in Visual Studio's Test Explorer window.

In order for Chutzpah to be able to run tests on your output script(s), you need to configure it to know in which directories to look. We do that with a `chutzpah.json` file. For reference, the solution structure looks like this:

<img src="https://raw.githubusercontent.com/BlaiseGratton/tfs_photos/master/solution_structure.png" width="200" />

We can configure the test runner to look in the output directory of the gulp tasks:

```
{
  "RootReferencePathMode": "SettingsFileDirectory",
  "Framework": "jasmine",
  "References": [
    {
      "Path": "../DeployTesting2/Scripts/dist/",
      "Include": "*.js"
    }
  ],
  "Tests": [
    {
      "Path": "js"
    }
  ]
}
```

This file is in the root of the test project, and thus we give it relative paths to look for both the transpiled JavaScript as well as in which directory our Jasmine tests are. At this point, running any of your Jasmine tests in the Test Explorer window should work, i.e. Chutzpah should be able to find & load your output files from `Scripts/dist/`.
<br/>
<br/>
<br/>
<a name="build"></a>
## Configuring Your Build
Now that we have everything installed locally, it's time to configure our .csproj file and build definition so that our source JavaScript/TypeScript files get transpiled and tested as part of the remote build process. 

TFS builds output files into a build staging area. The files output into the staging area then get copied to the final location (the 'drop' location). If a build were cued with the project as is, the `gulp build` task would run, since we configured it in a `<PreBuildEvent></PreBuildEvent>` tag. The output files would only be placed alongside the rest of the source code for your solution and would not even make it to the staging area, however, since TFS hasn't been instructed to look for them or copy them.

This copying can be accomplished by extending our .csproj configuration:

```
<Target Name="AfterBuild">
  <Message Text="Copying output files from gulp tasks" Importance="high" />
  <CreateItem Include="Scripts\dist\*">
    <Output TaskParameter="Include" ItemName="ScriptsToCopy" />
  </CreateItem>
  <Copy SourceFiles="@(ScriptsToCopy)"
    DestinationFolder="$(TF_BUILD_BINARIESDIRECTORY)\_PublishedWebsites\DeployTesting2\Scripts\dist\" />
</Target>
```

This takes the output files from the `src/` directory of the build and places them in the staging area (the `bin/` directory). When the staging area files get copied to the drop folder location, the output scripts will make it to the final directory. 
<br/>
<br/>

That's great - now when you run the deployed app, code that was developed in ES6 or TypeScript is getting transpiled and running from the remote build. But we want to also test that code when it builds, so we need Chutzpah to hook into the build process.

The first step is to make sure the `chutzpah.json` file makes it into the remote build. Right click it, select Properties, and make sure the `Copy to Output Directory` option is set to `Copy always`. 

Next, right click the build definition you are using and select `Edit Build Definition...`. Select the `Process` tab, and in the `Test` section, select `Automated tests`. Select the ellipsis which appears in the right column:

<img src="https://raw.githubusercontent.com/BlaiseGratton/tfs_photos/master/build_process_tests.png" />

In the window that pops up, click `Add...`. Check the box `Fail build on test failure`, and for `Test assembly file specification:`, type `**\*.js`. Hit `Okay` twice and then save your changes. If you'd like a different/more specific path specification for your test directory, you can try that, but another blog warned that this was generic `**\*.js` glob was the only way that worked for them. 

The last thing to configure is the reference path in your `chutzpah.json` file. The directory structure in the staging area is slightly different than the local setup, so for every local reference path you have, you need to add an additional reference path for the build directory. This usually involves including the `_PublishedWebsites` directory somewhere in the path, like so:

```
{
  "RootReferencePathMode": "SettingsFileDirectory",
  "Framework": "jasmine",
  "References": [
    {
      "Path": "../DeployTesting2/Scripts/dist/",
      "Include": "*.js"
    },
    {
      "Path": "./_PublishedWebsites/DeployTesting2/Scripts/dist",
      "Include": "*.js"
    }
  ],
  "Tests": [
    {
      "Path": "js"
    }
  ]
}
```

If you're having trouble with this configuration finding files, there is another way to include reference paths in the `chutzpah.json` file. You can also add comments, such as this:
```
/// <reference  path="./_PublishedWebsites/DeployTesting2/Scripts/dist/app.js"/>
```
I had no success with using `<reference/>` tags, while others have only had success with `<reference/>` tags but not by  defining references in the JSON itself. 
<br/>
<br/>
<br/>
Hopefully at this point, your builds are failing if the Jasmine tests are failing. Comments & questions can be submitted to my [gist](https://gist.github.com/BlaiseGratton/9a9ecdcba4736b367f182ef9619e8049) writeup of this process.
