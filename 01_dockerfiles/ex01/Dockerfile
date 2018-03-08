# **************************************************************************** #
#                                                                              #
#                                                         :::      ::::::::    #
#    Dockerfile                                         :+:      :+:    :+:    #
#                                                     +:+ +:+         +:+      #
#    By: aabelque <marvin@42.fr>                    +#+  +:+       +#+         #
#                                                 +#+#+#+#+#+   +#+            #
#    Created: 2018/02/22 17:37:37 by aabelque          #+#    #+#              #
#    Updated: 2018/02/22 18:29:19 by aabelque         ###   ########.fr        #
#                                                                              #
# **************************************************************************** #

FROM debian

RUN apt-get update -y && apt-get upgrade -y && apt-get install wget bzip2 -y

EXPOSE 9987/udp 30033 10011

WORKDIR /home/ts3

RUN wget http://dl.4players.de/ts/releases/3.0.13.8/teamspeak3-server_linux_amd64-3.0.13.8.tar.bz2 &&\
	bunzip2 *.bz2 &&\
	tar -xvf *.tar

WORKDIR teamspeak3-server_linux_amd64

ENTRYPOINT ["sh", "ts3server_minimal_runscript.sh", "createinifile=1"]
