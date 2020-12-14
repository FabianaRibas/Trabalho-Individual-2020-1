FROM ruby:2.5.7
ENV BUNDLER_VERSION=2.1.4
CMD echo "I'm running a ruby 2.5.7 image"

RUN apt-get update -qq && apt-get install -y nodejs postgresql-client
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev

WORKDIR /api

COPY ./api/Gemfile .
COPY ./api/Gemfile.lock .

RUN gem install bundler
RUN bundle install

COPY ./api/ /api

COPY ./api/entrypoint.sh /usr/bin/entrypoint.sh

RUN chmod +x /usr/bin/entrypoint.sh

# Start the main process.
ENTRYPOINT ["entrypoint.sh"]
