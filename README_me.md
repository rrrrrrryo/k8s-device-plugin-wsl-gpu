# Nvidia Device plugin for WSL
This repository was forked from [Nvidia-device-plugin](https://github.com/NVIDIA/k8s-device-plugin)  
I have made modification to this repository to make it useable on WSL.  
## Usage
It is assumed that microk8 is used.  
Building a kubernetes environment using microk8s.
1. Create Docker image
    ````
    $ docker build \
        -t k8s-device-plugin-wsl:devel \
        -f deployments/container/Dockerfile.ubuntu \
        .
    ````
2. Save Docker image
    ```
    $ docker save k8s-device-plugin-wsl:devel > k8s-device-plugin-wsl.tar
    ```
3. Import docker image to microk8s
    ```
    $ microk8s ctr images import k8s-device-plugin-wsl.tar
    ```
4. Create Pod
    ```
    $ kubectl apply -f nvidia-device-plugin.yml
    ```
5. Test
    ```
    $ cat <<EOF | kubectl apply -f -
    apiVersion: v1
    kind: Pod
    metadata:
      name: gpu-pod
    spec:
      restartPolicy: Never
      containers:
        - name: cuda-container
        image: nvcr.io/nvidia/k8s/cuda-sample:vectoradd-cuda10.2
        resources:
          limits:
            nvidia.com/gpu: 1 # requesting 1 GPU
      tolerations:
      - key: nvidia.com/gpu
        operator: Exists
        effect: NoSchedule
    EOF
    ```
    ```
    $ kubectl logs gpu-pod
        [Vector addition of 50000 elements]
        Copy input data from the host memory to the CUDA device
        CUDA kernel launch with 196 blocks of 256 threads
        Copy output data from the CUDA device to the host memory
        Test PASSED
        Done
    ```

## Notes
[You need to properly configure the `nvidia-container-runtime` with containerd.](./README.md#configure-containerd)  
[WSL manages GPU devices with `/dev/dxg`, so it does not support resource partitioning.](https://github.com/NVIDIA/k8s-device-plugin/issues/332)  
