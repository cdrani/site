---
title: "Simplify Rails App Deployment Using Docker"
seoTitle: "Simplify Rails Deployment with Docker"
seoDescription: "Optimize Rails deployment using Docker to manage multiple Ruby versions, eliminate setup issues, and boost development efficiency"
datePublished: Mon May 15 2023 06:04:56 GMT+0000 (Coordinated Universal Time)
cuid: clhofxlx4000g09mf9uxv40k8
slug: simplify-rails-app-deployment-using-docker
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/jOqJbvo1P9g/upload/06f53435135a6917bd9a48ce8afd3270.jpeg
tags: docker, rails, containers, dockerfile, docker-rails

---

I can honestly say that I have given up on managing multiple Ruby and Rails versions on my computer. Regardless of whether it's homebrew, asdf, rbenv, or any other new package manager, I've had enough. This applies to open-source projects, take-home assessments for interviews, and even pair-programming sessions. All Rails apps/projects should be Dockerized, and if they aren't, rest assured that a pull request is on its way to set it up accordingly by yours truly. In fact, in the upcoming [Rails 7.1 release](https://github.com/rails/rails/commit/4f3af4a67f227ed7998fed570b9aa671e1b74117), by default a Dockerfile will be included.

## Fresh Setup

By "Fresh", I mean just having Docker Desktop and a Text Editor - No Rails necessary.

1. Confirm installation of [Docker](https://docs.docker.com/get-docker/) - (Community Edition will work fine) - with
    
    `docker -v`
    

```bash
docker -v
# Docker version 23.0.5, build bc4487a
```

1. Set up a docker container with a ruby image in which to create the rails app.
    

```bash
docker run --rm -it -v "$PWD":/usr/src/app -w /usr/src/app ruby:3.2.2-bullseye bash -c "gem install rails && rails new . -d postgresql -T"
```

This command runs a Docker container using the `ruby:3.2.2-bullseye` image, with several options:

* `docker run`: Run a Docker container.
    
* `--rm`: Automatically remove the container when it exits.
    
* `-it`: Allocate a tty for interactive input and output.
    
* `-v "$PWD":/usr/src/app`: Mount the current directory as a volume inside the container at the path `/usr/src/app`.
    
* `-w /usr/src/app`: Set the working directory inside the container to `/usr/src/app`.
    
* `ruby:3.2.2-bullseye`: Use the `ruby:3.2.2-bullseye` image as the base image for the container.
    
* `/bin/bash`: Start an interactive shell inside the container.
    
* `-c "gem install rails && rails new . -d postgresql -T"`:
    
    Execute a shell command inside the container that installs Rails using `gem install` and then runs the `rails new` command within the current directory with the specified options - setting the database to PostgreSQL and optionally skipping tests setup.
    

1. We should now have a rails app generated on our machine. Now we want it dockerized, so we can run all the processes like the server, console, migrations, etc within containers. Let's start with the server and for that create a `Dockerfile` (also written as `dockerfile` - which is my preference).
    

```yaml
FROM ruby:3.2.2-bullseye

# Set the working directory inside the container
WORKDIR /app

# Update Dependencies
RUN apt-get update && \
    apt-get clean 

# Copy the Gemfile and Gemfile.lock from the host into the container
COPY Gemfile Gemfile.lock ./

# Install the RubyGems
RUN gem install bundler:2.4.13 && \
    bundle config --global frozen 1 && \
    bundle install --jobs 4 --retry 3

# Copy the rest of the application into the container
COPY . .

# Expose port 3000
EXPOSE 3000

# Start the Rails server
CMD ["rails", "server", "-b", "0.0.0.0"]
```

1. Now we need a container with a PostgreSQL image and connect it with the rails app on the same network. The easy approach is with utilizing a `docker-compose.yaml` (or `compose.yaml` - again preference) file.
    

```yaml
version: '3.9'
services:
  db:
    container_name: db
    image: postgres:14-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: attic_development
    volumes:
      - db_date:/var/lib/postgresql/data

  web:
    container_name: web
    build: .
    ports:
      - "3000:3000"
    depends_on:
      - db
    environment:
      DATABASE_URL: postgresql://postgres:postgres@db/attic_development
    volumes:
      - .:/app
      - gem_cache:/usr/local/bundle/gems

volumes:
  gem_cache:
  db_data:
```

The `compose.yaml` file is used to define a multi-container Docker application. It consists of a version number (in this case, version 3), and a list of services that make up the application.

The `services` section of the file contains two services:

* `db`: This service is defined using the official PostgreSQL Docker image tagged `14-alpine`. It sets three environment variables:
    
    * `POSTGRES_USER`: The username for the default PostgreSQL user.
        
    * `POSTGRES_PASSWORD`: The password for the default PostgreSQL user.
        
    * `POSTGRES_DB`: The name of the default database to be created.
        
* `web`: This service is defined to **build** a Docker image from the `dockerfile` in the current directory (`.`). It maps **port** `3000` of the host to port `3000` of the container. It depends on the `db` service, which means that the `db` service will be started before the `web` service. It also sets the `DATABASE_URL` environment variable to connect the Rails app to the PostgreSQL database. The `depends_on` section specifies that the `db` service should be started before the `web` service. The `environment` section sets the `DATABASE_URL` environment variable to the URL of the PostgreSQL database. The format of the URL is `postgresql://<username>:<password>@<hostname>/<database>`. In this case, the username is `postgres`, the password is `postgres`, the hostname is `db` (the name of the `db` service), and the database name is `attic_development`.
    

1. Now we can run both services together:
    

```yaml
docker compose up -d
```

The `-d` is for a detached mode, i.e. running in the background. In most cases this is the ideal mode as without the flag our terminal will stream the rails server logs, requiring us to either open a new terminal tab to run other commands for migrations, interact with the console, etc.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683790081061/f4ad0553-6a71-4a59-866e-9ddb95d7e701.png align="center")

### Common Commands:

Running various Rails commands within the web container:

```bash
docker compose run web rails c
docker compose run web rails g model|controller|migration ...
docker compose run web rails db:migrate
docker compose run web rails t
docker compose run web bundle exec rspec
docker compose run web bundle add|install|update|remove [gem]
```

The above `docker compose run web rails` should be saved into an alias such as:

```bash
alias dcr="docker compose run web rails"
alias dcb="docker compuse bundle"
alias dcbr="docker compose bundle exec rspec"
```

View `rails s` logs:

```bash
docker logs -f web
```

## Best Practices

The above is a minimal dockerized rails app sufficient for development. However, there are additional changes we would need to make to adhere to some best practices and make it ready for deployment in a production environment. For instance, we could use a slimmer image of Ruby, create and use a non-root user in our container, set up multi-stage builds, etc. There's a lot to glean from the examples in these posts for better [Dockerfile](https://lipanski.com/posts/dockerfile-ruby-best-practices) and [Docker Compose](https://nickjanetakis.com/blog/best-practices-around-production-ready-web-apps-with-docker-compose) files. Additionally, Docker has an [official post](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/) on this matter.

## Setup for Optimal Development/Production

This is most likely the most difficult part as there will likely be variances in how lean and performant one's production docker setup is in comparison to others found online. Furthermore, this comes with experience and testing. As in most cases where I am my own DevOps team, it's better to lean on the expertise of others who readily provide it. There are numerous rails-centric templates with fully-configured docker setup with GitHub actions for automated deployments on GitHub.

I came across [Ruby on Whales](https://evilmartians.com/chronicles/ruby-on-whales-docker-for-ruby-rails-development) post and it was truly a godsend. They clearly outline and explain their decisions for their particular setup and keep the post updated. Furthermore, they provide a template so anyone can duplicate their configurations. The script run in their generator can be found here. The setup is quite straightforward by running the below inside our rails app directory:

```bash
gem install rbytes
gem install dip
rbytes install https://railsbytes.com/script/z5OsoB
dip provision
```

The [rbytes](https://github.com/palkan/rbytes) command will install an interactive script to set up the docker environment in a **.dockerdev** folder and include a `dip.yml` file. The script itself can be found [here](https://railsbytes.com/public/templates/z5OsoB) for reference and forked for modification. For a full scope of the inner workings of the generator reference the [Ruby on Whales Github repo](https://github.com/evilmartians/ruby-on-whales).

[Dip](https://github.com/bibendi/dip) Dip (Docker Interaction Program) is a tool to simplify the previously complicated method of utilizing Docker Compose, making the process smoother. Review the `dip.yml` to see the commands we can run:

```bash
version: '7.1'

# Define default environment variables to pass
# to Docker Compose
environment:
  RAILS_ENV: development

compose:
  files:
    - .dockerdev/compose.yml
  project_name: attic

interaction:
  # This command spins up a Rails container with the required dependencies (such as databases),
  # and opens a terminal within it.
  runner:
    description: Open a Bash shell within a Rails container (with dependencies up)
    service: rails
    command: /bin/bash

  # Run a Rails container without any dependent services (useful for non-Rails scripts)
  bash:
    description: Run an arbitrary script within a container (or open a shell without deps)
    service: rails
    command: /bin/bash
    compose_run_options: [ no-deps ]

  # A shortcut to run Bundler commands
  bundle:
    description: Run Bundler commands
    service: rails
    command: bundle
    compose_run_options: [ no-deps ]

  rails:
    description: Run Rails commands
    service: rails
    command: bundle exec rails
    subcommands:
      s:
        description: Run Rails server at http://localhost:3000
        service: web
        compose:
          run_options: [ service-ports, use-aliases ]
      test:
        description: Run unit tests
        service: rails
        command: bundle exec rails test
        environment:
          RAILS_ENV: test

  yarn:
    description: Run Yarn commands
    service: rails
    command: yarn
    compose_run_options: [ no-deps ]

  psql:
    description: Run Postgres psql console
    service: postgres
    default_args: attic_development
    command: psql -h postgres -U postgres

provision:
  # We need the `|| true` part because some docker-compose versions
  # cannot down a non-existent container without an error,
  # see https://github.com/docker/compose/issues/9426
  - dip compose down --volumes || true
  - dip compose up -d postgres
  - dip bash -c bin/setup
```

The first command to run would be `dip provision` which as shown above is for a fresh setup - shutting down volumes, starting postgres service, and running the `bin/setup` script which will install/update gems, prepare the database, and restart the rails:

```bash
#!/usr/bin/env ruby
require "fileutils"

# path to our application root.
APP_ROOT = File.expand_path("..", __dir__)

def system!(*args)
  system(*args) || abort("\n== Command #{args} failed ==")
end

FileUtils.chdir APP_ROOT do
  # This script is a way to set up or update our development environment automatically.
  # This script is idempotent, so that we can run it at any time and get an expectable outcome.
  # Add necessary setup steps to this file.

  puts "== Installing dependencies =="
  system! "gem install bundler --conservative"
  system("bundle check") || system!("bundle install")

  # puts "\n== Copying sample files =="
  # unless File.exist?("config/database.yml")
  #   FileUtils.cp "config/database.yml.sample", "config/database.yml"
  # end

  puts "\n== Preparing database =="
  system! "bin/rails db:prepare"

  puts "\n== Removing old logs and tempfiles =="
  system! "bin/rails log:clear tmp:clear"

  puts "\n== Restarting application server =="
  system! "bin/rails restart"
end
```

We can run rails-specific commands defined in the `dip.yml` just like we did previously but prefixed with `dip`. Included are commands to target other services as well:

```bash
# snippet from dip.yml file
bundle:
  description: Run Bundler commands
  service: rails
  command: bundle
  compose_run_options: [ no-deps ]

rails:
  description: Run Rails commands
  service: rails
  command: bundle exec rails
  subcommands:
    s:
      description: Run Rails server at http://localhost:3000
      service: web
      compose:
        run_options: [ service-ports, use-aliases ]
    test:
      description: Run unit tests
      service: rails
      command: bundle exec rails test
      environment:
        RAILS_ENV: test
```

From the snippet above we can run the same rails-centric commands prefixed with `dip`, however, there is an option in the `.dockerdev/README.md` (plus available commands without referencing the `dip.yml` file) that makes the prefix unnecessary.

## Final Thoughts

I have found the interactive CLI option to be my go-to approach as it adheres to best practices and doubly gives the options to set the Rails and PostgreSQL versions (among other environment variables) which I usually hardcode. Currently working on various self and OSS projects with differing rails versions and I am at peace with integrating them with docker and contributing without the hassle of wrangling with the setup of multiple rails versions and gems issues based on my OS.