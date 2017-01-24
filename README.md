# recipe-pm2
This Abe recipe demonstrates how to deploy your Abe project admin in production with [pm2](https://github.com/Unitech/pm2).

## Add pm2 in dependencies in package.json

This add pm2 in the project dependencies. 

```javascript
{
  "dependencies": {
     "pm2": "2.2.*",
    },
}
``` 

Run `npm i` to install dependencies.

## Add a [npm](https://www.npmjs.com/) task to manage pm2 via npm.

This task deal with pm2 process to start the process. If the pm2 process is already startn this tasks stop, delete the process an start the process.

```javascript
'use strict';

var clc = require('cli-color');
var pm2 = require('pm2');

var dir = process.cwd();
if(process.env.ROOT) {
  dir = process.env.ROOT
}
dir = dir.replace(/\/$/, '')
console.log("run abe into", clc.green(dir), "folder");

var processFile = require(dir + '/pm2-config.json')
console.log(clc.green('[ pm2-config ]'), JSON.stringify(processFile))

var processName = processFile.name || 'abe'
pm2.connect((err) => {
  if (err instanceof Error) throw err
  var start = pm2.start

  pm2.list(function(err, process_list) {
    var found = false;

    Array.prototype.forEach.call(process_list, function(process) {
      if (process.name === processName) {
        found = true
      }
    })

    var cb = function() {
      if(typeof processFile.env === 'undefined' || processFile.env === null) {
        processFile.env = {}
      }
      processFile.env.ROOT = dir
      console.log(clc.green('[ pm2 ]'), 'start', 'pm2-config.json', JSON.stringify(processFile))
      pm2.start(
        processFile,
        function(err, proc) {
          if (err instanceof Error) throw err

          pm2.list((err, list) => {
             if (err instanceof Error) throw err
            Array.prototype.forEach.call(list, (item) => {
              console.log(clc.green('[ pm2 ]'), '{', '"pid":', item.pid + ',', '"process":', '"' + item.name + '"', '}')
            })
            process.exit(0);
          })
        })
    }

    if (!found) {
      cb()
    }else {
      console.log(clc.green('[ pm2 ]'), 'stop ', processName)
      pm2.delete(processName, function(err, proc) {
        if (err) throw new Error(err);
        console.log(clc.green('[ pm2 ]'), processName,  'server stopped')
        cb()
      });
    }
  });
})
```

## Add the tasks in package.json

This allow to add the npm task to run pm2 within your project.

```javascript
  "devDependencies": {},
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node path/to/start.js"
  },
```

## Add pm2-config.json

This is the config file for pm2. See pm2 process file [documentation](http://pm2.keymetrics.io/docs/usage/application-declaration/)

```javascript
{
  "name": "pm2processname",
  "script": "./node_modules/abe-cli/dist/server/app.js",
  "args": "serve",
  "nodeArgs": ["--harmony","--max-old-space-size=2048"], # You can pass node arguments here
  "log_date_format" : "YYYY-MM-DD HH:mm Z", # Pm2 log format
  "exec_mode": "fork", # Pm2 exec mode (fork or cluster)
  "instances": "-1", # if cluster mode number of intances -1 = server number of cpus -1 
  "env": {
    "PORT": "8012" # local port to run Abe
  }
}
```

Then run `npm start`


