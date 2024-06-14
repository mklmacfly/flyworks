# flyworks
Multiple Operating Systems and Architectures
jobs:
  build:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os:
          - ubuntu-latest
          - macos-latest
          - windows-latest
        node_version:
          - 12
          - 14
          - 16
        architecture:
          - x64
        # an extra windows-x86 run:
        include:
          - os: windows-2016
            node_version: 12
            architecture: x86
    name: Node ${{ matrix.node_version }} - ${{ matrix.architecture }} on ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - name: Setup node
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node_version }}
          architecture: ${{ matrix.architecture }}
      - run: npm ci
      - run: npm test
Publish to npmjs and GPR with npm
steps:
- uses: actions/checkout@v4
- uses: actions/setup-node@v4
  with:
    node-version: '14.x'
    registry-url: 'https://registry.npmjs.org'
- run: npm ci
- run: npm publish
  env:
    NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
- uses: actions/setup-node@v4
  with:
    registry-url: 'https://npm.pkg.github.com'
- run: npm publish
  env:
    NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
Publish to npmjs and GPR with yarn
steps:
- uses: actions/checkout@v4
- uses: actions/setup-node@v4
  with:
    node-version: '14.x'
    registry-url: <registry url>
- run: yarn install --frozen-lockfile
- run: yarn publish
  env:
    NODE_AUTH_TOKEN: ${{ secrets.YARN_TOKEN }}
- uses: actions/setup-node@v4
  with:
    registry-url: 'https://npm.pkg.github.com'
- run: yarn publish
  env:
    NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
Use private packages
steps:
- uses: actions/checkout@v4
- uses: actions/setup-node@v4
  with:
    node-version: '14.x'
    registry-url: 'https://registry.npmjs.org'
# Skip post-install scripts here, as a malicious
# script could steal NODE_AUTH_TOKEN.
- run: npm ci --ignore-scripts
  env:
    NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
# `npm rebuild` will run all those post-install scripts for us.
- run: npm rebuild && npm run prepare --if-present
Yarn2 configuration
Yarn2 ignores both .npmrc and .yarnrc files created by the action, so before installing dependencies from the private repo it is necessary either to create or to modify existing yarnrc.yml file with yarn config set commands.

Below you can find a sample "Setup .yarnrc.yml" step, that is going to allow you to configure a private GitHub registry for 'my-org' organisation.

steps:
- uses: actions/checkout@v4
- uses: actions/setup-node@v4
  with:
    node-version: '14.x'
- name: Setup .yarnrc.yml
  run: |
    yarn config set npmScopes.my-org.npmRegistryServer "https://npm.pkg.github.com"
    yarn config set npmScopes.my-org.npmAlwaysAuth true
    yarn config set npmScopes.my-org.npmAuthToken $NPM_AUTH_TOKEN
  env:
    NPM_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
- name: Install dependencies
  run: yarn install --immutable
  )
