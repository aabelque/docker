# **************************************************************************** #
#                                                                              #
#                                                         :::      ::::::::    #
#    Dockerfile                                         :+:      :+:    :+:    #
#                                                     +:+ +:+         +:+      #
#    By: aabelque <marvin@42.fr>                    +#+  +:+       +#+         #
#                                                 +#+#+#+#+#+   +#+            #
#    Created: 2018/03/03 13:25:43 by aabelque          #+#    #+#              #
#    Updated: 2018/03/05 10:13:03 by aabelque         ###   ########.fr        #
#                                                                              #
# **************************************************************************** #

FROM debian

RUN apt-get update -y \
	&& apt-get upgrade -y \
	&& apt-get install -y sudo \
	build-essential zlib1g-dev libyaml-dev libssl-dev libgdbm-dev libre2-dev \
	libreadline-dev libncurses5-dev libffi-dev curl openssh-server checkinstall \
	libxml2-dev libxslt-dev libcurl4-openssl-dev libicu-dev logrotate rsync \
	python-docutils pkg-config cmake git-core

RUN bash -c "debconf-set-selections <<< \"postfix postfix/mailname string localhost\"" \
	&& bash -c "debconf-set-selections <<< \"postfix postfix/main_mailer_type string 'Internet Site'\"" \
	&& apt-get install -y postfix

RUN apt-get remove -y ruby1.8 \
	&& mkdir /tmp/ruby && cd /tmp/ruby \
	&& curl --remote-name --progress https://cache.ruby-lang.org/pub/ruby/2.4/ruby-2.4.3.tar.gz \
	&& tar xvfz ruby-2.4.3.tar.gz \
	&& cd ruby-2.4.3 \
	&& ./configure --disable-install-rdoc \
	&& make \
	&& make install \
	&& gem install bundler --no-ri --no-rdoc

RUN rm -rf /usr/local/go \
	&& cd /tmp && curl --remote-name --progress https://storage.googleapis.com/golang/go1.8.3.linux-amd64.tar.gz \
	&& echo '1862f4c3d3907e59b04a757cfda0ea7aa9ef39274af99a784f5be843c80c6772  go1.8.3.linux-amd64.tar.gz' | shasum -a256 -c - \
	&& tar -C /usr/local -xzf go1.8.3.linux-amd64.tar.gz \
	&& ln -sf /usr/local/go/bin/go /usr/local/bin/ \
	&& ln -sf /usr/local/go/bin/godoc /usr/local/bin/ \
	&& ln -sf /usr/local/go/bin/gofmt /usr/local/bin/ \
  	&& rm go1.8.3.linux-amd64.tar.gz

RUN curl --location https://deb.nodesource.com/setup_8.x | sudo bash - \
	&& sudo apt-get install -y nodejs \
	&& curl --silent --show-error https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add - \
	&& echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list \
	&& sudo apt-get update -y \
	&& sudo apt-get install -y yarn

RUN adduser --disabled-login --gecos 'GitLab' git

RUN apt-get install -y postgresql postgresql-client libpq-dev postgresql-contrib \
	&& service postgresql start \
	&& sudo -u postgres psql -d template1 -c "CREATE USER git CREATEDB;" \
	&& sudo -u postgres psql -d template1 -c "CREATE EXTENSION IF NOT EXISTS pg_trgm;" \
		&& sudo -u postgres psql -d template1 -c "ALTER USER \"git\" WITH PASSWORD 'password';" \
	&& sudo -u postgres psql -d template1 -c "CREATE DATABASE gitlabhq_production OWNER git;"

RUN sudo apt-get install -y redis-server \
	&& sudo cp /etc/redis/redis.conf /etc/redis/redis.conf.orig \
	&& sed 's/^port .*/port 0/' /etc/redis/redis.conf.orig | sudo tee /etc/redis/redis.conf \
	&& echo 'unixsocket /var/run/redis/redis.sock' | sudo tee -a /etc/redis/redis.conf \
	&& echo 'unixsocketperm 770' | sudo tee -a /etc/redis/redis.conf \
	&& mkdir /var/run/redis \
	&& chown redis:redis /var/run/redis \
	&& chmod 755 /var/run/redis \
	&& if [ -d /etc/tmpfiles.d ]; then echo 'd  /var/run/redis  0755  redis  redis  10d  -' | sudo tee -a /etc/tmpfiles.d/redis.conf; fi \
	&& sudo usermod -aG redis git \
	&& sudo service redis-server start

WORKDIR /home/git

RUN sudo -u git -H git clone https://gitlab.com/gitlab-org/gitlab-ce.git -b 10-5-stable gitlab

WORKDIR /home/git/gitlab

#	&& sed -i '/    bin_path: \/usr\/bin\/git/c\    bin_path: \/usr\/local\/bin\/git/' config/gitlab.yml \

RUN sudo -u git -H cp config/gitlab.yml.example config/gitlab.yml \
	&& sed -i '/    email_reply_to: noreply@example.com/c\    email_reply_to: noreply@localhost/' config/gitlab.yml \
	&& sudo -u git -H cp config/secrets.yml.example config/secrets.yml \
	&& sudo -u git -H chmod 0600 config/secrets.yml \
	&& sudo chown -R git log/ \
	&& sudo chown -R git tmp/ \
	&& sudo chmod -R u+rwX,go-w log/ \
	&& sudo chmod -R u+rwX tmp/ \
	&& sudo chmod -R u+rwX tmp/pids/ \
	&& sudo chmod -R u+rwX tmp/sockets/ \
	&& sudo -u git -H mkdir public/uploads/ \
	&& sudo chmod 0700 public/uploads \
	&& sudo chmod -R u+rwX builds/ \
	&& sudo chmod -R u+rwX shared/artifacts/ \
	&& sudo chmod -R ug+rwX shared/pages/ \
	&& sudo -u git -H cp config/unicorn.rb.example config/unicorn.rb \
	&& sudo -u git -H cp config/initializers/rack_attack.rb.example config/initializers/rack_attack.rb \
	&& sudo -u git -H git config --global core.autocrlf input \
	&& sudo -u git -H git config --global gc.auto 0 \
	&& sudo -u git -H git config --global repack.writeBitmaps true \
	&& sudo -u git -H git config --global receive.advertisePushOptions true \
	&& sudo -u git -H cp config/resque.yml.example config/resque.yml \
	&& sudo -u git cp config/database.yml.postgresql config/database.yml \
	&& sudo -u git -H chmod o-rwx config/database.yml

#&& sed -i '/  url: unix:\/var\/run\/redis\/redis.sock/c\  url: redis:\/\/redis:6379/' config/resque.yml \

RUN sudo -u git cp config/database.yml.postgresql config/database.yml \
	&& sudo -u git -H chmod o-rwx config/database.yml \
	&& sed -i '/  password: "secure password"/c\  password: "password"' config/database.yml \
	&& cat -e config/database.yml

RUN sudo -u git -H bundle install --deployment --without development test mysql aws kerberos

RUN sudo -u git -H bundle exec rake gitlab:shell:install REDIS_URL=unix:/var/run/redis/redis.sock RAILS_ENV=production SKIP_STORAGE_VALIDATION=true

RUN sudo -u git -H bundle exec rake "gitlab:workhorse:install[/home/git/gitlab-workhorse]" RAILS_ENV=production

RUN service postgresql restart && service redis-server restart && yes yes | sudo -u git -H bundle exec rake gitlab:setup RAILS_ENV=production

RUN cp lib/support/init.d/gitlab /etc/init.d/gitlab \
	&& update-rc.d gitlab defaults 21

RUN sudo -u git -H bundle exec rake "gitlab:gitaly:install[/home/git/gitaly]" RAILS_ENV=production

RUN chmod 0700 /home/git/gitlab/tmp/sockets/private \
	&& chown git /home/git/gitlab/tmp/sockets/private

COPY docker-entrypoint.sh /docker-entrypoint.sh
RUN chmod +x /docker-entrypoint.sh

RUN cp lib/support/logrotate/gitlab /etc/logrotate.d/gitlab

RUN service postgresql restart && sudo -u git -H bundle exec rake gitlab:env:info RAILS_ENV=production

RUN sudo -u git -H bundle exec rake gettext:compile RAILS_ENV=production

RUN sudo -u git -H yarn install --production --pure-lockfile \
	&& sudo -u git -H bundle exec rake gitlab:assets:compile RAILS_ENV=production NODE_ENV=production

RUN sudo service gitlab start

COPY gitlab /etc/nginx/sites-available/gitlab

#&& cp lib/support/nginx/gitlab /etc/nginx/sites-available/gitlab \

RUN apt-get install -y nginx \	
	&& ln -s /etc/nginx/sites-available/gitlab /etc/nginx/sites-enabled/gitlab \
	&& rm -f /etc/nginx/sites-enabled/default 

EXPOSE 80 22

VOLUME /var/lib/postgresql/9.5/main
VOLUME /home/git/repositories

ENTRYPOINT /docker-entrypoint.sh
