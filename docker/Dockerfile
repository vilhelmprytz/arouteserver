FROM python:3.9

RUN mkdir /arouteserver
WORKDIR /arouteserver

RUN mkdir /bgpq4 && \
    cd /bgpq4 && \
    git clone https://github.com/bgp/bgpq4.git ./ && \
    ./bootstrap && \
    ./configure && \
    make && \
    make install

ADD requirements.txt ./

RUN pip install --upgrade pip
RUN pip install -r requirements.txt

COPY . ./
COPY docker/run.sh /root/run.sh
RUN pip install .

RUN arouteserver \
    setup --dest-dir /etc/arouteserver

RUN rm /etc/arouteserver/clients.yml

CMD /root/run.sh
