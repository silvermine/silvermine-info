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


[nvm]: https://github.com/nvm-sh/nvm
[nvmrc]: https://github.com/silvermine/standardization/blob/master/.nvmrc
[npm]: https://www.npmjs.com/
[check-node-version]: https://github.com/silvermine/standardization/blob/55d8b668abe5c63d4ea9a8184424eb3d2b7b55db/README.md#check-node-version
