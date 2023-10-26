# Configuring and running JobSet workloads with Kustomize


The examples in this folder show how to configure and run JobSet workloads using Kustomize. The `base_jobset` folder contains the base JobSet configuration that is used in overlays in the `tpu_hello_world` and `maxtext` folders.
Before running the examples, modify the `base_jobset\jobset.yaml` file to reflect the topology of TPU slices provisioned in your environment. For example, if you provisioned `v4-64` based node pools, update the `spec.replicatedJobs.template.spec.template.spec.nodeSelector` with `tpu-v4-podslice` and `2x4x4` values.

In addition, you need to install Kustomize. Please follow the instructions in the [Kustomize documentation](https://kubectl.docs.kubernetes.io/installation/kustomize/).


## TPU Hello World examples

In the `tpu_hello_world` folder you will find examples of experimenting with different data and model parallelism strategies. The examples use the `shardings.py` script from MaxText that is designed to make experimentation with different parallelism options easy for both single slice and multislice settings. For more information about parallelism strategies and TPU Multislice refer to the [Cloud TPU Multislice Overview](https://cloud.google.com/tpu/docs/multislice-introduction) article.

The `tpu_hello_world\single_slice` folder contains the example configuration of a single slice workload configured for  Interchip Interconnect (ICI) sharding using Fully Sharded Data Parallelism (FSDP). The `tpu_hello_world\multi-slice` is a configuration for a multi-slice workload with data parallelism (DP) over data-center network (DCN) connections and FSDP over ICI.

To adapt the samples to your environment update the `namespace` and `images` fields in the `kustomization.yaml` files to reflect your environment. Set the `namespace` field to a Kubernetes namespace created during setup. As your recall, the Kueue LocalQueue used to admit workloads has been provisioned in this namespace. Update the `newName` property of the `images` field with the name of MaxText training container image build in the previous steps.

To run a single slice example execute the following command from the `tpu_hello_world` folder:

```
kubectl apply -k single_slice
```

To run a Multislice sample execute:

```
kubectl apply -k multi_slice
```

Note that the multi-slice example is configured to run on two slices so you need at least two multi-host TPU node pools in your environment to execute it successfully.

You can review execution logs using [GKE Console](https://console.cloud.google.com/kubernetes/workload/overview) or from the command line using `kubectl`.

To get the Kueue workloads:

```
kubectl get workloads -n <YOUR NAMESPACE>
```

To get the JobSets:

```
kubectl get jobsets -n <YOUR NAMESPACE>
```

To get pods in your namespace, including pods started by your workload:

```
kubectl get pods -n <YOUR NAMESPACE>
```

Note, that if your workload failed the above command will not return the workload's pods as the JobSet operator cleans up all failed jobs.

If you want to review logs from the failed workload use [GKE Console](https://console.cloud.google.com/kubernetes/workload/overview).

To display logs for a pod:

```
kubectl logs <YOUR POD>
```

To remove your workload and all resources that it created execute:

```
kubectl delete -k single_slice
```

or

```
kubectl delete -k multi_slice
```

## MaxText pretraining examples

The `maxtext` folder contains examples of pretraining a MaxText 6.5 billion parameters model on the C4 dataset.

The `maxtext/base_maxtext` folder contains the base configuration of the JobSet workload. If you review the `maxtext\base_maxtext\jobset-spec-patch.yaml` you will notice that a JobSet resource is configured with two job templates. One (named `slice`) starts the MaxText trainer. The other (named `tensorboard`) starts the TensorBoard uploader. The runtime parameters to the MaxText trainer and the TensorBoard uploader are passed through environment variables that are set through the `maxtext-parameters` ConfigMap.

The `single-slice-6B` and `multi-slice-6B` folders contain the Kustomize overlays that customize the base MaxText JobSet configuration for running a single slice and multislice training workloads.

To run the samples in your environment




