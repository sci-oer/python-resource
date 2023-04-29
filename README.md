[![Deployment](https://github.com/sci-oer/MY_LANGUAGE-resource/actions/workflows/deployment.yml/badge.svg)](https://github.com/sci-oer/MY_LANGUAGE-resource/actions/workflows/deployment.yml)
![Docker Image Size (latest by date)](https://img.shields.io/docker/image-size/scioer/MY_LANGUAGE-resource?style=plastic)
![Docker Pulls](https://img.shields.io/docker/pulls/scioer/MY_LANGUAGE-resource?style=plastic)
![GitHub](https://img.shields.io/github/license/sci-oer/MY_LANGUAGE-resource?style=plastic)

----
# sci-oer Language Resource Template

This repository is meant to be used as a template to help get started to create a base image for
an sci-oer course for a new programming language.

## Usage

### Updating the README

1. Remove this block about using the template
2. Replace all occurrences of `MY_LANGUAGE` with the name of the language the resource is for in all files in this repository.
  * Note: if the github repository or Docker image is not in the `sci-oer` / `scioer` namespace then that will need to be updated as well.
3. Create github action secrets in the repository for `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN` to allow the generated docker image to be uploaded.
4. Update the Dockerfile with the language specific dependencies and jupyter labs kernel
5. Put some language specific documentation into the `./langdocs` directory

----

# sci-oer MY_LANGUAGE base-resource

This is the MY_LANGUAGE specific version of the sci-oer resource.
This extends the configuration defined in the [sci-oer/base-resource](https://github.com/sci-oer/base-resource) and adds the language specific dependencies.

## Building the container

```bash
docker build \
    --build-arg GIT_COMMIT=$(git rev-parse -q --verify HEAD) \
    --build-arg BUILD_DATE=$(date -u +"%Y-%m-%dT%H:%M:%SZ") \
    -t scioer/MY_LANGUAGE-resource:latest .
```


## Running the container

```bash
docker run --rm -it \
    -p 3000:3000 \
    -p 8888:8888 \
    -p 2222:22 \
    -p 8000:8000 \
    -v "$(pwd)/course:/course" \
    scioer/MY_LANGUAGE-resource:latest
```

This container is designed to be run in the foreground.
It will run a wiki.js server and a jupyter notebooks server in the background and provide the user with a bash shell

## Using the container


### Git
To configure git within the container this can be done manually by running the `git config` commands or by using the environment variables

`GIT_EMAIL='student@example.com'`, `GIT_NAME="My StudentName"`

These environment variables can be configured when you run the docker container

```bash
docker run -it --rm \
    -p 3000:3000 \
    -p 8888:8888 \
    -p 2222:22 \
    -p 8000:8000 \
    -e GIT_EMAIL='student@example.com' \
    -e GIT_NAME="My StudentName" \
    scioer/MY_LANGUAGE-resource:latest
```

### Wiki

The wiki can be found at http://localhost:3000

Username: admin@example.com
Password: password


### Jupyter Notebooks

The jupyter notebooks site can be found at http://localhost:8888

When you go to the jupyter notebooks page all of the builtin notebooks can be found in the `builtin` folder.
All of the notebooks that are in the `builtin` folder are provided by the container, they can be modified and the modifications will be persistent as long as the same volume mount is used.
If the volume is replaced then all the builtin notebooks will be replaced with fresh copies.
All of the notebooks that are created will be saved in the `course/jupyter/notebooks` directory.

Any user settings that are changed (such as dark mode) will also be persistent.

### Language docs

The language docs for MY_LANGUAGE have been built into this image and can be accessed at http://localhost:8000.

### ssh to work on files using external editor

This container runs an ssh daemon and exposes port 22.
You can ssh into this container by running `ssh -p 2222 student@127.0.0.1`.
You do not need a password to ssh into the container, but the password is `password` for any command that needs it.

Any files that are edited should be put in the `/course/work` directory to be saved to the volume mount.


Although you are able to ssh into this container, it is preferred to attach additional terminals to the container directly.
```bash
$ docker exec -it container_name bash
```

The name of the container can be gotten by running `docker ps` or it can be specified when the container is created by passing the `--name my_name` flag to `docker run`.

## Customization of this container

### How to configure custom wiki content

Any built in content to be included in the wiki must be added manually.
Unfortunately there is not currently an easy mechanism to automatically load markdown files from a directory into the wiki.

To Load custom content into the container the following process is suggested:

1. Start the container _with_ the volume mount `docker run -it --rm -v "$(pwd)/course:/course" scioer/MY_LANGUAGE-resource:latest`
2. Go to http://localhost:3000 and create all of the desired wiki pages and configurations
3. Exit the container
4. Replace the `database.sqlite` file with the new one from `course/wiki/database.sqlite`
5. Rebuild the container using the docker build command

Alternatively if there is a large number of wiki pages to import that already exist as markdown files an alternative process can be used.

1. Create the folder `course/wiki/files`
2. Place all of your markdown files to be imported into that folder, the files should have the following front matter so they can be imported with the desired title, tags, and if it should be published or not. Subfolder can also be used
```
---
title: Untitled Page
description:
isPublished: 1
tags: coma, separated, list
---
```
3. Start the container _with_ the volume mount `docker run -it --rm -v "$(pwd)/course:/course" scioer/MY_LANGUAGE-resources:latest`
4. Go to http://localhost:3000 and navigate to Administration > Storage > Local File System
5. Enable local file storage, set the Path to `/course/wiki/files`
6. Scroll to the bottom of the page and run `Import Everything`, now all of the wiki pages should be imported
7. Exit the container
8. Replace the `database.sqlite` file with the new one from `course/wiki/database.sqlite`
9. Rebuild the container using the docker build command

### How to configure builtin jupyter notebooks

Adding built in jupyter notebooks to the container is simpler.
Place all the desired files in the `builtinNotebooks` folder, then build the image.

### Startup message

To change the startup message that gets printed when the container starts edit the text in `motd.txt` to include the desired text

## Extending this container with custom container

This image is designed to be used as a base image for the [scioer-builder](https://pypi.org/project/scioer-builder/)
to generate a custom image for a course.

## Software License

This project is licensed under the GPLv3 license.
This is a strong copy left license that requires that any derivative work is released under the same license.
This was selected because the objective of this project is to provide a tool that can be used by others because it is something that is useful to us.
We believe that carrying that forward will be beneficial to the community.

#### wikiJS API Key:

Preset initialization API Key (valid until Jan 23, 2025):

```
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJhcGkiOjEsImdycCI6MSwiaWF0IjoxNjQyOTcyMTk5LCJleHAiOjE3Mzc2NDQ5OTksImF1ZCI6InVybjp3aWtpLmpzIiwiaXNzIjoidXJuOndpa2kuanMifQ.xkvgFfpYw2OgB0Z306YzVjOmuYzrKgt_fZLXetA0ThoAgHNH1imou2YCh-JBXSBCILbuYvfWMSwOhf5jAMKT7O1QJNMhs5W0Ls7Cj5tdlOgg-ufMZaLH8X2UQzkD-1o3Dhpv_7hs9G8xt7qlqCz_-DwroOGUGPaGW6wrtUfylUyYh86V9eJveRJqzZXiGFY3n6Z3DuzIVZtz-DoCHMaDceSG024BFOD-oexMCnAxTpk5OalEhwucaYHS2sNCLpmwiEGHSswpiaMq9-JQasVJtQ_fZ9yU_ZZLBlc0AJs1mOENDTI6OBZ3IS709byqxEwSPnWaF_Tk7fcGnCYk-3gixA
```
