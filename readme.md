# npm-build-cache

npm scripts for caching and retrieving node_modules.

## installation

it is recomended that npm-build-cache be installed globally:

```js
npm install -g npm-build-cache
```

## configuration

* `cacheInstall.host` sets the hostname of the cache server. Defaults to `localhost`
* `cacheInstall.path` sets the path to the cache on the cache server. Defaults to `/tmp/node_modules-cache/`

The configuration values can be set in one of two ways:

### as environment variables

```sh
export cacheInstallHost="modulecache.example.com" # sets cacheInstall.host
export cacheInstallPath="/path/to/module/cache" # sets cacheInstall.path
```

All scripts source `$HOME/.npm-cache-install` before they run, so you can set these values there.

### through package.json:

If `cacheInstall` is defined as a property in `package.json`, then `cacheInstallHost` and `cacheInstallPath` will both be overridden.

`package.json`:
```json
...,
"cacheInstall": {
    "host": "modulecache.example.com",
    "path": "/path/to/module/cache",
},
...
```

## usage

### `npm-cache-install`

run `npm-cache-install` in your package root instead of `npm install` to use the cache if it exists, or build the cache if the cache does not exist.

### the plumbing

`cache-install` calls a few separate scripts to do its work. They are described below and are also available for use on the command line.

#### `npm-build-cache-sign-install`

It is recomended that this script be run immediately after a clean install of `node_modules`. It will write the hash of your `package.json` in `node_modules/.npm-module-cache.hash`. `cache-modules` will fail if the hash of your current `package.json` does not equal the contents of this file.

#### `npm-cache-modules`

Uses ssh and scp to conditionally upload `node_modules` to `$cacheInstallHost` as node_modules-DEPS${shasum of package.json}-ARCH${shasum of uname -mprsv}.

Environment variables:
* `forceUpload` - if defined, the existence of `node_modules` on `cacheInstallHost` will be ignored and `node_modules` will always be uploaded.

Recomendations:

* configure authentication via ssh keys between your machine and `cacheInstallHost` so you don't have to type your password every time this script is run.
* Install a cronjob on the cache host that removes `node_modules` directories with old access times.


#### `npm-restore-modules`

rsyncs the cached `node_modules` from the cache host to your machine, according to the current package.json and your machine's architecture.

Notes:
* This script will `touch` the `node_modules` directory on the cache host before rsyncing it so it can be identified as recently used.
