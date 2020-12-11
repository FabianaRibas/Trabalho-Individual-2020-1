FROM ruby:2.5.7
ENV BUNDLER_VERSION=2.1.4
CMD echo "I'm running a ruby 2.5.7 image"
RUN apt-get update -qq && apt-get install -y nodejs postgresql-client
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev
COPY . .
WORKDIR /api
RUN gem install bundler
RUN bundle install

# Start the main process.
CMD ["rails", "server"]
CMD ["bundle", "exec", "rake", "db:drop"]
CMD ["bundle", "exec", "rake", "db:create"]
CMD ["bundle", "exec", "rake", "db:migrate"]
