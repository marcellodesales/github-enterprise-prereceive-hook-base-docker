# Github Pre-receive Hook: Base Images

A collection of base images based on the requirements from the Dockerfile for the Github Enterprise Pre-receive Hooks https://help.github.com/enterprise/2.6/admin/guides/developer-workflow/creating-a-pre-receive-hook-script/#testing-pre-receive-scripts-locally

[![resolution](http://dockeri.co/image/marcellodesales/github-enterprise-prereceive-hook-base "Github Enterprise Pre-Receive Hook Base Image")](https://hub.docker.com/r/intuit/spring-cloud-config-validator/)

# Base Images

* Use one of the base images for the implementation of the hooks:

## Golang

* Golang 1.13.x

```console
$ docker run -ti marcellodesales/git-pre-receive-hook:golang go version
go version go1.13.4 linux/amd64
```

## Python

* Python 3 Base

```console
$ docker run -ti marcellodesales/git-pre-receive-hook:python3 python --version
Python 3.6.9
```

* Python 2 Base

```console
$ docker run -ti marcellodesales/git-pre-receive-hook:python2 python --version
Python 2.7.16
```

# Build

```console
$ docker-compose build
Building marcellodesales-git-pre-receive-hook-python2
Step 1/5 : FROM marcellodesales/git-sshd-server
 ---> 5ca61c1ab080
Step 2/5 : LABEL maintainer="marcello.desales@gmail.com"
 ---> Using cache
 ---> 28897c326ab3
Step 3/5 : RUN apk add --no-cache python &&     python -m ensurepip &&     rm -r /usr/lib/python*/ensurepip &&     pip install --upgrade pip setuptools &&     rm -r /root/.cache
 ---> Using cache
 ---> be1de5727086
Step 4/5 : WORKDIR /home/git
 ---> Using cache
 ---> d4482b5fa6f5
Step 5/5 : CMD ["/usr/sbin/sshd", "-D"]
 ---> Using cache
 ---> a517a25b2ee5

Successfully built a517a25b2ee5
Successfully tagged marcellodesales/git-pre-receive-hook:python2
Building marcellodesales-git-pre-receive-hook-python3
Step 1/5 : FROM marcellodesales/git-sshd-server
 ---> 5ca61c1ab080
Step 2/5 : LABEL maintainer="marcello.desales@gmail.com"
 ---> Using cache
 ---> 28897c326ab3
Step 3/5 : RUN apk add --no-cache python3 &&     python3 -m ensurepip &&     rm -r /usr/lib/python*/ensurepip &&     pip3 install --upgrade pip setuptools &&     if [ ! -e /usr/bin/pip ]; then ln -s pip3 /usr/bin/pip ; fi &&     if [[ ! -e /usr/bin/python ]]; then ln -sf /usr/bin/python3 /usr/bin/python; fi &&     rm -r /root/.cache
 ---> Using cache
 ---> c1bd0babe9cd
Step 4/5 : WORKDIR /home/git
 ---> Using cache
 ---> 81a620673d38
Step 5/5 : CMD ["/usr/sbin/sshd", "-D"]
 ---> Using cache
 ---> 79bdc846a728

Successfully built 79bdc846a728
Successfully tagged marcellodesales/git-pre-receive-hook:python3
Building marcellodesales-git-pre-receive-hook-golang
Step 1/11 : FROM marcellodesales/git-sshd-server
 ---> 5ca61c1ab080
Step 2/11 : LABEL maintainer="marcello.desales@gmail.com"
 ---> Using cache
 ---> 28897c326ab3
Step 3/11 : RUN apk add --no-cache 		ca-certificates
 ---> Using cache
 ---> 2895a1280db7
Step 4/11 : RUN [ ! -e /etc/nsswitch.conf ] && echo 'hosts: files dns' > /etc/nsswitch.conf
 ---> Using cache
 ---> 4b2e05ca3755
Step 5/11 : ENV GOLANG_VERSION 1.13.4
 ---> Using cache
 ---> e7537f5b6c69
Step 6/11 : ENV GOLANG_VERSION_CHECKSUM 95dbeab442ee2746b9acf0934c8e2fc26414a0565c008631b04addb8c02e7624
 ---> Using cache
 ---> cec20e3fdef9
Step 7/11 : RUN set -eux; 	apk add --no-cache --virtual .build-deps 		bash 		gcc 		musl-dev 		openssl 		go 	; 	export 		GOROOT_BOOTSTRAP="$(go env GOROOT)" 		GOOS="$(go env GOOS)" 		GOARCH="$(go env GOARCH)" 		GOHOSTOS="$(go env GOHOSTOS)" 		GOHOSTARCH="$(go env GOHOSTARCH)" 	; 	apkArch="$(apk --print-arch)"; 	case "$apkArch" in 		armhf) export GOARM='6' ;; 		x86) export GO386='387' ;; 	esac; 	wget -O go.tgz "https://golang.org/dl/go${GOLANG_VERSION}.src.tar.gz"; 	tar -C /usr/local -xzf go.tgz; 	rm go.tgz; 		cd /usr/local/go/src; 	./make.bash; 		rm -rf 	/usr/local/go/pkg/bootstrap 		/usr/local/go/pkg/obj 	; 	apk del .build-deps; 		export PATH="/usr/local/go/bin:$PATH"; 	go version
 ---> Using cache
 ---> 4f25306f0fb2
Step 8/11 : ENV GOPATH /go
 ---> Using cache
 ---> b35569859951
Step 9/11 : ENV PATH $GOPATH/bin:/usr/local/go/bin:$PATH
 ---> Using cache
 ---> d570af0898e3
Step 10/11 : RUN mkdir -p "$GOPATH/src" "$GOPATH/bin" && chmod -R 777 "$GOPATH"
 ---> Using cache
 ---> 627e0e9d7430
Step 11/11 : WORKDIR /home/git
 ---> Using cache
 ---> 04b6376984ea

Successfully built 04b6376984ea
Successfully tagged marcellodesales/git-pre-receive-hook:golang
```

# Extensions

You can create your own Github Pre-receive hook. Just create another image `FROM` this image.
The example below shows adding `python` to the docker image in order to execute a python script.

```
$ docker build -f Dockerfile.spring-cloud-config-validation -t springboot-config-verification . 
Sending build context to Docker daemon 210.4 kB
Step 1 : FROM github-enterprise-pre-receive-hook-base
 ---> 174b3be1d2c9
Step 2 : MAINTAINER Marcello_deSales@intuit.com
 ---> Using cache
 ---> 5cafeb0f185b
Step 3 : RUN apk add --no-cache py-pip &&   pip install pyyaml pyjavaproperties
 ---> Using cache
 ---> 96d7af8e0851
Successfully built 96d7af8e0851
```

# Setup for Verification

You must run a container that holds the data, which means, having a script that does the validation.

```
$ docker run --name data springboot-config-verification /bin/true
```

After that, you can simply copy the verification script to the running container. This is step 7 from the
github Enterprise documentation https://help.github.com/enterprise/2.6/admin/guides/developer-workflow/creating-a-pre-receive-hook-script/#testing-pre-receive-scripts-locally.

```
$ docker cp validate-config-files.py data:/home/git/test.git/hooks/pre-receive
```

At this point, you can start a github server that mimics Github Enterprise as follows:

```
$ docker run -d -p 52311:22 --volumes-from data springboot-config-verification 
7966ae3a6b83184cb38b6cbd1bbef22b35a52955f035f87b4f7a53b541352c2b

$ docker ps
CONTAINER ID        IMAGE                            COMMAND               CREATED             STATUS              PORTS                   NAMES
7966ae3a6b83        springboot-config-verification   "/usr/sbin/sshd -D"   7 seconds ago       Up 6 seconds        0.0.0.0:52311->22/tcp   ecstatic_panini
```

Before you can push to the server, copy the ssh keys from the Git server locally so you can connect to it.

```
$ docker cp data:/home/git/.ssh/id_rsa id_rsa_from_container
```

* Make sure to remember that commands that needs to connect to the Git server requires the relative or full path to `id_rsa_from_container`.

## Subsequent Work

And finally, you can quickly iterate and test your verification script using the following:

```
$ docker run --name gitrepo -e GITHUB_PULL_REQUEST_HEAD=534b172 -e GITHUB_PULL_REQUEST_BASE=f394274 -d -p 52311:22 --volumes-from data -e GIT_DIR=/home/git/test.git springboot-config-verification
```

And then, you can quickly re-execute using:

```
docker stop gitrepo && docker rm gitrepo && docker run --name gitrepo -e GITHUB_PULL_REQUEST_HEAD=534b172 -e GITHUB_PULL_REQUEST_BASE=f394274 -d -p 52311:22 --volumes-from data -e GIT_DIR=/home/git/test.git springboot-config-verification
gitrepo
gitrepo
afe70ca2e5b51757a5cf8ea2126c662a94b6ba49a6bb093080666baaf7e02551
```

This is the point where you have an ssh server simulating a git server and ready to execute the `pre-receive` hook you can create.

# Executing checks locally

After setting up the ssh server above, you are ready to verify your script. You need to add the ssh server as a test remote origin to any directory from a cloned Github repository.

Determine which ip address locally represents your machine. ifconfig can help you.

* Linux: try eth0 or eth1

```
$ git remote add test git@$(ifconfig eth0 | grep 'inet addr:' | cut -d: -f2 | awk '{ print $1}'):test.git
```

* Mac: try en0 or en1 depending on wifi or ethernet.

```
$ git remote add test git@$(ipconfig getifaddr en0):test.git
```

The command gets the ip address of the host and adds as the server. You can verify if the server is reachable using the following command.

> Note that you need to `id_rsa_from_container` you copied from the data container described in previous section.

```
$ GIT_SSH_COMMAND="ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -p 52311 -i PATH_TO_ID/id_rsa_from_container" git remote show test
Warning: Permanently added '[172.16.188.135]:52311' (ECDSA) to the list of known hosts.
* remote test
  Fetch URL: git@172.16.188.135:test.git
  Push  URL: git@172.16.188.135:test.git
  HEAD branch: (unknown)
```

At this point, you are ready to proceed. Execute the push command and the pre-commit hook you developed will be automatically verifying your code. Note that if the pre-receive hook fails, the commit won't be accepted.

```
$ GIT_SSH_COMMAND="ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -p 52311 -i PATH_TO_ID/id_rsa_from_container" git push -u test master        
Warning: Permanently added '[172.16.188.135]:52311' (ECDSA) to the list of known hosts.
Counting objects: 214, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (100/100), done.
Writing objects: 100% (214/214), 21.71 KiB | 0 bytes/s, done.
Total 214 (delta 126), reused 184 (delta 109)
remote: ##################################################
remote: ###### Intuit Spring Cloud Config Validator ######
remote: ##################################################
remote: Processing base=0000000000000000000000000000000000000000 commit=6ff408ac89564c994925c46847d775fff940caa3 ref=refs/heads/master
remote: => Validating SHA 6ff408ac89564c994925c46847d775fff940caa3
remote: ✘ File .matrix-android.json is NOT valid: Extra data: line 2 column 11 - line 29 column 1 (char 11 - 394)
remote: ✔ File .matrix-ios.json is valid!
To git@172.16.188.135:test.git
 ! [remote rejected] master -> master (pre-receive hook declined)
error: failed to push some refs to 'git@172.16.188.135:test.git'
```
