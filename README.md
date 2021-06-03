# Docker

## What is it?

Docker delivers software in packages called *containers*, which can be run locally or on servers. Containers are isolated from one another and bundle their own software, libraries, and configuration files (https://docker.com). They are more robust than virtual environments, but more lightweight and efficient than virtual machines. You can read more about that [in this Medium article](https://stephen-odaibo.medium.com/docker-containers-python-virtual-environments-virtual-machines-d00aa9b8475).

## Creating an image

Containers are created from layered read-only templates called *images* by adding a read-write layer on top. There are three ways to add a new image to your machine. 
1. Build it from a *Dockerfile*.  
Dockerfiles can be created in any text editor and are pretty intuitive to write. Good starting points are the [DeepLabCut container](https://github.com/DeepLabCut/Docker4DeepLabCut2.0) and the Dockerfile in the jupyter folder here. After you've created your Dockerfile all you need to do is run `docker build -t image_name .` (note the dot in the end). This will add everything in your current directory to the container, so if you want it empty, put the Dockerfile in an empty folder.
2. Pull it from [dockerhub](https://hub.docker.com/).  
You can run `docker pull image_name` or just skip this step altogether, Docker will pull automatically when you try to run a container from an image that isn't already on the machine.
3. Create it from a container.  
It's important to remember that containers are supposed to be temporary and after a system reboot all information that only existed in containers will disappear, so taking care of your data and saving the environment regularly is good practice! In order to save a container just run `docker commit container_name image_name:tag`. You can also push it to dockerhub with `docker push image_name`. Note that when you commit a container, all processes running inside it are paused!

## Running a container

After you've got your image in order to start working you just need to run the container. The command for that is `docker run --name=container_name image_name`. Here are a few useful options:
- `-it` runs the container interactively and opens the shell,
- `--rm` destroys the container as soon as you exit,
- `-d` runs it in detached mode, without taking up the terminal,
- `-w` sets the working directory in the container. 

One option I would like to talk about in a bit more detail is `-v`. As I've already mentioned, containers are temporary and **if your data is only saved in a container it could very easily get lost**. In addition, by default containers are completely isolated from the host machine, so you can't just save important things there instead. The solution to that problem is mounting *data volumes*. That basically means synchronising a directory in the container with a directory on your computer, or, alternatively, with an abstract data space that can then be accessed by other containers. That way everything in that folder is accessible both from the container and from the host machine and nothing happens to the data if the container is removed. In order to set this up just use the `-v /host/path:/container/path` option. Note that the paths need to be absolute if you want to use a folder on your machine. Using `-v abstract:/container_path` will create a data volume named abstract that is not connected to your host system. If you want to mount multiple data volumes, just repeat the option several times: `-v /host/path/1:/container/path/1 -v /host/path/2:/container/path/2`.

When you are done working with a container, you can print `exit` to stop it or press `Ctrl+p, Ctrl+q` to quit the terminal without interrupting the process. To come back to it later, use `docker attach container_name`. If the container has been stopped, you'll need to run `docker start container_name` first. You can also run another process in an existing container with `docker exec`, for example `docker exec -it container_name bash`.

You can always see the list of all images on the machine with `docker images` and the list of all containers with `docker ps -a`. Please **don't forget to remove the containers you are not using** with `docker rm container_name`!

## Using Jupyter

In order to run a jupyter lab in your container you only really need to install jupyter and then run
- `ssh -L <host port>:localhost:<remote port> user@remote` on your computer,
- `docker run -it -p <remote port>:<container port> --name <container name>` on the server,
- `jupyter lab --ip 0.0.0.0 --port <container port> --allow-root` inside the container.

However, there are many ways to improve that process. [Here](https://u.group/thinking/how-to-put-jupyter-notebooks-in-a-dockerfile/) is a nice tutorial and 
[here](https://jupyter-docker-stacks.readthedocs.io/en/latest/) is the page of Jupyter Docker Stacks, images specifically designed with jupyter and data science in mind.  
Alternatively, you can use the Dockerfiles in the jupyter_cuda_torch and jupyter_cuda_tf folders. They are short and easy to understand and to modify even if you are very new to Docker. Building from that Dockerfile will create an image with basic data science python libraries as well as git, nano and unzip already installed and a working combination of your DL library of choice and CUDA drivers. A container run from that image will automatically start a jupyter notebook at port 8889, so you would only need to run the commands on your computer and on the server with `<container port>=8889`. 

Note that by default the image will be build with the latest versions of all libraries, but you can specify them in the Dockerfile if needed. You can create multiple images with different library versions for different purposes. Just remember that images have a layered structure that is utilized to make storage efficient. That means that if you want to create multiple images that differ by a few Dockerfile lines, those lines should be closer to the end of the file.  

One more thing to remember is that if you are planning to use GPUs, you should use `nvidia-docker` instead of regular `docker`. All the commands stay the same and you can even keep using images created in regular docker.

Here is an example use pulling everything together:
- `nvidia-docker run --name=jupyter -it -p <remote port>:8889 -v ~/data:/home -w /home -d  jupyter_base` runs a GPU-enabled container named `jupyter` on top of the `jupyter_base` image in detached mode (I am assuming the image has the instructions to start a notebook at port 8889 automatically). It connects the `~/data` folder on the host machine to the `/home` folder on the container in order to keep the important data safe and makes `/home` the working directory in the container. 
- `docker exec -it jupyter bash` opens the shell of that container without interfering with the notebook. You can run another process and type `exit` when you are done, it will not stop the notebook.
- `docker attach -it jupyter` connects you to the output of the notebook server. Here in order to quit without stopping the process you need to press `Ctrl+p, Ctrl+q`.


D. Merkel: Docker: lightweight linux containers for consistent development and deployment Linux J., 2014 (2014), p. 2
