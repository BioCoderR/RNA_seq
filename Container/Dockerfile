########################################################
# Dockerfile to build RNA Seq Pipeline container images
# Based on Ubuntu 16.04
#######################################################

#Set the base image to Ubuntu
FROM ubuntu:16.04

# File Author / Maintainer
MAINTAINER Pranav Swaroop Gundla <pranav@plantik.bio>

# Install OpenJDK 8 JRE
RUN apt-get update && apt-get install --yes \
    openjdk-8-jre \
    perl \
    unzip

# Download FastQC
ADD http://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.11.5.zip /tmp/

# Install FastQC
RUN cd /usr/local && \
    unzip /tmp/fastqc_*.zip && \
    chmod 755 /usr/local/FastQC/fastqc && \
    ln -s /usr/local/FastQC/fastqc /usr/local/bin/fastqc && \
    rm -rf /tmp/fastqc_*.zip

ENTRYPOINT ["fastqc"]

# UMI-tools
ARG PACKAGE_VERSION=0.5.3
ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install --yes python-pip && \
    pip install --upgrade pip && \
    pip install umi_tools==$PACKAGE_VERSION && \
    apt clean && \
    rm -rf /var/lib/apt/lists/*
    
ARG DEBIAN_FRONTEND=noninteractive
WORKDIR /usr/local

# Update the repository sources list
RUN apt-get update && \
    apt-get -y install wget python gzip unzip && \
    wget https://github.com/ComputationalSystemsBiology/EoulsanDockerFiles/raw/master/TrimAdapt/cutadapt-1.8.1.tar.gz && \
    tar -xzf cutadapt-1.8.1.tar.gz && \
    ln -s /usr/local/cutadapt-1.8.1 /tmp/ && \
    rm cutadapt-1.8.1.tar.gz && \
    wget https://github.com/ComputationalSystemsBiology/EoulsanDockerFiles/raw/master/TrimAdapt/trim_galore_v0.4.1.zip && \
    unzip trim_galore_v0.4.1.zip -d . && \
    rm trim_galore_v0.4.1.zip && \
    ln -s /usr/local/trim_galore_zip/trim_galore /usr/local/bin/ && \
    touch /usr/local/bin/rename_trim_galore_output.sh && \
    ln -s /usr/local/bin/rename_trim_galore_output.sh /RenameOutput.sh && \
    chmod +x /usr/local/bin/rename_trim_galore_output.sh && \
    printf '#/bin/bash\nb=$(basename "$2")\ne=${b#*.}\nd=$(dirname "$2")\nmv "$d/$(basename "$b" .$e)_$1.fq" "$3"\n' >> /usr/local/bin/rename_trim_galore_output.sh

WORKDIR /

# Cutadapt
ARG PACKAGE_VERSION=2.10
ARG BUILD_PACKAGES="build-essential"
ARG DEBIAN_FRONTEND=noninteractive

RUN apt update && \
    apt install --yes $BUILD_PACKAGES && \
    apt install --yes python3-pip \
                      libpython3-dev && \
    pip3 install "cutadapt==$PACKAGE_VERSION" && \
    apt autoremove --purge --yes && \
    apt clean && \
    rm -rf /var/lib/apt/lists/*  
    
# Bowtie 2
# Update the repository sources list
RUN apt-get update
# Install compiler and perl stuff
RUN apt-get install --yes build-essential gcc-multilib apt-utils zlib1g-dev git
# Install STAR
WORKDIR /tmp
RUN git clone https://github.com/BenLangmead/bowtie2.git
WORKDIR /tmp/bowtie2
RUN git checkout v2.2.4
# Patch Makefile
RUN sed -i 's/ifneq (,$(findstring 13,$(shell uname -r)))/ifneq (,$(findstring Darwin 13,$(shell uname -sr)))/' Makefile
RUN sed -i 's/RELEASE_FLAGS = -O3 -m64 $(SSE_FLAG) -funroll-loops -g3/RELEASE_FLAGS = -O3 -m64 $(SSE_FLAG) -funroll-loops -g3 -static/' Makefile
# Compile
RUN make
RUN cp -p bowtie2 bowtie2-* /usr/local/bin
# Cleanup
RUN rm -rf /tmp/bowtie2
RUN apt-get clean
RUN apt-get remove --yes --purge build-essential gcc-multilib apt-utils zlib1g-dev vim git
WORKDIR /root

# bwa
# Update the repository sources list
RUN apt-get update

# Install compiler and perl stuff
RUN apt-get install --yes build-essential gcc-multilib apt-utils zlib1g-dev wget

# Get source code from git
RUN apt-get install --yes git
WORKDIR /tmp
RUN git clone https://github.com/lh3/bwa.git
WORKDIR /tmp/bwa
RUN git checkout v0.7.15

# Compile
RUN make
RUN cp -p bwa /usr/local/bin

# Cleanup
RUN rm -rf /tmp/bwa
RUN apt-get clean
RUN apt-get remove --yes --purge build-essential gcc-multilib apt-utils zlib1g-dev wget


# htseq
ARG PACKAGE_VERSION=0.11.0
ARG DEBIAN_FRONTEND=noninteractive
ARG BUILD_PACKAGES=""


RUN apt-get update && \
    apt-get install --yes \
              $BUILD_PACKAGES \
              python-pip && \
    pip install htseq==$PACKAGE_VERSION && \
    apt clean && \
    rm -rf /var/lib/apt/lists/*

#    pip install --upgrade pip && \

# STAR
# Install compiler and perl stuff
RUN apt-get install --yes \
 build-essential \
 gcc-multilib \
 apt-utils \
 zlib1g-dev

# Install git
RUN apt-get install --yes git

# Install STAR
WORKDIR /usr/local/
RUN pwd
RUN git clone https://github.com/alexdobin/STAR.git
WORKDIR /usr/local/STAR/
RUN pwd
RUN git checkout 2.5.2b 
WORKDIR /usr/local/STAR/source 
RUN pwd
RUN make STAR
ENV PATH /usr/local/STAR/source:$PATH

# tophat 2
# Update the repository sources list, install wget, unzip, python
RUN apt-get update && apt-get install --yes wget unzip python
# Working directory in /bin
WORKDIR /bin
# Download TopHat2
RUN wget http://ccb.jhu.edu/software/tophat/downloads/tophat-2.0.14.Linux_x86_64.tar.gz
# Download Bowtie2
RUN wget --default-page=bowtie2-2.2.3-linux-x86_64.zip http://sourceforge.net/projects/bowtie-bio/files/bowtie2/2.2.3/bowtie2-2.2.3-linux-x86_64.zip/
# Unzip the archive
RUN tar zxvf tophat-2.0.14.Linux_x86_64.tar.gz
# Remove the archive
RUN rm tophat-2.0.14.Linux_x86_64.tar.gz
# Unzip the archive
RUN unzip bowtie2-2.2.3-linux-x86_64.zip
# Remove the archive
RUN rm bowtie2-2.2.3-linux-x86_64.zip
# Working directory in Bowtie2
WORKDIR /bin/bowtie2-2.2.3
# Symbolic link from "bowtie" to "bowtie2"
RUN ln -s bowtie2 bowtie
# Change in PATH
ENV PATH $PATH:/bin/tophat-2.0.14.Linux_x86_64
ENV PATH $PATH:/bin/bowtie2-2.2.3
# Remove wget and unzip
RUN apt-get purge --yes wget unzip
# Working directory 
WORKDIR /
# HISAT
# Update the repository sources list
RUN apt-get update

# Install compiler and perl stuff
RUN apt-get install --yes build-essential git

# Install HISAT
WORKDIR /tmp
RUN git clone https://github.com/infphilo/hisat.git
WORKDIR /tmp/hisat
RUN git checkout master

# Compile
RUN make
RUN cp -p hisat hisat-* /usr/local/bin

# Cleanup
RUN rm -rf /tmp/hisat
RUN apt-get clean
RUN apt-get remove --yes --purge build-essential gcc-multilib apt-utils zlib1g-dev vim git

# Set default working path
WORKDIR /root

# Default command to execute at startup of the container
CMD hisat

# Cufflinks
FROM biocontainers/biocontainers:latest

LABEL base.image="biocontainers:latest"
LABEL version="1"
LABEL software="Cufflinks"
LABEL software.version="2.2.1"
LABEL description="Transcriptome assembly and differential expression analysis for RNA-Seq"
LABEL website="http://cole-trapnell-lab.github.io/cufflinks/"
LABEL documentation="http://cole-trapnell-lab.github.io/cufflinks/manual/"
LABEL license="https://github.com/cole-trapnell-lab/cufflinks/blob/master/LICENSE"
LABEL tags="Genomics"

RUN conda install cufflinks

USER root

# bed tools
ARG PACKAGE_VERSION=2.27.1
ARG BUILD_PACKAGES="git openssl python build-essential zlib1g-dev"
ARG DEBIAN_FRONTEND=noninteractive

# Update the repository sources list
RUN apt-get update && \
    apt-get install --yes \
              $BUILD_PACKAGES && \
    cd /tmp && \
    git clone https://github.com/arq5x/bedtools2.git && \
    cd bedtools2 && \
    git checkout v$PACKAGE_VERSION && \
    make && \
    mv bin/* /usr/local/bin && \
    cd / && \
    rm -rf /tmp/* && \
    apt remove --purge --yes \
              $BUILD_PACKAGES && \
    apt autoremove --purge --yes && \
    apt clean && \
    rm -rf /var/lib/apt/lists/*
   
# samtools    
# Update the repository sources list and install samtools package
RUN apt update && \
    apt install --yes wget libcurl3-gnutls && \
    wget http://mirrors.kernel.org/ubuntu/pool/universe/s/samtools/samtools_1.4.1-1build1_amd64.deb && \
    wget http://mirrors.kernel.org/ubuntu/pool/universe/h/htslib/libhts2_1.5-1_amd64.deb && \
    dpkg -i samtools_*.deb libhts2_*.deb && \
    rm *.deb && \
    apt clean
   
# install CRAN dependancies

RUN apt-get install -y \
 r-cran-bitops \
 r-cran-xml \
 r-cran-dbi \
 r-cran-rcurl \
 r-cran-rsqlite
    
# DESeq
# Set the base image to Ubuntu
FROM r-base:3.2.3


# Default command to execute at startup of the container
CMD R --no-save

# Update the repository sources list
RUN apt-get update
RUN apt-get install -y \
 apt-utils

# Add CRAN source to apt
RUN echo "deb http://cran.r-project.org/bin/linux/ubuntu trusty/" > /etc/apt/sources.list.d/cran.list

# install CRAN dependancies

RUN apt-get install -y \
 r-cran-bitops \
 r-cran-xml \
 r-cran-dbi \
 r-cran-rcurl \
 r-cran-rsqlite
    


# Install DESeq2 and FactoMineR package  
RUN R -e 'BiocManager::install("DESeq2")' && R -e 'BiocManager::install("FactoMineR")' && R -e 'devtools::install_github("kassambara/factoextra")'

# Cleanup
RUN apt-get clean
















