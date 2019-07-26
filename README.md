# A MODIFIED VERSION OF [json-schema-ref-parser](https://github.com/APIDevTools/json-schema-ref-parser).

The modifications are to address the incompatability of this library with Angular versions > 6. See [this issue](https://github.com/APIDevTools/json-schema-ref-parser/issues/129) over in the real repository.

Because I do not need to resolve URLs for my current usages, I'm just removing the HTTP(s) resolver. This "fix" works for me, but SHOULD NOT BE CONSIDERED A REAL FIX!

I'm publishing it locally using [Verdaccio](https://verdaccio.org/) so as to not confuse anyone looking for the **[real version](https://github.com/APIDevTools/json-schema-ref-parser)**

## USAGE

### 1. Publish this library locally
 1. Clone this repository
 2. Modify `package.json` to set the library name and version as desired.
 3. Make sure [Verdaccio](https://verdaccio.org/) is configured and running. (So you don't publish this to the real NPM.)
 4. Publish this hacky package: 
    ```
      cd json-schema-ref-parser
      npm publish
    ```
### 2. Configure your Angular app
Inside your angular app:
 1. Install this package using whatever name you published it with.
 2. *Modify `index.html` by adding the following to the `<head>` section:
     ```html
       <script>
         var global = global || window;
       </script>
     ```
 3. *Modify `polyfills.ts` by adding the following:
      ```typescript
      global.Buffer = global.Buffer || require('buffer').Buffer;
      ```
 4. Import it using:
      ```typescript
      import $RefParser from "json-schema-ref-parser";
      ```
 5. Parse your file using something else, such as [`js-yaml`](https://github.com/nodeca/js-yaml)
 5. Use `$RefParser.dereference()`
 
### 3. Example Usage
```typescript
        const raw = {
            foo: 1,
            bar: 'test',
            deref: {
                $ref: '#/foo'
            }
        };

        const asString = JSON.stringify(raw);
        const parsed = yaml.safeLoad(asString);
        // DO NOT USE THE FOLLOWING:
        //  -- It would trigger the resolvers that were removed.
        // $RefParser.parse(asString, {resolve: {external: true}}, (err, schema) => {
        //     console.log(err, schema);
        //     done();
        // });

        $RefParser.dereference(parsed, (err, schema) => {
            console.log(err, schema);
        });
``` 
The above example outputs the following:
```
null, Object{foo: 1, bar: 'test', deref: 1}
 ```
 
# Summary of Changes
All changes are hacky. This fork is designed to work for my purposes with minimal effor. Do not expect this to be a good
solution! This is a **bad** solution.


### Changes to make it work in the browser:
It seems like this library was designed to be run on the back-end, not in the browser. I'm not familiar enough with Node
to know if these changes are OK or not.
  1. Added `process` to the package dependencies.
  1. Added `const process = require('process')` to the files that use it

### Changes to make it work with newer versions of Angular:
  1. Removed the imports of `http` and `https` from `lib/resolvers/http.js`
  1. Commented out the code that performs an HTTP GET and replaced it with a hardcoded promise that immediately rejects
     with an error message indicating the feature was removed for Angular support.
  1. Changed the syntax of the export in `index.d.ts` to avoid the need for `import * as` 

  
 



JSON Schema $Ref Parser
============================
#### Parse, Resolve, and Dereference JSON Schema $ref pointers

[![Build Status](https://api.travis-ci.com/APIDevTools/json-schema-ref-parser.svg?branch=master)](https://travis-ci.com/APIDevTools/json-schema-ref-parser)
[![Coverage Status](https://coveralls.io/repos/github/APIDevTools/json-schema-ref-parser/badge.svg?branch=master)](https://coveralls.io/github/APIDevTools/json-schema-ref-parser)

[![npm](https://img.shields.io/npm/v/json-schema-ref-parser.svg)](https://www.npmjs.com/package/json-schema-ref-parser)
[![Dependencies](https://david-dm.org/APIDevTools/json-schema-ref-parser.svg)](https://david-dm.org/APIDevTools/json-schema-ref-parser)
[![License](https://img.shields.io/npm/l/json-schema-ref-parser.svg)](LICENSE)


[![OS and Browser Compatibility](https://apidevtools.org/img/badges/ci-badges-with-ie.svg)](https://travis-ci.com/APIDevTools/json-schema-ref-parser)


The Problem:
--------------------------
You've got a JSON Schema with `$ref` pointers to other files and/or URLs.  Maybe you know all the referenced files ahead of time.  Maybe you don't.  Maybe some are local files, and others are remote URLs.  Maybe they are a mix of JSON and YAML format.  Maybe some of the files contain cross-references to each other.

```javascript
{
  "definitions": {
    "person": {
      // references an external file
      "$ref": "schemas/people/Bruce-Wayne.json"
    },
    "place": {
      // references a sub-schema in an external file
      "$ref": "schemas/places.yaml#/definitions/Gotham-City"
    },
    "thing": {
      // references a URL
      "$ref": "http://wayne-enterprises.com/things/batmobile"
    },
    "color": {
      // references a value in an external file via an internal reference
      "$ref": "#/definitions/thing/properties/colors/black-as-the-night"
    }
  }
}
```


The Solution:
--------------------------
JSON Schema $Ref Parser is a full [JSON Reference](https://tools.ietf.org/html/draft-pbryan-zyp-json-ref-03) and [JSON Pointer](https://tools.ietf.org/html/rfc6901) implementation that crawls even the most complex [JSON Schemas](http://json-schema.org/latest/json-schema-core.html) and gives you simple, straightforward JavaScript objects.

- Use **JSON** or **YAML** schemas &mdash; or even a mix of both!
- Supports `$ref` pointers to external files and URLs, as well as [custom sources](https://apidevtools.org/json-schema-ref-parser/docs/plugins/resolvers.html) such as databases
- Can [bundle](https://apidevtools.org/json-schema-ref-parser/docs/ref-parser.html#bundlepath-options-callback) multiple files into a single schema that only has _internal_ `$ref` pointers
- Can [dereference](https://apidevtools.org/json-schema-ref-parser/docs/ref-parser.html#dereferencepath-options-callback) your schema, producing a plain-old JavaScript object that's easy to work with
- Supports [circular references](https://apidevtools.org/json-schema-ref-parser/docs/#circular-refs), nested references, back-references, and cross-references between files
- Maintains object reference equality &mdash; `$ref` pointers to the same value always resolve to the same object instance
- [Tested](https://travis-ci.com/APIDevTools/json-schema-ref-parser) in Node and all major web browsers on Windows, Mac, and Linux


Example
--------------------------

```javascript
$RefParser.dereference(mySchema, (err, schema) => {
  if (err) {
    console.error(err);
  }
  else {
    // `schema` is just a normal JavaScript object that contains your entire JSON Schema,
    // including referenced files, combined into a single object
    console.log(schema.definitions.person.properties.firstName);
  }
}
```

Or use `async`/`await` syntax instead. The following example is the same as above:

```javascript
try {
  let schema = await $RefParser.dereference(mySchema);
  console.log(schema.definitions.person.properties.firstName);
}
catch(err) {
  console.error(err);
}
```

For more detailed examples, please see the [API Documentation](https://apidevtools.org/json-schema-ref-parser/docs/)



Installation
--------------------------
Install using [npm](https://docs.npmjs.com/about-npm/):

```bash
npm install json-schema-ref-parser
```



Usage
--------------------------
When using Json-Schema-Ref-Parser in Node.js apps, you'll probably want to use **CommonJS** syntax:

```javascript
const $RefParser = require("json-schema-ref-parser");
```

When using a transpiler such as [Babel](https://babeljs.io/) or [TypeScript](https://www.typescriptlang.org/), or a bundler such as [Webpack](https://webpack.js.org/) or [Rollup](https://rollupjs.org/), you can use **ECMAScript modules** syntax instead:

```javascript
import $RefParser from "json-schema-ref-parser";
```



Browser support
--------------------------
Json-Schema-Ref-Parser supports recent versions of every major web browser.  Older browsers may require [Babel](https://babeljs.io/) and/or [polyfills](https://babeljs.io/docs/en/next/babel-polyfill).

To use Json-Schema-Ref-Parser in a browser, you'll need to use a bundling tool such as [Webpack](https://webpack.js.org/), [Rollup](https://rollupjs.org/), [Parcel](https://parceljs.org/), or [Browserify](http://browserify.org/). Some bundlers may require a bit of configuration, such as setting `browser: true` in [rollup-plugin-resolve](https://github.com/rollup/rollup-plugin-node-resolve).



API Documentation
--------------------------
Full API documentation is available [right here](https://apidevtools.org/json-schema-ref-parser/docs/)


Contributing
--------------------------
I welcome any contributions, enhancements, and bug-fixes.  [File an issue](https://github.com/APIDevTools/json-schema-ref-parser/issues) on GitHub and [submit a pull request](https://github.com/APIDevTools/json-schema-ref-parser/pulls).

#### Building/Testing
To build/test the project locally on your computer:

1. __Clone this repo__<br>
`git clone https://github.com/APIDevTools/json-schema-ref-parser.git`

2. __Install dependencies__<br>
`npm install`

3. __Run the build script__<br>
`npm run build`

4. __Run the tests__<br>
`npm test`


License
--------------------------
JSON Schema $Ref Parser is 100% free and open-source, under the [MIT license](LICENSE). Use it however you want.

Big Thanks To
--------------------------
Thanks to these awesome companies for their support of Open Source developers ❤

[![Travis CI](https://jsdevtools.org/img/badges/travis-ci.svg)](https://travis-ci.com)
[![SauceLabs](https://jsdevtools.org/img/badges/sauce-labs.svg)](https://saucelabs.com)
[![Coveralls](https://jsdevtools.org/img/badges/coveralls.svg)](https://coveralls.io)
