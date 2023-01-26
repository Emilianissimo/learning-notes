# Node JS (only)

### Server on pure Node
Simple server -> example https://github.com/Emilianissimo/nodejs-seerver-example
JSON and HTML server, working simple af in case we have http modeule, for deeply desc follow the link

### Modules
Path - using for working with PATH (lmao) and able with file/its path as string, getting extension, names and also parsing it as well.
FS - need to operate filesystem. Storing bytes as file and read them and et cetera...
OS - providing utilities to operate OS, like getting arch, reading something like freemem, cpu stuff, very close to OS from python (lmao, the same purpose).
HTTP - is something, that we see in server example.
Events - just events, like in React/Vue and other frame_work_name, like:
```js
import { EventEmitter } from 'node:events';

class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter();
myEmitter.on('event', () => {
  console.log('an event occurred!');
});
myEmitter.emit('event');
```

### Module system
Like in Petuhon, Node.js modules are just files, which exporting something. You cannot access anything if you not exporting it.
```js
//main.js
var hello = require('./hello');
hello.world();

//hello.js 
exports.world = function() {
  console.log('Hello World');
}
// or main.js
var Hello = require('./hello'); 
hello = new Hello(); 
hello.setName('BYVoid'); 
hello.sayHello(); 

// hello.js
function Hello() { 
	var name; 
	this.setName = function(thyName) { 
		name = thyName; 
	}; 
	this.sayHello = function() { 
		console.log('Hello ' + name); 
	}; 
}; 
module.exports = Hello;
```
# Command line args
To take command line args, we need just to call process.args -> it returns an array of passing args to this main func (running file).
