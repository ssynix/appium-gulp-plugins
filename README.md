appium-gulp-plugins
===================

Custom plugins used accross appium modules

## transpile plugin

Babel compilation (via Traceur), sourcemaps and file renaming functionality in
one plugin. `.es7.js` and `.es6.js` files will be automatically renamed to `.js
files`. The necessary sourcemaps and traceur comments and imports are also
automatically added.

### usage

1/ Configure gulp as below:

``` js
var gulp = require('gulp'),
Transpiler = require('appium-gulp-plugins').Transpiler;

gulp.task('transpile', function () {
  var transpiler = new Transpiler();
  // babel options are configurable in transpiler.babelOpts

  return gulp.src('test/fixtures/es7/**/*.js')
    .pipe(transpiler.stream())
    .pipe(gulp.dest('build'));
});
```

2/ in your code you need to mark the main and mocha files as below:

- main: add `// transpile:main` at the beginning of the file ([example here](https://github.com/appium/appium-gulp-plugins/blob/master/test/fixtures/es7/lib/run.es7.js)) .
- mocha: add `// transpile:mocha` at the beginning of the file ([example here](https://github.com/appium/appium-gulp-plugins/blob/master/test/fixtures/es7/test/a-specs.es7.js))

Regular lib files do not need any extra comments.

### Type assertions

Type assertions are not yet supported, but if you use Flow you can pass in an
option to the traspiler:

```js
var transpiler = new Transpiler({'rtts-assert': true});
```

This will leave the type annotations un-stripped. You may specify type in your
code like in the following:

```js
// The regular way
let a = function(t:string, n:number):string {return 'let's type code.'};

// Within comments
let a = function(ti/*:string*/, n/*:number*/)/*:string*/ {return 'let's type code.'};
```

## watch plugin

There are some issues with Gulp 3.x error handling which cause the default
gulp-watch to hang. This plugin is a small hack which solves that by respawning
the whole process on error. This should not be needed in gulp 4.0.

### usage

```
var gulp = require('gulp'),
    spawnWatcher = require('./index').spawnWatcher.use(gulp);

spawnWatcher.configure('watch', ['lib/**/*.js','test/**/*.js','!test/fixtures'], function() {
  // this is the watch action
  return runSequence('test');
});
```

The test function in `spawnWatcher.configure` should return a promise.

### error handling

The spawn needs to catch error as soon as they happen. To do so use the
`spawnWatcher.handleError` method, for instance:

```js
// add error handling where needed
gulp.task('transpile', function () {
  return gulp.src('test/es7/**/*.js')
    .pipe(transpile())
    .on('error', spawnWatcher.handleError)
    .pipe(gulp.dest('build'));
});

gulp.task('test', ['transpile'] , function () {
  return gulp.src('build/test/a-specs.js')
    .pipe(mocha())
    .on('error', spawnWatcher.handleError);
});
```

### clear terminal

Terminal is cleared by default. To avoid that call:

```js
spawnWatcher.clear(false);
```

### notification

Native notification is enabled by default. To disable it use the
`--no-notif` option.

## hacking this package

### watch

```
npm run watch
```

### test

```
npm test
```
