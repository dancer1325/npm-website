---
title: scripts
section: 7
description: How npm handles the "scripts" field
github_repo: npm/cli
github_branch: latest
github_path: docs/lib/content/using-npm/scripts.md
redirect_from:
  - /cli-documentation/misc/scripts
  - /cli-documentation/using-npm/scripts
  - /cli-documentation/v10/misc/scripts
  - /cli-documentation/v10/using-npm/scripts
  - /cli/misc/scripts
  - /cli/using-npm/scripts
  - /cli/v10/misc/scripts
  - /misc/scripts
  - /using-npm/scripts
---

### Description

* `package.json`'s `"scripts"`
  * ALLOWED values
    * built-in scripts + 's preset life cycle events
    * arbitrary scripts
  * ways to run
    * `npm run-script <stage>` or
    * `npm run <stage>`
    * `npm explore <pkg> -- npm run <stage>`
      * run scripts -- from -- dependencies 

### Pre & Post Scripts

* "pre" or "post" scripts -- via using -- "pre" or "post" prefixes + scriptName
    ```json
    {
      "scripts": {
        "precompress": "{{ executes BEFORE the `compress` script }}",
        "compress": "{{ run command to compress files }}",
        "postcompress": "{{ executes AFTER `compress` script }}"
      }
    }
    ```
  * if you run `npm run compress` -> will execute `precompress` + `compress` + `postcompress`

### Life Cycle Scripts

* == 👀SPECIAL life cycle scripts / happen | CERTAIN situations👀

* **prepare**  
  - requirements
    - `npm@4.0.0`
  - runs
    - BEFORE package is packed
      - == | `npm publish` & `npm pack`
    - | local `npm install` / WITHOUT arguments
    - [AFTER `prepublish`, BEFORE `prepublishOnly`]
  - if a package is installed -- via -- git -> 
    - 's `dependencies` & `devDependencies` -- will be -- installed
    - | BEFORE package is packaged & installed, prepare script -- will be -- run
  - goal
    - -- replace -- `prepublish`
  - | `npm@7`
    - run | background  
      - if you want to see the output -> run with `--foreground-scripts`

* **prepublish** 
  * | `npm@1.1.71`,
    * if you run `npm publish` OR `npm install` -> npm CLI runs `prepublish` script
      * Reason: 🧠 convenient way to prepare a package for use 🧠
  * ❌deprecated❌
    * Reason: 🧠[confusing](https://github.com/npm/npm/issues/10074)🧠
  * [here](#prepare-and-prepublish)
  * use case
    * operations | your package
      * / 
        * BEFORE using it
        * NOT dependent on the target system's OS or architecture
        * done 1! time
      * _Example:_
        * CoffeeScript source code -- is compiled into -- JavaScript  
          * -> You can -- depend on -- `coffee-script` as a `devDependency` == NOT need to install it
        * from JavaScript source code -- create -- minified versions
          * NOT need to include minifiers | your package == reducing the size
        * fetching remote resources / -- used by -- your package
          * NOT demand to install `curl` or `wget`

* **prepublishOnly**
  * transitional strategy BETWEEN `prepare` & `prepublish` 
  * if you run `npm publish` -> npm CLI runs `prepublishOnly` script | BEFORE the package is prepared & packed 
  * use cases
    * run the tests 1! / ensure they're in good shape

* **prepack**
  - | BEFORE packing a tarball
    - == | `npm pack` / `npm publish`/ install a git dependency
      - ⚠️`npm run pack` != `npm pack`⚠️
        - `npm run pack` == arbitrary user defined script name
        - `npm pack` == CLI defined command

* **postpack**
  - | [AFTER generating the tarball, BEFORE moving tarball -- to -- its final destination]

* **dependencies**
  - | AFTER operations / modify "node_modules/"
  - ❌ NOT run | global mode❌

#### Dependencies

* TODO:
The `dependencies` script is run any time an `npm` command causes changes to the `node_modules` directory. 
It is run AFTER the changes have been applied and the `package.json` and `package-lock.json` files have been updated.

### Life Cycle Operation Order

#### [`npm cache add`](/cli/v10/commands/npm-cache)

- `prepare`

#### [`npm ci`](/cli/v10/commands/npm-ci)

- modules installed | "node_modules"
- `preinstall`
- `install`
- `postinstall`
- `prepublish`
- `preprepare`
- `prepare`
- `postprepare`

#### [`npm diff`](/cli/v10/commands/npm-diff)

- `prepare`

#### [`npm install`](/cli/v10/commands/npm-install)

These also run when you run `npm install -g <pkg-name>`

- `preinstall`
- `install`
- `postinstall`
- `prepublish`
- `preprepare`
- `prepare`
- `postprepare`

If there is a `binding.gyp` file in the root of your package and you haven't defined your own `install` or `preinstall` scripts, 
npm will default the `install` command to compile using node-gyp via `node-gyp rebuild`

These are run from the scripts of `<pkg-name>`

#### [`npm pack`](/cli/v10/commands/npm-pack)

- `prepack`
- `prepare`
- `postpack`

#### [`npm publish`](/cli/v10/commands/npm-publish)

- `prepublishOnly`
- `prepack`
- `prepare`
- `postpack`
- `publish`
- `postpublish`

#### [`npm rebuild`](/cli/v10/commands/npm-rebuild)

- `preinstall`
- `install`
- `postinstall`
- `prepare`

`prepare` is only run if the current directory is a symlink (e.g. with linked packages)

#### [`npm restart`](/cli/v10/commands/npm-restart)

If there is a `restart` script defined, these events are run, otherwise `stop` and `start` are both run if present, including their `pre` and `post` iterations)

- `prerestart`
- `restart`
- `postrestart`

#### [`npm run <user defined>`](/cli/v10/commands/npm-run-script)

- `pre<user-defined>`
- `<user-defined>`
- `post<user-defined>`

#### [`npm start`](/cli/v10/commands/npm-start)

- `prestart`
- `start`
- `poststart`

If there is a `server.js` file in the root of your package, then npm will default the `start` command to `node server.js`. `prestart` and `poststart` will still run in this case.

#### [`npm stop`](/cli/v10/commands/npm-stop)

- `prestop`
- `stop`
- `poststop`

#### [`npm test`](/cli/v10/commands/npm-test)

- `pretest`
- `test`
- `posttest`

#### [`npm version`](/cli/v10/commands/npm-version)

- `preversion`
- `version`
- `postversion`

#### A Note on a lack of [`npm uninstall`](/cli/v10/commands/npm-uninstall) scripts

While npm v6 had `uninstall` lifecycle scripts, npm v7 does not. Removal of a package can happen for a wide variety of reasons, and there's no clear way to currently give the script enough context to be useful.

Reasons for a package removal include:

- a user directly uninstalled this package
- a user uninstalled a dependant package and so this dependency is being uninstalled
- a user uninstalled a dependant package but another package also depends on this version
- this version has been merged as a duplicate with another version
- etc.

Due to the lack of necessary context, `uninstall` lifecycle scripts are not implemented and will not function.

### User

When npm is run as root, scripts are always run with the effective uid and gid of the working directory owner.

### Environment

Package scripts run in an environment where many pieces of information are made available regarding the setup of npm and the current state of the process.

#### path

If you depend on modules that define executable scripts, like test suites, then those executables will be added to the `PATH` for executing the scripts. So, if your package.json has this:

```json
{
  "name": "foo",
  "dependencies": {
    "bar": "0.1.x"
  },
  "scripts": {
    "start": "bar ./test"
  }
}
```

then you could run `npm start` to execute the `bar` script, which is exported into the `node_modules/.bin` directory on `npm install`.

#### package.json vars

The package.json fields are tacked onto the `npm_package_` prefix. So, for instance, if you had `{"name":"foo", "version":"1.2.5"}` in your package.json file, then your package scripts would have the `npm_package_name` environment variable set to "foo", and the `npm_package_version` set to "1.2.5". You can access these variables in your code with `process.env.npm_package_name` and `process.env.npm_package_version`, and so on for other fields.

See [`package.json`](/cli/v10/configuring-npm/package-json) for more on package configs.

#### current lifecycle event

Lastly, the `npm_lifecycle_event` environment variable is set to whichever stage of the cycle is being executed. So, you could have a single script used for different parts of the process which switches based on what's currently happening.

Objects are flattened following this format, so if you had `{"scripts":{"install":"foo.js"}}` in your package.json, then you'd see this in the script:

```bash
process.env.npm_package_scripts_install === "foo.js"
```

### Examples

For example, if your package.json contains this:

```json
{
  "scripts": {
    "install": "scripts/install.js",
    "postinstall": "scripts/install.js"
  }
}
```

then `scripts/install.js` will be called for the install and post-install stages of the lifecycle. Since `scripts/install.js` is running for two different phases, it would be wise in this case to look at the `npm_lifecycle_event` environment variable.

If you want to run a make command, you can do so. This works just fine:

```json
{
  "scripts": {
    "preinstall": "./configure",
    "install": "make && make install",
    "test": "make test"
  }
}
```

### Exiting

Scripts are run by passing the line as a script argument to `sh`.

If the script exits with a code other than 0, then this will abort the process.

Note that these script files don't have to be Node.js or even JavaScript programs. They just have to be some kind of executable file.

### Best Practices

- Don't exit with a non-zero error code unless you _really_ mean it. 
  - If the failure is minor or only will prevent some optional features, then it's better to just print a warning and exit successfully.
- Try not to use scripts to do what npm can do for you. 
  - Read through [`package.json`](/cli/v10/configuring-npm/package-json) to see all the things that you can specify and enable by simply describing your package appropriately. 
  - In general, this will lead to a more robust and consistent state.
- Inspect the env to determine where to put things. For instance, if the `npm_config_binroot` environment variable is set to `/home/user/bin`, then don't try to install executables into `/usr/local/bin`. The user probably set it up that way for a reason.
- Don't prefix your script commands with "sudo". If root permissions are required for some reason, then it'll fail with that error, and the user will sudo the npm command in question.
- `install`
  - ❌NOT use it❌
    - -> barely use `preinstall` 
    - EXCEPT TO
      - compilation / must be done | target architecture 
  - use better
    - | compilation, `.gyp` file
    - | anything else, `prepare` 
    
- Scripts are run from the root of the package folder, regardless of what the current working directory is when `npm` is invoked. 
  - If you want your script to use different behavior based on what subdirectory you're in, you can use the `INIT_CWD` environment variable, which holds the full path you were in when you ran `npm run`.

### See Also

- [npm run-script](/cli/v10/commands/npm-run-script)
- [package.json](/cli/v10/configuring-npm/package-json)
- [npm developers](/cli/v10/using-npm/developers)
- [npm install](/cli/v10/commands/npm-install)
