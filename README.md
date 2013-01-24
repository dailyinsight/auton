# Auton
## Your plastic pal who's fun to code with

Auton is an automated node.js development environment & resource compiler which is meant to be flexible and extensible. A few things you can pull off with Auton:

- Automatically regenerate files when updated
- Immediately running a linter or associated test when
- CommonJS to AMD translation for server/browser parity
- Monitor your resource compiling and server debug output in the same window!

Auton's file watcher API (currently using watchr underneath) provides an API that should feel comfortable to anyone used to programming in express.

```javascript
var Auton = require('auton').Auton;
var mw    = Auton.Middleware;

var robot = new Auton( {
  root   : __dirname ,
  server : { path: 'server.js' }
} );

robot.watch('src/js', 'src/css', 'src/templates');

robot.use( /\.(js(on)?|styl|less|css)$/, mw.read() ); // run for all matched files
robot.use( '*.js', mw.jshint() );

robot.file('src/js/**/*.js',    mw.destination(['src/js',  'public/_/js' ]), mw.checkAge(), mw.commonJsToAmd(), mw.uglifyjs(), mw.save(), robot.server.middleware() );
robot.file('src/css/**/*.styl', mw.destination(function(input) { return input.replace(/\.styl$/,'.css').replace('src/css','public/_/css'); }), mw.checkAge(), mw.stylus(), mw.cssmin(), mw.save() );
robot.file('src/vendor/**/*',   mw.copy({ destination: ['src/vendor','public/_/vendor']}) );

robot.server.start();
robot.watcher.start();
robot.scanAll();
```

```javascript
  function someMiddleware( options ) {
    function( file, next ) {
      var filename = file.path;            // all middleware always have this
      var fullpath = file.fullpath;        // same with this

      var destname = file.destination;     // these are normally generated by Auton.Middleware.destination()
      var fullpath = file.fulldestination; // but any middleware could manually set them

      var root     = file.root;            // Auton's root directory

      // In addition, middleware can cause Auton to output using one of the following. Auton's UI may throw these messages away or display them depending on user preference:
      this.success("It Worked!");
      this.error("Something bad happened!");
      this.info("Helpful Information");
      this.warning("Caution: something you maybe didn't expect is happening");
      this.debug("probably more information than is needed");
      this.debug2("apparently you really want to see a ridiculous amount of information about what's happening. if you care about this you're probably fixing something. and you're probably me.");
    }
  };
```


## Built-In Middleware
### Auton.Middleware.read()
- reads contents of file located at file.path into file.data.
###

### Auton.Middleware.destination()
This middleware sets file.destination to a path which is calculated from file.path.
```javascript


  // using a static filename
  robot.file('somefile.js',   mw.destination( {filename: 'outputfile.js'} )); // somefile.js        -> outputfile.js
  robot.file('somefile.js',   mw.destination('outputfile.js'              )); // somefile.js        -> outputfile.js

  // mw.destination(a,b) and mw.destination([a,b]) get turned into file.replace(a,b)
  robot.file('somedir/*.js',  mw.destination( ['somedir/', 'outputdir/']  )); // somedir/file.js    -> outputdir/file.js
  robot.file('banana-man.js', mw.destination( /[aeiou]/g,  'oo'           )); // banana-man.js      -> boonoonoo-moon.js
  robot.file('src/**/*.js',   mw.destination( {pattern: p, replace: r}    )); // equivalent to mw.destination( p, r );

  // "mw.destination( someFunction );" is a shortcut for "file.destination = someFunction( file.path );"
  var _coffee = function(_input) {
    return _input
      .replace(/\.coffee/g, '.js')
      .replace('src', 'public')
    ;
  };
  robot.file('src/foo.coffee', mw.destination( { transform: _coffee } )); // function which takes input filename and returns output filename
  robot.file('src/foo.coffee', mw.destination( _coffee                )); // shortcut for above, both do: src/foo.coffee -> public/foo.js
```

Both mw.copy() and mw.save() use this middleware to determine the proper file to save to. Underneath, all it does is set file.destination = newPath, so you can replace a call to mw.destination() with a middleware which sets file.destination to a string value of your choice and both .save() and .copy() would listen to you (unless they're called with a custom destination, in which case this field is ignored. see below.)

### Auton.Middleware.save()

### Auton.Middleware.copy()

### Auton.Middleware.checkAge()

### Auton.Middleware.last()

### Auton.Middleware.commonJsToAmd()

Performs a CommonJS to AMD translation, useful for code meant to run both natively under node and in the browser through an AMD-compatible loader such as require.js.