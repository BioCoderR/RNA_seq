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

# Update the repository sources list and install the essential packages
RUN apt-get update && \
    apt-get install --yes python-pip && \
    pip install --upgrade pip && \
    pip install umi_tools==$PACKAGE_VERSION && \
    rm -rf /var/lib/apt/lists/* 
    
# Trim Galore!
WORKDIR /usr/local
# Update the repository sources list
RUN apt-get update && \
    apt-get install -y wget && \
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

# Cutadapt
ARG PACKAGE_VERSION=2.10
ARG BUILD_PACKAGES="build-essential"
ARG DEBIAN_FRONTEND=noninteractive
# Update the repository sources list
RUN apt update && \
    apt install --yes $BUILD_PACKAGES && \
    apt install --yes python3-pip \
                      libpython3-dev && \
    pip3 install "cutadapt==$PACKAGE_VERSION" && \
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

# bwa
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

# htseq
ARG PACKAGE_VERSION=0.11.0
ARG DEBIAN_FRONTEND=noninteractive
ARG BUILD_PACKAGES=""
# Update the repository sources list
RUN apt-get update && \
    apt-get install --yes \
              $BUILD_PACKAGES \
              python-pip && \
    pip install htseq==$PACKAGE_VERSION && \
    rm -rf /var/lib/apt/lists/*

#    pip install --upgrade pip && \

# STAR
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

# HISAT
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
    rm -rf /var/lib/apt/lists/*
    
# SAM Tools   
# set the environment variables
ENV samtools_version 1.10
ENV bcftools_version 1.10
ENV htslib_version 1.10

# run update and install necessary packages
RUN apt-get update -y && apt-get install -y \
    bzip2 \
    build-essential \
    zlib1g-dev \
    libncurses5-dev \
    libncursesw5-dev \
    libnss-sss \
    libbz2-dev \
    liblzma-dev \
    vim \
    less \
    libcurl4-openssl-dev \
    wget

# download the suite of tools
WORKDIR /usr/local/bin/
RUN wget https://github.com/samtools/samtools/releases/download/${samtools_version}/samtools-${samtools_version}.tar.bz2
RUN wget https://github.com/samtools/bcftools/releases/download/${bcftools_version}/bcftools-${bcftools_version}.tar.bz2
RUN wget https://github.com/samtools/htslib/releases/download/${htslib_version}/htslib-${htslib_version}.tar.bz2

# extract files for the suite of tools
RUN tar -xjf /usr/local/bin/samtools-${samtools_version}.tar.bz2 -C /usr/local/bin/
RUN tar -xjf /usr/local/bin/bcftools-${bcftools_version}.tar.bz2 -C /usr/local/bin/
RUN tar -xjf /usr/local/bin/htslib-${htslib_version}.tar.bz2 -C /usr/local/bin/

# run make on the source
RUN cd /usr/local/bin/htslib-${htslib_version}/ && ./configure
RUN cd /usr/local/bin/htslib-${htslib_version}/ && make
RUN cd /usr/local/bin/htslib-${htslib_version}/ && make install

RUN cd /usr/local/bin/samtools-${samtools_version}/ && ./configure
RUN cd /usr/local/bin/samtools-${samtools_version}/ && make
RUN cd /usr/local/bin/samtools-${samtools_version}/ && make install

RUN cd /usr/local/bin/bcftools-${bcftools_version}/ && make
RUN cd /usr/local/bin/bcftools-${bcftools_version}/ && make install
# Working directory default
WORKDIR /
