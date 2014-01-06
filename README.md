Heroku buildpack: Lineman
=========================

This is a [Heroku buildpack](http://devcenter.heroku.com/articles/buildpacks) for [Lineman](https://github.com/testdouble/lineman) apps.

Usage
-----

Example usage:

    $ heroku create --stack cedar --buildpack http://github.com/testdouble/heroku-buildpack-lineman.git

Or, for an existing heroku app:

    $ heroku config:set BUILDPACK_URL=http://github.com/testdouble/heroku-buildpack-lineman.git

And the output will look like this when you push to heroku:

    $ git push heroku master
    ...
    -----> Heroku receiving push
    -----> Fetching custom buildpack
    -----> Lineman app detected
    -----> Vendoring node 0.4.7
    -----> Installing dependencies with npm ....
    ...
    -----> Building runtime environment
    -----> Building static web assets with lineman
    Running "configure" task
    
    Running "common" task
    
    Running "coffee:compile" (coffee) task
    File generated/js/app.coffee.js created.
    File generated/js/spec.coffee.js created.
    >> Destination not written because compiled files were empty.
    
    Running "less:compile" (less) task
    File generated/css/app.less.css created.
    
    Running "jshint:files" (jshint) task
    >> 0 files lint free.
    
    Running "handlebars:compile" (handlebars) task
    >> Destination not written because compiled files were empty.
    
    Running "jst:compile" (jst) task
    File "generated/template/underscore.js" created.
    
    Running "configure" task
    
    Running "concat:js" (concat) task
    File "generated/js/app.js" created.
    
    Running "concat:spec" (concat) task
    File "generated/js/spec.js" created.
    
    Running "concat:css" (concat) task
    File "generated/css/app.css" created.
    
    Running "images:dev" (images) task
    Copying images to 'generated/img'
    
    Running "webfonts:dev" (webfonts) task
    Copying webfonts to 'generated/webfonts'
    
    Running "homepage:dev" (homepage) task
    Homepage HTML written to 'generated/index.html'
    
    Running "dist" task
    
    Running "uglify:js" (uglify) task
    File "dist/js/app.min.js" created.
    Uncompressed size: 38578 bytes.
    Compressed size: 4452 bytes gzipped (13217 bytes minified).
    
    Running "cssmin:compress" (cssmin) task
    File dist/css/app.min.css created.
    Uncompressed size: 69 bytes.
    Compressed size: 45 bytes gzipped (57 bytes minified).
    
    Running "images:dist" (images) task
    Copying images to 'dist/img'
    
    Running "webfonts:dist" (webfonts) task
    Copying webfonts to 'dist/webfonts'
    
    Running "homepage:dist" (homepage) task
    Homepage HTML written to 'dist/index.html'
    
    Done, without errors.
           Web assets built
    -----> Bundling Apache v2.2.19
    -----> Discovering process types
           Procfile declares types   -> (none)
           Default types for Lineman -> web
    
    -----> Compiled slug size: 34.8MB
    -----> Launching... done, v9
           http://pacific-fortress-3824.herokuapp.com deployed to Heroku
    
    To git@heroku.com:pacific-fortress-3824.git
       9b88fd7..e4bfa19  HEAD -> master

Apache Modules
-----

Custom Apache modules saved to a directory named `apache_modules` in your app root will be deployed to Heroku. You'll add need to make the corresponding change to `conf/httpd.conf`, adding a line to load the module. Here's an example that will load the `headers_module`:

```
LoadModule headers_module  modules/mod_headers.so
```

__Building Apache Modules__

To build an Apache module, we'll use a Heroku one-off dyno:

```
$ heroku run bash
```

Once your one-off dyno is up and running, download a copy the Apache source, matching the version you'll be running in production. This buildpack uses 2.2.19.

```
$ curl http://archive.apache.org/dist/httpd/httpd-2.2.19.tar.gz | tar zx
```

Change to the Apache source directory:

```
$ cd httpd-2.2.19
```

Build the module with `apxs`

```
$ /app/apache/bin/apxs -i -c -Wl,lz modules/metadata/mod_headers.c
```

Last, copy (scp works great) the compiled module from the one-off dyno (`/app/apache/modules`) and commit it to your app's `apache_modules` directory (you might need to create this directory).
