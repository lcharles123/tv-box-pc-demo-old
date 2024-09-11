FROM debian:bullseye-slim

WORKDIR /root

RUN apt-get update && apt-get install -y nano dh-make lintian tree git

RUN apt-get install -y procps devscripts

COPY <<"EOF" /root/.bashrc
DEBEMAIL="myname@ufmg.br"
DEBFULLNAME="myname"
export DEBEMAIL DEBFULLNAME
EOF

RUN git clone https://github.com/lcharles123/videoboard /root/videoboard-0.1

#RUN cd /root/videoboard-0.1 && dh_make --indep --createorig
#RUN cd /root/videoboard-0.1 && debuild -us -uc

CMD [ "/bin/bash" ]
