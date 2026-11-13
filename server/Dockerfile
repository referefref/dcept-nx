# Domain Controller Enticing Password Tripwire (DCEPT) - Dockerfile
# Python 3 version (dcept-nx). The original image used Ubuntu 16.04, Python 2 and
# John the Ripper 1.8.0-jumbo-1, none of which build or install today.

FROM debian:12-slim

ENV DEBIAN_FRONTEND=noninteractive PYTHONUNBUFFERED=1

RUN echo "wireshark-common wireshark-common/install-setuid boolean false" | debconf-set-selections && \
    apt-get update && \
    apt-get install -y --no-install-recommends tshark tcpreplay python3 python3-venv ca-certificates \
        git build-essential libssl-dev zlib1g-dev && \
    rm -rf /var/lib/apt/lists/*

# John the Ripper (community "jumbo" version, which supports krb5pa-sha1), pinned to a commit
ARG JOHN_COMMIT=5184bc0b8c105f8c5978b112e723d65b2de3e63e
RUN git init -q /tmp/john && cd /tmp/john && \
    git fetch -q --depth 1 https://github.com/openwall/john "${JOHN_COMMIT}" && git checkout -q FETCH_HEAD && \
    cd src && ./configure --disable-openmp >/dev/null && make -s -j"$(nproc)" >/dev/null && \
    mkdir -p /opt/john && cp -r /tmp/john/run /opt/john/run && rm -rf /tmp/john && \
    # DCEPT reads John's output: turn off colour codes and admin highlighting
    sed -i 's/^UseColors = Y/UseColors = N/; s/^MarkAdminCracks = Y/MarkAdminCracks = N/' /opt/john/run/john.conf

RUN mkdir -p /opt/dcept/var && \
    printf '#!/bin/sh\nexec /opt/john/run/john --session=/opt/dcept/var/john "$@"\n' > /opt/dcept/john && \
    chmod 755 /opt/dcept/john

COPY requirements.txt /opt/dcept/requirements.txt
RUN python3 -m venv /opt/dcept/venv && /opt/dcept/venv/bin/pip install --no-cache-dir -r /opt/dcept/requirements.txt

COPY dcept.py Cracker.py GenerationServer.py ConfigReader.py alert.py dcept.cfg example.pcap /opt/dcept/

WORKDIR /opt/dcept
CMD ["/opt/dcept/venv/bin/python", "/opt/dcept/dcept.py"]
