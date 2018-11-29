# javascript deferreds for simplicity

Simplicity... possibly.

I had a situation where I have a complex options object passed to a function and I wanted to unwrap it, like so:

```javascript
function blah(options) {
  const {
     option1,
     option2
  } = options;
  console.log(option1, option2);
}
```
But what if you need a default for one of those options?

```javascript
function blah(options) {
  const {
     option1 = "twelve",
     option2
  } = options;
  console.log(option1, option2);
}
```

But what if the default needs to be based on something that isn't available till later? For example, a url? 

That is perfectly feasible. The situation I was in was with a url based password store. The caller might want to pass in a default url for the password store... but during dev want it computed to the local, default store.

So how?

You need to make the value lazy somehow. Fortunately we have a good way of doing this even with strings: valueOf.

So if we do this:

```javascript
function blah(options) {
  const {
     option1 = "twelve",
     option2
  } = options;
  console.log(option1.valueOf(), option2);
}
```

we have a chance.

How do we make the value actually deferred though?

```javascript
const DeferredHolder = function () {
    this.value = undefined;
    this.setValue = function (value) { this.value = value; };
};
DeferredHolder.prototype.valueOf = function () {
    return this.value;
};

const defaultKeepieUrl = new DeferredHolder();

function blah(options) {
  const defaultValue = new DeferredHolder();
  const {
     option1 = defaultValue,
     option2
  } = options;
  
  defaultValue.setValue("twenty");
  console.log(option1.valueOf(), option2);
}
```

Would produce:

```
twenty undefined
```

So here's a more thorough run through:

```javascript

const DeferredHolder = function () {
    this.hostPort = undefined;
    this.setHost = function (hostPort) { this.hostPort = hostPort; };
};
DeferredHolder.prototype.valueOf = function () {
    return this.hostPort;
};

const defaultKeepieUrl = new DeferredHolder();

const {
    keepieUrl = defaultKeepieUrl
} = { keepieUrl: "string!" };

console.log(keepieUrl.valueOf());


const {
    keepieUrl:keepers1 = defaultKeepieUrl
} = { keepieUrlX: "string!" };

console.log(keepers1.valueOf());


defaultKeepieUrl.setHost("localhost:5000");
const {
    keepieUrl:keepers2 = defaultKeepieUrl,
    second
} = { keepieUrlX: "string!" };

console.log(keepers2.valueOf(), second);
```

Produces:

```
string!
undefined
localhost:5000 undefined
```

Ta da.
