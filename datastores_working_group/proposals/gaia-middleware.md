Creating a community of open source projects built on top of Gaia necessitates an architecture that factors in addon composability and interoperability.

Imagine this scenario:
I'm running an app using Gaia, and I've decided to use two opensource libraries built on top of Gaia:
  1. GaiaBackups: In essence, a module that the developer configures, allowing them to automatically create a backup data file anytime a user updates the file.
  2. NiceGaia: A client-side MongoDB implementation which supports basic queries, a la `NiceGaia.addCollection("foo");`

Within the restraints of the current spec, both "GaiaBackups" and "NiceGaia" might consider writing wrapper functions around `putFile`, so that instead of calling `blockstack.js`'s version, you call `NiceGaia.addCollection("foo")`. That's actually perfectly acceptable for NiceGaia because it's what their library probably requires, but what if `GaiaBackups` has the same type of API, i.e., one that functions as a wrapper for the core API? It might be possible, but it would be rather ugly.

Instead, consider this architecture inspired by Redux and Express ([more info here](https://github.com/reactjs/redux/blob/master/docs/advanced/Middleware.md)):
```
// somewhere in app bootstrapping
const gaiaBackups = initGaiaBackups({ backupLast: 10 })
blockstack.createGaia(
  blockstack.applyMiddleware([gaiaBackups, etc...])
)

// somewhere in app
NicaGaia.addCollection("foo") // calls blockstack.putFile with new data
// But wait! gaiaBackups runs before putFile to run the additional operations without any developer action required.
```
