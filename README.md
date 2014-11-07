sane
----

I've been driven to insanity by node filesystem watcher wrappers.
Sane aims to be fast, small, and reliable file system watcher. It does that by:

* By default stays away from fs polling because it's very slow and cpu intensive
* Uses `fs.watch` by default and sensibly works around the various issues
* Maintains a consistent API across different platforms
* Where `fs.watch` is not reliable you have the choice of using the following alternatives:
  * [the facebook watchman library](https://facebook.github.io/watchman/)
  * polling

## Install

```
$ npm install sane
```

## How to choose a mode

Don't worry too much about choosing the correct mode upfront because sane
maintains the same API across all modes and will be easy to switch.

* If you're only supporting Linux and OS X, `watchman` would be the most reliable mode
* If you're using node > v0.10.0 use the default mode
* If you're running OS X and you're watching a lot of directories and you're running into https://github.com/joyent/node/issues/5463, use `watchman`
* If you're in an environment where native file system events aren't available (like Vagrant), you should use polling
* Otherwise, the default mode should work well for you

## API

### sane(dir, options)

Watches a directory and all it's descendant directories for changes, deletions, and additions on files and directories.

```js
var watcher = sane('path/to/dir', ['**/*.js', '**/*.css']);
watcher.on('ready', function () { console.log('ready') });
watcher.on('change', function (filepath, root, stat) { console.log('file changed', filepath); });
watcher.on('add', function (filepath, root, stat) { console.log('file added', filepath); });
watcher.on('delete', function (filepath, root) { console.log('file deleted', filepath); });
// close
watcher.close();
```

options:

* `glob`: a single string glob pattern or an array of them.
* `poll`: puts the watcher in polling mode. Under the hood that means `fs.watchFile`.
* `watchman`: makes the watcher use [watchman](https://facebook.github.io/watchman/)

For the glob pattern documentation, see [minimatch](https://github.com/isaacs/minimatch).
If you choose to use `watchman` you'll have to [install watchman yourself](https://facebook.github.io/watchman/docs/install.html)).

### sane.NodeWatcher(dir, options)

The default watcher class. Uses `fs.watch` under the hood, and takes the same options as `sane(options, dir)`.

### sane.WatchmanWatcher(dir, options)

The watchman watcher class. Takes the same options as `sane(options, dir)`.

### sane.PollWatcher(dir, options)

The polling watcher class. Takes the same options as `sane(options, dir)` with the addition of:

* interval: indicates how often the files should be polled. (passed to fs.watchFile)

### sane.{Node|Watchman|PollWatcher}#close

Stops watching.

### sane.{Node|Watchman|PollWatcher} events

Emits the following events:

All events are passed the file/dir path relative to the root directory
* `ready` when the program is ready to detect events in the directory
* `change` when a file changes
* `add` when a file or directory has been added
* `delete` when a file or directory has been deleted

## License

MIT
