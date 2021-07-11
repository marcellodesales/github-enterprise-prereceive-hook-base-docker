FROM marcellodesales/git-sshd-server
LABEL maintainer="marcello.desales@gmail.com"

RUN apk add --no-cache python && \
    python -m ensurepip && \
    rm -r /usr/lib/python*/ensurepip && \
    pip install --upgrade pip setuptools && \
    rm -r /root/.cache

WORKDIR /home/git

CMD ["/usr/sbin/sshd", "-D"]
