# shipit-captain 

> Run [Shipit](https://github.com/shipitjs/shipit) tasks from Gulp, or any task runner. Includes [Inquirer.js](https://github.com/SBoudrias/Inquirer.js) prompts, [CLI arguments](https://github.com/bcoe/yargs), customized logging, and more.

![shipit-captain demo](docs/demo.gif)

## Why?
Shipit comes with its own [CLI](https://github.com/shipitjs/shipit#launch-command), but I wanted to integrate Shipit tasks into our existing task workflow, be it Gulp, Grunt, or anything else.

`shipit-captain` will let you easily do things like [set default environments](https://github.com/shipitjs/shipit/issues/38), log confirmation prompts, and easily integrate into Gulp tasks.

## Install

```sh
$ npm install --save shipit-captain
```

## Usage

You can organize your config files any way you like. Below is my preference, as it still allows `shipit-cli` commands to work, as well as those intended for `shipit-captain`. The only requirement is you must separate your `shipit.config` exports.

### Example `shipitfile.js`
```js
module.exports = require('./config/shipit').init;
```

### Example `config/shipit.js`

```js
var config = {
  default: {
    workspace: '/tmp/github-monitor',
    deployTo: '/tmp/deploy_to',
    repositoryUrl: 'https://github.com/user/repo.git',
    ignores: ['.git', 'node_modules'],
    keepReleases: 2,
    deleteOnRollback: false,
    key: '/path/to/key',
    shallowClone: true
  },
  staging: {
    servers: 'user@myserver.com'
  }
};
module.exports.config = config;
module.exports.init = function(shipit) {
  require('shipit-shared')(shipit);
  shipit.initConfig(config);
}
```

### Example `gulpfile.js`
```js
var gulp = require('gulp');
var shipitCaptain = require('shipit-captain');

// With no options, will run shipit-deploy task by default.
gulp.task('shipit', function(cb) {
  shipitCaptain(shipitConfig, cb);
});

// Run other after Shipit tasks are completed 
gulp.task('myTask', ['shipit'], function(cb) {
  console.log('Shipit tasks are done!');
  cb();
});

// Pass options 
var options = {
  init: require('config/shipit').init,
  run: ['deploy', 'clean'],
  targetEnv: 'staging',
}

gulp.task('deploy', function(cb) {
  shipitCaptain(shipitConfig, options, cb);
});
// 

```

## API

### captain(shipitConfig, [options], [cb])

------

#### shipitConfig

`@param {object} shipitConfig`

> The config object you would normally pass to `shipit.initConfig`.

##### Gulp example:

```bash
gulp shipit -e production
```

------

#### options.run

`@param {string|string[]} [options.run=[]]`

> A string or array of strings of Shipit tasks to run. If not set, user will be prompted for a task to run from all available tasks.

> Users may set `options.run` manually, or by passing the `-r` or `--run` argument via the CLI. If set via CLI, comma-separate multiple tasks names.

##### Gulp example:

```bash
gulp shipit --run deploy,myOtherTask
```

------

#### options.availableEnvs

`@param {string[]} [options.availableEnvs]`
 
> By default this will be set to any environments defined in `shipitConfig`. This shouldn't normally need to be set.

------

#### options.confirm

`@param {boolean} [options.confirm=true]`
 
> Set to `false` to bypass the confirmation prompt.

------

#### options.logItems

`{function} [options.logItems(options, shipit)]`

##### Gulp example:

```js
var options = {
  logItems: function(options, shipit) {
    return {
      'Environment': options.targetEnv,
      'Branch': shipit.config.branch,
    };
  },
};

gulp.task('shipit', function(cb) {
  shipitCaptain(shipitConfig, options, cb);
});

```

------

#### options.init

`{function} [options.init(shipit)]`

Require Shipit plugins or anything else you would have in your [`shipitfile`](https://github.com/shipitjs/shipit#example-shipitfilejs).

`shipit.initConfig` will be called automatically if it has not already been called.

##### Gulp example:

```js
var options = {
  init: function(options, shipit) {
    require('shipit-deploy')(shipit);
    require('shipit-shared')(shipit);
  }
};

gulp.task('shipit', function(cb) {
  shipitCaptain(shipitConfig, options, cb);
});
```

------

#### cb

`{function} cb`

Optional callback function, called when all Shipit tasks are complete.

```js
var gulp   = require('gulp');
var shipitCaptain = require('shipit-captain');

gulp.task('shipit', function(cb) {
  shipitCaptain(shipitConfig, cb);
});
```

## License

MIT © [Tim kelty](http://fusionary.com)
