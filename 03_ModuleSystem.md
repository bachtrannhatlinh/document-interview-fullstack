# 3. MODULE SYSTEM - INTERVIEW GUIDE

## 📜 LỊCH SỬ & EVOLUTION

### Tại sao Module System quan trọng?
```javascript
// Before modules - Global Namespace Pollution
var userName = 'John'; // Global variable
function getUserName() { return userName; } // Global function

// Problems:
// 1. Name conflicts
// 2. No encapsulation  
// 3. Dependency management hell
// 4. No code reusability
```

### Evolution Timeline
```
2009: CommonJS (Server-side modules)
2015: ES6 Modules (Browser standard)
2017: Node.js experimental ESM support
2020: Node.js stable ESM support (v14+)
2023: ESM becomes default in modern tooling
```

---

## 🔧 COMMONJS IN-DEPTH

### Module Wrapper Mechanism
```javascript
// Node.js wraps every module in this function:
(function(exports, require, module, __filename, __dirname) {
  // Your module code here
  console.log('Current file:', __filename);
  console.log('Current directory:', __dirname);
  console.log('Module object:', module);
});

// Practical example:
// utils.js
console.log('Module ID:', module.id);
console.log('Module filename:', module.filename);
console.log('Module parent:', module.parent);
console.log('Module children:', module.children);
```

### exports vs module.exports
```javascript
// ❌ Common mistake
exports = { name: 'John' }; // This breaks the reference!

// ✅ Correct ways
exports.name = 'John';
exports.age = 25;

// Or
module.exports = { name: 'John', age: 25 };

// Understanding the reference
console.log(exports === module.exports); // true initially

// When exports reference is broken:
exports = { name: 'Jane' };
console.log(exports === module.exports); // false now!

// utils.js - Demonstrating different export patterns
module.exports = function Calculator() {
  this.add = (a, b) => a + b;
};

// Or named exports
module.exports.add = (a, b) => a + b;
module.exports.subtract = (a, b) => a - b;

// Or mixed
module.exports = {
  add: (a, b) => a + b,
  PI: 3.14159
};
```

### Require Cache & Singleton Pattern
```javascript
// cache-demo.js
console.log('Module executed!');
let counter = 0;

module.exports = {
  increment: () => ++counter,
  getCount: () => counter
};

// app.js
const counter1 = require('./cache-demo'); // "Module executed!" logs
const counter2 = require('./cache-demo'); // No log - cached!

console.log(counter1 === counter2); // true - same instance

counter1.increment();
console.log(counter2.getCount()); // 1 - shared state

// Cache inspection
console.log(require.cache);
console.log(Object.keys(require.cache));

// Hot reload - clear cache
delete require.cache[require.resolve('./cache-demo')];
const counter3 = require('./cache-demo'); // "Module executed!" logs again
```

### Circular Dependencies
```javascript
// a.js
console.log('A starting');
exports.done = false;
const b = require('./b');
console.log('in A, B.done =', b.done);
exports.done = true;
console.log('A done');

// b.js  
console.log('B starting');
exports.done = false;
const a = require('./a');
console.log('in B, A.done =', a.done);
exports.done = true;
console.log('B done');

// main.js
const a = require('./a');
const b = require('./b');

/* Output:
A starting
B starting
in B, A.done = false  // A not fully loaded yet!
B done
in A, B.done = true
A done
*/

// ✅ Better pattern - avoid circular deps
// Use dependency injection or event emitters
const EventEmitter = require('events');
const bus = new EventEmitter();

// moduleA.js
const bus = require('./eventBus');
bus.on('moduleB:ready', (data) => {
  console.log('Module B is ready:', data);
});

// moduleB.js
const bus = require('./eventBus');
bus.emit('moduleB:ready', { status: 'initialized' });
```

---

## 🔍 MODULE RESOLUTION ALGORITHM

### Node.js Resolution Steps
```javascript
// When you do: require('lodash')
// Node.js searches in this order:

// 1. Core modules (built-in)
require('fs');        // ✅ Core module
require('path');      // ✅ Core module

// 2. File (with extensions)
require('./math');         // Tries math.js, math.json, math.node
require('./math.js');      // Direct file

// 3. Directory (with package.json or index)
require('./utils');        // Tries utils/package.json -> main field
                          // Or utils/index.js

// 4. node_modules (current -> parent -> parent...)
require('lodash');         // ./node_modules/lodash
                          // ../node_modules/lodash  
                          // ../../node_modules/lodash
                          // etc.

// Resolution debugging
console.log(require.resolve('lodash'));
console.log(require.resolve('./math.js'));
```

### package.json Fields Priority
```json
{
  "name": "my-package",
  "version": "1.0.0",
  
  // Module resolution fields (priority order):
  "exports": {
    ".": {
      "import": "./lib/index.mjs",
      "require": "./lib/index.cjs",
      "types": "./types/index.d.ts"
    },
    "./utils": "./lib/utils.js"
  },
  
  "main": "./lib/index.js",      // Fallback for older Node
  "module": "./lib/index.mjs",   // Used by bundlers
  "browser": "./lib/browser.js", // Browser-specific
  "types": "./types/index.d.ts", // TypeScript definitions
  
  // Conditional exports
  "exports": {
    ".": {
      "node": "./lib/node.js",
      "browser": "./lib/browser.js", 
      "default": "./lib/index.js"
    }
  }
}
```

### Custom Require Extensions (Legacy)
```javascript
// ❌ Deprecated but still works
require.extensions['.txt'] = function(module, filename) {
  const fs = require('fs');
  const content = fs.readFileSync(filename, 'utf8');
  module.exports = content;
};

// Now you can:
const readme = require('./README.txt');

// ✅ Modern alternative: Use loaders or bundlers
```

---

## 🚀 ES MODULES (ESM) ADVANCED

### Enabling ESM in Node.js
```json
// package.json - Method 1
{
  "type": "module"
}

// Now all .js files are ESM
// Use .cjs for CommonJS files
```

```javascript
// Method 2: Use .mjs extension
// math.mjs
export const add = (a, b) => a + b;
export default class Calculator {
  multiply(a, b) { return a * b; }
}

// app.mjs
import Calculator, { add } from './math.mjs';
const calc = new Calculator();
```

### ESM Characteristics
```javascript
// 1. Static Analysis - Imports are hoisted
console.log(add(2, 3)); // ✅ Works! Hoisted
import { add } from './math.mjs';

// 2. Read-only bindings
import { counter } from './counter.mjs';
counter++; // ❌ TypeError: Assignment to constant variable

// 3. Strict mode by default
// No need for 'use strict'

// 4. Different this context
console.log(this); // undefined in ESM, {} in CommonJS
```

### import.meta Object
```javascript
// Current module URL
console.log(import.meta.url);
// file:///path/to/current/module.js

// Resolve relative paths
const configPath = new URL('./config.json', import.meta.url);
console.log(configPath.pathname);

// Dynamic import with import.meta.resolve (experimental)
const specifier = await import.meta.resolve('./utils.js');
const utils = await import(specifier);
```

### Dynamic Imports
```javascript
// Code splitting / lazy loading
async function loadMath() {
  if (needComplexMath) {
    const { Calculator } = await import('./advanced-math.js');
    return new Calculator();
  }
  return null;
}

// Conditional loading
const dbModule = process.env.NODE_ENV === 'test' 
  ? await import('./mock-db.js')
  : await import('./real-db.js');

// Import with error handling
try {
  const plugin = await import(`./plugins/${pluginName}.js`);
  plugin.initialize();
} catch (error) {
  console.error('Plugin loading failed:', error);
}

// JSON imports (requires assert)
import config from './config.json' assert { type: 'json' };
console.log(config.apiUrl);
```

### Top-level await
```javascript
// ESM allows top-level await
const response = await fetch('https://api.example.com/config');
const config = await response.json();

export { config };

// This affects module loading order!
// Modules with top-level await load asynchronously
```

---

## 🔄 COMMONJS ↔ ESM INTEROPERABILITY

### ESM importing CommonJS
```javascript
// CJS module: math.cjs
module.exports = {
  add: (a, b) => a + b,
  PI: 3.14159
};
module.exports.default = module.exports; // For ESM compatibility

// ESM importing CJS
import math from './math.cjs';         // Gets the whole module.exports
import { add } from './math.cjs';      // ❌ Named imports don't work
                                       // Unless cjs exports are statically analyzable

// Correct way:
import math from './math.cjs';
const { add, PI } = math;
```

### CommonJS importing ESM
```javascript
// ESM module: utils.mjs
export const helper = () => 'helper';
export default { version: '1.0.0' };

// CJS trying to import ESM
const utils = require('./utils.mjs'); // ❌ Error: require of ES modules not supported

// ✅ Solution: Use dynamic import
async function loadESM() {
  const utils = await import('./utils.mjs');
  console.log(utils.default.version);
  console.log(utils.helper());
}

// Or use createRequire
import { createRequire } from 'module';
const require = createRequire(import.meta.url);
const oldModule = require('./old-cjs-module');
```

### Dual Package Strategy
```javascript
// package.json for dual package
{
  "name": "my-dual-package",
  "exports": {
    ".": {
      "import": "./esm/index.js",
      "require": "./cjs/index.js"
    }
  },
  "main": "./cjs/index.js",
  "type": "module"
}

// esm/index.js
export function add(a, b) { return a + b; }

// cjs/index.js
function add(a, b) { return a + b; }
module.exports = { add };

// Build scripts often generate both versions
```

---

## 🏗️ BUILT-IN MODULES & NATIVE ADDONS

### Core Modules Categories
```javascript
// File System
import { readFile, writeFile } from 'fs/promises';
import { createReadStream } from 'fs';

// Path utilities
import { join, resolve, dirname } from 'path';
import { fileURLToPath } from 'url';

// HTTP/Networking
import { createServer } from 'http';
import { request } from 'https';

// Crypto
import { createHash, randomBytes } from 'crypto';

// Utilities
import { promisify, inspect, format } from 'util';

// Process & OS
import { platform, cpus, totalmem } from 'os';
import process from 'process';

// Streams
import { Transform, pipeline } from 'stream';

// Worker Threads
import { Worker, isMainThread, parentPort } from 'worker_threads';
```

### Native Addons (.node files)
```javascript
// Native addon resolution
const binding = require('./build/Release/addon.node');

// Or using N-API
const addon = require('bindings')('addon');

// Checking if native addon is available
try {
  const nativeModule = require('native-module');
  console.log('Native module loaded');
} catch (error) {
  console.log('Falling back to JavaScript implementation');
  const jsModule = require('./js-fallback');
}
```

---

## 📦 NPM & DEPENDENCY MANAGEMENT

### Package Installation Types
```bash
# Local dependencies (saved to package.json)
npm install lodash                    # Production dependency
npm install --save-dev jest          # Development dependency
npm install --save-optional imagemin # Optional dependency

# Global installation
npm install -g nodemon

# Peer dependencies (package.json)
{
  "peerDependencies": {
    "react": ">=16.0.0"
  }
}
```

### Semantic Versioning & Version Ranges
```json
{
  "dependencies": {
    "lodash": "4.17.21",      // Exact version
    "express": "^4.18.0",     // Compatible version (4.x.x)
    "mongoose": "~6.5.0",     // Approximately equivalent (6.5.x)
    "chalk": ">=4.0.0",       // Greater than or equal
    "debug": "4.1.0 - 4.3.0"  // Range
  }
}
```

### Lock Files & Security
```javascript
// package-lock.json ensures exact versions
// npm audit for vulnerabilities
// npm audit fix for automatic fixes

// Checking package integrity
const crypto = require('crypto');
const fs = require('fs');

function verifyPackage(packagePath, expectedHash) {
  const content = fs.readFileSync(packagePath);
  const hash = crypto.createHash('sha256').update(content).digest('hex');
  return hash === expectedHash;
}
```

---

## 🔧 BUNDLERS & TREE SHAKING

### How Tree Shaking Works
```javascript
// utils.js - Only export what's needed
export const used = () => 'I am used';
export const unused = () => 'I am unused'; // Will be removed

// main.js
import { used } from './utils.js'; // Only 'used' is bundled

// For CommonJS - Tree shaking is harder
const utils = require('./utils'); // Entire module is included
const { used } = utils;

// ESM enables better static analysis
```

### Bundle Analysis
```javascript
// webpack.config.js
module.exports = {
  resolve: {
    mainFields: ['browser', 'module', 'main'] // Priority order
  },
  optimization: {
    usedExports: true,    // Mark unused exports
    sideEffects: false,   // Enable tree shaking
  }
};

// package.json
{
  "sideEffects": ["*.css", "./src/polyfills.js"] // Files with side effects
}
```

---

## 🛡️ SECURITY & BEST PRACTICES

### Module Security Policies
```javascript
// --experimental-policy flag
// policy.json
{
  "resources": {
    "./app.js": {
      "integrity": "sha384-..."
    }
  },
  "scopes": {
    "./node_modules/": {
      "integrity": true
    }
  }
}

// Run with: node --experimental-policy=policy.json app.js
```

### Preventing Prototype Pollution
```javascript
// Dangerous dynamic require
const userInput = req.body.module; // "__proto__"
const module = require(userInput);  // ❌ Potential security issue

// ✅ Safe approach - whitelist
const allowedModules = {
  'lodash': require('lodash'),
  'moment': require('moment')
};

const module = allowedModules[userInput];
if (!module) {
  throw new Error('Module not allowed');
}
```

### Exports Field Security
```json
{
  "exports": {
    ".": "./public/index.js",
    "./public/*": "./public/*"
  }
  // Users cannot access: ./internal/secret.js
}
```

---

## 🎯 INTERVIEW QUESTIONS & SCENARIOS

### **Question 1: exports vs module.exports**
```javascript
// What will this output?
// file1.js
exports.a = 1;
module.exports.b = 2;
exports = { c: 3 };
module.exports.d = 4;

// main.js
const obj = require('./file1');
console.log(obj);

// Answer: { a: 1, b: 2, d: 4 }
// 'c' is lost because exports reference was broken
```

### **Question 2: Circular Dependency**
```javascript
// a.js
console.log('A start');
exports.done = false;
const b = require('./b');
console.log('A got:', b.done);
exports.done = true;

// b.js
console.log('B start');
exports.done = false;
const a = require('./a');
console.log('B got:', a.done);
exports.done = true;

// main.js
require('./a');

// What's the output and why?
// Answer: 
// A start
// B start  
// B got: false (A not fully loaded)
// A got: true
```

### **Question 3: Module Resolution**
```javascript
// Given this structure:
// /project
//   /node_modules
//     /lodash
//   /lib  
//     /utils
//       /node_modules
//         /debug
//       index.js
//   app.js

// From /lib/utils/index.js:
require('lodash');  // Finds where?
require('debug');   // Finds where?  
require('fs');      // Finds where?

// Answer:
// lodash: /project/node_modules/lodash
// debug: /lib/utils/node_modules/debug  
// fs: Core module (built-in)
```

### **Question 4: ESM vs CommonJS Loading**
```javascript
// Which loads first and why?

// a.mjs
console.log('A loading');
await new Promise(resolve => setTimeout(resolve, 100));
console.log('A loaded');
export const a = 1;

// b.mjs  
console.log('B loading');
import { a } from './a.mjs';
console.log('B loaded', a);
export const b = 2;

// main.mjs
import { a } from './a.mjs';
import { b } from './b.mjs';

// Answer: A loading -> A loaded -> B loading -> B loaded
// ESM resolves dependencies before execution
```

### **Question 5: Dynamic Import vs Require**
```javascript
// Performance comparison - explain the difference

// Method A: CommonJS
const fs = require('fs');
if (someCondition) {
  const heavy = require('./heavy-module'); // Loaded upfront
}

// Method B: ESM Dynamic Import
import fs from 'fs';
if (someCondition) {
  const heavy = await import('./heavy-module'); // Lazy loaded
}

// Which is better for performance and why?
// Answer: Method B - dynamic import is truly lazy
```

---

## ✅ MODULE SYSTEM CHECKLIST

### **Core Concepts:**
- [ ] Module wrapper function và built-in variables
- [ ] exports vs module.exports differences  
- [ ] Require cache và singleton pattern
- [ ] Circular dependency handling
- [ ] Module resolution algorithm

### **ESM Features:**
- [ ] Static vs dynamic imports
- [ ] import.meta object usage
- [ ] Top-level await implications
- [ ] Conditional exports
- [ ] Import assertions (JSON, CSS)

### **Interoperability:**
- [ ] CommonJS ↔ ESM interop rules
- [ ] createRequire() usage
- [ ] Dual package strategies
- [ ] Migration paths

### **Advanced Topics:**
- [ ] Native addons (.node files)
- [ ] Custom loaders/transformers
- [ ] Security policies
- [ ] Bundle optimization strategies
- [ ] Tree shaking mechanisms

### **Production Concerns:**
- [ ] Dependency management (npm/yarn/pnpm)
- [ ] Lock file importance
- [ ] Security auditing
- [ ] Performance optimization
- [ ] Caching strategies

---

## 🚀 PRACTICAL EXERCISES

1. **Build Module Loader**: Create custom require() implementation
2. **Dependency Resolver**: Build npm-like module resolution
3. **Circular Dependency Detector**: Analyze module graphs
4. **ESM to CommonJS Transformer**: Build transpiler
5. **Module Bundler**: Create simple Webpack-like tool

---

## 📚 ESSENTIAL RESOURCES

- **Node.js Modules Guide**: [Official Documentation](https://nodejs.org/api/modules.html)
- **ES Modules Guide**: [MDN Documentation](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)
- **Module Resolution**: [Node.js Algorithm](https://nodejs.org/api/modules.html#modules_all_together)
- **Package.json Fields**: [Complete Reference](https://docs.npmjs.com/cli/v7/configuring-npm/package-json)
- **Books**: "Understanding ECMAScript 6" by Nicholas Zakas, "Node.js Design Patterns" by Mario Casciaro
