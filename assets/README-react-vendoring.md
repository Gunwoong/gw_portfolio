# React/ReactDOM vendoring

`support.js` (the DC runtime) normally loads React 18.3.1 and ReactDOM 18.3.1
from `unpkg.com` at page-load time, and only skips that fetch if
`window.React` and `window.ReactDOM` are already defined
(see `loadReactUmd()` in `support.js`).

Relying on a third-party CDN for a hard runtime dependency means the entire
page fails to render — falling back to the raw, un-interpolated
`{{ t.heroTitle }}`-style template markup — whenever `unpkg.com` is slow,
blocked by a corporate proxy/firewall, blocked by an ad blocker, or down.
That's a real risk for a portfolio site being shared with recruiters, who
are often on locked-down corporate networks.

To remove that dependency, `react.production.min.js` and
`react-dom.production.min.js` in this folder are vendored copies of the
exact versions `support.js` expects (verified byte-for-byte against the
SRI hashes `REACT_SRI` / `REACT_DOM_SRI` in `support.js`) and are loaded
locally in `index.html` **before** `support.js`. Since `window.React` /
`window.ReactDOM` are already set by the time `support.js` runs its
`loadReactUmd()` check, the CDN fetch is skipped entirely and the page
renders with zero external script dependencies.

## Updating the version

If `support.js` is ever regenerated with a newer React/ReactDOM version
(check `REACT_URL` / `REACT_DOM_URL` near the top of the file), re-vendor
with:

```sh
npm pack react@<version> react-dom@<version>
tar xzf react-<version>.tgz && cp package/umd/react.production.min.js assets/
tar xzf react-dom-<version>.tgz && cp package/umd/react-dom.production.min.js assets/
```

Then verify the SRI hash still matches what `support.js` expects:

```sh
openssl dgst -sha384 -binary assets/react.production.min.js | openssl base64 -A
openssl dgst -sha384 -binary assets/react-dom.production.min.js | openssl base64 -A
```
