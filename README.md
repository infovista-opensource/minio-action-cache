# minio-action-cache

This action allows to cache your dependencies in Minio (or other S3-compatible service).

Self-hosted runners supported and encouraged.

## Usage

1. Deploy a https://min.io server

2. Create `.github/workflows/my-cachable-workflow.yml` file

```yaml
name: yarn cache

on:
  push:
    branches: [master]
  pull_request:
    branches: [master]
  workflow_dispatch:

jobs:
  dependencies:
    runs-on: [self-hosted]
    steps:
      - uses: actions/checkout@v3
      - name: Restore yarn cache
        id: cache
        uses: infovista-opensource/minio-action-cache@main
        with:
          endpoint: 'onpremcache.domain.com' # optional, default s3.amazonaws.com
          port: 443 # minio port
          insecure: false # optional, use http instead of https. default false
          accessKey: "minioadmin" # required
          secretKey: "minioadmin" # required
          bucket: "bucket-name" # required
          key: ${{ runner.os }}-yarn-${{ hashFiles('**/yarn.lock') }}
          path: |
            node_modules
            ~/node_modules

      - name: Yarn install
        if: steps.cache.outputs.cache-hit != 'true'
        run: yarn install

      - name: Check that dependency installed
        run: yarn ts-jest -v
```

## TODO

- [ ] fallback key management
- [ ] check cache content (all restored files hash) to see if cache has to be reuploaded

## Changes from upstream

Add standalone restore/save sub-actions in order to avoid the [random post-steps order issue](https://github.com/actions/runner/issues/1657).
