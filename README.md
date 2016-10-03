# Github Pre-receive Hook: Base Image

This is the base image from the Dockerfile https://help.github.com/enterprise/2.6/admin/guides/developer-workflow/creating-a-pre-receive-hook-script/#testing-pre-receive-scripts-locally

# Build

```
$ docker build -f Dockerfile.pre-receive -t github-enterprise-pre-receive-hook-base .
Sending build context to Docker daemon 208.9 kB
Step 1 : FROM pre-receive.dev
 ---> 174b3be1d2c9
Step 2 : MAINTAINER Marcello_deSales@intuit.com
 ---> Running in e3965f484166
 ---> 5cafeb0f185b
Removing intermediate container e3965f484166
Step 3 : RUN apk add --no-cache py-pip &&   pip install pyyaml pyjavaproperties
 ---> Running in 38674126a032
fetch http://alpine.gliderlabs.com/alpine/v3.3/main/x86_64/APKINDEX.tar.gz
fetch http://alpine.gliderlabs.com/alpine/v3.3/community/x86_64/APKINDEX.tar.gz
(1/7) Installing libbz2 (1.0.6-r4)
(2/7) Installing libffi (3.2.1-r2)
(3/7) Installing gdbm (1.11-r1)
(4/7) Installing sqlite-libs (3.9.2-r0)
(5/7) Installing python (2.7.12-r0)
(6/7) Installing py-setuptools (18.8-r0)
(7/7) Installing py-pip (7.1.2-r0)
Executing busybox-1.24.2-r0.trigger
OK: 81 MiB in 33 packages
Collecting pyyaml
  Downloading PyYAML-3.12.tar.gz (253kB)
Collecting pyjavaproperties
  Downloading pyjavaproperties-0.6.tar.gz
Installing collected packages: pyyaml, pyjavaproperties
  Running setup.py install for pyyaml
  Running setup.py install for pyjavaproperties
Successfully installed pyjavaproperties-0.6 pyyaml-3.12
You are using pip version 7.1.2, however version 8.1.2 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
 ---> 96d7af8e0851
Removing intermediate container 38674126a032
Successfully built 96d7af8e0851
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

Before you can push to the server, copy the ssh keys locally.

```
$ docker cp data:/home/git/.ssh/id_rsa .
```

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

```
$ git remote add test git@$(ifconfig eth0 | grep 'inet addr:' | cut -d: -f2 | awk '{ print $1}'):test.git
```

The command gets the ip address of the host and adds as the server. You can verify if the server is reachable using the following command.

> Note that you need to `id_rsa` you copied from the data container described in previous section.

```
$ GIT_SSH_COMMAND="ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -p 52311 -i ../id_rsa" git remote show test
Warning: Permanently added '[172.16.188.135]:52311' (ECDSA) to the list of known hosts.
* remote test
  Fetch URL: git@172.16.188.135:test.git
  Push  URL: git@172.16.188.135:test.git
  HEAD branch: (unknown)
```

At this point, you are ready to proceed. Execute the push command and the pre-commit hook you developed will be automatically verifying your code. Note that if the pre-receive hook fails, the commit won't be accepted.

```
$ GIT_SSH_COMMAND="ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -p 52311 -i ../id_rsa" git push -u test master              
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
