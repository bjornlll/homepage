---
layout: post
title: serverless-seed v0.3.14
categories: news
author: frank
---

We just rolled out an update to the [serverless-seed](https://www.npmjs.com/package/serverless-seed) plugin to support manually specified artifacts.

The [serverless-seed](https://www.npmjs.com/package/serverless-seed) plugin optimizes Serverless Framework builds in Seed by [incrementally deploying changes]({% link _docs/incremental-lambda-deploys.md %}).

It does this by tracking the artifacts generated by Serverless Framework. This [`v0.3.14`](https://github.com/seed-run/serverless-seed/releases/tag/v0.3.14) update supports cases where you might be manually specifying the path to the artifact in your `serverless.yml`.

You can upgrade by running:

``` bash
$ npm install --save-dev --save-exact serverless-seed@0.3.14
``` 

Read more about [Incremental Lambda Deploys]({% link _docs/incremental-lambda-deploys.md %}) over on the docs.