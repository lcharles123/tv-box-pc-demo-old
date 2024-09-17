# tv-box-pc-demo
This repository contains 2 Docker images:
## build-server:
This is a container specification that must receive the code and transform it in a package

### How to use?
You need Docker installed.

Use provided Makefile, `make` will print it.

This uses dh-make to build a .deb package, for that it must receive the code of a git repository in root of the container

## package-server
About:
This is a package repository that stores packages for using with a package manager (apt, aptitude)

How to configure the server?
- You need Docker installed.
- Just use provided Makefile, `make` will print it.

How to send packages to the server?
- You need to copy to root directory inside the container, it can be done in two ways:
	1. Copying to a shared folder with docker container
	2. Using ssh (sftp, scp) to transfer to docker container.
- After copying, run the following commands inside docker for uploading to repository: `reprepro includedeb bookworm <file.deb>`

How to use the repository?
- Modify /etc/apt/sources.list or include new list file inside /etc/apt/sources.list.d with the url and tags of the repository: `deb [trusted=yes] http://nerdvm.mywire.org:8080/ bookworm main`
- Do apt-get update and the packages will be available to install or download using apt.
- You can search a specific package using this command
    `apt-cache search <package_name>`





# References:
- https://github.com/streadway/hello-debian/blob/master/README.md
- https://lazyfrosch.github.io/training-debian-packaging/dev/dh_make/
- https://blog.packagecloud.io/how-to-build-debian-packages-for-simple-shell-scripts/





