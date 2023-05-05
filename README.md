# Action Node.js with `node_modules` cache

Composite GitHub Action for the setup of Node.js via [actions/setup-node](https://github.com/actions/setup-node) and managed `node_modules` caching via [actions/cache](https://github.com/actions/cache). This offers an alternative to the typical cache target of the `npm` module cache under `~/.npm`.

Why may I want this?

- Defeat time consuming `npm install|ci` operations where modules are performing `postinstall` tasks.
- Cache paths _in addition_ to `node_modules` for modules which perform tooling installs as part of their `npm install` - for example the [cypress](https://www.npmjs.com/package/cypress) module, which downloads and installs the [`cypress` runner binary](https://docs.cypress.io/guides/references/advanced-installation#Binary-cache).

This method _may not work_ or be advantageous for every workflow - often `npm ci` will perform important install operations that can't be successfully cached - but in the right situations may offer a good speed advantage.

The Action ensures cache artifact keys include **both** Node.js and `npm` versions - ensuring the `node_modules` cached is closely tied to the versions which generated them.

- [Usage](#usage)
- [Setting unique cache keys](#setting-unique-cache-keys)
- [Action outputs](#action-outputs)

## Usage

Cache `node_modules` at the root of a repository, using `/package-lock.json` as the hash:

```yaml
steps:
  - name: Setup Node.js with node_modules cache
    id: node-with-cache
    uses: magnetikonline/action-node-modules-cache@v1
    with:
      node-version: 18.x
  - name: Install npm packages
    if: steps.node-with-cache.outputs.cache-hit != 'true'
    run: npm ci
```

Determine requested Node.js version from file (e.g. `.nvmrc`) using `node-version-file` input:

```yaml
steps:
  - name: Setup Node.js with node_modules cache
    id: node-with-cache
    uses: magnetikonline/action-node-modules-cache@v1
    with:
      node-version-file: .nvmrc
  - name: Install npm packages
    if: steps.node-with-cache.outputs.cache-hit != 'true'
    run: npm ci
```

Set alternative paths to cache (`node-modules-path`) and compute hash key from file (`package-lock-path`):

```yaml
steps:
  - name: Setup Node.js with node_modules cache
    id: node-with-cache
    uses: magnetikonline/action-node-modules-cache@v1
    with:
      node-version-file: .nvmrc
      node-modules-path: path/to/node_modules
      package-lock-path: path/to/package-lock.json
  - name: Install npm packages
    if: steps.node-with-cache.outputs.cache-hit != 'true'
    run: npm ci
```

Cache additional paths(s) alongside `node_modules`) - handy for modules such as [cypress](https://www.npmjs.com/package/cypress) and the installed [`cypress` runner binary](https://docs.cypress.io/guides/references/advanced-installation#Binary-cache):

```yaml
steps:
  - name: Setup Node.js with node_modules cache
    id: node-with-cache
    uses: magnetikonline/action-node-modules-cache@v1
    with:
      node-version-file: .nvmrc
      additional-cache-path: |
        ~/.cache/Cypress
  - name: Install npm packages
    if: steps.node-with-cache.outputs.cache-hit != 'true'
    run: npm ci
```

## Setting unique cache keys

An optional `cache-key-suffix` input allows control of the generated cache key. This might be handy where multiple `npm install|ci` operations/jobs are performed across workflow definitions - resulting in the need for unique `node_modules` cache artifacts:

```yaml
steps:
  - name: Cache key suffix
    uses: magnetikonline/action-node-modules-cache@v1
    with:
      node-version-file: .nvmrc
      cache-key-suffix: apples

# cache key:   ${{ runner.os }}-nodemodules-apples-${{ hashFiles('package-lock.json') }}
# restore key: ${{ runner.os }}-nodemodules-apples-
```

## Action outputs

Outputs of `cache-hit` and `cache-key` are provided - the latter can be used to always save cache, regardless of the execution result of the rest of the workflow:

```yaml
steps:
  - name: Setup Node.js with node_modules cache
    id: node-with-cache
    uses: magnetikonline/action-node-modules-cache@v1
    with:
      node-version-file: .nvmrc

  # further steps...

  - name: Save node_modules cache
    if: always()
    uses: actions/cache/save@v3
    with:
      path: |
        node_modules
      key: ${{ steps.node-with-cache.outputs.cache-key }}
```
