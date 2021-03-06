# Inside Context

So you're making a plugin. Let's get started.

```js
module.exports = async function (context) {
  
  // great! now what?

}
```

Here's what's available inside `context` object.

name            | provides the...                                     | 3rd party 
----------------|-----------------------------------------------------|--------
**parameters**  | command line arguments and options                  | minimist
**config**      | configuration options from the app or plugin        | 
**print**       | tools to print output to the command line           | colors, ora
**template**    | code generation from templates                      | ejs
**prompt**      | tools to acquire extra command line user input      | enquirer
**filesystem**  | ability to copy, move & delete files & directories  | fs-jetpack
**system**      | ability to execute & copy to the clipboard          | shelljs, execa
**http**        | ability to talk to the web                          | apisauce
**strings**     | some string helpers like case conversion, etc.      | lodash & ramda

If this is starting to sound like a scripting language, then good.  That's exactly how to think of it.  Except
we're not inventing another language.  And we're still running in a `node.js` environment, so you can do whatever you want.

# context.parameters
Information about how the command was invoked. Most of the time this is from the comand line, however, 
it is possible to call it from the API if you're writing your own CLI.

Check out this example of creating a new Reactotron plugin.

```sh
glue reactotron plugin MyAwesomePlugin full --comments --lint standard
```

name           | type   | purpose                           | from the example above
---------------|--------|-----------------------------------|------------------------------------
**pluginName** | string | the pluginName used                | `'reactotron'`
**command**    | string | the command used                  | `'plugin'`
**string**     | string | the command arguments as a string | `'MyAwesomePlugin full'`
**array**      | array  | the command arguments as an array | `['MyAwesomePlugin', 'full']`
**first**      | string | the 1st argument                  | `'MyAwesomePlugin'`
**second**     | string | the 2nd argument                  | `'full'`
**third**      | string | the 3rd argument                  | `undefined`
**options**    | object | command line options              | `{comments: true, lint: 'standard'}`

### context.parameters.options
Options are the command line flags. Always exists however it may be empty.

```sh
gluegun say hello --loud -v --wave furiously
```

```js
module.exports = async function (context) {
  context.parameters.options // { loud: true, v: true, wave: 'furiously' }
}
```

### context.parameters.string
Everything else after the command as a string.

```sh
gluegun say hello there
```

```js
module.exports = async function (context) {
  context.parameters.string // 'hello there'
}
```

### context.parameters.array
Everything else after the command, but as an array.

```sh
gluegun reactotron plugin full
```

```js
module.exports = async function (context) {
  context.parameters.array // ['plugin', 'full']
}
```

### context.parameters.first / .second / .third
The first, second, and third element in `array`. It is provided as a shortcut, and there isn't one, 
this will be `undefined`.

```sh
gluegun reactotron plugin full
```

```js
module.exports = async function (context) {
  context.parameters.first  // 'plugin'
  context.parameters.second // 'full'
  context.parameters.third  // undefined
}
```

# context.config
This is an object. Each plugin will have it's own root level key. 

It takes the plugin's defaults, and merges the user's changes overtop.

```

```js
module.exports = async function (context) {
  context.config.myPlugin // { fun: true, level: 10 }
}
```

# context.print
Features for allowing you to print to the console.


### context.print.info
Prints an informational message.  Use this as your goto.

```js
context.print.info('Hello.  I am a chatty plugin.')
```

### context.print.success
Print a "something good just happened" message.

```js
context.print.success('We did it!')
```

### context.print.warning
Prints a warning message.  Use this when you feel a disturbance in the force.

```js
context.print.warning('Your system does not have Yarn installed.  It\'s awesome.')
```

### context.print.error
Prints an error message.  Use this when something goes Pants-On-Head wrong.
What does that mean?  Well, if your next line of code isn't `process.exit(0)`, then
it was probably a warning.

```js
context.print.warning('Out of disk space.  lol.')
```

### context.print.debug
Only used for debugging your plugins. You can pass this function a string or an object.

```js
context.print.debug(someObject, 'download status')
``` 

The `message` parameter is object you would like to see.

The `title` is an optional title message which is handy if you've got a lot of debug messages
and you're losing track of which one is which.

### context.print.colors
An object for working with printing colors on the command line. It is from the `colors` NPM package, 
however we define a theme to make things a bit consistent.

Some available functions include:

function           | use when you want...
------------------ | ------------------------------------------------------
`colors.success()` | the user to smile
`colors.error()`   | to say something has failed
`colors.warn()`    | to point out that something might be off
`colors.info()`    | to say something informational
`colors.muted()`   | you need to say something secondary

Each take a `string` parameter and return a `string`. 

One gotcha here is that the length of the string is longer than you think
because of the embedded color codes that disappear when you print them. 🔥

### context.print.spin
Creates a spinner for long running tasks on the command line.  It's [ora](https://github.com/sindresorhus/ora)!

Here's an example of how to work with it:

```js
// a spinner starts with the text you provide
const spinner = context.print.spin('Time for fun!')
await context.system.run('sleep 5')
```

🚨 Important 🚨 - Make sure you don't print anything else while a spinner is going.  You need to stop it first.

There's a few ways to stop it.

```js
// stop it & clear the text
spinner.stop()

// stop it, leave a checkmark, and optional new text
spinner.succeed('woot!')

// stop it, leave an X, and optional new text
spinner.fail('womp womp.')

// stop it, leave a custom label, and optional new text
spinner.stopAndPersist({ symbol: '🚨', text: 'osnap!' })
```

Once stopped, you can start it again later.

```js
spinner.start()
```

You can change the color of the spinner by setting:

```js
spinner.color = 'cyan'
```

The text can also be set with the normal printing colors.

```js
spinner.text = context.print.colors.green('i like trees')
```

# context.template
Features for generating files based on a template.

### context.template.generate

**async** - don't forget to prefix calls to this function with `await` if you want to wait until it finishes before continuing.

Generates a new file based on a template.

```js
module.exports = async function (context) {
  
  const name = context.parameters.first
  const semicolon = context.options.useSemicolons && ';'

  await context.template.generate({
    template: 'component.njk',
    target: `app/components/${name}-view.js`,
    props: { name, semicolon }
  })

}
```

Note: `generate()` will always overwrite the target if given.  Make sure to prompt your users if that's
the behaviour you're after.

option           | type    | purpose                              | notes
-----------------|---------|--------------------------------------|----------------------------
`template`       | string  | path to the EJS template             | relative from plugin's `templates` directory 
`target`         | string  | path to create the file              | relative from user's working directory  
`props`          | object  | more data to render in your template | 
`directory`      | string  | where to find templates              | an absolute path (optional)

`generate()` returns the string that was generated in case you didn't want to render to a target.


# context.prompt

### context.prompt.ask

**async**

This is the lovely and talented [enquirer](https://github.com/enquirer/enquirer).  Here's some examples:

```js
// a thought-provoking question
const askAge = { 
  type: 'input', 
  name: 'age', 
  message: 'How old are you?'
  }

// now let's get to what we REALLY want to know...
const askShoe = { 
  type: 'input', 
  name: 'shoe', 
  message: 'What shoes are you wearing?', 
  choices: ['Clown', context.prompt.separator(), 'Other'] 
  }

// ask the question
const { age, shoe } = await context.prompt.ask([askAge, askShoe])
```

### context.prompt.separator

Call `context.prompt.separator()` to return a separator you can use in some of your questions.

### context.prompt.confirm

**async**

A pre-built prompt which asks a yes or no question.

##### parameters

`message` is a `string` required for displaying a message to user.  It's the question you're asking. 

##### returns

`true` or `false`


# context.filesystem

A set of functions & values to work with files and directories.  The majority of these functions come
straight from [fs-jetpack](https://github.com/szwacz/fs-jetpack), a fantastic API for working with the
file system.  All jetpack-based functions have an equivalent `*Async` version if you need it.

### context.filesystem.separator

This value is the path separator `\` or `/` depending on the OS.

```js
context.filesystem.seperator // '/' on posix but '\' on windows
```

### context.filesystem.eol

This value is the end of line byte sequence.

```js
context.filesystem.eol // '\n' on posix but '\r\n' on windows 
```

### context.filesystem.append

[Appends](https://github.com/szwacz/fs-jetpack#appendpath-data-options) data to the end of a file.

### context.filesystem.copy

[Copies](https://github.com/szwacz/fs-jetpack#copyfrom-to-options) a file or a directory.

### context.filesystem.cwd

Gets the [current working directory](https://github.com/szwacz/fs-jetpack#createreadstreampath-options).

### context.filesystem.dir

[Ensures a directory exists](https://github.com/szwacz/fs-jetpack#dirpath-criteria) and creates a new jetpack
instance with it's `cwd` pointing there. 

### context.filesystem.exists

Checks to see if file or directory [exists](https://github.com/szwacz/fs-jetpack#existspath).

### context.filesystem.file

[Ensures a file exists](https://github.com/szwacz/fs-jetpack#filepath-criteria).

### context.filesystem.find

[Finds](https://github.com/szwacz/fs-jetpack#findpath-searchoptions) files or directories.

### context.filesystem.inspect

[Grabs information](https://github.com/szwacz/fs-jetpack#inspectpath-options) about a file or directory.

### context.filesystem.inspectTree

[Grabs nested information](https://github.com/szwacz/fs-jetpack#inspecttreepath-options) about a set of files or directories.

### context.filesystem.list

[Gets a directory listing](https://github.com/szwacz/fs-jetpack#listpath), like `ls`.

### context.filesystem.move

[Moves](https://github.com/szwacz/fs-jetpack#movefrom-to) files and directories.

### context.filesystem.path

[Grabs path parts](https://github.com/szwacz/fs-jetpack#pathparts) as a string.

### context.filesystem.read

[Reads](https://github.com/szwacz/fs-jetpack#readpath-returnas) the contents of a file as a string or JSON.

### context.filesystem.remove

[Deletes](https://github.com/szwacz/fs-jetpack#removepath) a file or directory.

### context.filesystem.rename

[Renames](https://github.com/szwacz/fs-jetpack#renamepath-newname) a file or directory.

### context.filesystem.symlink

[Makes a symbolic link](https://github.com/szwacz/fs-jetpack#symlinksymlinkvalue-path) to a file or directory. 

### context.filesystem.write

[Writes](https://github.com/szwacz/fs-jetpack#writepath-data-options) data to a file.




# context.system

### context.system.run

**async**

Runs a shell command and returns the output as a string.

The first parameter `commandLine` is the shell command to run.  It can have pipes! The
second parameter is `options`, object. This is a promise wrapped around node.js `child-process.exec`
[api call](https://nodejs.org/api/child_process.html#child_process_child_process_exec_command_options_callback). 

You can also pass `trim: true` inside the options parameter to have the output automatically trim all
starting and trailing spaces.

Should the process fail, an `error` will be thrown with properties such as:

property | type   | purpose
---------|--------|------------------------------------------------------------
code     | number | the exit code
cmd      | string | the command we asked to run
stderr   | string | any information the process wrote to `stderr`
killed   | bool   | if the process was killed or not
signal   | number | the signal number used to off the process (if killed)

```js
const nodeVersion = context.system.run('node -v', { trim: true })
```

### context.system.which

Retursn the full path to a command on your system if located on your path.

```js
const whereIsIt = context.system.which('npm')
```

### context.system.open
:(

### context.system.readFromClipboard

Grabs the text currently on the clipboard and returns it as a string.

```js
const password = context.system.readFromClipboard() // ya, don't be a bad person
```

### context.system.writeToClipboard

Copies the given text to the clipboard.

```js
context.system.writeToClipboard('8675309')
```

# context.strings

Provides some helper functions to work with strings.  This list is also added to the available filters
inside your EJS templates.

function    | parameters          | usage
------------|---------------------|---------------------------------------
identity    | value               | returns itself
isBlank     | value               | true if its null, or trimmed to empty
isNotString | value               | true if it's not `typeof 'string'`
camelCase   | value               | thisIsCamelCase
kebabCase   | value               | this-is-kebab-case
snakeCase   | value               | this_is_snake_case
upperCase   | value               | THIS IS UPPER CASE
lowerCase   | value               | this is lower case
startCase   | value               | This is start case
upperFirst  | value               | Changes the first character to upper case
lowerFirst  | value               | changes the first character to lower case 
pascalCase  | value               | ThisIsPascalCase
pad         | value, length, char | Pads a string to a length with by filling char
padStart    | value, length, char | Pads the start of a string
padEnd      | value, length, char | Pads the end of string
trim        | value, length, char | Removes whitespace from the edges
trimStart   | value               | Removes whitespace from the front
trimEnd     | value               | Removes whitespace from the back
repeat      | value, count        | Repeats a value, count times


# context.http

Gives you the ability to talk to HTTP(s) web and API servers using [apisauce](https://github.com/skellock/apisauce) which
is a thin wrapper around [axios](https://github.com/mzabriskie/axios).

### context.http.create

This creates an `apisauce` client.  It takes 1 parameter called `options` which is an object.

```js
const api = context.http.create({
  baseURL: 'https://api.github.com',
  headers: {'Accept': 'application/vnd.github.v3+json'}
})
```

Once you have this api object, you can then call `HTTP` verbs on it. All verbs are `async` so don't forget your `await` call.

```js
// GET
const { ok, data } = await api.get('/repos/skellock/apisauce/commits')

// and others
api.get('/repos/skellock/apisauce/commits')
api.head('/me')
api.delete('/users/69')
api.post('/todos', {note: 'jump around'}, {headers: {'x-ray': 'machine'}})
api.patch('/servers/1', {live: false})
api.put('/servers/1', {live: true})
```

