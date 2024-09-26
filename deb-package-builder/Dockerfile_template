FROM debian:bookworm-slim

WORKDIR /root

RUN apt-get update && apt-get install -y nano git procps dh-make devscripts build-essential lintian

RUN apt-get install -y python3-pip
# this is required to use python3.11+ without venv
#RUN mv /usr/lib/python3.11/EXTERNALLY-MANAGED /usr/lib/python3.11/EXTERNALLY-MANAGED.old

ENV DEBEMAIL="myname@ufmg.br"
ENV DEBFULLNAME="Myname"
ARG BASE_DIR="/root"
RUN mkdir deb_packages
# this will be printed on every shell init
COPY <<"EOF" /help.sh 
echo -e "\tWelcome to deb package builder in iterative mode.
\t\tThis container image comes with some tools installed: dh_make, devscripts
\t\tYou can use them to automate package build 
\t\tman pages: https://dyn.manpages.debian.org/bookworm/dh-make/dh_make.1.en.html
\t\t           https://manpages.debian.org/testing/devscripts/devscripts.1.en.html
\t\tHow to create a hello-wolrd debian package:
\t\t           https://github.com/lcharles123/hello-debian-package
"
EOF
RUN chmod +x /help.sh
RUN echo "source /help.sh" >> .bashrc

COPY <<"EOF" /do_build.sh 
#!/bin/bash


trap "echo -e ' \nAbort.' && exit" INT
### Parse args
echo -e ""
echo -e "This is a instance of deb-package-builder."
echo -e ""
usage() {
    echo -e '\tUsage: "docker run <this_image>"  build|upgrade|iterative  <source_url>  <upstream_version>  [dependency_1 ...]'
    echo -e '\t       make drun arg="build|upgrade|iterative  <source_url>  <upstream_version>  [dependency_1 ...]"'
    echo -e "\t\t"
    echo -e "\t\t<source url> must be a valid git repository"
    echo -e "\t\tdependency_1... must exist on APT repository of the container"
    echo -e "\t\t"    
    exit 1
}
action=""
case "$1" in
    build)
        action=build
        ;;
    upgrade)
        action=upgrade
        ;;
    iterative)
        action=iterative
        ;;
    *)
        usage
        ;;
esac
[[ $# -lt 3 ]] && [ "$action" != iterative ] && usage
source_url="$2"
upstream_version="$3"
shift 3
dependencies_to_install=$(echo "$@")
repo_name=$(basename "$source_url")
repo_name="$BASE_DIR"/${repo_name%.git}
versioned_repo_name="$repo_name"_"$upstream_version"
if [ "$action" == iterative ] ;
then
    echo -e "Entering iterative mode:"
    bash
    exit
fi
echo -e "\tYou entered the following parameters:"
echo -e "\t\tAction(build|upgrade): "$action
echo -e "\t\tRepository: "$source_url
echo -e "\t\tVersion: "$upstream_version
echo -e "\t\tDependencies: ""${dependencies_to_install[@]}"
echo -e "\t\tFinal package: ""$versioned_repo_name"".deb"
echo -e "\t"
echo -e "\tPress ENTER to continue if are all ok or CTRL + C to abort..." 
read continue
echo -e "\t"
echo -e "\tThe container will try to do the work for you and output the ""$versioned_repo_name"".deb file."
echo -e "\t"

### Now  are the proccess of building the package  ###
if ! [ -d "$repo_name" ] ; 
then
    echo -e '\tGetting the repository from "$source_url"...'
    git clone "$source_url" || (echo "***ERROR***: input url isn't a valid git repository." && exit 1)
fi
echo -e "\t"
# TODO try to generate debian files if not present
# this may fail but can bootstrap the generation of these files 
if ! [ -d debian ] && ! [ -d DEBIAN ] ; 
    then
    echo -e "\tGenerating debian-control files using dh-make..."
    cd "$repo_name" && git archive HEAD | gzip > ../"$versioned_repo_name".tar.gz
    dh_make -p "$versioned_repo_name" -s -f ../"$versioned_repo_name".tar.gz

    #TODO call nano to edit debian/files?
    #the important are: 
    #
    #https://www.debian.org/doc/manuals/maint-guide/dreq.en.html
    #
    echo -e "\tYou can edit the debian files now or press CTRL+X to exit editor..."
    read -p "\tPress ENTER to continue" enter
    cd "$repo_name"/debian && nano control copyright rules install 
    #
    #exemple here
    #https://github.com/spotify/dh-virtualenv/issues/326
    #
    # i.e. for different architetures: source, all
    #Source: helloworld  
    #Maintainer: myname <myname@example.com>
    #Build-Depends: debhelper (>= 8.0.0)
    #Standards-Version: 3.9.3
    #Section: utils
    #
    #Package: helloworld  
    #Priority: extra  
    #Architecture: any
    #Depends: ${shlibs:Depends}, ${misc:Depends}  
    #Description: a simple helloworld package

    # edit changelog file
    # changelog file
    # created already
    dch -i --create
    #nano debian/changelog
    #https://blog.packagecloud.io/buildling-debian-packages-with-debuild/
fi

#If it arrives here, means that may be ok to install dependencies and do the build
if [ -n "$dependencies_to_install" ] ;
then
    echo -e "\tInstaling dependencies..."
    apt update && apt install -y "$dependencies_to_install"
fi

#build without bureaucracy
cd "$repo_name" && dpkg-buildpackage -b
# Now the .deb package are in root folder, 
echo -e "\tDONE"
echo -e "\tNow the .deb package are in /root folder"
echo -e "\tYou can get it on the host machine in the configured virtual volume"

EOF

RUN chmod +x /do_build.sh
ENTRYPOINT [ "/do_build.sh" ]

