Follow official docker rails guide
https://docs.docker.com/compose/rails/

And also check https://github.com/rails/webpacker/blob/master/docs/docker.md

## Step1
```bash
mkdir rails6-docker-starter
cd rails6-docker-starter
touch Dockerfile
```

From rails webpacker guide...
https://github.com/rails/webpacker/blob/master/docs/docker.md

`Dockerfile`
```
FROM ruby:2.6

RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
    && echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list

RUN apt-get update -qq && apt-get install -y nodejs postgresql-client yarn
RUN mkdir /myapp
WORKDIR /myapp
COPY Gemfile /myapp/Gemfile
COPY Gemfile.lock /myapp/Gemfile.lock
RUN bundle install
COPY . /myapp

# Add a script to be executed every time the container starts.
COPY entrypoint.sh /usr/bin/
RUN chmod +x /usr/bin/entrypoint.sh
ENTRYPOINT ["entrypoint.sh"]
EXPOSE 3000

# Start the main process.
CMD ["rails", "server", "-b", "0.0.0.0"]
```

## Step2
Make `Gemfile`

`Gemfile`
```
source 'https://rubygems.org'
gem 'rails', '~> 6.0.0'
```

## Step3
Make `Gemfile.locK`

## Step4
Make `entrypoint.sh`

```
#!/bin/bash
set -e

# Remove a potentially pre-existing server.pid for Rails.
rm -f /myapp/tmp/pids/server.pid

# Then exec the container's main process (what's set as CMD in the Dockerfile).
exec "$@"
```

## Step5
Official docs guide and Rails webpacker docker example.

https://github.com/rails/webpacker/blob/master/docs/docker.md

`docker-compose.yml`
```yaml
version: '3'
services:
  db:
    image: postgres
    volumes:
      - ./tmp/db:/var/lib/postgresql/data
  web:
    build: .
    command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"
    volumes:
      - .:/myapp
    ports:
      - "3000:3000"
    depends_on:
      - db
  webpacker:
    build: .
    env_file:
      - '.env.docker'
    command: ./bin/webpack-dev-server
    volumes:
      - .:/myapp
    ports:
      - '3035:3035'
```

`env.docker`
```
NODE_ENV=development
RAILS_ENV=development
WEBPACKER_DEV_SERVER_HOST=0.0.0.0
```


## Step6

`terminal`
```bash
docker-compose run web rails new . --force --no-deps --database=postgresql
```


## Step7

`terminal`
```bash
docker-compose build
```


## Step8
`config/database.yml`
```yml
default: &default
  adapter: postgresql
  encoding: unicode
  host: db
  username: postgres
  password:
  pool: 5

development:
  <<: *default
  database: myapp_development


test:
  <<: *default
  database: myapp_test
```

## Step9

`terminal`
```bash
docker-compose run web rake db:create
```

## Step 10
```bash
docker-compose up
```

And 

visit

http://localhost:3000/


## Refs
https://docs.docker.com/compose/rails/