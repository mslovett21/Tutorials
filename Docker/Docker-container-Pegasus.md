# Docker for ML Workflows
##  Components of Dockerfile

###  Base Image
Choose base image that has the required CUDA and Pytorch versions (or Tensorflow)
Try using official docker images. 
```
FROM pytorch/pytorch:1.6.0-cuda10.1-cudnn7-runtime
```

### OS Update
Update the OS and install any software needed via package manager.
```
RUN apt-get update
RUN apt-get install ffmpeg libsm6 libxext6 -y
```

## Create directory for your application
```
RUN mkdir app
WORKDIR /app
```
### Python packages
```
RUN python3 -m pip install --upgrade pip
RUN pip3 install \
    opencv-python \
    optuna==2.0.0 \
    pandas \
    matplotlib \
    torch \
    numpy \
    Pillow \
    bs4 \
    scikit-learn \
    torchvision \
    pytorchtools \
    joblib\
    scikit-image

```
### Copy over the executable scripts
Create the bin directory in the docker and copy the transformation scripts there
```
RUN mkdir ./bin
COPY data_aug.py ./bin
RUN chmod 777 ./bin/data_aug.py
COPY plot_class_distribution.py ./bin
RUN chmod 777 ./bin/plot_class_distribution.py
COPY plot_images.py ./bin
RUN chmod 777 ./bin/plot_images.py
COPY rename_file.py ./bin
RUN chmod 777 ./bin/rename_file.py
COPY train_model.py ./bin
RUN chmod 777 ./bin/train_model.py
COPY hpo_train.py ./bin
RUN chmod 777 ./bin/hpo_train.py
```


### Build the Image

```
DOCKER_BUILDKIT=1 docker build . -t test
```

### Tag the Image

```
docker tag test patkraw/mask-detecton-wf:latest
```

### Push the docker image to DockerHub

```
docker push patkraw/mask-detecton-wf:latest
```


### Add Container to the transformation catalog

```
# Container for all the jobs
tc = TransformationCatalog()
mask_detection_wf_cont = Container(
                "mask_detection_wf",
                Container.DOCKER,
                image="docker://patkraw/mask-detecton-wf:latest",
                arguments="--runtime=nvidia --shm-size=15gb"
            )

tc.add_containers(mask_detection_wf_cont)
```

### Set attributes of the transformations

```
dist_plot = Transformation(
                "dist_plot",
                site = "condorpool",
                pfn = os.path.join(os.getcwd(),"bin/plot_class_distribution.py"),
                is_stageable = False,
                container = mask_detection_wf_cont 
            )

```

### Execute the scripts in the docker without Pegasus to make sure the Dockerfile is correct

```
docker run --rm -ti -v `pwd`:/host test /host/data_aug.py
```