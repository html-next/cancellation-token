# Cancellation Token

Tokens are extremely lightweight and easy to grok, their power lies in
their composability. The entire implementation is this:

```js
export default class Token {
  constructor(parent) {
    this._parent = parent;
    this._cancelled = false;
  }

  get cancelled() {
    return this._cancelled || (this._cancelled = this._parent ? this._parent.cancelled : false);
  }

  cancel() {
    this._cancelled = true;
  }
}
```

## Usage

Each example builds on the previous in order.

**create**
```js
import Token from 'cancellation-token';

let token = new Token();
```

**conditional work**
```js
let jobs = [];

function cancellableJob(job, token) {
  return function() {
    if (!token.cancelled) {
      job();
    }
  };
}

function schedule(job) {
  let token = new Token();
  
  jobs.push(cancellableJob(job, token);
  
  return token;
}
```

**cancelling work**
```js
let scheduledJob = schedule(() => { alert("hello world!"); });
scheduledJob.cancel();
```

**inheritance**
```js
let grandParentToken = new Token();
let parentToken = new Token(grandParentToken);

function scheduleWithParent(job, parentToken) {
  let token = new Token(parentToken);
  
  jobs.push(cancellableJob(job, token);
  
  return token;
}

// cancel the task and all descendants
grandParentToken.cancel();
```