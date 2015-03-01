# lisp-json-to-js
> Transpiles lisp code represented as object to JS-code

## Install
```
$ npm install --save lisp-json-to-js
```
or, to install it globally:
```
$ npm install -g lisp-json-to-js
```

## Functions
### `.-`
Gets or sets a property.
```json
[".-", "document", ["`", "body"]]
[".-", "obj", ["`", "prop"], "newValue"]
```
compiles to:
```js
env["document"]["body"]
env["obj"]["prop"] = env["newValue"]
```

### `.`
Calls a method.
```json
[".", "document", ["`", "querySelector"], ["`", "body"]]
```
compiles to:
```js
env["document"]["querySelector"]("body")
```

### `` ` ``
Returns the first argument.
```json
["`", "querySelector"]
["`", ["example1", "example2"]]
```
compile to:
```js
"querySelector"
["example1", "example2"]
```

### `=`
Checks for strict equality.
```json
["=", "a", "b"]
```
compiles to:
```js
env["a"] === env["b"]
```

### `def`
Defines a variable.
```json
["def", "var", "val"]
```
compiles to:
```js
env["var"] = env["val"]
```

### `do`
Executes multiple functions and returns the last.
```json
["do", ["def", "var", ["`", "val"]], ["+", "var", ["`", "val2"]]]
```
compiles to:
```js
(function() {
    env["var"] = "val";
    return env["var"] + "val2";
})()
```

### `fn`
A function expression.
```json
["fn", ["arg1", "&", "arg 2"], [".", "env", ["`", "arg1"], "arg 2"]]
```
compiles to:
```js
(function() {
    var __args = arguments;
    return (function(env) {
        env["arg1"] = __args[0];
        env["arg 2"] = __args[2];
        return env["env"]["arg1"](env["arg 2"]);
    })(Object.create(env));
})
```
Note: the __args and the IIFE are required because variable names can
contain symbols which would be invalid in JavaScript, such as whitespaces.

### `if`
Returns the second argument if the first argument is true, the third argument
otherwise.
```json
["if", ["=", "var1", "var2"], ["`", "isTrue"], ["`", "isFalse"]]
```
compiles to:
```js
((env["var1"] === env["var2"])
    ? "isTrue"
    : "isFalse")
```

### `js`
Executes javascript code.
```json
["js", "document.body.appendChild(new Text('test'))"]
```
compiles to:
```js
document.body.appendChild(new Text('test'))
```

### `let`
Creates a new lexical scope, defines variables in it and then executes code in it.
```json
["let", ["a", 5, "b", 10], ["+", "a", "b"]]
```
compiles to:
```js
(function(env) {
    env["a"] = 5;
    env["b"] = 10;
    return env["a"] + env["b"];
})(Object.create(env))
```

### `try` and `catch`
A try/catch block. Note that the second argument must be a call to catch.
```json
["try", "test", ["catch", "e", ["list", ["`", "Error: "], "e"]]]
```
compiles to:
```js
try {
    env["test"]
} catch(__e) {
    (function(env) {
        env["e"] = env["__e"];
        return env["list"]("Error: ", env["e"]);
    })(Object.create(env))
}
```
