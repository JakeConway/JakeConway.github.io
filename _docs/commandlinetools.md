---
title: Command line tools (Docker)
permalink: /docs/commandlinetools/
---
For the tools that run on the command line, I've set up dockers with all of the requirements installed. **One thing of note is that due to the growing size and quantity of data being generated, most of our work involving computational methods for data generation is typically performed on the cloud. However, when performing a deep dive into one or a few samples, it is feasible run these methods locally depending on the size of the input files.** This will be the case with the examples listed throughout the documentation.

## Installing Docker
Docker is available for Mac, Windows, and Linux distributions. Instructions for installing Docker can be found on the [Docker website](https://docs.docker.com/get-docker/).

## Downloading Docker images
Docker Hub is the most popular repository service (similar to GitHub) for hosting and sharing Docker images. Once Docker is installed, Docker images can be downloaded from Docker Hub using the Docker **pull** command. As an example, we're going to pull the GATK3.7 image from the [Broad Institue's Docker Hub page](https://hub.docker.com/u/broadinstitute). The code below can be interpreted as "pull the `3.7-0` version of the Docker image from the `broadinstitute/gatk3` repository".

```
docker pull broadinstitute/gatk3:3.7-0
```

## Running Docker containers
To run a Docker container (or virtualized run-time environment) based off a Docker image, we first need to obtain the Image ID. To see a list of images in your local Docker environment and their attributes (including the Image ID), the following command line code can be used:
```
docker images
```

To run a container based off the `broadinstitute/gatk3:3.7-0` image, copy and paste the Image ID output by `docker images` into the following line of code:
```
docker run -it $image_id /bin/bash
```

After running the line above, you will now be in the virtual run-time environment that contains GATK3.7 and all required dependencies. To check this, we can run the following line of code to get the help documentation for ContEst, a GATK tool used to determine cross-individual contamination of DNA samples.
```
java -jar /usr/local/bin/GenomeAnalysisTK.jar -T ContEst -h
```
To copy files from your local computer into a docker container, we first need to get the Container ID. To get the Container ID we can use the following line of code:
```
docker ps
```
Once you've obtained the Container ID, the following line of code can be used to copy a file from your local computer into the Docker container:
```
docker cp /local_path/file.txt $container_id:/path_in_container/file.txt
```
