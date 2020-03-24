# Neuroevolution of Self-Interpretable Agents

This repository contains the code to reproduce the results presented in the orignal [paper](https://attentionagent.github.io/). 

## Dependencies

* CarRacing-v0: run the command `pip3 install -r requirements.txt` and install all the required packages.
* DoomTakeCover: change `gym[box2d]==0.15.3` to `gym==0.9.4`, then run `pip3 install -r requirements.txt`.

Installing DoomTakeCover locally may be challenging due to some software incompatibility problems.  
We have prepared a docker image to save your time, run the following commands to pull and connect to the image.  
```
# Download the image.
docker image pull docker.io/braintok/self-attention-agent

# Connect to the image, you can run the training/test commands in the container now.
docker run -it braintok/self-attention-agent /bin/bash
```

## Training

### Training locally
To train on a local machine or in a local container, run the following command:
```
# Train CarRacing locally with 4 worker processes.
bash train_local.sh -c configs/CarRacing.gin -n 4

# Train DoomTakeCover locally with 4 worker processes.
bash train_local.sh -c configs/TakeCover.gin -n 4
```

You can modify the corresponding `.gin` file to change parameters.  
For instance, changing `cma.CMA.population_size = 256` to `cma.CMA.population_size = 128` in `configs/CarRacing.gin` shrinks the population size to half.

### Training on a cluster

Running the code on a distributed computing platform such as GKE can speed up the training a lot, see this [blog](https://cloud.google.com/blog/products/ai-machine-learning/how-to-run-evolution-strategies-on-google-kubernetes-engine) for more information about running evolution algorithms on GKE.  

Once you have a Kubernetes cluster ready (either locally or from a cloud service provider), you can wrap the code into an image and submit a job to the cluster.  
The following commands are only tested on GKE but should be general, steps 1 and 2 are *only* necessary for the first time or when you have made code modifications.

#### Step 1. Prepare an image.
You can use our prepared docker image `docker.io/braintok/self-attention-agent`, in this case you can skip this step.  
If you want to train CarRacing with our docker image, run the following commands first.
```
# Change gym version.
docker run -it braintok/self-attention-agent /bin/bash
pip3 uninstall gym
pip3 install gym[box2d]==0.15.3

# Press ctrl+p ctrl+q to detach from the container.

# Check the container ID.
docker container ls

# Save the container as a new image.
docker container commit {container-id-from-last-command} braintok/self-attention-agent:carracing
```
If you decide to create your own image, you can run the following commands in the repository's root directory.
```
docker build -t {image-name}:{version} .
```

#### Step 2. Upload the image to docker hub or GCP.
Run the following commands to upload your image.
```
# Tag the image, you can look up the image name and version with `docker image ls`.
docker image tag {image-name}:{version} docker.io/{your-docker-account-name}/self-attention-agent

# Push the image to a remote registry.
docker image push docker.io/{your-docker-account-name}/self-attention-agent
```

#### Step 3. Submit a job to Kubernetes.
TODO

## Evaluate pre-trained models

We have included our pre-trained models for both CarRacing and DoomTakeCover in this repository.  
You can run the following commands to evaluate the trained agent, change `pretrained/CarRacing` to `pretrained/TakeCover` to see results in DoomTakeCover.
```
# Evaluate for 100 episodes.
python3 test_solution.py --log-dir=pretrained/CarRacing/ --n-episodes=100

# Evaluate with GUI.
python3 test_solution.py --log-dir=pretrained/CarRacing/ --render

# Evaluate with GUI, also show attention patch visualization.
python3 test_solution.py --log-dir=pretrained/CarRacing/ --render --overplot

# Evaluate with GUI, save videos and screenshots.
python3 test_solution.py --log-dir=pretrained/CarRacing/ --render --overplot --save-screens
```

## Citation
For attribution in academic contexts, please cite this work as

```
@inproceedings{attentionagent2020,
  author    = {Yujin Tang and Duong Nguyen and David Ha},
  title     = {Neuroevolution of Self-Interpretable Agents},
  booktitle = {Proceedings of the Genetic and Evolutionary Computation Conference},
  url       = {https://attentionagent.github.io},
  note      = "\url{https://attentionagent.github.io}",
  year      = {2020}
}
```

## Disclaimer

This is not an official Google product.
