# metalsmith-keymaster
A general-purpose [Metalsmith]("http://www.metalsmith.io/") plugin to copy or add file properties, and save under new keys

### Usage

`use(keymaster(from, to, filter))`

**from** is required and defines the value to be added/copied to the file object (here named **fileData**):
 - if `from` is '.', use `fileData.contents.toString()`.
 - if `from` is a String, use `fileData[from]` (shallow copy).
 - if `from` is a function, use `from(fileData, filePath, metalsmith)`.   

**to** is required, the value derived from **from** will be placed there, _i.e._  `fileData[to] = value;`

**filter** is optional and filters which files will be processed
 - if missing, process all files.
 - if an array, use as the match array for [multimatch](https://www.npmjs.com/package/multimatch)
 - if a string or Regex, only process matching filePaths.
 - if a function, only process when filter(filePath, data, metalsmith) returns true.


### Possible uses:

##### You have existing markdown files and template files, but the names of some fields have changed over time and are incompatible.
_e.g._, if the template now uses {{ userName }} instead of {{ name }}, but you don't want to redo all the YAML:

`.use(keymaster('name', 'userName'))`

##### You want to preserve `.contents` before the next step processes them.
_e.g._, to preserve the raw contents before markdown changes them, add the following before `use(markdown())`:

    .use(keymaster('.', 'raw'))  // save contents
       // then
    .use(markdown())            // before markdown changes them


##### You are lazy and want a quick-and-dirty plugin of your own.
This plugin serves as an excellent framework for writing your own plugin.  _e.g._, to quickly hack your own not-very-smart excerpt plugin:

    .use(keymaster(function(data) {
                    return data.contents.toString().substring(0, 50);
                  },
                  'excerpt'))


### Notes, Todos, and Caveats

If you want a deep copy, pass in a function for from and do it there.

Currently only supports a _single_ level of keys, _i.e._ `from` can be `"foo"` but not `"foo.bar"`.  If you want deeper use the function version for from.

To copy multiple fields, just use keymaster multiple times, _e.g._

    .use(keymaster('foo', 'bar'))
    .use(keymaster('abc', 'xyz'))

**Warning:** The code does not check if the property already exists, so be careful not to overwrite something you need!

### More Examples

Instead of using [metalsmith-filenames](https://www.npmjs.com/package/metalsmith-filenames), use

    .use(keymaster(function(data, filePath) {
                     return filePath;
                  },
                  'yourKeyHere'));

   and you have the options to filter files or select the key.