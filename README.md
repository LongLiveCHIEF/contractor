# NPM Contractor

A chef/puppet like tool that allows you to use npm scripts as your build tool, even with complicated tasks, by creating "contracts". After creating these contracts, they can be published to npm using the `npm-contract-` to be automatically parsed and utilized by `npm install`

(I intend to fork npm, and add support for a "contracts" object in package.json, but this should work even if it's not added to the npm tool itself")

### Benefits

- get the benefits of "configuration" (ala Grunt), with the native javascript composability of gulp
- uses native OS streaming/piping, stdout/stdin/stderr
- automatically allows you to use modules that don't have registered cli's as part of your npm build system
- can add your contracts to your `.gitignore` opening up the possibility of OS specific builds
- can utilize the output of *any other* build tool (grunt, gulp, broccolli), to allow you to fork and utilize modules from other teams and run quickly integrate their build process into yours

## How it works

A "contract" is simply a javascript module, that takes input, and does something with it, optionally returning output. Each `contract` file under the `projects/contracts` folder, will be added to the projects local `contractor-cli`, as a command available. 

Each `contract` is passed any output piped to it from other `npm scripts`, `contracts` you chain, or aruguments you pass. When a contract is added using `--save`, it will modify the `package.json`'s `script` object.

Internally, contractor will use `npm run` and it's conventions to pipe output, execute commands, etc...

### Example

```
- project/
 | - contracts/
   | - build.js
   | - linter.js
   | - register.js
   | - server.js
   | - watch.js
```


```bash
$> contracts register // parses your /path/to/project/node_modules/contrator/contracts, and adds files as cli executable command options to the contract binary
```

```json
{
  "scripts": {
    "start":"contract server.env=dev&then:watch" 
  }
}
```

Example `contracts/server.js` file:

```js
var http = require('http-server');
var contractor = require('contractor');
var contract = contractor.contract;
 
contract.setState("developing"); // the current "state" of your project, more to come on this later
 
// contract.getOptions() will parse out of arguments passed to the script
var contract.options = contract.getOptions(); // assigns any options passed in cli to contract.options and made available to the contractor
 
var port = contract.options.port = options.port; // takes the aruguments, parses out the port=###
var ssl = contract.options.ssl = options['-S'];
 
function serverScript(){
   var script = 'http-server';
   
   if(port) script += ' -p ' + port;
   if(ssl) script += ' -S ' + ssl;
   return script;
}
 
module.exports = serverScript();
```
