Benchmarks
==========

Below we document key performance benchmarks for common AIR tasks and workflows.

Bulk Ingest
-----------

This task uses the DummyTrainer module to ingest 200GiB of synthetic data.

We test out the performance across different cluster sizes.

- `Bulk Ingest Script`_
- `Bulk Ingest Cluster Configuration`_

For this benchmark, we configured the nodes to have reasonable disk size and throughput to account for object spilling.

.. code-block:: yaml

    aws:
        BlockDeviceMappings:
            - DeviceName: /dev/sda1
              Ebs:
                Iops: 5000
                Throughput: 1000
                VolumeSize: 1000
                VolumeType: gp3

.. list-table::

    * - **Cluster Setup**
      - **Performance**
      - **Disk Spill**
      - **Command**
    * - 1 m5.4xlarge node (1 actor)
      - 390 s (0.51 GiB/s)
      - 205 GiB
      - `python data_benchmark.py --dataset-size-gb=200 --num-workers=1`
    * - 5 m5.4xlarge nodes (5 actors)
      - 70 s (2.85 GiB/S)
      - 206 GiB
      - `python data_benchmark.py --dataset-size-gb=200 --num-workers=5`
    * - 20 m5.4xlarge nodes (20 actors)
      - 3.8 s (52.6 GiB/s)
      - 0 GiB
      - `python data_benchmark.py --dataset-size-gb=200 --num-workers=20`


XGBoost Batch Prediction
------------------------

This task uses the BatchPredictor module to process different amounts of data
using an XGBoost model.

We test out the performance across different cluster sizes and data sizes.

- `XGBoost Prediction Script`_
- `XGBoost Cluster Configuration`_

.. TODO: Add script for generating data and running the benchmark.

.. list-table::

    * - **Cluster Setup**
      - **Data Size**
      - **Performance**
      - **Command**
    * - 1 m5.4xlarge node (1 actor)
      - 10 GB (26M rows)
      - 275 s (94.5k rows/s)
      - `python xgboost_benchmark.py --size 10GB`
    * - 10 m5.4xlarge nodes (10 actors)
      - 100 GB (260M rows)
      - 331 s (786k rows/s)
      - `python xgboost_benchmark.py --size 100GB`


XGBoost training
----------------

This task uses the XGBoostTrainer module to train on different sizes of data
with different amounts of parallelism.

XGBoost parameters were kept as defaults for xgboost==1.6.1 this task.


- `XGBoost Training Script`_
- `XGBoost Cluster Configuration`_

.. list-table::

    * - **Cluster Setup**
      - **Data Size**
      - **Performance**
      - **Command**
    * - 1 m5.4xlarge node (1 actor)
      - 10 GB (26M rows)
      - 692 s
      - `python xgboost_benchmark.py --size 10GB`
    * - 10 m5.4xlarge nodes (10 actors)
      - 100 GB (260M rows)
      - 693 s
      - `python xgboost_benchmark.py --size 100GB`


GPU image batch prediction
----------------------------------------------------

This task uses the BatchPredictor module to process different amounts of data
using a Pytorch pre-trained ResNet model.

We test out the performance across different cluster sizes and data sizes.

- `GPU image batch prediction script`_

.. list-table::

    * - **Cluster Setup**
      - **Data Size**
      - **Performance**
      - **Command**
    * - 1 g3.8xlarge node
      - 1 GB (1623 images)
      - 72.59 s (22.3 images/sec)
      - `python gpu_batch_prediction.py --data-size-gb=1`
    * - 1 g3.8xlarge node
      - 20 GB (32460 images)
      - 1213.48 s (26.76 images/sec)
      - `python gpu_batch_prediction.py --data-size-gb=20`
    * - 4 g3.16xlarge nodes
      - 100 GB (162300 images)
      - 885.98 s (183.19 images/sec)
      - `python gpu_batch_prediction.py --data-size-gb=100`


GPU image training
------------------------

This task uses the TorchTrainer module to train different amounts of data
using an Pytorch ResNet model.

We test out the performance across different cluster sizes and data sizes.

- `GPU image training script`_

.. note::

    For multi-host distributed training, on AWS we need to ensure ec2 instances are in the same VPC and 
    all ports are open in the secure group.


.. list-table::

    * - **Cluster Setup**
      - **Data Size**
      - **Performance**
      - **Command**
    * - 1 g3.8xlarge node (1 worker)
      - 1 GB (1623 images)
      - 79.76 s (2 epochs, 40.7 images/sec)
      - `python pytorch_training_e2e.py --data-size-gb=1`
    * - 1 g3.8xlarge node (1 worker)
      - 20 GB (32460 images)
      - 1388.33 s (2 epochs, 46.76 images/sec)
      - `python pytorch_training_e2e.py --data-size-gb=20`
    * - 4 g3.16xlarge nodes (16 workers)
      - 100 GB (162300 images)
      - 434.95 s (2 epochs, 746.29 images/sec)
      - `python pytorch_training_e2e.py --data-size-gb=100 --num-workers=16`


.. _`Bulk Ingest Script`: https://github.com/ray-project/ray/blob/a30bdf9ef34a45f973b589993f7707a763df6ebf/release/air_tests/air_benchmarks/workloads/data_benchmark.py#L25-L40
.. _`Bulk Ingest Cluster Configuration`: https://github.com/ray-project/ray/blob/a30bdf9ef34a45f973b589993f7707a763df6ebf/release/air_tests/air_benchmarks/data_20_nodes.yaml#L6-L15
.. _`XGBoost Training Script`: https://github.com/ray-project/ray/blob/a241e6a0f5a630d6ed5b84cce30c51963834d15b/release/air_tests/air_benchmarks/workloads/xgboost_benchmark.py#L40-L58
.. _`XGBoost Prediction Script`: https://github.com/ray-project/ray/blob/a241e6a0f5a630d6ed5b84cce30c51963834d15b/release/air_tests/air_benchmarks/workloads/xgboost_benchmark.py#L63-L71
.. _`XGBoost Cluster Configuration`: https://github.com/ray-project/ray/blob/a241e6a0f5a630d6ed5b84cce30c51963834d15b/release/air_tests/air_benchmarks/xgboost_compute_tpl.yaml#L6-L24
.. _`GPU image batch prediction script`: https://github.com/ray-project/ray/blob/cec82a1ced631525a4d115e4dc0c283fa4275a7f/release/air_tests/air_benchmarks/workloads/gpu_batch_prediction.py#L18-L49
.. _`GPU image training script`: https://github.com/ray-project/ray/blob/cec82a1ced631525a4d115e4dc0c283fa4275a7f/release/air_tests/air_benchmarks/workloads/pytorch_training_e2e.py#L95-L106