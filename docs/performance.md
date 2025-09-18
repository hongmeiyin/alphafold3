# Performance 表现，性能

## Data Pipeline 数据管道

The runtime of the data pipeline (i.e. genetic sequence search and template
search) can vary significantly depending on the size of the input and the number
of homologous sequences found, as well as the available hardware – the disk
speed can influence genetic search speed in particular.
数据管道的运行时间（即遗传序列搜索和模板搜索）可能会因输入的大小、找到的同源序列的数量以及可用的硬件而异，磁盘速度尤其会影响遗传搜索速度。

If you would like to improve performance, it's recommended to increase the disk
speed (e.g. by leveraging a RAM-backed filesystem), or increase the available
CPU cores and add more parallelisation. This can help because AlphaFold 3 runs
genetic search against 4 databases in parallel, so the optimal number of cores
is the number of cores used for each Jackhmmer process times 4. Also note that
for sequences with deep MSAs, Jackhmmer or Nhmmer may need a substantial amount
of RAM beyond the recommended 64 GB of RAM.
如果你想提高性能，建议提高磁盘速度（例如通过利用RAM支持的文件系统），或者增加可用的CPU内核并增加更多的并行化。这可能会有所帮助，因为AlphaFold 3并行对4个数据库进行遗传搜索，因此最佳的核心数量是每个Jackhmmer过程使用的核心数量乘以4。还要注意，对于具有深度MSA的序列，Jackhmmer或Nhmmer可能需要超出建议的64 GB RAM的大量RAM。

## Model Inference 模型推理

Table 8 in the Supplementary Information of the
[AlphaFold 3 paper](https://nature.com/articles/s41586-024-07487-w) provides
compile-free inference timings for AlphaFold 3 when configured to run on 16
NVIDIA A100s, with 40 GB of memory per device. In contrast, this repository
supports running AlphaFold 3 on a single NVIDIA A100 with 80 GB of memory in a
configuration optimised to maximise throughput.
AlphaFold 3论文补充信息中的表8提供了在16个NVIDIA A100（每个设备40 GB内存）上运行AlphaFold 3的无编译推理时间。相比之下，此代码库支持在单个NVIDIA A100（80 GB内存）上运行AlphaFold 3，并采用优化配置以最大化吞吐量。

We compare compile-free inference timings of these two setups in the table below
using GPU seconds (i.e. multiplying by 16 when using 16 A100s). The setup in
this repository is more efficient (by at least 2×) across all token sizes,
indicating its suitability for high-throughput applications.
我们在下表中使用GPU秒数（即使用16个A100时乘以16）比较了这两种设置的无编译推理时间。此代码库中的设置在所有token尺寸下都更高效（至少2倍），表明其适合高通量应用。

Num Tokens | 1 A100 80 GB (GPU secs) | 16 A100 40 GB (GPU secs) | Improvement
:--------- | ----------------------: | -----------------------: | ----------:
1024       | 62                      | 352                      | 5.7×
2048       | 275                     | 1136                     | 4.1×
3072       | 703                     | 2016                     | 2.9×
4096       | 1434                    | 3648                     | 2.5×
5120       | 2547                    | 5552                     | 2.2×

## Running the Pipeline in Stages 分阶段运行流程

The `run_alphafold.py` script can be executed in stages to optimise resource
utilisation. This can be useful for:
run_alphafold.py脚本可以分阶段执行以优化资源利用。这在以下场景非常有用：

1.  Splitting the CPU-only data pipeline from model inference (which requires a
    GPU), to optimise cost and resource usage.
    将仅需CPU的数据流程与需要GPU的模型推理分离，以优化成本和资源使用。
1.  Generating the JSON output file from the data pipeline only run and then
    using it for multiple different inference only runs across seeds or across
    variations of other features (e.g. a ligand or a partner chain).
    从仅运行数据流程生成JSON输出文件，然后将其用于多个不同的仅推理运行（跨种子或其他特征变体，如配体或伴侣链）。
1.  Generating the JSON output for multiple individual monomer chains (e.g. for
    chains A, B, C, D), then running the inference on all possible chain pairs
    (AB, AC, AD, BC, BD, CD) by creating dimer JSONs by merging the monomer
    JSONs. By doing this, the MSA and template search need to be run just 4
    times (once for each chain), instead of 12 times.
    为多个单独单体链（如链A、B、C、D）生成JSON输出，然后通过合并单体JSON创建二聚体JSON，对所有可能的链对（AB、AC、AD、BC、BD、CD）运行推理。通过这种方式，MSA和模板搜索只需运行4次（每个链一次），而不是12次。

### Data Pipeline Only 仅运行数据流程

Launch `run_alphafold.py` with `--norun_inference` to generate Multiple Sequence
Alignments (MSAs) and templates, without running featurisation and model
inference. This stage can be quite costly in terms of runtime, CPU, and RAM use.
The output will be JSON files augmented with MSAs and templates that can then be
directly used as input for running inference.
使用--norun_inference参数启动run_alphafold.py，以生成多序列比对（MSA）和模板，而不运行特征化和模型推理。此阶段在运行时间、CPU和RAM使用方面可能相当昂贵。输出将是增加了MSA和模板的JSON文件，这些文件可以直接用作运行推理的输入。

### Featurisation and Model Inference Only 仅运行特征化与模型推理

Launch `run_alphafold.py` with `--norun_data_pipeline` to skip the data pipeline
and run only featurisation and model inference. This stage requires the input
JSON file to contain pre-computed MSAs and templates (or they must be explicitly
set to empty if you want to run MSA and template free).
使用--norun_data_pipeline参数启动run_alphafold.py，以跳过数据流程，仅运行特征化和模型推理。此阶段要求输入JSON文件包含预先计算好的MSA和模板（或者如果要运行无MSA和无模板模式，必须显式将其设置为空）。

## Accelerator Hardware Requirements 加速器硬件要求

We officially support the following configurations, and have extensively tested
them for numerical accuracy and throughput efficiency:
我们正式支持以下配置，并对其数值精度和吞吐量效率进行了广泛测试：

-   1 NVIDIA A100 (80 GB)
-   1 NVIDIA H100 (80 GB)

We compare compile-free inference timings of both configurations in the
following table:
我们在下表中比较了两种配置的免编译推理时间：

Num Tokens | 1 A100 80 GB (seconds) | 1 H100 80 GB (seconds)
:--------- | ---------------------: | ---------------------:
1024       | 62                     | 34
2048       | 275                    | 144
3072       | 703                    | 367
4096       | 1434                   | 774
5120       | 2547                   | 1416

### Other Hardware Configurations 其他硬件配置

#### NVIDIA A100 (40 GB)

AlphaFold 3 can run on inputs of size up to 4,352 tokens on a single NVIDIA A100
(40 GB) with the following configuration changes:
通过以下配置更改，AlphaFold 3可以在单个NVIDIA A100（40 GB）上运行最大4352个令牌的输入：

1.  Enabling [unified memory](#unified-memory). 启用统一内存
1.  Adjusting `pair_transition_shard_spec` in `model_config.py`:
调整model_config.py中的pair_transition_shard_spec：
    ```py
      pair_transition_shard_spec: Sequence[_Shape2DType] = (
          (2048, None),
          (3072, 1024),
          (None, 512),
      )
    ```

The format of entries in `pair_transition_shard_spec` is
`(num_tokens_upper_bound, shard_size)`. Setting `shard_size=None` means there is
no upper bound.
pair_transition_shard_spec中的条目格式为(令牌数上限, 分片大小)。设置shard_size=None表示没有上限。
For the example above:
对于上述示例：

*   `(2048, None)`: for sequences up to 2,048 tokens, do not shard
*   (2048, None)：对于最多2,048个令牌的序列，不进行分片
*   `(3072, 1024)`: for sequences up to 3,072 tokens, shard in chunks of 1,024
*   (3072, 1024)：对于最多3,072个令牌的序列，以1,024为块进行分片
*   `(None, 512)`: for all other sequences, shard in chunks of 512
*   (None, 512)：对于所有其他序列，以512为块进行分片

While numerically accurate, this configuration will have lower throughput
compared to the set up on the NVIDIA A100 (80 GB), due to less available memory.
虽然在数值上是准确的，但由于可用内存较少，此配置的吞吐量将低于NVIDIA A100（80 GB）的设置。

#### NVIDIA V100

There are known numerical issues with CUDA Capability 7.x devices. To work
around the issue, set the ENV XLA_FLAGS to include
`--xla_disable_hlo_passes=custom-kernel-fusion-rewriter`.
已知CUDA计算能力7.x设备存在数值问题。要解决此问题，请设置环境变量XLA_FLAGS，包含--xla_disable_hlo_passes=custom-kernel-fusion-rewriter

With the above flag set, AlphaFold 3 can run on inputs of size up to 1,280
tokens on a single NVIDIA V100 using [unified memory](#unified-memory).
设置上述标志后，AlphaFold 3可以在单个NVIDIA V100上使用统一内存运行最多1280个令牌的输入。

#### NVIDIA P100

AlphaFold 3 can run on inputs of size up to 1,024 tokens on a single NVIDIA P100
with no configuration changes needed.
AlphaFold 3可以在单个NVIDIA P100上运行高达1024个令牌的输入，无需更改配置

#### Other devices

Large-scale numerical tests have not been performed on any other devices but
they are believed to be numerically accurate.
尚未在其他设备上进行大规模数值测试，但相信其在数值上是准确的

There are known numerical issues with CUDA Capability 7.x devices. To work
around the issue, set the environment variable `XLA_FLAGS` to include
`--xla_disable_hlo_passes=custom-kernel-fusion-rewriter`.
已知CUDA计算能力7.x设备存在数值问题。要解决此问题，请设置环境变量XLA_FLAGS，包含--xla_disable_hlo_passes=custom-kernel-fusion-rewriter

## Compilation Buckets 编译分桶

To avoid excessive re-compilation of the model, AlphaFold 3 implements
compilation buckets: ranges of input sizes using a single compilation of the
model.
为避免模型过度重新编译，AlphaFold 3实现了编译分桶：使用单个模型编译处理一定范围内的输入尺寸。

When featurising an input, AlphaFold 3 determines the smallest bucket the input
fits into, then adds any necessary padding. This may avoid re-compiling the
model when running inference on the input if it belongs to the same bucket as a
previously processed input.
在对输入进行特征化时，AlphaFold 3会确定输入适合的最小分桶，然后添加必要的填充。如果输入与先前处理的输入属于同一分桶，这可以避免在运行推理时重新编译模型。

The configuration of bucket sizes involves a trade-off: more buckets leads to
more re-compilations of the model, but less padding.
分桶大小的配置需要权衡：更多分桶会导致更多模型重新编译，但填充更少。

By default, the largest bucket size is 5,120 tokens. Processing inputs larger
than this maximum bucket size triggers the creation of a new bucket for exactly
that input size, and a re-compilation of the model. In this case, you may wish
to redefine the compilation bucket sizes via the `--buckets` flag in
`run_alphafold.py` to add additional larger bucket sizes. For example, suppose
you are running inference on inputs with token sizes: `5132, 5280, 5342`. Using
the default bucket sizes configured in `run_alphafold.py` will trigger three
separate model compilations, one for each unique token size. If instead you pass
in the following flag to `run_alphafold.py`
默认情况下，最大分桶大小为5,120个令牌。处理大于此最大分桶大小的输入会触发为该特定输入尺寸创建新分桶，并重新编译模型。这种情况下，您可能希望通过run_alphafold.py中的--buckets标志重新定义编译分桶大小，以添加更大的分桶尺寸。例如，假设您正在处理令牌尺寸为5132, 5280, 5342的输入。使用run_alphafold.py中配置的默认分桶大小将触发三次单独的模型编译，每次对应一个独特的令牌尺寸。如果您改为向run_alphafold.py传递以下标志：

```
--buckets 256,512,768,1024,1280,1536,2048,2560,3072,3584,4096,4608,5120,5376
```

when running inference on the above three input sizes, the model will be
compiled only once for the bucket size `5376`. **Note:** for this specific
example with input sizes `5132, 5280, 5342`, passing in `--buckets 5376` is
sufficient to achieve the desired compilation behaviour. The provided example
with multiple buckets illustrates a more general solution suitable for diverse
input sizes.
在处理上述三个输入尺寸时，模型将仅针对分桶大小5376编译一次。注意：对于输入尺寸为5132, 5280, 5342的此特定示例，传入--buckets 5376足以实现所需的编译行为。提供的包含多个分桶的示例展示了一种适用于不同输入尺寸的更通用解决方案。

## Additional Flags 附加标志

### Compilation Time Workaround with XLA Flags 使用XLA标志解决编译时间问题

To work around a known XLA issue causing the compilation time to greatly
increase, the following environment variable must be set (it is set by default
in the provided `Dockerfile`).
为解决已知的XLA问题导致编译时间大幅增加，必须设置以下环境变量（提供的Dockerfile中默认设置）。

```sh
ENV XLA_FLAGS="--xla_gpu_enable_triton_gemm=false"
```

### CUDA Capability 7.x GPUs CUDA计算能力7.x GPU

For all CUDA Capability 7.x GPUs (e.g. V100) the environment variable
`XLA_FLAGS` must be changed to include
`--xla_disable_hlo_passes=custom-kernel-fusion-rewriter`. Disabling the Tritron
GEMM kernels is not necessary as they are not supported for such GPUs.
对于所有CUDA计算能力7.x GPU（如V100），环境变量XLA_FLAGS必须更改为包含--xla_disable_hlo_passes=custom-kernel-fusion-rewriter。不需要禁用Tritron GEMM内核，因为它们在此类GPU上不受支持。
```sh
ENV XLA_FLAGS="--xla_disable_hlo_passes=custom-kernel-fusion-rewriter"
```

### GPU Memory GPU内存

The following environment variables (set by default in the `Dockerfile`) enable
folding a single input of size up to 5,120 tokens on a single A100 (80 GB) or a
single H100 (80 GB):
以下环境变量（在Dockerfile中默认设置）支持在单个A100（80 GB）或单个H100（80 GB）上处理最大5,120个令牌的单个输入：

```sh
ENV XLA_PYTHON_CLIENT_PREALLOCATE=true
ENV XLA_CLIENT_MEM_FRACTION=0.95
```

#### Unified Memory 统一内存

If you would like to run AlphaFold 3 on inputs larger than 5,120 tokens, or on a
GPU with less memory (an A100 with 40 GB of memory, for instance), we recommend
enabling unified memory. Enabling unified memory allows the program to spill GPU
memory to host memory if there isn't enough space. This prevents an OOM, at the
cost of making the program slower by accessing host memory instead of device
memory. To learn more, check out the
[NVIDIA blog post](https://developer.nvidia.com/blog/unified-memory-cuda-beginners/).
如果您想在大于5,120个令牌的输入上运行AlphaFold 3，或在内存较小的GPU上运行（例如40 GB内存的A100），我们建议启用统一内存。启用统一内存允许程序在GPU内存不足时将内存溢出到主机内存。这可以防止OOM（内存不足），但代价是通过访问主机内存而非设备内存使程序变慢。了解更多，请查看NVIDIA博客文章。

You can enable unified memory by setting the following environment variables in
your `Dockerfile`:
您可以通过在Dockerfile中设置以下环境变量来启用统一内存：

```sh
ENV XLA_PYTHON_CLIENT_PREALLOCATE=false
ENV TF_FORCE_UNIFIED_MEMORY=true
ENV XLA_CLIENT_MEM_FRACTION=3.2
```

### JAX Persistent Compilation Cache JAX持久编译缓存

You may also want to make use of the JAX persistent compilation cache, to avoid
unnecessary recompilation of the model between runs. You can enable the
compilation cache with the `--jax_compilation_cache_dir <YOUR_DIRECTORY>` flag
in `run_alphafold.py`.
您可能还希望利用JAX持久编译缓存，以避免在运行之间不必要的模型重新编译。您可以使用run_alphafold.py中的--jax_compilation_cache_dir <您的目录>标志启用编译缓存。

More detailed instructions are available in the
[JAX documentation](https://jax.readthedocs.io/en/latest/persistent_compilation_cache.html#persistent-compilation-cache),
and more specifically the instructions for use on
[Google Cloud](https://jax.readthedocs.io/en/latest/persistent_compilation_cache.html#persistent-compilation-cache).
In particular, note that if you would like to make use of a non-local
filesystem, such as Google Cloud Storage, you will need to install
[`etils`](https://github.com/google/etils) (this is not included by default in
the AlphaFold 3 Docker container).
更详细的说明可在JAX文档中找到，特别是关于在Google Cloud上使用的说明。特别注意，如果您想使用非本地文件系统（如Google Cloud Storage），需要安装etils（AlphaFold 3 Docker容器默认不包含此库）。
