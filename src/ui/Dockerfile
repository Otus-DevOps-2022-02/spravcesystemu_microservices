FROM ubuntu:16.04

RUN apt-get update \
    && apt-get install --no-install-recommends -y ruby-full ruby-dev build-essential \
    && gem install bundler --no-ri --no-rdoc \
    && apt-get -y autoclean

WORKDIR /app

COPY Gemfile* /app/

RUN bundle install

COPY . /app

ENV POST_SERVICE_HOST post
ENV POST_SERVICE_PORT 5000
ENV COMMENT_SERVICE_HOST comment
ENV COMMENT_SERVICE_PORT 9292

CMD ["puma"]
