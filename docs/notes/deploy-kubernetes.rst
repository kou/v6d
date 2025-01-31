Deploying on Kubernetes
=======================

.. _deploying-on-kubernetes:

For better leveraging the scale-in/out capability of Kubernetes for worker pods of
a data analytical job, vineyard could be deployed on Kubernetes to as a DaemonSet
in Kubernetes cluster.

Cross-Pod IPC
^^^^^^^^^^^^^

Vineyard pods shares memory with worker pods using a UNIX domain socket with fine-grained
access control.

The UNIX domain socket can be either mounted on ``hostPath`` or via a ``PersistentVolumeClaim``.
When users bundle vineyard and the workload to the same pod, the UNIX domain socket
could also be shared using an ``emptyDir``.

**Note** that when deploying vineyard on Kubernetes, it usually can only be connected
from containers in the same pod, or pods on the same hosts.

Deploying with Helm
^^^^^^^^^^^^^^^^^^^

Before installing the vineyard operator, you should install cert-manager at first.

.. code:: shell

   helm repo add jetstack https://charts.jetstack.io
   helm install \
      cert-manager jetstack/cert-manager \
      --namespace cert-manager \
      --create-namespace \
      --version v1.9.1 \
      --set installCRDs=true

Vineyard operator has been integrated with `Helm <https://helm.sh/>`_. Deploy it as follows.

.. code:: shell

   helm repo add vineyard https://vineyard.oss-ap-southeast-1.aliyuncs.com/charts/
   helm install vineyard-operator vineyard/vineyard-operator

Install vineyardd as follows.

.. code:: shell

   curl https://raw.githubusercontent.com/v6d-io/v6d/main/k8s/test/e2e/vineyardd.yaml | kubectl apply -f -

Deploying using Python API
^^^^^^^^^^^^^^^^^^^^^^^^^^

For environments where the Helm is not available, vineyard provides a Python API for
deploying on Kubernetes,

.. code:: python

    from vineyard.deploy.kubernetes import start_vineyardd

    resources = start_vineyardd()

The deployed etcd pods, vineyard daemonset and services can be viewed as:

.. code:: shell

    $ kubectl -n vineyard get pods
    NAME             READY   STATUS    RESTARTS   AGE
    etcd0            1/1     Running   0          10s
    vineyard-pwkcn   1/1     Running   0          10s

    $ kubectl -n vineyard get daemonsets
    NAME       DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
    vineyard   1         1         1       1            1           <none>          14s

    $ kubectl -n vineyard get services
    NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
    etcd-for-vineyard   ClusterIP   ........        <none>        2379/TCP   18s
    vineyard-rpc        ClusterIP   ........        <none>        9600/TCP   18s

Later the cluster can be stoped by releasing the resources with

.. code:: python

    from vineyard.deploy.kubernetes import delete_kubernetes_objects

    delete_kubernetes_objects(resources)

For more details about the API usage, please refer to the :ref:`vineyard-python-deployment-api`.
