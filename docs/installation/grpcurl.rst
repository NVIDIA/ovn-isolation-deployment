====================================================
Tool Installation for Interacting with gRPC servers
====================================================

grpcurl
----------

"grpcurl" is a command-line tool that allows you to interact with gRPC servers.
It is essentially a curl-like tool specifically designed for gRPC servers.

**grpcurl install**::

    wget https://github.com/fullstorydev/grpcurl/releases/download/v1.8.9/grpcurl_1.8.9_linux_x86_64.tar.gz
    tar -xvzf grpcurl_1.8.9_linux_x86_64.tar.gz
    mv grpcurl /usr/local/bin/
or::

    go install github.com/fullstorydev/grpcurl/cmd/grpcurl@latest