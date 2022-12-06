# 1 - Dependencies management

***git branch name:*** dependencies

## Theory [2]

As usual, we will start with a few theoretical questions:

* [0.5] What is Docker, and how it differs from dependencies management systems? From virtual machines?

*Docker* is an instrument for buildings isolated virtual user spaces called *containers* inside one operating system with the help of program called *Docker Engine*. Containers have their own file spaces, adjusted dependencies and installed software. Nevertheless, there are ways for the containers to exchange files.

*Dependencies management systems* isolate different sets of software and their dependencies, working on the level of libraries (packages) inside the only one user space, thus the file system is common for all *environments* of such a management system. Thus, Docker container is an exemplar of an operating system inside which you can build several environments with different sets of software.

Virtual machine emulates *guest operating systems* inside a *host operating system*. Guest operating systems are managed by a *hypervisor* and represent all the features of real operating systems, in particular they need much memory to be allocated in logical drives, require computational power in addition to what is used by a host operating system, provide higher isolation of user space. In this regard, Docker container is much more flexible: you don't need to know in advance how much space you need for it, they have channel to exchange file. All the containers work on one kernel what makes them more efficient.

* [0.5] What are the advantages and disadvantages of using containers over other approaches?

**Advantages:**

1. Require less computational power and time then virtual machines.
2. Safer, faster and more stable then dependencies management systems.
3. Environments are restricted by packages available in a corresponding dependencies management systems, while  you can put any version of software into Dockerfile to build a container.
4. Docker allows to work with indipendent, not overlapping file systems.

**Disadvantages:**

1. Good adjustment of the work with Docker needs highly qualified DevOps specialists to make dealing with containers so effective.
2. Problems with backward compatibility because Docker is developed very fast.
3. The process of removing Docker containers takes a lot of time and requires a lot of input/output operations.

* [0.5] Explain how Docker works: what are Dockerfiles, how are containers created, and how are they run and destroyed?

Docker allows to divide the workspace into several relatively independent user spaces called containers. To create a new container you need to have an *image*, i.e. template with all required software and dependencies on the basis of which new containers are created. To create an image you can write a *Dockerfile* which incudes all the necessary information about the image and commands which will be launched while creating a new container. For example, the next code in a Dockerfile will install Perl and copy the `/home/usr/folder` folder from external environment to a new container when the container is created. It also prescribes to print 'Welcome!' each time the container is run:

```
RUN apt -y install perl
COPY /home/usr/folder /home/usr/folder
CMD echo 'Welcome!'
```

To create image from Dockerfile use the command `docker build [OPTIONS] PATH`, where `PATH` is the path to the Dockerfile. For more information see [this](https://docs.docker.com/engine/reference/commandline/build/).

To launch the Docker container from image use `docker run [OPTIONS] IMAGE [COMMAND] [ARG...]`, where `IMAGE` is the title of image. For more information see [this](https://docs.docker.com/engine/reference/commandline/run/).

To remove the Docker container use `docker rm [OPTIONS] CONTAINER [CONTAINER...]`, where `CONTAINER` is the title of container. For more information see [this](https://docs.docker.com/engine/reference/commandline/rm/).

* [0.25] Name and describe at least one Docker competitor (i.e., a tool based on the same containerization technology).

**Podman** (Pod Manager tool) is a container engine developed by Red Hat. It is the main alternative to Docker. It is fully compatible with Docker, so that you even may assign alias: `alias docker=podman` and use it as you used Docker. All the commands are alike! But still there are some differences. Podman is available only in Linux while Docker is developed for Linux, FreeBSD, MacOS and Windows. Podman does not have a daemon unlike Docker which uses it to manage all containers under its control. Another feature of Podman that is not yet available in Docker is the ability to create and run so-called *pods* — groups of containers deployed together.

* [0.25] What is conda? How it differs from apt, yarn, and others?

Conda is a dependencies management system inside Anaconda or Miniconda which are distributions including a set of popular free libraries. Conda can run locally and independently for each user. It does not require administrator rights, unlike brew and apt. Unlike Composer for PHP, pip for Python or npm for JavaScript, conda is not restricted by programming language. It manages packages united by the problems of data science and machine learning. Moreover, this package manager also provides extended functionality for working with environments.

## Problem [6.5]

The problem itself is relatively simple. 

Imagine that you developed an excellent RNA-seq analysis pipeline and want to share it with the world. Based on your experience, you are confident that the popularity of the pipeline will be proportional to its ease of use. So, you decided to help your future users and to pack all dependencies in a Conda environment and a Docker container.

Here is the list of tools and their versions that are used in your work:
* [fastqc](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/), v0.11.9
* [STAR](https://github.com/alexdobin/STAR), v2.7.10b
* [samtools](https://github.com/samtools/samtools), v1.16.1
* [picard](https://github.com/broadinstitute/picard), v2.27.5
* [salmon](https://github.com/COMBINE-lab/salmon), commit tag 1.9.0
* [bedtools](https://github.com/arq5x/bedtools2), v2.30.0
* [multiqc](https://github.com/ewels/MultiQC), v1.13

### Anaconda

* [1] Install conda, create a new virtual environment, and install all necessary packages.

I use Linux Ubuntu and I already have Miniconda installed, but to install it one can use the following commands:

`wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh`

`bash Miniconda3-latest-Linux-x86_64.sh`

Create new totaly empty environment called 'dependencies' by the command:

`conda create --name dependencies --no-default-packages`

Activate a new environment:

`conda activate dependencies`

Add channels through which I will install new packages:

`conda config --add channels bioconda`

`conda config --add channels conda-forge`

Install the particular versions of the tools:

`conda install fastqc=0.11.9`

`conda install star=2.7.10b`

`conda install bedtools=2.30.0`

`conda install samtools=1.16.1`

`conda install salmon=1.9.0`

`conda install multiqc=1.13`

Export the environment to the environment.yml file:

`conda env export --from-history > environment.yml`

The content of the **environment.yml** file:

```
name: dependencies
channels:
  - conda-forge
  - bioconda
  - defaults
dependencies:
  - samtools=1.16.1
  - salmon=1.9.0
  - multiqc=1.13
  - fastqc=0.11.9
  - star=2.7.10b
  - bedtools=2.30.0
prefix: /home/snitkin/anaconda3/envs/dependencies
```

* [0.75] You won't be able to install some tools - that's fine. List their names, and explain what should be done to make them conda-friendly ([conda-forge](https://conda-forge.org/docs/maintainer/adding_pkgs.html) channel, [bioconda](https://bioconda.github.io/contributor/workflow.html) channel).

I failed to install picard of the 2.27.5 version, because the latest version of it in Conda is 2.27.4. Not all the packages and their versions can be found in Conda.

To add your package to Conda-Forge or Bioconda channels you should generate a recipe (special file including information about your tool and dependencies), include all necessary tests and check that all tests pass successfully, fork and clone a corresponding repository from GitHub, add your project there and make Pull Request to the `main` branch, wait while your package is available in Conda. 
 
* [0.25] [Export](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#exporting-the-environment-yml-file) the environment ([example](https://github.com/nf-core/clipseq/blob/master/environment.yml)) to the file and verify that it can be [rebuilt](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#creating-an-environment-from-an-environment-yml-file) from the file without problems.

Delete the dependencies environment:

`conda env remove -n dependencies`

Recover the invironment on the basis of the environment.yml file:

`conda env create -n dependencies --file environment.yml`

After download, check that all the packages of particular versions are there with `conda list`. Yes, this is it!

### Docker
* [3] Create a Dockerfile for a container with **all** required dependencies. Don't forget about comments; test that all tools are accessible and work inside the container. Hints:
 - If needed, grant rights to execute downloaded/compiled binaries using chmod (`chmod a+x BINARY_NAME`)
 - Move all executables to $PATH folders (e.g.`/usr/local/bin`) to make them accessible without specifying the full path.
 - Typical command to run a container interactively (`-it`) and delete on exit(`--rm`): `docker run --rm -it name:tag`
 
[Install Docker](https://docs.docker.com/engine/install/ubuntu/):

```
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release


sudo mkdir -p /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
 
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
sudo apt-get update
  
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

Create a Dockerfile (see the content of the Dockerfile below):

`nano Dockerfile`

Create an image from the Dockerfile:

`sudo docker build -t dependencies .`

Launch the container from image:

`sudo docker run -it dependencies bash`

*Warning! Execute `. /.bashrc` command right after the container is run for correct work of all aliases.*

**Dockerfile before linter**

```
FROM ubuntu:latest
MAINTAINER "snitkin.d@list.ru"
# apt-transport-https & wget
RUN apt update && \
    apt -y install apt-transport-https wget
# Unzip
RUN apt install unzip
# Java
RUN apt -y install openjdk-11-jdk xvfb
# Perl
RUN apt -y install perl
# Create /.bashrc for aliases
# Execute ". /.bashrc" command right after the container is run
RUN touch /.bashrc
# FastQC v0.11.9
RUN wget https://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.11.9.zip && \
    unzip fastqc_v0.11.9.zip && \
    rm fastqc_v0.11.9.zip && \
    chmod a+x /FastQC/fastqc && \
    echo 'alias fastqc="/FastQC/fastqc"' >> /.bashrc # && \
# STAR v2.7.10b
RUN wget https://github.com/alexdobin/STAR/releases/download/2.7.10b/STAR_2.7.10b.zip && \
    unzip ./STAR_2.7.10b.zip && \
    rm ./STAR_2.7.10b.zip && \
    chmod a+x ./STAR_2.7.10b/Linux_x86_64_static/STAR ?? \
    mv ./STAR_2.7.10b/Linux_x86_64_static/STAR /bin/STAR && \
    rm -r ./STAR_2.7.10b
# samtools v1.16.1
RUN wget https://github.com/samtools/samtools/archive/refs/tags/1.16.1.zip -O ./samtools-1.16.1.zip && \
    unzip ./samtools-1.16.1.zip && \
    rm ./samtools-1.16.1.zip && \
    mv ./samtools-1.16.1/misc /samtools && \
    rm -r ./samtools-1.16.1 && \
    echo 'alias samtools="/samtools/samtools.pl"' >> /.bashrc
# picard v2.27.5
RUN wget https://github.com/broadinstitute/picard/releases/download/2.27.5/picard.jar -O /bin/picard.jar && \
    chmod a+x /bin/picard.jar && \
    echo 'alias picard="java -jar /bin/picard.jar"' >> /.bashrc
# bedrools v2.30.0
RUN wget https://github.com/arq5x/bedtools2/releases/download/v2.30.0/bedtools.static.binary -O /bin/bedtools.static.binary && \
    chmod a+x /bin/bedtools.static.binary && \
    echo 'alias bedtools="/bin/bedtools.static.binary"' >> /.bashrc
# Python 3 with pip
RUN apt -y install python3-pip
# MultiQC v1.13
RUN pip install multiqc==1.13
# salmon v1.9.0 with libgomp1 and libtbb12 needed for its functioning
RUN wget https://github.com/COMBINE-lab/salmon/releases/download/v1.9.0/salmon-1.9.0_linux_x86_64.tar.gz && \
    tar -zxvf ./salmon-1.9.0_linux_x86_64.tar.gz && \
    rm salmon-1.9.0_linux_x86_64.tar.gz && \
    chmod a+x ./salmon-1.9.0_linux_x86_64/bin/salmon && \
    mv ./salmon-1.9.0_linux_x86_64/bin/salmon /bin/salmon && \
    rm -r ./salmon-1.9.0_linux_x86_64 && \
    apt install libgomp1 libtbb12
```

* [1] Use [hadolint](https://hadolint.github.io/hadolint/) and remove as many reported warnings as possible.

The list of changes:

1. Specified the version of Ubuntu.
2. MAINTAINER is deprecated. Replaced with the `LABEL author`.
3. Replaced `apt` with `apt-get` because `apt` is meant to be an end-user tool.
4. Added versions to all the installed packages.
5. Added `--no-install-recommends` to exclude all the unnecessary packages.

* [0.5] Add relevant [labels](https://docs.docker.com/engine/reference/builder/#label), e.g. maintainer, version, etc. ([hint](https://medium.com/@chamilad/lets-make-your-docker-image-better-than-90-of-existing-ones-8b1e5de950d))

Added version and description labels.

**Dockerfile after linter**

*Warning! Execute `. /.bashrc` command right after the container is run for correct work of all aliases.*

```
FROM ubuntu:22.04
LABEL author="snitkin.d@list.ru"
LABEL version="1.0"
LABEL description="Container for the homework 1 in Computing Infrastructure in Bioinformatics Problems, HSE, 2022"
# Apt packages installation
RUN apt-get update && \
    # apt-utils
    apt-get -y --no-install-recommends install apt-utils=2.4.8 && \
    # apt-transport-https
    apt-get -y --no-install-recommends install apt-transport-https=2.4.8 && \
    # Wget
    apt-get -y --no-install-recommends install wget=1.21.2-2ubuntu1 && \
    # Unzip
    apt-get -y --no-install-recommends install unzip=6.0-26ubuntu3.1 && \
    # Python 3 with pip
    apt-get -y --no-install-recommends install python3-pip=22.0.2+dfsg-1 && \
    # Java
    apt-get -y --no-install-recommends install openjdk-11-jdk=11.0.17+8-1ubuntu2~22.04 && \
    apt-get -y --no-install-recommends install xvfb=2:21.1.3-2ubuntu2.3 && \
    # Perl
    apt-get -y --no-install-recommends install perl=5.34.0-3ubuntu1.1 && \
    # libgomp1 & libtbb12 (needed for salmon)
    apt-get -y --no-install-recommends install libgomp1=12.1.0-2ubuntu1~22.04 && \
    apt-get -y --no-install-recommends install libtbb12=2021.5.0-7ubuntu2 && \
    # Delete the apt-get lists
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
# Create /.bashrc for aliases
# Execute ". /.bashrc" command right after the container is run
RUN touch /.bashrc
# FastQC v0.11.9
RUN wget https://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.11.9.zip && \
    unzip fastqc_v0.11.9.zip && \
    rm fastqc_v0.11.9.zip && \
    chmod a+x /FastQC/fastqc && \
    echo 'alias fastqc="/FastQC/fastqc"' >> /.bashrc # && \
# STAR v2.7.10b
RUN wget https://github.com/alexdobin/STAR/releases/download/2.7.10b/STAR_2.7.10b.zip && \
    unzip ./STAR_2.7.10b.zip && \
    rm ./STAR_2.7.10b.zip && \
    chmod a+x ./STAR_2.7.10b/Linux_x86_64_static/STAR ?? \
    mv ./STAR_2.7.10b/Linux_x86_64_static/STAR /bin/STAR && \
    rm -r ./STAR_2.7.10b
# samtools v1.16.1
RUN wget https://github.com/samtools/samtools/archive/refs/tags/1.16.1.zip -O ./samtools-1.16.1.zip && \
    unzip ./samtools-1.16.1.zip && \
    rm ./samtools-1.16.1.zip && \
    mv ./samtools-1.16.1/misc /samtools && \
    rm -r ./samtools-1.16.1 && \
    echo 'alias samtools="/samtools/samtools.pl"' >> /.bashrc
# picard v2.27.5
RUN wget https://github.com/broadinstitute/picard/releases/download/2.27.5/picard.jar -O /bin/picard.jar && \
    chmod a+x /bin/picard.jar && \
    echo 'alias picard="java -jar /bin/picard.jar"' >> /.bashrc
# bedrools v2.30.0
RUN wget https://github.com/arq5x/bedtools2/releases/download/v2.30.0/bedtools.static.binary -O /bin/bedtools.static.binary && \
    chmod a+x /bin/bedtools.static.binary && \
    echo 'alias bedtools="/bin/bedtools.static.binary"' >> /.bashrc
# MultiQC v1.13
RUN pip install multiqc==1.13
# salmon v1.9.0
RUN wget https://github.com/COMBINE-lab/salmon/releases/download/v1.9.0/salmon-1.9.0_linux_x86_64.tar.gz && \
    tar -zxvf ./salmon-1.9.0_linux_x86_64.tar.gz && \
    rm salmon-1.9.0_linux_x86_64.tar.gz && \
    chmod a+x ./salmon-1.9.0_linux_x86_64/bin/salmon && \
    mv ./salmon-1.9.0_linux_x86_64/bin/salmon /bin/salmon && \
    rm -r ./salmon-1.9.0_linux_x86_64
```

## Extra points [1.5]

You will be awarded extra points for the following:
* [0.5] Using [multi-stage builds](https://docs.docker.com/build/building/multi-stage/) in Docker. E.g. to build reat and copy only the executable to the final image.

* [0.75] Minimizing the size of the final Docker image. That is, removing all intermediates, unnecessary binaries/caches, etc. Don't forget to compare & report the final size before and after all the optimizations.

* [0.25] Create an extra Dockerfile that starts from [a conda base image](https://hub.docker.com/r/continuumio/anaconda3) and builds everything from your conda environment file. 

Hint: `conda env create --quiet -f environment.yml && conda clean -a` ([example](https://github.com/nf-core/clipseq/blob/master/Dockerfile))


**Dockerfile with Miniconda and created environment**

environment.yml file is created while building the docker image and removed in the end. Miniconda is chosen to minimize the size of the image. I tried to create containers with and without `conda clean -a`.


```
FROM ubuntu:22.04
LABEL author="snitkin.d@list.ru"
LABEL version="1.0"
LABEL description="Container with Miniconda for the homework 1 in Computing Infrastructure in Bioinformatics Problems, HSE, 2022"
RUN apt-get update && \
    # Install apt-utils
    apt-get -y --no-install-recommends install apt-utils=2.4.8 && \
    # Install wget
    apt-get -y --no-install-recommends install wget=1.21.2-2ubuntu1 && \
    # Delete the apt-get lists
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
# Creating environment.yml for Conda
RUN printf "name: dependencies \n\
channels: \n\
  - conda-forge \n\
  - bioconda \n\
  - defaults \n\
dependencies: \n\
  - samtools=1.16.1 \n\
  - salmon=1.9.0 \n\
  - multiqc=1.13 \n\
  - fastqc=0.11.9 \n\
  - star=2.7.10b \n\
  - bedtools=2.30.0 \n\
prefix: /home/snitkin/anaconda3/envs/dependencies" >> environment.yml
# Install miniconda
ENV CONDA_DIR /opt/conda
RUN wget --quiet --no-check-certificate https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda.sh && \
    /bin/bash ~/miniconda.sh -b -p /opt/conda
# Put conda in path so we can use conda activate
ENV PATH=$CONDA_DIR/bin:$PATH
# Create the environment and clean unnecessary packages
RUN conda env create -n dependencies --file environment.yml && \
    conda clean -a
# Create conda environment and remove environment.yml file
RUN rm environment.yml && \
    rm ~/miniconda.sh
```

Let's compare our images by size of the containers created:

|Docker image|Container size|
| ----------- | ----------- |
|before linter|1.9GB|
|after linter|1.52GB|
|with Miniconda|3.07GB|
|with Miniconda (`conda clean -a`)|1.91GB|

When I created the Dockerfile before linter I already tried to delete all unnecessary files. But adding the `--no-install-recommends` flag helped to reduce total size even more - by 0.38GB. I also aggregated the installation of all the apt packages and after it deleted the apt-get lists.

No doubt, Miniconda itself takes up quite a lot of memory, especially without using `conda clean -a`.
