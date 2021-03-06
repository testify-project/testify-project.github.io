# Testify Site
[![Build Status](https://travis-ci.org/testify-project/testify-project.github.io.svg?branch=master)](https://travis-ci.org/testify-project/testify-project.github.io)
[![Stories in Progress](https://badge.waffle.io/testify-project/testify-project.github.io.svg?label=In%20Progress&title=In%20Progress)](http://waffle.io/testify-project/testify-project.github.io)
[![Stories in Ready](https://badge.waffle.io/testify-project/testify-project.github.io.svg?label=ready&title=Ready)](http://waffle.io/testify-project/testify-project.github.io)
[![Join the chat at https://gitter.im/testify-project/testify](https://badges.gitter.im/testify-project/testify.svg)](https://gitter.im/testify-project/testify?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
[![CodecovIO](https://codecov.io/github/testify-project/testify-project.github.io/coverage.svg?branch=master)](https://codecov.io/github/testify-project/testify-project.github.io?branch=master)
[![License](https://img.shields.io/badge/license-Apache%20License%202-lightgrey.svg)](https://github.com/testify-project/testify-project.github.io/blob/master/LICENSE)

## Local Site Development
You can set up a local version of your Jekyll GitHub Pages site to test changes to your site locally.

### Install Prerequisites
1. [Install Ruby 2.1.0 or higher](https://www.ruby-lang.org/en/documentation/installation/)
1. [Install NodeJS 4.2.6 or higher](https://nodejs.org/en/download/package-manager/)
1. Install Bundler
```bash
gem install bundler
```
1. Install Bundles
```bash
sudo bundle install
```
1. Install Gulp
```bash
sudo npm install --global gulp-cli
```
1. Install Dependencies
```bash
npm install
```

### Generate and Run Site Locally
#### Clean, Build, and Run Site
```bash
gulp
```
#### Clean Site
```bash
gulp clean
```
#### Build Site
```bash
gulp  build
```
#### Run Site
```bash
gulp run
```

### Use GUI Admin Tool Site
1. Enable jekyll-admin gem in the Gemfile
```ruby
gem 'jekyll-admin', group: :jekyll_plugins
```
1. Run Admin
```bash
gulp admin
```
1. Go to [http://127.0.0.1:4098/admin](http://127.0.0.1:4098/admin)