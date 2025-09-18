# AlphaFold 3 Output AlphaFold 3 输出

## Output Directory Structure 输出目录结构

For every input job, AlphaFold 3 writes all its outputs in a directory called by
the sanitized version of the job name. E.g. for job name "My first fold (TEST)",
AlphaFold 3 will write its outputs in a directory called `My_first_fold_TEST`
(the case is respected). If such directory already exists, AlphaFold 3 will
append a timestamp to the directory name to avoid overwriting existing data
unless `--force_output_dir` is passed.
对于每个输入作业，AlphaFold 3将其所有输出写入一个由作业名称的净化版本调用的目录中。例如，对于作业名称“My first fold（TEST）”，AlphaFold 3将把其输出写入名为“My_first_fold_TEST”的目录中（视情况而定）。如果这样的目录已经存在，AlphaFold 3将在目录名后附加一个时间戳，以避免覆盖现有数据，除非传递了`-force_output_dir`。

The following structure is used within the output directory:
输出目录中使用了以下结构：

*   Sub-directories with results for each sample and seed. There will be
    *num\_seeds* \* *num\_samples* such sub-directories. The naming pattern is
    `seed-<seed value>_sample-<sample number>`. Each of these directories
    contains a confidence JSON, summary confidence JSON, and the mmCIF with the
    predicted structure.
    每个样本和种子的结果子目录。共有num_seeds * num_samples个子目录。命名模式为seed-<种子值>_sample-<样本编号>。每个目录包含置信度JSON、摘要置信度JSON以及预测结构的mmCIF文件。
*   Distogram for each seed: `seed-<seed value>_distogram/distogram.npz`. The
    Numpy zip file contains a single key: `distogram`. The distogram can be
    large, its shape is `(num_tokens, num_tokens, 64)` and dtype `np.float16`
    (almost 3 GiB for a 5,000-token input). Only saved if AlphaFold 3 is run
    with `--save_distogram=true`.
    每种子的距离分布图：seed-<种子值>_distogram/distogram.npz。该Numpy压缩文件包含单个键：distogram。距离分布图可能很大，其形状为(num_tokens, num_tokens, 64)，数据类型为np.float16（对于5,000个token的输入，约3 GiB）。仅在AlphaFold 3以--save_distogram=true运行时保存。
*   Embeddings for each seed: `seed-<seed value>_embeddings/embeddings.npz`. The
    Numpy zip file contains 2 keys: `single_embeddings` and `pair_embeddings`.
    The embeddings can be large, their shapes are `(num_tokens, 384)` for
    `single_embeddings`, and `(num_tokens, num_tokens, 128)` for
    `pair_embeddings`. Their dtype is `np.float16` (almost 6 GiB for a
    5,000-token input). Only saved if AlphaFold 3 is run with
    `--save_embeddings=true`.
    每种子的嵌入表示：seed-<种子值>_embeddings/embeddings.npz。该Numpy压缩文件包含2个键：single_embeddings和pair_embeddings。嵌入表示可能很大，single_embeddings的形状为(num_tokens, 384)，pair_embeddings的形状为(num_tokens, num_tokens, 128)，数据类型为np.float16（对于5,000个token的输入，约6 GiB）。仅在AlphaFold 3以--save_embeddings=true运行时保存。
*   Top-ranking prediction mmCIF: `<job_name>_model.cif`. This file contains the
    predicted coordinates and should be compatible with most structural biology
    tools. We do not provide the output in the PDB format, the CIF file can be
    easily converted into one if needed.
    顶级预测mmCIF：<任务名称>_model.cif。该文件包含预测坐标，应与大多数结构生物学工具兼容。我们不提供PDB格式输出，如需可轻松将CIF文件转换。
*   Top-ranking prediction confidence JSON: `<job_name>_confidences.json`.
*   顶级预测置信度JSON：<任务名称>_confidences.json。
*   Top-ranking prediction summary confidence JSON:
    `<job_name>_summary_confidences.json`.
    顶级预测摘要置信度JSON：<任务名称>_summary_confidences.json。
*   Job input JSON file with the MSA and template data added by the data
    pipeline: `<job_name>_data.json`.
    数据流程添加了MSA和模板数据后的任务输入JSON文件：<任务名称>_data.json。
*   Ranking scores for all predictions: `ranking_scores.csv`. The prediction
    with highest ranking is the one included in the root directory.
    所有预测的排名分数：ranking_scores.csv。排名最高的预测即根目录中包含的预测。
*   Output terms of use: `TERMS_OF_USE.md`.
*   输出使用条款：TERMS_OF_USE.md。

Below is an example AlphaFold 3 output directory listing for a job called "Hello
Fold", that has been ran with 1 seed and 5 samples:
下面是一个名为“Hello Fold”的作业的AlphaFold 3输出目录示例，该作业已使用1个种子和5个示例运行：

```txt
hello_fold/
├── seed-1234_distogram                        # Only if --save_distogram=true.
│   └── hello_fold_seed-1234_distogram.npz     # Only if --save_distogram=true.
├── seed-1234_embeddings                       # Only if --save_embeddings=true.
│   └── hello_fold_seed-1234_embeddings.npz    # Only if --save_embeddings=true.
├── seed-1234_sample-0/
│   ├── hello_fold_seed-1234_sample-0_confidences.json
│   ├── hello_fold_seed-1234_sample-0_model.cif
│   └── hello_fold_seed-1234_sample-0_summary_confidences.json
├── seed-1234_sample-1/
│   ├── hello_fold_seed-1234_sample-1_confidences.json
│   ├── hello_fold_seed-1234_sample-1_model.cif
│   └── hello_fold_seed-1234_sample-1_summary_confidences.json
├── seed-1234_sample-2/
│   ├── hello_fold_seed-1234_sample-2_confidences.json
│   ├── hello_fold_seed-1234_sample-2_model.cif
│   └── hello_fold_seed-1234_sample-2_summary_confidences.json
├── seed-1234_sample-3/
│   ├── hello_fold_seed-1234_sample-3_confidences.json
│   ├── hello_fold_seed-1234_sample-3_model.cif
│   └── hello_fold_seed-1234_sample-3_summary_confidences.json
├── seed-1234_sample-4/
│   ├── hello_fold_seed-1234_sample-4_confidences.json
│   ├── hello_fold_seed-1234_sample-4_model.cif
│   └── hello_fold_seed-1234_sample-4_summary_confidences.json
├── TERMS_OF_USE.md
├── hello_fold_confidences.json
├── hello_fold_data.json
├── hello_fold_model.cif
├── hello_fold_ranking_scores.csv
└── hello_fold_summary_confidences.json
```

## Confidence Metrics 置信度矩阵

Similar to AlphaFold 2 and AlphaFold-Multimer, AlphaFold 3 outputs include
confidence metrics. The main metrics are:
与AlphaFold 2和AlphaFold-Multimer类似，AlphaFold 3的输出包含置信度指标。主要指标包括：

*   **pLDDT:** a per-atom confidence estimate on a 0-100 scale where a higher
    value indicates higher confidence. pLDDT aims to predict a modified LDDT
    score that only considers distances to polymers. For proteins this is
    similar to the
    [lDDT-Cα metric](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3799472/) but
    with more granularity as it can vary per atom not just per residue. For
    ligand atoms, the modified LDDT considers the errors only between the ligand
    atom and polymers, not other ligand atoms. For DNA/RNA a wider radius of 30
    Å is used for the modified LDDT instead of 15 Å.
    pLDDT：每个原子的置信度估计，范围0-100，值越高表示置信度越高。pLDDT旨在预测一种改良的LDDT分数，仅考虑与聚合物的距离。对于蛋白质，这类似于lDDT-Cα指标，但更精细，因为它可以每个原子变化而不仅仅是每个残基。对于配体原子，改良的LDDT仅考虑配体原子与聚合物之间的误差，而不考虑其他配体原子。对于DNA/RNA，改良的LDDT使用30 Å的更大半径而不是15 Å。
*   **PAE (predicted aligned error)**: an estimate of the error in the relative
    position and orientation between two tokens in the predicted structure.
    Higher values indicate higher predicted error and therefore lower
    confidence. For proteins and nucleic acids, PAE score is essentially the
    same as AlphaFold 2, where the error is measured relative to frames
    constructed from the protein backbone. For small molecules and
    post-translational modifications, a frame is constructed for each atom from
    its closest neighbors from a reference conformer.
    PAE（预测对齐误差）：预测结构中两个token之间相对位置和方向误差的估计。值越高表示预测误差越大，因此置信度越低。对于蛋白质和核酸，PAE分数与AlphaFold 2基本相同，其中误差相对于蛋白质骨架构建的框架进行测量。对于小分子和翻译后修饰，为每个原子从其参考构象的最邻近原子构建框架。
*   **pTM and ipTM scores**: the predicted template modeling (pTM) score and the
    interface predicted template modeling (ipTM) score are both derived from a
    measure called the template modeling (TM) score. This measures the accuracy
    of the entire structure
    ([Zhang and Skolnick, 2004](https://doi.org/10.1002/prot.20264);
    [Xu and Zhang, 2010](https://doi.org/10.1093/bioinformatics/btq066)). A pTM
    score above 0.5 means the overall predicted fold for the complex might be
    similar to the true structure. ipTM measures the accuracy of the predicted
    relative positions of the subunits within the complex. Values higher than
    0.8 represent confident high-quality predictions, while values below 0.6
    suggest a failed prediction. ipTM values between 0.6 and 0.8 are a gray zone
    where predictions could be correct or incorrect. The TM score is very strict
    for small structures or short chains, so pTM assigns values less than 0.05
    when fewer than 20 tokens are involved; for these cases PAE or pLDDT may be
    more indicative of prediction quality.
    pTM和ipTM分数：预测的模板建模（pTM）分数和界面预测模板建模（ipTM）分数均源自称为模板建模（TM）分数的度量。这衡量整个结构的准确性（Zhang and Skolnick, 2004; Xu and Zhang, 2010）。pTM分数高于0.5表示复合物的整体预测折叠可能与真实结构相似。ipTM衡量复合物内亚基预测相对位置的准确性。值高于0.8代表可靠的高质量预测，而低于0.6则表示预测失败。0.6到0.8之间的ipTM值是一个灰色区域，预测可能正确也可能错误。TM分数对于小结构或短链非常严格，因此当涉及少于20个token时，pTM分配小于0.05的值；对于这些情况，PAE或pLDDT可能更能指示预测质量。

For detailed description of these confidence metrics see the
[AlphaFold 3 paper](https://www.nature.com/articles/s41586-024-07487-w). For
protein components, the
[AlphaFold: A Practical guide](https://www.ebi.ac.uk/training/online/courses/alphafold/inputs-and-outputs/evaluating-alphafolds-predicted-structures-using-confidence-scores/)
course for structures provides additional tutorials on the confidence metrics.
有关这些置信度指标的详细描述，请参阅AlphaFold 3论文。对于蛋白质组分，AlphaFold: A Practical guide课程提供了关于置信度指标的其他教程。

If you are interested in a specific entity or interaction, then there are
confidences available in the outputs which are specific to each chain or
chain-pair, as opposed to the full complex. See below for more details on all
the confidence metrics that are returned.
如果您对特定实体或相互作用感兴趣，输出中提供了针对每个链或链对的置信度，而不是整个复合物。有关返回的所有置信度指标的更多详细信息，请参见下文。

## Multi-Seed and Multi-Sample Results 多种子与多样本结果

By default, the model samples five predictions per seed. The top-ranked
prediction across all samples and seeds is available at the top-level of the
output directory. All samples along with their associated confidences are
available in subdirectories of the output directory.
默认情况下，模型对每个种子采样五个预测。所有样本和种子中排名最高的预测可在输出目录的顶层获取。所有样本及其相关置信度均可在输出目录的子目录中找到。

For ranking of the full complex use the `ranking_score` (higher is better). This
score uses overall structure confidences (pTM and ipTM), but also includes terms
that penalize clashes and encourage disordered regions not to have spurious
helices – these extra terms mean the score should only be used to rank
structures.
对于完整复合物的排名，请使用ranking_score（分数越高越好）。该分数使用整体结构置信度（pTM和ipTM），但也包含惩罚 clashes（原子冲突）和鼓励无序区域不形成虚假螺旋的项——这些额外项意味着该分数仅应用于结构排名。

If you are interested in a specific entity or interaction, you may want to rank
by a metric specific to that chain or chain-pair, as opposed to the full
complex. In that case, use the per chain or per chain-pair confidence metrics
described below for ranking.
如果您对特定实体或相互作用感兴趣，可能需要使用针对该链或链对的度量进行排名，而不是针对完整复合物。在这种情况下，请使用下文描述的每链或每链对置信度指标进行排名。

## Metrics in Confidences JSON 置信度JSON中的指标

For each predicted sample we provide two JSON files. One contains summary
metrics – summaries for either the whole structure, per chain or per chain-pair
– and the other contains full 1D or 2D arrays.
对于每个预测样本，我们提供两个JSON文件。一个包含摘要指标——针对整个结构、每条链或每对链的摘要——另一个包含完整的一维或二维数组。

Summary outputs:
摘要输出：

*   `ptm`: A scalar in the range 0-1 indicating the predicted TM-score for the
    full structure.
    ptm：0-1范围内的标量，表示整个结构的预测TM分数。
*   `iptm`: A scalar in the range 0-1 indicating predicted interface TM-score
    (confidence in the predicted interfaces) for all interfaces in the
    structure.
    iptm：0-1范围内的标量，表示结构中所有界面的预测界面TM分数（界面预测置信度）。
*   `fraction_disordered`: A scalar in the range 0-1 that indicates what
    fraction of the prediction structure is disordered, as measured by
    accessible surface area, see our
    [paper](https://www.nature.com/articles/s41586-024-07487-w) for details.
    0-1范围内的标量，表示预测结构中无序部分的比例，通过可及表面积测量，详见我们的论文。
*   `has_clash`: A boolean indicating if the structure has a significant number
    of clashing atoms (more than 50% of a chain, or a chain with more than 100
    clashing atoms).
    has_clash：布尔值，表示结构是否存在显著数量的原子冲突（超过50%的链原子，或链中超过100个冲突原子）。
*   `ranking_score`: A scalar in the range \[-100, 1.5\] that can be used for
    ranking predictions, it incorporates `ptm`, `iptm`, `fraction_disordered`
    and `has_clash` into a single number with the following equation: 0.8 × ipTM
    \+ 0.2 × pTM \+ 0.5 × disorder − 100 × has_clash.
    ranking_score：[-100, 1.5]范围内的标量，可用于预测排名，它通过以下公式将ptm、iptm、fraction_disordered和has_clash合并为单个数字：0.8 × ipTM + 0.2 × pTM + 0.5 × disorder − 100 × has_clash。
*   `chain_pair_pae_min`: A \[num_chains, num_chains\] array. Element (i, j) of
    the array contains the lowest PAE value across rows restricted to chain i
    and columns restricted to chain j. This has been found to correlate with
    whether two chains interact or not, and in some cases can be used to
    distinguish binders from non-binders.
    chain_pair_pae_min：[num_chains, num_chains]数组。数组元素(i, j)包含限于链i的行和限于链j的列中的最低PAE值。这已被发现与两条链是否相互作用相关，在某些情况下可用于区分结合体与非结合体。
*   `chain_pair_iptm`: A \[num_chains, num_chains\] array. Off-diagonal element
    (i, j) of the array contains the ipTM restricted to tokens from chains i and
    j. Diagonal element (i, i) contains the pTM restricted to chain i. Can be
    used for ranking a specific interface between two chains, when you know that
    they interact, e.g. for antibody-antigen interactions
    chain_pair_iptm：[num_chains, num_chains]数组。数组的非对角线元素(i, j)包含限于链i和j的token的ipTM。对角线元素(i, i)包含限于链i的pTM。当已知两条链相互作用时（如抗体-抗原相互作用），可用于排名特定界面。
*   `chain_ptm`: A \[num_chains\] array. Element i contains the pTM restricted
    to chain i. Can be used for ranking individual chains when the structure of
    that chain is most of interest, rather than the cross-chain interactions it
    is involved with.
    chain_ptm：[num_chains]数组。元素i包含限于链i的pTM。当最关心该链的结构而非其涉及的链间相互作用时，可用于排名单个链。
*   `chain_iptm:` A \[num_chains\] array that gives the average confidence
    (interface pTM) in the interface between each chain and all other chains.
    Can be used for ranking a specific chain, when you care about where the
    chain binds to the rest of the complex and you do not know which other
    chains you expect it to interact with. This is often the case with ligands.
    chain_iptm：[num_chains]数组，给出每条链与所有其他链之间界面的平均置信度（界面pTM）。当关心链与复合物其余部分的结合位置但不知道预期与哪些其他链相互作用时，可用于排名特定链。这在配体中常见。

Full array outputs:
完整数组输出：

*   `pae`: A \[num\_tokens, num\_tokens\] array. Element (i, j) indicates the
    predicted error in the position of token j, when the prediction is aligned
    to the ground truth using the frame of token i.
    pae：[num_tokens, num_tokens]数组。元素(i, j)表示当使用token i的框架将预测与真实结构对齐时，token j位置的预测误差。
*   `atom_plddts`: A \[num_atoms\] array, element i indicates the predicted
    local distance difference test (pLDDT) for atom i in the prediction.
    atom_plddts：[num_atoms]数组，元素i表示预测中原子i的预测局部距离差异测试（pLDDT）。
*   `contact_probs`: A \[num_tokens, num_tokens\] array. Element (i, j)
    indicates the predicted probability that token i and token j are in contact
    (8 Å between the representative atom for each token), see
    [paper](https://www.nature.com/articles/s41586-024-07487-w) for details.
    contact_probs：[num_tokens, num_tokens]数组。元素(i, j)表示token i和j接触的预测概率（每个token的代表原子之间8 Å），详见论文。
*   `token_chain_ids`: A \[num_tokens\] array indicating the chain ids
    corresponding to each token in the prediction.
    token_chain_ids：[num_tokens]数组，指示预测中每个token对应的链ID。
*   `atom_chain_ids`: A \[num_atoms\] array indicating the chain ids
    corresponding to each atom in the prediction.
    atom_chain_ids：[num_atoms]数组，指示预测中每个原子对应的链ID。

## Embeddings 嵌入

AlphaFold 3 can be run with `--save_embeddings=true` to save the embeddings for
each seed. The file is in the
[compressed Numpy `.npz` format](https://numpy.org/doc/stable/reference/generated/numpy.savez_compressed.html)
and can be loaded using `numpy.load` as a dictionary-like object with two
arrays:
AlphaFold 3可以通过--save_embeddings=true参数运行，以保存每种子的嵌入表示。文件采用压缩Numpy .npz格式，可使用numpy.load加载为类字典对象，包含两个数组：

*   `single_embeddings`: A \`[num\_tokens, 384\] array containing the embeddings
    for each token.
    single_embeddings：`[num_tokens, 384]数组，包含每个token的嵌入表示
*   `pair_embeddings`: A \[num\_tokens, num\_tokens, 128\] array containing the
    pairwise embeddings between all tokens.
    pair_embeddings：[num_tokens,num_tokens,128]数组，包含所有token之间的成对嵌入表示

You can use for instance the following Python code to load the embeddings:
您可以使用以下Python代码加载嵌入表示：

```py
import numpy as np

with open('embeddings.npz', 'rb') as f:
  embeddings = np.load(f)
  single_embeddings = embeddings['single_embeddings']
  pair_embeddings = embeddings['pair_embeddings']
```

## Chirality checks 手性检查

In the AlphaFold 3 paper Posebusters results, a penalty was applied to the
ranking score if the ligand of interest contained chiral errors. By running
multiple seeds and using this chiral aware ranking, chiral error rates were
greatly reduced.
在AlphaFold 3论文的Posebusters结果中，如果目标配体包含手性错误，会对排名分数施加惩罚。通过运行多个种子并使用这种手性感知排名，手性错误率大大降低。

We provide the method `compare_chirality` in
[`model/scoring/chirality.py`](https://github.com/google-deepmind/alphafold3/blob/main/src/alphafold3/model/scoring/chirality.py)
to replicate these chiral checks. Chirality is checked against CCD structures if
available, otherwise users can supply custom RDKit Mol objects for comparison.
我们在model/scoring/chirality.py中提供了compare_chirality方法以复现这些手性检查。如果有CCD结构可用，将针对其检查手性；否则，用户可以提供自定义的RDKit Mol对象进行比较。
