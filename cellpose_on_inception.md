# data segmentation and processing tutorial

## A quick introduction to the server infrastructure at Pasteur

Pasteur has several services to store experimental data. They have recently migrated to `Maestro` and so normally your images you want to segment are stored on the server.

The segmentation of the images is currently done using Cellpose. This program consumes a lot of memory and requires GPUs, segmenting directly on your personal computer can be very time-consuming. A second set of servers, named [Inception](https://gitlab.pasteur.fr/inception-gpulab/wiki), can be used to do the segmentation. Cellpose is installed on Inception.

## Connecting to `Inception`

To be able to connect to `Inception`, you first need to program an `Inception` account. Write to **Tru Huyn** for more information.

When that is done, in the terminal, type:

```
ssh $PASTEURNAME@adm.inception.hubbioit.pasteur.fr
```

(My $PASTEURNAME is gronteix for example)

You can now check what is on your `Inception` folders, create new folders, and move files around.

## Moving your data from `Maestro` to `Inception`

To move the data, open a terminal window, and connect to `tars`:

```
ssh $PASTEURNAME@tars.pasteur.fr
```

Now that you are connected, you can move the data from `Maestro` to `Inception`:

```
scp -r $PATH_ON_MAESTRO adm.inception.hubbioit.pasteur.fr:$PATH_ON_INCEPTION
```

Otherwise, maestro storage space can be mounted on gpulab via sshfs. 
```
sshfs $PASTEURNAME@maestro.pasteur.fr:/pasteur/zeus/projets/p02/Anchor/ ~/Anchor
```
The only problem is that this mount point won't be visible across the cluster nodes and sshfs command should be rerun on the computing node. 

```
ssh gpulab01
sshfs $PASTEURNAME@maestro.pasteur.fr:/pasteur/zeus/projets/p02/Anchor/ ~/Anchor
```

To avoid entering the passwords averytime, run `ssh-copy-id $PASTEURNAME@maestro.pasteur.fr`

## Creating a Cellpose container on `Inception`

Cellpose needs to run within a specific environment. All Python libraries can't be installed on the `Inception` server, and so before running a specific container needs to be launched. Your code will be executed within this environment, where all the appropriate packages should be available.

The isies way to get singularity container is to use Docker container. Simply locate the pull address on dockerhub and run

```
singularity pull docker://aaristov85/spheroidgraphs:tagname spheroid-graph.sif
```

`Inception` uses singularity to run the containers. If you have any issues contact Andrey Aristov or Tru Huyn for more information.

First make sure you have a container on your account. The directory architecture should be:

```
.
├── containers                    # Container folder
│   └── container file            # Singularity file
└── your files
```

To launch the Cellpose container, in the open `Inception` terminal, type:

```
srun --pty --gres=gpu:1 singularity shell --nv containers/c7-conda-cellpose-gpu-cu101mkl-2020-09-03-1149.sif
```

This command opens the container that is in the `containers` folder on the home repository on your space in `Inception`. You can now start segmenting the images.

## Segmenting the images using Cellpose

Within you singularity terminal you can use Cellpose to segment images. Normally all your files should be available in the `Inception` server. Type in:

```
python -m cellpose --dir=$PATH_TO_IMAGES --pretrained_model nuclei --diameter 0. --use_gpu --do_3D --fast_mode
```

The exact options are explained in the [Cellpose documentation](https://cellpose.readthedocs.io/en/latest/). Go have look!

The segmented images should be stored on some location on the `Inception` server. TO move them to `Maestro`, you can copy the steps in the "Moving your data" section.

# Using `Griottes` on `Maestro`

Sometimes images can be too heavy to be opened immediately on you own computer. This can be solved by conducting the heavier operations online.

## Connecting to Jupyter on Maestro

First connect to `Maestro`:

```
ssh $PASTEURNAME@maestro.pasteur.fr
```
Then on the server open the singularity container:
```
srun --mem=100G singularity run -B /pasteur,$MYSCRATCH/.cache:/home/jovyan/.cache spheroids-graphs.sif
```
This will open a local environment with Jupyter Notebook and the `Griottes` package installed. If you need extra packages, you can update by creating a new singularity container and moving it to your `Maestro` homepage. Ask Andrey Aristov if you have any issues with this.

Now within the singularity container, you can launch the Jupyter Notebook:

```
jupyter notebook
```

On your computer, open a new terminal, and establish a ssh tunnel to `Maestro`:

```
ssh -NL 8888:localhost:8888 $PASTEURNAME@maestro-$SERVERNUMBER.maestro.pasteur.fr
```

The $SERVERNUMBER is visible in the text below the singularity command. It should be a number between 1000 and 1050. If the command is successful no text appears and you can copy-paste the URL below the singularity command to your browser.

Make sure no other jupyter notebook windows are already open on your computer as they can compete for the same port.

