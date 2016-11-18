# *TerraFERMA* Docker

TerraFERMA is the *Transparent Finite Element Rapid Model Assembler*, a software system for the  rapid and reproducible construction and exploration of  coupled  multi-physics models.  

This repository contains Dockerfiles for building Docker images with TerraFERMA and its dependencies installed.

To install Docker please follow the instructions at
[docker.com](https://docs.docker.com/engine/getstarted/step_one/).

Once Docker is installed the images can be run using:

    docker run -ti terraferma/dev:latest

If you want to share your current working directory with the Docker container
use the following command:

    docker run -ti -v $(pwd):/home/tfuser/shared terraferma/dev:latest

Additional arguments are necessary to open the TerraFERMA GUI, diamond, within Docker.  
From linux operating systems use:

    docker run -ti -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix terraferma/dev:latest

Above we use the `dev:latest` image and tag, which supplies the most complete and up to date installation but we supply a range of
Docker images:

 * `dev` - a full installation of TerraFERMA and its dependencies
 * `dev-env` - a development environment with all dependencies installed but no pre-installed TerraFERMA
 * `base` - a simple image setting up the user and basic environment

## Acknowledgements

Our Docker images and build process is largely based off that used by [FEniCS](https://bitbucket.org/fenics-project/docker).

