# Ronin (Coffeescript Fork)

Toolkit for building shining CLI programs in Node.js using coffeescript.

**This is the coffeescript fork of [Ronin](https://github.com/vdemedes/ronin).** The changes are to make it work with .coffee files instead of .js. I'm actively using this project so I will do my best to keep it up-to-date with the original project.

I don't use the middleware functionality so I didn't convert that to coffeescript. It uses a named function which we can't really do in coffeescript other than with just plain escaped JS. I'm pretty sure it should work if your middleware files are `.js`. If not feel free to submit a pull request to patch that.

## Installation

```
npm install ronin-coffeescript --global
```

## Getting Started

### Creating basic structure

Execute the following command to generate basic skeleton for your program:

```
ronin new hello-world
```

Ronin will create a hello-world directory (if it does not exists or empty) and put everything that's needed to start developing your CLI tool immediately.

Once you `cd` into the app, you can generate the scaffolding for a new command with:

```
ronin g command <command-name>
```

### Initialization

Here's how to initialize CLI program using Ronin:

```coffeescript
ronin = require 'ronin'
program = ronin __dirname
program.run()
```

### Creating commands

Next, to setup some commands, simply create folders and files.
The structure you create, will be reflected in your program.
For example, if you create such folders and files:

```
commands/
--  apps.coffee
--  apps/
    -- add.coffee
    -- remove.coffee
--  keys/
    -- dump.coffee
```

In result, Ronin, will generate these commands for you automatically:

```
$ hello-world apps
$ hello-world apps add
$ hello-world apps remove
$ hello-world keys dump
```

Each folder is treated like a namespace and each file like a command, where file name is command name.

To actually create handlers for those commands, in each file, Command should be defined:

```coffeescript
Command = require('ronin').Command

AppsAddCommand = module.exports = Command.extend
    desc: 'This command adds application'
    run: (name) ->
        // create an app with name given in arguments
```

To run this command, execute:

```
$ hello-world apps add great-app
```

Whatever arguments passed to command after command name, will be passed to .run() method in the same order they were written.

#### Specifying options

You can specify options and their properties using *options* object.

```coffeescript
AppsDestroyCommand = module.exports = Command.extend
    desc: 'This command removes application'
    options:
        name: 'string'
        force:
            type: 'boolean'
            alias: 'f'
    run: (name, force) ->
        unless force
            throw new Error '--force option is required to remove application'
        // remove app
```

**Note**: Options will be passed to .run() method in the same order they were defined.

#### Customizing help

By default, Ronin generates help for each command and for whole program automatically.
If you wish to customize the output, override .help() method in your command (program help can not be customized at the moment):

```coffeescript
HelloCommand = Command.extend
    help: ->
        "Usage: #{@programName} #{@name} [OPTIONS]"
    desc: 'Hello world'
```

#### Customizing command delimiter

By default, Ronin separates sub-commands with a space.
If you want to change that delimiter, just specify this option when initializing Ronin:

```coffeescript
program = ronin()

program.set
    path: __dirname
    delimiter: ':'

program.run()
```

After that, `apps create` command will become `apps:create`.


### Middleware

There are often requirements to perform the same operations/checks for many commands.
For example, user authentication.
In order to avoid code repetition, Ronin implements middleware concept.
Middleware is just a function, that accepts the same arguments as .run() function + callback function.
Middleware functions can be asynchronous, it makes no difference for Ronin.

Let's take a look at this example:

```javascript
var UsersAddCommand = Command.extend({
    use: ['auth', 'beforeRun'],
    
    run: function (name) {
        // actual users add command
    },
    
    beforeRun: function (name, next) {
        // will execute before .run()
        
        // MUST call next() when done
        next();
    }
});
```

In this example, we've got 2 middleware functions: auth and beforeRun.
Ronin allows you to write middleware functions inside commands or inside `root/middleware` directory.
So in this example, Ronin will detect that `beforeRun` function is defined inside a command and `auth` function will be `require`d from `root/middleware/auth.js` file.

**Note**: To interrupt the whole program and stop execution, just throw an error.


## Tests

```
npm test
```

## License

Ronin is released under the MIT License.
