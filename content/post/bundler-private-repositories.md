---
title: "Bundler: Private Repositories"
subtitle: "Installing a gem from a private GitHub repository on Heroku"
date: 2016-02-29T14:56:33+02:00
tags: ["github", "heroku", "ruby"]
summary: "This is a little guide on how to install private gems hosted in GitHub on the Heroku platform."
---

This is a little guide on how to install private gems hosted in GitHub on the Heroku platform.

### 0. Getting started

To follow this guide, you need Bundler 1.11+.

{{< highlight none >}}
$ bundle --version
Bundler version 1.11.2
{{</ highlight >}}

### 1. Get your OAuth Token from GitHub

To authenticate Bundler to GitHub, you will need an OAuth token. You can follow this [guide](https://help.github.com/articles/creating-an-access-token-for-command-line-use/) to create a new one.

### 2. Setting up your credentials

To set up your credentials, use `bundle-config` like:

{{< highlight none >}}
$ bundle config GITHUB__COM myoauthtoken:x-oauth-basic
{{</ highlight >}}

If you want to apply this configuration only to your current project, do this instead:

{{< highlight none >}}
$ bundle config --local GITHUB__COM myoauthtoken:x-oauth-basic
{{</ highlight >}}

### 3. Install your dependencies

Add the private gem to your `Gemfile`:

{{< highlight ruby >}}
gem "mygem", git: "https://github.com/user/mygem.git"
{{</ highlight >}}

Install the gem dependencies using `bundle install`:

{{< highlight none >}}
$ bundle install
Fetching https://github.com/user/mygem.git
Fetching gem metadata from https://rubygems.org/.........
Fetching version metadata from https://rubygems.org/..
Using mygem 1.0.0 from https://github.com/user/mygem.git (at master@bc4adfb)
Using bundler 1.11.2
Bundle complete! 1 Gemfile dependency, 2 gems now installed.
{{</ highlight >}}

Notice that your credentials aren’t stored in the Gemfile or the generated Gemfile.lock.

### 4. Setting up Heroku

> UPDATE: Heroku now uses Bundler 1.11+, you can skip this step.

At the moment of writting, Heroku’s default Ruby buildpack uses an older version of Bundler (1.9.7), but you can use the latest version of the buildpack with:

{{< highlight none >}}
$ heroku buildpacks:remove heroku/ruby
$ heroku buildpacks:add https://github.com/heroku/heroku-buildpack-ruby
{{</ highlight >}}

Then, add your GitHub credentials to Heroku:

{{< highlight none >}}
$ heroku config:add BUNDLE_GITHUB__COM=myoauthtoken:x-oauth-basic
{{</ highlight >}}

Finally, push your changes to create a new release.

{{< highlight none >}}
$ git push heroku master
# ...
remote: Running: bundle install --without development:test --path vendor/bundle --binstubs vendor/bundle/bin -j4 --deployment
remote: Using bundler 1.11.2
remote: Using mygem 1.0.0 from https://github.com/user/mygem.git (at master@bc4adfb)
remote: Bundle complete! 1 Gemfile dependency, 2 gems now installed.
{{</ highlight >}}
