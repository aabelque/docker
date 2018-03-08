# **************************************************************************** #
#                                                                              #
#                                                         :::      ::::::::    #
#    Dockerfile                                         :+:      :+:    :+:    #
#                                                     +:+ +:+         +:+      #
#    By: aabelque <marvin@42.fr>                    +#+  +:+       +#+         #
#                                                 +#+#+#+#+#+   +#+            #
#    Created: 2018/03/02 18:22:15 by aabelque          #+#    #+#              #
#    Updated: 2018/03/02 18:44:26 by aabelque         ###   ########.fr        #
#                                                                              #
# **************************************************************************** #

FROM ubuntu

RUN apt-get update -y \
	&& apt-get upgrade -y \
	&& apt-get -y install vim cmake gcc git clang binutils

CMD /bin/bash
