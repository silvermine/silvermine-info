# Node.js and NPM

## Version Management

We use a tool called [NVM][nvm] to manage Node versions on our development machines.

Each project has a `.nvmrc` file in its root directory that specifies the version of
Node that the project requires developers to use.

We try to use the same Node version across all of our projects when possible. The
Silvermine-standard Node version is defined in [the `.nvmrc` file in the root directory of
the `silvermine-info` repository][nvmrc].

Some projects, however, may not be updated as often as others. This means that developers
need to make sure that the version of Node they're using when working in any given project
is the same as the one the project expects.

Run:

```bash
nvm use
```

from the root directory of each project before starting work to ensure you are using the
correct Node version.

## Package Management

We use [NPM][npm] to manage packages in our projects.

We always specify exact package versions (with no fuzzy-match characters) in our
package.json files:

✅ Good:

```json
{
   "dependencies": {
      "package-name": "1.2.3",
      "another-name": "2.0.0"
   }
}
```

❌ Bad:

```json
{
   "dependencies": {
      "package-name": "^1.2.3",
      "another-name": "~2"
   }
}
```

When installing NPM packages, use the `-E` / `--save-exact` flag to ensure that the
exact version of the package is saved in the package.json file:

```bash
npm install -E package-name
```

We also try to use a consistent version of NPM across projects. This helps ensure that all
development CI/CD, and production environments are consistent in how they install
dependencies and run the code.

We prefer to use the default version of NPM that comes with each Node version that we use.
At times, we may upgrade NPM because of a defect or security vulnerability in the default
version of NPM.

The standard NPM version is enforced in each project via our standard
[check-node-version][check-node-version] script.

### Package Management for Tauri Plugins

Tauri plugins are an exception to the exact-version rule. The `@tauri-apps/api` package
**must** be declared as a `peerDependency` with a permissive caret range (e.g. `^2.9.0`).

#### Why a peer dependency instead of a production dependency?

When a plugin lists `@tauri-apps/api` as a production dependency, NPM may install a
separate, nested copy inside the plugin's `node_modules`. If that copy differs from the
version the host app uses, subtle runtime bugs can occur. Declaring it as a `peerDependency`
ensures the plugin shares the host app's copy.

Tauri's own first-party plugins use production dependencies, but the dependency is
automatically bumped as part of their release process. Unofficial third-party plugins like
ours do not benefit from this and cannot keep in lockstep with Tauri's releases, making a
peer dependency the correct choice.

#### Why a wide caret range?

Using a permissive caret range allows consuming applications to use any compatible version
of `@tauri-apps/api` without the plugin causing an install failure or forcing a version
upgrade. The minimum version should reflect the oldest version the plugin is known to work
with. If we discover a specific incompatibility, we can raise the minimum while still
accepting all newer minor and patch releases.

✅ Good:

```json
{
   "peerDependencies": {
      "@tauri-apps/api": "^2.9.0"
   }
}
```

❌ Bad — production dependency (causes nested duplicate):

```json
{
   "dependencies": {
      "@tauri-apps/api": "2.9.1"
   }
}
```

❌ Bad — exact peer dependency (breaks consuming projects on a different minor version):

```json
{
   "peerDependencies": {
      "@tauri-apps/api": "2.9.1"
   }
}
```

This exception applies **only** to `@tauri-apps/api` in `peerDependencies`. All other
dependency types (`dependencies`, `devDependencies`) in Tauri plugins should continue to use
exact versions as described above.

[nvm]: https://github.com/nvm-sh/nvm
[nvmrc]: https://github.com/silvermine/standardization/blob/master/.nvmrc
[npm]: https://www.npmjs.com/
[check-node-version]: https://github.com/silvermine/standardization/blob/55d8b668abe5c63d4ea9a8184424eb3d2b7b55db/README.md#check-node-version
