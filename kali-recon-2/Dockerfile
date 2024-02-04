FROM golang:1.19 as builder
RUN go install -v github.com/owasp-amass/amass/v3/...@master
RUN go install -v github.com/tomnomnom/anew@latest
RUN go install -v github.com/tomnomnom/assetfinder@latest
RUN go install -v github.com/tomnomnom/httprobe@latest
RUN go install -v github.com/tomnomnom/fff@latest
RUN go install -v github.com/jaeles-project/gospider@latest
RUN go install -v github.com/ffuf/ffuf@latest
RUN go install -v github.com/OJ/gobuster/v3@latest
RUN go install -v github.com/projectdiscovery/httpx/cmd/httpx@v1.2.8
RUN apt-get update && apt-get install -y masscan

FROM ubuntu:20.04
COPY --from=builder go/bin/amass /usr/bin/amass
COPY --from=builder go/bin/anew /usr/bin/anew
COPY --from=builder go/bin/assetfinder /usr/bin/assetfinder
COPY --from=builder go/bin/httprobe /usr/bin/httprobe
COPY --from=builder go/bin/fff /usr/bin/fff
COPY --from=builder go/bin/gospider /usr/bin/gospider
COPY --from=builder go/bin/ffuf /usr/bin/ffuf
COPY --from=builder go/bin/gobuster /usr/bin/gobuster
COPY --from=builder go/bin/httpx /go/bin/httpx
COPY --from=builder /usr/bin/masscan /usr/bin/masscan

RUN apt-get update
ENV DEBIAN_FRONTEND noninteractive
RUN apt install -y mongodb-clients python3 python3-pip dnsutils nmap whois whatweb dnsenum nikto
RUN apt install -y wget unzip curl libcurl4-openssl-dev libssl-dev git

# Install additional Python tools 
RUN pip3 install arjun
RUN pip3 install dirsearch 
RUN pip3 install wfuzz

ENV VERSION=1.4.3
RUN apt-get update && \
    apt-get install -y net-tools iputils-ping

# Install Chromium
WORKDIR /opt/chromium
RUN apt-get install -yq gconf-service libasound2 libatk1.0-0 libc6 libcairo2 libcups2 libdbus-1-3 \
    libexpat1 libfontconfig1 libgcc1 libgconf-2-4 libgdk-pixbuf2.0-0 libglib2.0-0 libgtk-3-0 libnspr4 \
    libpango-1.0-0 libpangocairo-1.0-0 libstdc++6 libx11-6 libx11-xcb1 libxcb1 libxcomposite1 \
    libxcursor1 libxdamage1 libxext6 libxfixes3 libxi6 libxrandr2 libxrender1 libxss1 libxtst6 \
    ca-certificates fonts-liberation libappindicator1 libnss3 lsb-release xdg-utils

RUN git clone https://github.com/scheib/chromium-latest-linux
WORKDIR /opt/chromium/chromium-latest-linux
RUN ./update.sh && ln -s /opt/chromium/chromium-latest-linux/latest/chrome /usr/bin/chromium

# install aquatone binary
WORKDIR /opt/aquatone
RUN wget https://github.com/michenriksen/aquatone/releases/download/v${VERSION}/aquatone_linux_amd64_${VERSION}.zip && \
    unzip aquatone_linux_amd64_${VERSION}.zip && \
    cp aquatone /usr/bin

# install cargo and rustscan, metabigor
RUN curl https://sh.rustup.rs -sSf | sh -s -- -y
RUN /bin/bash -c 'source "$HOME/.cargo/env" && cargo install rustscan'
RUN wget https://github.com/j3ssie/metabigor/releases/download/v1.2.3/metabigor_v1.2.3_linux_amd64.tar.gz
RUN tar -xvf metabigor_v1.2.3_linux_amd64.tar.gz && mv metabigor /usr/bin
RUN apt install -y nano time openssh-server
RUN mkdir -p /var/run/sshd

# authorize SSH connection with root account
RUN echo 'PermitRootLogin yes' >> /etc/ssh/sshd_config
# change password root
RUN echo "root:kali"|chpasswd
# RUN service ssh restart
RUN ssh-keygen -A

# Set up passwordless SSH
RUN mkdir -p /root/.ssh && \
    ssh-keygen -t rsa -b 2048 -f /root/.ssh/id_rsa -N '' && \
    cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys && \
    rm -rf /root/.ssh/id_rsa.pub && \
    chmod 600 /root/.ssh/authorized_keys && \
    chmod 700 /root

# Copy the custom sshd_config file to the container
COPY config/sshd_config /etc/ssh/sshd_config

RUN mkdir /root/wordlists
COPY ./fuzz-Bo0oM.txt /root/wordlists
COPY ./common.txt /root/wordlists
WORKDIR /root/wordlists 
COPY ./SecLists /root/wordlists/SecLists
COPY ./dirbuster /root/wordlists/dirbuster
WORKDIR /root
CMD ["/usr/sbin/sshd", "-D","-f","/etc/ssh/sshd_config"]


# git clone github.com/projectdiscovery/httpx httpx-new; cd httpx-new/cmd/httpx; go build .; mv httpx /usr/local/bin/; httpx -h
# Clone and build httpx
# RUN git clone https://github.com/projectdiscovery/httpx.git && \
#     cd httpx/cmd/httpx && \
#     go build -o /usr/bin/httpx .
#how to add php 8.0 have mysqli in dockerfile with ubuntu 20.04
