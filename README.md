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

Assumed that bundler is included within Ruby 2.7 or later.

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

## Misc

- You can see the available dip commands via `dip ls`.
- .vscode contains a minimum set of conf and extensions. You can discard.
- If you encounter any issues around caching, try checking around bootsnap and spring gem.
