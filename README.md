# Quick start for Rails 6 + Webpacker + SQLite3 on Docker

> The repository is based on the article [Ruby on Whales: Dockerizing Ruby and Rails development — Martian Chronicles, Evil Martians’ team blog](https://evilmartians.com/chronicles/ruby-on-whales-docker-for-ruby-rails-development). Note that the article will be continuously updated.

The docker-compose.yml is for building a minimum and configurable development environment for latest Rails 6 with the following, in a very short time.

- Webpacker
- SQLite3

You don't need to tweak the Dockerfile to update gems, packs, databases.

Note that current config is just for building a dev env, not for production container.

## Requirement

- Docker and Docker Compose
- [dip](https://github.com/bibendi/dip) for managing docker-compose.yml

## Step

### Preparation

- Clone this repository to your local environment.
- You can discard the .git `rm -rf .git` if needed.
- Adjust the values at the top of docker-compose.yml if needed:

```yaml
x-var: &APP_IMAGE_TAG
  "my_app:1.0.0"
x-var: &RUBY_VERSION
  "2.7.0-slim-buster"
x-var: &NODE_MAJOR
  12
x-var: &YARN_VERSION
  1.21.1
```

Assumed that bundler is included within Ruby 2.7 or later. If you want to specify other version of bundler, uncomment the following as well:

- `BUNDLER_VERSION` in docker-compose.yml
- `&& gem install bundler:$BUNDLER_VERSION` in .dockerdev/Dockerfile.

Now you can perform quick/custom installation:

### A. Quick Install

- `dip provision`

> Note: `--skip-listen` option is specified in the command to avoid the issue on macOS:
> ref: [Code is not reloaded in dev with Docker on OS X · Issue \#25186 · rails/rails](https://github.com/rails/rails/issues/25186). Perhaps you can remove the option for Linux environments.

### B. Custom Install

- `dip compose build` to build a container.
- `dip bundle install` to install gems for Rails.
- `dip bundle exec rails new . --webpacker <options as you like>`.
  - To macOS user: add `--skip-listen`.
- `dip yarn install` to install yarn.
- Perform the following manually to activate local access via Docker:

```sh
dip sh -c "sed -i -e \"3i\   config.hosts << 'localhost'\" config/environments/development.rb"
dip sh -c "sed -i -e \"4i\   config.web_console.whitelisted_ips = '0.0.0.0/0'\" config/environments/development.rb"
```

---

That's all. Now you can run `rails s` command via `dip rails s`. You don't need to add `bundle exec` any more.

## Recommended gems

The following Gemfile contains recommended gems. Just pick them up from under "Additional gems".

```ruby
# frozen_string_literal: true

source 'https://rubygems.org'
git_source(:github) { |repo| "https://github.com/#{repo}.git" }

ruby '2.7.0'

# Bundle edge Rails instead: gem 'rails', github: 'rails/rails'
gem 'rails', '~> 6.0.2', '>= 6.0.2.1'
# Use sqlite3 as the database for Active Record
gem 'sqlite3', '~> 1.4'
# Use Puma as the app server
gem 'puma', '~> 4.1'
# Use SCSS for stylesheets
gem 'sass-rails', '>= 6'
# Transpile app-like JavaScript. Read more: https://github.com/rails/webpacker
gem 'webpacker', '~> 4.0'
# Turbolinks makes navigating your web application faster. Read more: https://github.com/turbolinks/turbolinks
gem 'turbolinks', '~> 5'
# Build JSON APIs with ease. Read more: https://github.com/rails/jbuilder
gem 'jbuilder', '~> 2.7'
# Use Redis adapter to run Action Cable in production
# gem 'redis', '~> 4.0'
# Use Active Model has_secure_password
# gem 'bcrypt', '~> 3.1.7'

# Use Active Storage variant
# gem 'image_processing', '~> 1.2'

# Reduces boot times through caching; required in config/boot.rb
gem 'bootsnap', '>= 1.4.2', require: false

# Additional gems
gem "rack-attack"

group :development, :test do
  # Call 'byebug' anywhere in the code to stop execution and get a debugger console
  gem "byebug", platforms: [:mri, :mingw, :x64_mingw]

  # Additional gems
  gem "better_errors"
  gem "binding_of_caller"
  gem "brakeman"
  gem "bullet"
  gem "bundler-audit"
  gem "license_finder"
  gem "nullalign"
  gem "rack-mini-profiler"
  gem "rails_best_practices", require: false
  gem "rubocop"
  gem "stackprof"
end

group :development do
  # Access an interactive console on exception pages or by calling 'console' anywhere in the code.
  gem 'web-console', '>= 3.3.0'
  # Spring speeds up development by keeping your application running in the background. Read more: https://github.com/rails/spring
  gem 'spring'
end

group :test do
  # Adds support for Capybara system testing and selenium driver
  gem "capybara", ">= 2.15"
  gem "selenium-webdriver"
  # Easy installation and use of web drivers to run system tests with browsers
  gem "webdrivers"

  # Additional gems
  gem "minitest-reporters"
  gem "simplecov", require: false
end

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]

```

## Misc

- You can see the available dip commands via `dip ls`.
- .vscode contains a minimum set of conf and extensions. You can discard.
- If you encounter any issues around caching, try checking bootsnap and spring gem.

## Webpacker + Bootstrap + font-awesome

You can configure Bootstrap 4 and font-awesome on Webpacker by running the following script:

```sh
dip yarn add bootstrap jquery popper.js @fortawesome/fontawesome-free

mkdir app/javascript/src
mkdir app/javascript/images

echo "@import '~bootstrap/scss/bootstrap';" > app/javascript/src/application.sass
echo "@import '~@fortawesome/fontawesome-free/scss/fontawesome';" >> app/javascript/src/application.sass

dip sh -c 'sed -i -e "s/stylesheet_link_tag/stylesheet_pack_tag/g" app/views/layouts/application.html.erb'

dip sh -c "sed -i -e \"10i\import 'bootstrap';\" app/javascript/packs/application.js"
dip sh -c "sed -i -e \"11i\import '../src/application.sass';\" app/javascript/packs/application.js"
dip sh -c "sed -i -e \"12i\import '@fortawesome/fontawesome-free/js/all';\" app/javascript/packs/application.js"
```

Then you can remove app/assets and deactivate Sprockets if unnecessary.
