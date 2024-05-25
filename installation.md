# Installation

## Install on Windows

### Using installer

1. Download the latest `.msi` file from [Github](https://github.com/SMTP2Graph/SMTP2Graph/releases)
2. Run the setup
3. Create a file called `config.yml` with a [minimal config](config.md#minimal) in `C:\ProgramData\SMTP2Graph`
4. Start the SMTP2Graph service

### Manual

1. Download the latest release from [Github](https://github.com/SMTP2Graph/SMTP2Graph/releases)
2. Put the downloaded exe in a folder where you have write access
3. Create a file called `config.yml` next to the exe with a [minimal config](config.md#minimal)
4. Now you can launch the file

## Install on Linux

1. Download the latest release from [Github](https://github.com/SMTP2Graph/SMTP2Graph/releases)
2. Put the downloaded binary in a folder where you have write access
    - Also make sure you have execute permissions on the binary
3. Create a file called `config.yml` next to the binary with a [minimal config](config.md#minimal)
4. Now you can launch the file

## Run with Docker

1. Create a folder and put an SMTP2Graph config in it
    - SMTP2Graph will also store logs and the message queue in this folder
2. Mount your folder to `/data` in the container and expose port 587
3. Run the image (example: `docker run -p 587:587 -v ./smtp2graph:/data smtp2graph/smtp2graph:latest`)

[![Docker version](https://img.shields.io/docker/v/smtp2graph/smtp2graph?style=for-the-badge&logo=docker)](https://hub.docker.com/r/smtp2graph/smtp2graph)
