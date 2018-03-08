# **************************************************************************** #
#                                                                              #
#                                                         :::      ::::::::    #
#    Dockerfile                                         :+:      :+:    :+:    #
#                                                     +:+ +:+         +:+      #
#    By: aabelque <marvin@42.fr>                    +#+  +:+       +#+         #
#                                                 +#+#+#+#+#+   +#+            #
#    Created: 2018/02/23 12:27:10 by aabelque          #+#    #+#              #
#    Updated: 2018/03/02 15:52:53 by aabelque         ###   ########.fr        #
#                                                                              #
# **************************************************************************** #

FROM ruby

RUN apt-get update -y && apt-get upgrade -y
RUN apt-get install nodejs -y
RUN gem install rails

ONBUILD COPY . /opt/app/

ONBUILD RUN bundle update && bundle install
ONBUILD RUN rake db:migrate
ONBUILD RUN rake db:setup
ONBUILD RUN rake db:seed

WORKDIR /opt/app/
