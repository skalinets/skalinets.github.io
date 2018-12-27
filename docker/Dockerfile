FROM jekyll/jekyll:3.8
COPY *.crt /usr/local/share/ca-certificates/
RUN update-ca-certificates
WORKDIR /srv/jekyll
COPY Gemfile* ./
RUN bundle install
EXPOSE 4000 80
