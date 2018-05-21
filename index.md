---
layout: default
---

<script type="text/javascript">
const originalOnError = window.onerror;
function setupRunner(name, fn) {
  const testRunner = document.querySelector('[data-run='+name+']');
  const testOutput = document.querySelector('[data-output='+name+']');
  testRunner.addEventListener('click', function () {
    console.log(fn);
    window.onerror = function(message, source, lineno, colno, error) {
      const stack = error && error.stack;
      const onerror = JSON.stringify({
        message: message,
        source: source,
        lineno: lineno,
        colno: colno
      }, null, 2);
      const errObj = JSON.stringify({
        message: error && error.message,
        sourceURL: error && error.sourceURL,
        line: error && error.line,
        column: error && error.column
      }, null, 2);
      testOutput.innerHTML = `
=== Stack ===
${stack}

=== window.onerror args ===
${onerror}

=== window.onerror error arg ===
${errObj}
`
      return true;
    };
    fn();
    window.onerror = originalOnError;
  });
}
</script>

Errors which are not caught in a try/catch will bubble up the window's error handler.
  This is your last chance to log the error for reporting.
  Some browsers report slightly different information, or will not report any information at all.
  This page will interactively explain these intricacies, and acts as a "test runner".

## Inline function on same page

This function is inlined into this page, so it is served from the same domain. Error reporting should be correct.

```javascript
function inlineFn () {
  throw new Error('inlineFn-error');
}
```

<button data-run="inlineFn">Run inlineFn()</button>

<pre><code class="json" data-output="inlineFn"></code></pre>

<script type="text/javascript">
function inlineFn () {
  throw new Error('inline-error');
}
setupRunner("inlineFn", inlineFn);
</script>

| Browser | window.onerror<br>msg | window.onerror<br>url  | window.onerror<br>line:col | error<br>msg | error<br>url | error<br>line:col |
|:------|:------|:------|:------|:------|:------|:------|
| Chrome | Uncaught Error: inline-error | https://safari-cors-errors.framba.ch/ | 79:3 | inline-error |  |
| Firefox | Error: inline-error | https://safari-cors-errors.framba.ch/ | 79:9 | inline-error |  |
| Safari | Error: inline-error | https://safari-cors-errors.framba.ch/ | 79:34 | inline-error | https://safari-cors-errors.framba.ch/ | 79:18 |

## Generated function on same page

An inline script generates a new function from a string and executes it.

```javascript
function newFn () {
  const fn = new Function("throw new Error('newFn-error')");
  fn();
}
```

<button data-run="newFn">Run newFn()</button>

<pre><code class="json" data-output="newFn"></code></pre>

<script type="text/javascript">
function newFn () {
  const fn = new Function("throw new Error('newFn-error')");
  fn();
}
setupRunner("newFn", newFn);
</script>

| Browser | window.onerror<br>msg | window.onerror<br>url | window.onerror<br>line:col | error<br>msg | error<br>url<br>line:col |
|:------|:------|:------|:------|:------|:------|:------|
| Chrome | Uncaught Error: newFn-error | https://safari-cors-errors.framba.ch/ | 3:1 | newFn-error |  |
| Firefox | Error: newFn-error | https://safari-cors-errors.framba.ch/ line 143 > Function | 3:7 | newFn-error |  |
| Safari | Error: newFn-error |  | 2:16 | newFn-error | | 2:16 |

## Calling an external function

<p>This function is defined on a different domain, with crossorigin="anonymous" and CORS enabled.
  Safari shows "Script Error.", ostensibly for secrity reasons.</p>

```javascript
// Defined externally
function externalFn() {
  throw new Error("externalFn-error");
}
```

<button data-run="externalFn">Run externalFn()</button>

<pre><code class="json" data-output="externalFn"></code></pre>

<script type="text/javascript">
setupRunner("externalFn", externalFn);
</script>

| Browser | window.onerror<br>msg | window.onerror<br>url | window.onerror<br>line:col | error<br>msg | error<br>url<br>line:col |
|:------|:------|:------|:------|:------|:------|:------|
| Chrome | Uncaught Error: external-cors-error | https://safari-cors-external.framba.ch/external-fn.js | 2:9 | external-cors-error |  |
| Firefox | Error: external-cors-error | https://safari-cors-external.framba.ch/external-fn.js | 2:9 | external-cors-error |  |
| Safari | Script error. |  |  |  |  |  |

## Calling and rethrowing an external function

<p>This function is defined on a different domain, with crossorigin="anonymous" and CORS enabled.
  On our domain, we try/catch the function call, but immediately rethrow the error.</p>

```javascript
// Defined externally
function externalFn() {
  throw new Error("externalFn-error");
}

// Defined inline
function rethrowExternalFn() {
  try {
    externalFn();
  } catch (e) {
    throw e;
  }
}
```

<button data-run="rethrowExternalFn">Run rethrowExternalFn()</button>

<pre><code class="json" data-output="rethrowExternalFn"></code></pre>

<script type="text/javascript">
function rethrowExternalFn() {
  try {
    externalFn();
  } catch (e) {
    throw e;
  }
}
setupRunner("rethrowExternalFn", rethrowExternalFn);
</script>

| Browser | window.onerror<br>msg | window.onerror<br>url | window.onerror<br>line:col | error<br>msg | error<br>url<br>line:col |
|:------|:------|:------|:------|:------|:------|:------|
| Chrome | Uncaught Error: external-cors-error | https://safari-cors-errors.framba.ch/ | 283:5 | external-cors-error |  |
| Firefox | Error: external-cors-error | https://safari-cors-external.framba.ch/external-fn.js | 2:9 | external-cors-error |  |
| Safari | Error: external-cors-error | https://safari-cors-errors.framba.ch/ | 283:12 | external-cors-error | https://safari-cors-external.framba.ch/external-fn.js | 2:18 |
