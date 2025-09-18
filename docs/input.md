# AlphaFold 3 Input

## Specifying Input Files 指定输入文件

You can provide inputs to `run_alphafold.py` in one of two ways:您可以通过以下两种方式之一向`run_alphafold.py `提供输入：

-   Single input file: Use the `--json_path` flag followed by the path to a
    single JSON file.
    单个输入文件：使用`--json_path`标志，后跟单个json文件的路径。
-   Multiple input files: Use the `--input_dir` flag followed by the path to a
    directory of JSON files.
    多个输入文件：使用`--input_dir`标志，后跟JSON文件目录的路径。

## Input Format 输入格式

AlphaFold 3 uses a custom JSON input format differing from the
[AlphaFold Server JSON input format](https://github.com/google-deepmind/alphafold/tree/main/server).
See [below](#alphafold-server-json-compatibility) for more information.
AlphaFold 3 使用一种不同于 AlphaFold Server JSON 输入格式的自定义 JSON 格式。更多信息请见下文。

The custom AlphaFold 3 format allows:
自定义的 AlphaFold 3 格式允许：

*   Specifying protein, RNA, and DNA chains, including modified residues.
*   指定蛋白质、RNA 和 DNA 链，包括修饰过的残基。
*   Specifying custom multiple sequence alignment (MSA) for protein and RNA
    chains.
    为蛋白质和 RNA 链指定自定义的多序列比对（MSA）。
*   Specifying custom structural templates for protein chains.
*   为蛋白质链指定自定义的结构模板。
*   Specifying ligands using
    [Chemical Component Dictionary (CCD)](https://www.wwpdb.org/data/ccd) codes.
    使用化学成分词典（CCD）代码指定配体。
*   Specifying ligands using SMILES.
*   使用 SMILES 字符串指定配体。
*   Specifying ligands by defining them using the CCD mmCIF format and supplying
    them via the [user-provided CCD](#user-provided-ccd).
    通过使用 CCD mmCIF 格式定义配体，并通过用户提供的 CCD 来提供它们。
*   Specifying covalent bonds between entities.
*   指定实体之间的共价键。
*   Specifying multiple random seeds.
*   指定多个随机种子。    


## AlphaFold Server JSON Compatibility AlphaFold Server JSON 兼容性

The [AlphaFold Server](https://alphafoldserver.com/) uses a separate
[JSON format](https://github.com/google-deepmind/alphafold/tree/main/server)
from the one used here in the AlphaFold 3 codebase. In particular, the JSON
format used in the AlphaFold 3 codebase offers more flexibility and control in
defining custom ligands, branched glycans, and covalent bonds between entities.
AlphaFold Server 使用的 JSON 格式与 AlphaFold 3 代码库中使用的格式是分开的。特别地，AlphaFold 3 代码库中使用的 JSON 格式在定义自定义配体、支链聚糖和实体间的共价键方面提供了更大的灵活性和控制力。

We provide a converter in `run_alphafold.py` which automatically detects the
input JSON format, denoted `dialect` in the converter code. The converter
denotes the AlphaFoldServer JSON as `alphafoldserver`, and the JSON format
defined here in the AlphaFold 3 codebase as `alphafold3`. If the detected input
JSON format is `alphafoldserver`, then the converter will translate that into
the JSON format `alphafold3`.
我们在 run_alphafold.py 中提供了一个转换器，它可以自动检测输入的 JSON 格式（在转换器代码中称为 dialect，即“方言”）。转换器将 AlphaFold Server 的 JSON 标记为 alphafoldserver，而将 AlphaFold 3 代码库中定义的 JSON 格式标记为 alphafold3。如果检测到输入的 JSON 格式是 alphafoldserver，转换器会将其翻译成 alphafold3 格式。

### Multiple Inputs 多重输入

The top-level of the `alphafoldserver` JSON format is a list, allowing
specification of multiple inputs in a single JSON. In contrast, the `alphafold3`
JSON format requires exactly one input per JSON file. Specifying multiple inputs
in a single `alphafoldserver` JSON is fully supported.
alphafoldserver JSON 格式的顶层是一个列表，允许在单个 JSON 文件中指定多个输入。相比之下，alphafold3 JSON 格式要求每个 JSON 文件只有一个输入。完全支持在单个 alphafoldserver JSON 中指定多个输入。

Note that the converter distinguishes between `alphafoldserver` and `alphafold3`
JSON formats by checking if the top-level of the JSON is a list or not. In
particular, if you pass in a `alphafoldserver`-style JSON without a top-level
list, then this is considered incorrect and `run_alphafold.py` will raise an
error.
请注意，转换器通过检查 JSON 的顶层是否为列表来区分 alphafoldserver 和 alphafold3 格式。特别地，如果您传入一个没有顶层列表的 alphafoldserver 风格的 JSON，这将被视为不正确，run_alphafold.py 将会报错。

### Glycans 聚糖 (Glycans)

If the JSON in `alphafoldserver` format specifies glycans, the converter will
raise an error. This is because translating glycans specified in the
`alphafoldserver` format to the `alphafold3` format is not currently supported.
如果 alphafoldserver 格式的 JSON 指定了聚糖，转换器将会报错。这是因为目前不支持将 alphafoldserver 格式中指定的聚糖翻译成 alphafold3 格式。

### Random Seeds 随机种子 (Random Seeds)

The `alphafoldserver` JSON format allows users to specify `"modelSeeds": []`, in
which case a seed is chosen randomly for the user. On the other hand, the
`alphafold3` format requires users to specify a seed.
alphafoldserver JSON 格式允许用户指定 "modelSeeds": []，在这种情况下，系统会为用户随机选择一个种子。而 alphafold3 格式则要求用户必须明确指定一个种子。

The converter will choose a seed randomly if `"modelSeeds": []` is set when
translating from `alphafoldserver` JSON format to `alphafold3` JSON format. If
seeds are specified in the `alphafoldserver` JSON format, then those will be
preserved in the translation to the `alphafold3` JSON format.
当从 alphafoldserver 格式转换为 alphafold3 格式时，如果设置了 "modelSeeds": []，转换器会随机选择一个种子。如果在 alphafoldserver 格式中指定了种子，这些种子将在转换到 alphafold3 格式时被保留。

### Ions 离子

While AlphaFold Server treats ions and ligands as different entity types in the
JSON format, AlphaFold 3 treats ions as ligands. Therefore, to specify e.g. a
magnesium ion, one would specify it as an entity of type `ligand` with
`ccdCodes: ["MG"]`.
虽然AlphaFold Server将离子和配体视为JSON格式的不同实体类型，但AlphaFold 3将离子视为配体。因此，要指定例如镁离子，可以将其指定为具有“ccdCodes：[“MG”]”的“配体”类型的实体。

### Sequence IDs 序列IDs

The `alphafold3` JSON format requires the user to specify a unique identifier
(`id`) for each entity. On the other hand, the `alphafoldserver` does not allow
specification of an `id` for each entity. Thus, the converter automatically
assigns one.
`alphafold3` JSON格式要求用户为每个实体指定一个唯一的标识符（`id`）。另一方面，“alphafoldserver”不允许为每个实体指定“id”。因此，转换器会自动分配一个。

The converter iterates through the list provided in the `sequences` field of the
`alphafoldserver` JSON format, assigning an `id` to each entity using the
following order ("reverse spreadsheet style"):
转换器会遍历alphafoldserver JSON格式中sequences字段提供的列表，并按照以下顺序（"反向电子表格风格"）为每个实体分配id：

```
A, B, ..., Z, AA, BA, CA, ..., ZA, AB, BB, CB, ..., ZB, ...
```

For any entity with `count > 1`, an `id` is assigned arbitrarily to each "copy"
of the entity.
对于任何“计数>1”的实体，都会为该实体的每个“副本”任意分配一个“id”。

## Top-level Structure 顶层结构

The top-level structure of the input JSON is: 输入 JSON 的顶层结构是：

```json
{
  "name": "Job name goes here",
  "modelSeeds": [1, 2],  # At least one seed required.// 至少需要一个种子
  "sequences": [
    {"protein": {...}},
    {"rna": {...}},
    {"dna": {...}},
    {"ligand": {...}}
  ],
  "bondedAtomPairs": [...],  # Optional.// 可选
  "userCCD": "...",  # Optional, mutually exclusive with userCCDPath. // 可选, 与 userCCDPath 互斥
  "userCCDPath": "...",  # Optional, mutually exclusive with userCCD. // 可选, 与 userCCD 互斥
  "dialect": "alphafold3",  # Required. // 必需
  "version": 3  # Required. // 必需
}
```



    
    
    
    
    
    


The fields specify the following:各字段说明如下：

*   `name: str`: The name of the job. A sanitised version of this name is used
    for naming the output files.
    name: str: 任务的名称。一个经过清理的版本将用于命名输出文件。
*   `modelSeeds: list[int]`: A list of integer random seeds. The pipeline and
    the model will be invoked with each of the seeds in the list. I.e. if you
    provide *n* random seeds, you will get *n* predicted structures, each with
    the respective random seed. You must provide at least one random seed.
    modelSeeds: list[int]: 一个整数随机种子列表。程序将使用列表中的每个种子分别调用模型。即，如果您提供 n 个种子，您将得到 n 个预测结构。必须提供至少一个种子。
*   `sequences: list[Protein | RNA | DNA | Ligand]`: A list of sequence
    dictionaries, each defining a molecular entity, see below.
    sequences: list[...]: 一个序列字典列表，每个字典定义一个分子实体（见下文）。
*   `bondedAtomPairs: list[Bond]`: An optional list of covalently bonded atoms.
    These can link atoms within an entity, or across two entities. See more
    below.
    bondedAtomPairs: list[Bond]: (可选) 共价键合的原子对列表。可以连接同一实体内或跨两个实体的原子。
*   `userCCD: str`: An optional string with user-provided chemical components
    dictionary. This is an expert mode for providing custom molecules when
    SMILES is not sufficient. This should also be used when you have a custom
    molecule that needs to be bonded with other entities - SMILES can't be used
    in such cases since it doesn't give the possibility of uniquely naming all
    atoms. It can also be used to provide a reference conformer for cases where
    RDKit fails to generate a conformer. See more below.
    userCCD: str: (可选) 包含用户提供的化学成分词典的字符串。这是一种专家模式，用于在 SMILES 不足以定义自定义分子时使用。当您的自定义分子需要与其他实体键合时也应使用此选项，因为 SMILES 无法唯一命名所有原子。它还可用于为 RDKit 生成构象失败的情况提供参考构象。
*   `userCCDPath: str`: An optional path to a file that contains the
    user-provided chemical components dictionary instead of providing it inline
    using the `userCCD` field. The path can be either absolute, or relative to
    the input JSON path. The file must be in the
    [CCD mmCIF format](https://www.wwpdb.org/data/ccd#mmcifFormat), and could be
    either plain text, or compressed using gzip, xz, or zstd.
    userCCDPath: str: (可选) 指向包含用户提供的化学成分词典的文件的路径，替代在 userCCD 字段中内联提供。路径可以是绝对路径或相对路径。文件必须是 CCD mmCIF 格式，可以是纯文本或压缩文件（gzip, xz, zstd）。
*   `dialect: str`: The dialect of the input JSON. This must be set to
    `alphafold3`. See
    [AlphaFold Server JSON Compatibility](#alphafold-server-json-compatibility)
    for more information.
    dialect: str: 输入 JSON 的“方言”。必须设置为 alphafold3。
*   `version: int`: The version of the input JSON. This must be set to 1 or 2.
    See
    [AlphaFold Server JSON Compatibility](#alphafold-server-json-compatibility)
    and [versions](#versions) below for more information.
    version: int: 输入 JSON 的版本。必须设置为 1, 2 或 3。

## Versions 版本 (Versions)

The top-level `version` field (for the `alphafold3` dialect) can be either `1`,
`2`, or `3`. The following features have been added in respective versions:
顶层 version 字段（针对 alphafold3 方言）可以是 1、2 或 3。各版本新增功能如下：

*   `1`: the initial AlphaFold 3 input format.
*   1: 初始的 AlphaFold 3 输入格式。
*   `2`: added the option of specifying external MSA and templates using newly
    added fields `unpairedMsaPath`, `pairedMsaPath`, and `mmcifPath`.
    2: 新增了 unpairedMsaPath, pairedMsaPath, 和 mmcifPath 字段，允许指定外部的 MSA 和模板文件。
*   `3`: added the option of specifying external user-provided CCD using newly
    added field `userCCDPath`.
    3: 新增了 userCCDPath 字段，允许指定外部的用户自定义 CCD 文件。

## Sequences 序列

The `sequences` section specifies the protein chains, RNA chains, DNA chains,
and ligands. Every entity in `sequences` must have a unique ID. IDs don't have
to be sorted alphabetically.
sequences 部分指定了蛋白质、RNA、DNA 链和配体。每个实体必须有唯一的 ID。

### Protein 蛋白

Specifies a single protein chain.

```json
{
  "protein": {
    "id": "A",
    "sequence": "PVLSCGEWQL",
    "modifications": [
      {"ptmType": "HY3", "ptmPosition": 1},
      {"ptmType": "P1L", "ptmPosition": 5}
    ],
    "unpairedMsa": ...,  # Mutually exclusive with unpairedMsaPath.// 与 unpairedMsaPath 互斥
    "unpairedMsaPath": ...,  # Mutually exclusive with unpairedMsa. // 与 unpairedMsa 互斥
    "pairedMsa": ...,  # Mutually exclusive with pairedMsaPath. // 与 pairedMsaPath 互斥
    "pairedMsaPath": ...,  # Mutually exclusive with pairedMsa. // 与 pairedMsa 互斥
    "templates": [...]
  }
}
```

The fields specify the following:
这些字段指定了以下内容：

*   `id: str | list[str]`: An uppercase letter or multiple letters specifying
    the unique IDs for each copy of this protein chain. The IDs are then also
    used in the output mmCIF file. Specifying a list of IDs (e.g. `["A", "B",
    "C"]`) implies a homomeric chain with multiple copies.
    id: str | list[str]: 链的唯一 ID（大写字母）。如果是同源多聚体，可提供一个列表 ["A", "B"]。
*   `sequence: str`: The amino-acid sequence, specified as a string that uses
    the 1-letter standard amino acid codes.
    sequence: str: 氨基酸序列（单字母代码）。
*   `modifications: list[ProteinModification]`: An optional list of
    post-translational modifications. Each modification is specified using its
    CCD code and 1-based residue position. In the example above, we see that the
    first residue won't be a proline (`P`) but instead `HY3`.
    翻译后修饰的可选列表。每个修改都使用其CCD代码和基于1的残基位置指定。在上面的例子中，我们看到第一个残基不是脯氨酸（"P"），而是"HY3"。
*   `unpairedMsa: str`: An optional multiple sequence alignment for this chain.
    This is specified using the A3M format (equivalent to the FASTA format, but
    also allows gaps denoted by the hyphen `-` character). See more details
    below.
    此链的可选多序列比对。这是使用A3M格式指定的（相当于FASTA格式，但也允许用连字符“-”表示的间隙）。详见下文。
*   `unpairedMsaPath: str`: An optional path to a file that contains the
    multiple sequence alignment for this chain instead of providing it inline
    using the `unpairedMsa` field. The path can be either absolute, or relative
    to the input JSON path. The file must be in the A3M format, and could be
    either plain text, or compressed using gzip, xz, or zstd.
    指向包含此链的多序列比对的文件的可选路径，而不是使用`unpaidMsa `字段内联提供它。路径可以是绝对的，也可以是相对于输入JSON路径的。文件必须为A3M格式，可以是纯文本，也可以使用gzip、xz或zstd压缩。
*   `pairedMsa: str`: We recommend *not* using this optional field and using the
    `unpairedMsa` for the purposes of pairing. See more details below.
    我们建议*不要*使用此可选字段并使用“unpaidMsa”进行配对。请参阅下面的更多详细信息。
*   `pairedMsaPath: str`: An optional path to a file that contains the multiple
    sequence alignment for this chain instead of providing it inline using the
    `pairedMsa` field. The path can be either absolute, or relative to the input
    JSON path. The file must be in the A3M format, and could be either plain
    text, or compressed using gzip, xz, or zstd.
    指向包含此链的多序列比对的文件的可选路径，而不是使用`pairedMsa`字段内联提供它。路径可以是绝对的，也可以是相对于输入JSON路径的。文件必须为A3M格式，可以是纯文本，也可以使用gzip、xz或zstd压缩。
*   `templates: list[Template]`: An optional list of structural templates. See
    more details below.
    结构模板的可选列表。详见下文。

### RNA

Specifies a single RNA chain. 指定单个RNA链

```json
{
  "rna": {
    "id": "A",
    "sequence": "AGCU",
    "modifications": [
      {"modificationType": "2MG", "basePosition": 1},
      {"modificationType": "5MC", "basePosition": 4}
    ],
    "unpairedMsa": ...,  # Mutually exclusive with unpairedMsaPath.
    "unpairedMsaPath": ...  # Mutually exclusive with unpairedMsa.
  }
}
```

The fields specify the following:
这些字段指定了以下内容：

*   `id: str | list[str]`: An uppercase letter or multiple letters specifying
    the unique IDs for each copy of this RNA chain. The IDs are then also used
    in the output mmCIF file. Specifying a list of IDs (e.g. `["A", "B", "C"]`)
    implies a homomeric chain with multiple copies.
    一个大写字母或多个字母，指定此RNA链每个副本的唯一ID。然后，这些ID也会在输出mmCIF文件中使用。指定一个ID列表（例如`["A"、"B"、"C"]`）意味着一个具有多个拷贝的同源多聚体链。
*   `sequence: str`: The RNA sequence, specified as a string using only the
    letters `A`, `C`, `G`, `U`.
    RNA序列，指定为仅使用字母"A"、"C"、"G"、"U"的字符串。
*   `modifications: list[RnaModification]`: An optional list of modifications.
    Each modification is specified using its CCD code and 1-based base position.
    一个可选的修饰列表。每个修饰都使用其CCD代码和基于1的基准位置指定。
*   `unpairedMsa: str`: An optional multiple sequence alignment for this chain.
    This is specified using the A3M format. See more details below.
    此链的可选多序列比对。这是使用A3M格式指定的。详见下文。
*   `unpairedMsaPath: str`: An optional path to a file that contains the
    multiple sequence alignment for this chain instead of providing it inline
    using the `unpairedMsa` field. The path can be either absolute, or relative
    to the input JSON path. The file must be in the A3M format, and could be
    either plain text, or compressed using gzip, xz, or zstd.
    指向包含此链的多序列比对的文件的可选路径，而不是使用“unpaidMsa”字段内联提供它。路径可以是绝对的，也可以是相对于输入JSON路径的。文件必须为A3M格式，可以是纯文本，也可以使用gzip、xz或zstd压缩。

### DNA

Specifies a single DNA chain.指定单个DNA链

```json
{
  "dna": {
    "id": "A",
    "sequence": "GACCTCT",
    "modifications": [
      {"modificationType": "6OG", "basePosition": 1},
      {"modificationType": "6MA", "basePosition": 2}
    ]
  }
}
```

The fields specify the following:
这些字段指定了以下内容：

*   `id: str | list[str]`: An uppercase letter or multiple letters specifying
    the unique IDs for each copy of this DNA chain. The IDs are then also used
    in the output mmCIF file. Specifying a list of IDs (e.g. `["A", "B", "C"]`)
    implies a homomeric chain with multiple copies.
    一个大写字母或多个字母，指定此DNA链每个副本的唯一ID。然后，这些ID也会在输出mmCIF文件中使用。指定一个ID列表（例如`["A"、"B"、"C"]`）意味着一个具有多个拷贝的同源多聚体链。
*   `sequence: str`: The DNA sequence, specified as a string using only the
    letters `A`, `C`, `G`, `T`.
    DNA序列，指定为仅使用字母"A"、"C"、"G"、"T"的字符串。
*   `modifications: list[DnaModification]`: An optional list of modifications.
    Each modification is specified using its CCD code and 1-based base position.
    一个可选的修饰列表。每个修饰都使用其CCD代码和基于1的基准位置指定。

### Ligands

Specifies a single ligand. Ligands can be specified using 3 different formats:
指定单个配体。配体可以使用3种不同的格式指定：

1.  [CCD code(s)](https://www.wwpdb.org/data/ccd). This is the easiest way to
    specify ligands. Supports specifying covalent bonds to other entities. CCD
    from 2022-09-28 is used. If multiple CCD codes are specified, you may want
    to specify a bond between these and/or a bond to some other entity. See the
    [bonds](#bonds) section below.
    这是指定配体的最简单方法。支持指定与其他实体的共价键。使用2022年9月28日的CCD。如果指定了多个CCD代码，您可能需要指定这些代码之间的bond和/或与其他实体的bond。请参阅下面的[bonds]（#bonds）部分。
2.  [SMILES string](https://en.wikipedia.org/wiki/Simplified_Molecular_Input_Line_Entry_System).
    This enables specifying ligands that are not in CCD. If using SMILES, you
    cannot specify covalent bonds to other entities as these rely on specific
    atom names - see the next option for what to use for this case.
    这使得能够指定不在CCD中的配体。如果使用SMILES，则不能指定与其他实体的共价键，因为这些键依赖于特定的原子名称——请参阅下一个选项以了解在这种情况下使用什么。
3.  User-provided CCD + custom ligand codes. This enables specifying ligands not
    in CCD, while also supporting specification of covalent bonds to other
    entities and backup reference coordinates for when RDKit fails to generate a
    conformer. This offers the most flexibility, but also requires careful
    attention to get all of the details right.
    用户提供的CCD+自定义配体代码。这使得可以指定不在CCD中的配体，同时还支持指定与其他实体的共价键，以及当RDKit无法生成构象时的备份参考坐标。这提供了最大的灵活性，但也需要仔细注意所有细节。

```json
{
  "ligand": {
    "id": ["G", "H", "I"],
    "ccdCodes": ["ATP"]
  }
},
{
  "ligand": {
    "id": "J",
    "ccdCodes": ["LIG-1337"]
  }
},
{
  "ligand": {
    "id": "K",
    "smiles": "CC(=O)OC1C[NH+]2CCC1CC2"
  }
}
```

The fields specify the following:
这些字段指定了以下内容：

*   `id: str | list[str]`: An uppercase letter (or multiple letters) specifying
    the unique ID of this ligand. This ID is then also used in the output mmCIF
    file. Specifying a list of IDs (e.g. `["A", "B", "C"]`) implies a ligand
    that has multiple copies.
    指定此配体唯一ID的大写字母（或多个字母）。然后，此ID也用于输出mmCIF文件。指定ID列表（例如`["A", "B", "C"]`）意味着配体具有多个拷贝。
*   `ccdCodes: list[str]`: An optional list of CCD codes. These could be either
    standard CCD codes, or custom codes pointing to the
    [user-provided CCD](#user-provided-ccd).
    CCD代码的可选列表。这些可以是标准CCD代码，也可以是指向[用户提供的CCD]（#用户提供的CCD）的自定义代码。
*   `smiles: str`: An optional string defining the ligand using a SMILES string.
    The SMILES string must be correctly JSON-escaped.
    使用SMILES字符串定义配体的可选字符串。SMILES串必须正确进行JSON转义。

Each ligand may be specified using CCD codes or SMILES but not both, i.e. for a
given ligand, the `ccdCodes` and `smiles` fields are mutually exclusive.
每个配体可以使用CCD代码或SMILES指定，但不能同时使用两者，即对于给定的配体，“ccdCodes”和“SMILES”字段是互斥的。

#### SMILES string JSON escaping SMILES字符串JSON转义

The SMILES string must be correctly JSON-escaped, in particular the backslash
character must be escaped as two backslashes, otherwise the JSON parser will
fail with a `JSONDecodeError`. For instance, the following SMILES string
`CCC[C@@H](O)CC\C=C\C=C\C#CC#C\C=C\CO` has to be specified as:
SMILES字符串必须正确进行JSON转义，特别是反斜杠字符必须转义为两个反斜杠，否则JSON解析器将失败，并出现"JSONDecodeError"。例如，以下SMILES字符串“CCC[C@@H]（O）CC\C=C\C=C\C#CC#C\C=C:\CO”必须指定为：
```json
{
  "ligand": {
    "id": "A",
    "smiles": "CCC[C@@H](O)CC\\C=C\\C=C\\C#CC#C\\C=C\\CO"
  }
}
```

You can JSON-escape the SMILES string using the
[`jq`](https://github.com/jqlang/jq) command-line tool which should be easily
installable on most Linux systems:
您可以使用[`jq`]对SMILES字符串进行JSON转义(https://github.com/jqlang/jq)命令行工具，应易于在大多数Linux系统上安装：

```bash
jq -R . <<< 'CCC[C@@H](O)CC\C=C\C=C\C#CC#C\C=C\CO'  # Replace with your SMILES.
```

Alternatively, you can use this Python code:
或者，您可以使用以下Python代码：

```python
import json

smiles = r'CCC[C@@H](O)CC\C=C\C=C\C#CC#C\C=C\CO'  # Replace with your SMILES.
print(json.dumps(smiles))
```

#### Reference structure construction with SMILES 使用SMILES构建参考结构

For some ligands and some random seeds, RDKit might fail to generate a
conformer, indicated by the `Failed to construct RDKit reference structure`
error message. In this case, you can either provide a reference structure for
the ligand using the [user-provided CCD Format](#user-provided-ccd-format), or
try increasing the number of RDKit conformer iterations using the
`--conformer_max_iterations=...` flag.
对于某些配体和随机种子，RDKit可能无法生成构象异构体，此时会出现"Failed to construct RDKit reference structure"（未能构建RDKit参考结构）的错误提示。这种情况下，您可以选择使用用户提供的CCD格式为配体提供参考结构，或者尝试通过--conformer_max_iterations=...参数增加RDKit构象迭代次数。

### Ions 离子

Ions are treated as ligands, e.g. a magnesium ion would simply be a ligand with
`ccdCodes: ["MG"]`.
离子被当作配体处理。例如，镁离子可以指定为 {"ligand": {"id": "MG", "ccdCodes": ["MG"]}}。

## Multiple Sequence Alignment 多序列比对

Protein and RNA chains allow setting a custom Multiple Sequence Alignment (MSA).
If not set, the data pipeline will automatically build MSAs for protein and RNA
entities using Jackhmmer/Nhmmer search over genetic databases as described in
the paper.
蛋白质和RNA链允许设置自定义的多序列比对（MSA）。如果没有设置，数据管道将使用Jackhmmer/Nhmmer搜索遗传数据库，自动为蛋白质和RNA实体构建MSA，如论文所述。

### RNA Multiple Sequence Alignment RNA多序列比对

RNA `unpairedMsa` can be either:
RNA'unpaidMsa'可以是：

1.  Unset (or set explicitly to `null`). AlphaFold 3 will build MSA for this RNA
    chain automatically. This is the recommended option.
    取消设置（或显式设置为"null"）。AlphaFold 3将自动为该RNA链构建MSA。这是推荐的选项。
3.  Set to an empty string (`""`). AlphaFold 3 won't build the MSA for this RNA
    chain and the MSA input to the model will be just the RNA chain (equivalent
    to running MSA-free for this RNA chain)
    设置为空字符串（""）。AlphaFold 3不会为该RNA链构建MSA，模型的MSA输入将只是RNA链（相当于对该RNA链运行无MSA）。
5.  Set to a non-empty A3M string. AlphaFold 3 will use the provided MSA for
    this RNA chain.
    设置为非空的A3M字符串。AlphaFold 3将使用此RNA链提供的MSA。

### Protein Multiple Sequence Alignment 蛋白多序列比对

For protein chains, the situation is slightly more complicated due to paired and
unpaired MSA (see [MSA Pairing](#msa-pairing) below for more details).
对于蛋白质链，由于配对和未配对的MSA，情况稍微复杂一些（有关更多详细信息，请参阅下面的[MSA配对]（#MSA配对））。

The following combinations are valid for a given protein chain:
以下组合适用于给定的蛋白质链：

1.  Both `unpairedMsa` and `pairedMsa` fields are unset (or set explicitly to
    `null`), AlphaFold 3 will build both MSAs automatically. This is the
    recommended option.
    "unpaidMsa"和"pairedMsa"字段均未设置（或明确设置为"null"），AlphaFold 3将自动构建这两个MSA。这是推荐的选项。
2.  The `unpairedMsa` is set to to a non-empty A3M string, `pairedMsa` set to an
    empty string (`""`). AlphaFold 3 won't build MSA, will use the `unpairedMsa`
    as is and run `pairedMSA`-free.
    当unpairedMsa设置为非空A3M字符串，且pairedMsa设置为空字符串("")时：AlphaFold 3将不会构建多序列比对（MSA），而是直接使用提供的unpairedMsa，并在无配对MSA的模式下运行。
3.  The `pairedMsa` is set to to a non-empty A3M string, `unpairedMsa` set to an
    empty string (`""`). AlphaFold 3 won't build MSA, will use the `pairedMsa`
    and run `unpairedMSA`-free. **This option is not recommended**, see
    [MSA Pairing](#msa-pairing) below.
    当pairedMsa设置为非空A3M字符串，且unpairedMsa设置为空字符串("")时：AlphaFold 3将不会构建多序列比对，直接使用提供的pairedMsa，并在无非配对MSA的模式下运行。此选项不推荐使用，详见下文MSA配对章节。
4.  Both `unpairedMsa` and `pairedMsa` fields are set to an empty string (`""`).
    AlphaFold 3 will not build the MSA and the MSA input to the model will be
    just the query sequence (equivalent to running completely MSA-free).
    当unpairedMsa和pairedMsa均设置为空字符串("")时：AlphaFold 3将完全不构建多序列比对，模型的MSA输入将仅为查询序列（相当于完全无MSA的运行模式）。
5.  Both `unpairedMsa` and `pairedMsa` fields are set to a custom non-empty A3M
    string, AlphaFold 3 will use the provided MSA instead of building one as
    part of the data pipeline. This is considered an expert option.
    当unpairedMsa和pairedMsa均设置为自定义的非空A3M字符串时：AlphaFold 3将使用提供的MSA数据，而不再通过数据流程自动构建。这被视为专家级选项。

Note that both `unpairedMsa` and `pairedMsa` have to either be *both* set (i.e.
non-`null`), or both unset (i.e. both `null`, explicitly or implicitly).
Typically, when setting `unpairedMsa`, you will set the `pairedMsa` to an empty
string (`""`). For example this will run the protein chain A with the given MSA,
but without any templates (template-free):
请注意：unpairedMsa和pairedMsa必须同时设置（即均为非null值）或同时不设置（即显式或隐式均为null）。通常情况下，当设置unpairedMsa时，您需要将pairedMsa设为空字符串（""）。例如以下配置将使用给定的MSA运行蛋白质链A的预测，且不使用任何模板（无模板模式）：

```json
{
  "protein": {
    "id": "A",
    "sequence": ...,
    "unpairedMsa": "The A3M you want to run with",
    "pairedMsa": "",
    "templates": []
  }
}
```

When setting your own MSA, you have to make sure that:
在设置自己的MSA时，您必须确保：

1.  The MSA is in the A3M format. This means adhering to the FASTA format while
    also allowing lowercase characters denoting inserted residues and hyphens
    (`-`) denoting gaps in sequences.
    MSA需采用A3M格式。这意味着需遵循FASTA格式规范，同时允许使用小写字符表示插入残基，连字符（-）表示序列中的缺失。
2.  The first sequence is exactly equal to the query sequence.
    第一条序列必须与查询序列完全一致。
3.  If all insertions are removed from MSA hits (i.e. all lowercase letters are
    removed), all sequences have exactly the same length as the query (they form
    an exact rectangular matrix).
    若从MSA匹配序列中移除所有插入残基（即删除所有小写字母），所有序列必须与查询序列保持完全相同的长度（形成严格的矩形矩阵）。

### MSA Pairing MSA配对

MSA pairing matters only when folding multiple chains (multimers), since we need
to find a way to concatenate MSAs for the individual chains along the sequence
dimension. If done naively, by simply concatenating the individual MSA matrices
along the sequence dimension and padding so that all MSAs have the same depth,
one can end up with rows in the concatenated MSA that are formed by sequences
from different organisms.
MSA配对仅在进行多链（多聚体）折叠时才有意义，因为我们需要找到一种方法将各个链的MSA沿序列维度进行拼接。如果简单地通过沿序列维度直接拼接单个MSA矩阵并进行填充以使所有MSA具有相同深度，最终可能导致拼接后的MSA中出现由不同生物体序列组成的行。

It may be desirable to ensure that across multiple chains, sequences in the MSA
that are from the same organism end up in the same MSA row. AlphaFold 3
internally achieves this by looking for the UniProt organism ID in the
`pairedMsa` and pairing sequences based on this information.
通常需要确保跨多链时，来自同一生物体的MSA序列位于同一MSA行中。AlphaFold 3通过在pairedMsa中查找UniProt生物体ID并基于此信息进行序列配来实现这一点。

We recommend users do the pairing manually or use the output of an appropriate
software and then provide the MSA using only the `unpairedMsa` field. This
method gives exact control over the placement of each sequence in the MSA, as
opposed to relying on name-matching post-processing heuristics used for
`pairedMsa`.
我们建议用户手动进行配对或使用相应软件的输出结果，然后仅通过unpairedMsa字段提供MSA。这种方法可以精确控制每个序列在MSA中的位置，而不依赖于pairedMsa所使用的名称匹配后处理启发式方法。

When setting `unpairedMsa` manually, the `pairedMsa` must be explicitly set to
an empty string (`""`).
手动设置unpairedMsa时，必须将pairedMsa显式设置为空字符串（""）。

Make sure to run with `--resolve_msa_overlaps=false`. This prevents
deduplication of the unpaired MSA within each chain against the paired MSA
sequences. Even if you set `pairedMsa` to an empty string, the query sequence(s)
will still be added in there and the deduplication procedure could destroy the
carefully crafted sequence positioning in the unpaired MSA.
请确保使用--resolve_msa_overlaps=false参数运行。这可以防止在每个链内针对配对MSA序列进行未配对MSA的去重处理。即使将pairedMsa设置为空字符串，查询序列仍会被添加进去，而去重过程可能会破坏未配对MSA中精心设计的序列定位。

For instance, if there are two chains `DEEP` and `MIND` which we want to be
paired on organism A and C, we can achieve it as follows:
例如，如果有两个链DEEP和MIND，我们希望将它们与生物体A和C配对，可以通过以下方式实现：

```txt
> query
DEEP
> match 1 (organism A)
D--P
> match 2 (organism B)
DD-P
> match 3 (organism C)
DD-P
```

```txt
> query
MIND
> match 1 (organism A)
M--D
> Empty hit to make sure pairing is achieved
----
> match 2 (organism C)
MIN-
```

The resulting MSA when chains are concatenated will then be:
链拼接时产生的MSA将是：

```txt
> query
DEEPMIND
> match 1 + match 1
D--PM--D
> match 2 + padding
DD-P----
> match 3 + match 2
DD-PMIN-
```

## Structural Templates 结构模板

Structural templates can be specified only for protein chains:
只能为蛋白质链指定结构模板：

```json
"templates": [
  {
    "mmcif": ...,  # Mutually exclusive with mmcifPath.
    "mmcifPath": ...,  # Mutually exclusive with mmcif.
    "queryIndices": [0, 1, 2, 4, 5, 6],
    "templateIndices": [0, 1, 2, 3, 4, 8]
  }
]
```

The fields specify the following:
这些字段指定了以下内容：

*   `mmcif: str`: A string containing the single chain protein structural
    template in the mmCIF format.
    包含mmCIF格式单链蛋白质结构模板的字符串
*   `mmcifPath: str`: An optional path to a file that contains the mmCIF with
    the structural template instead of providing it inline using the `mmcifPath`
    field. The path can be either absolute, or relative to the input JSON path.
    The file must be in the mmCIF format, and could be either plain text, or
    compressed using gzip, xz, or zstd.
    一个可选的文件路径，包含结构模板的 mmCIF 数据，用于替代在 mmcifPath 字段中直接提供内联数据。该路径可以是绝对路径，也可以是相对于输入 JSON 文件路径的相对路径。文件必须为mmCIF格式，支持纯文本或使用gzip/xz/zstd压缩格式
*   `queryIndices: list[int]`: O-based indices in the query sequence, defining
    the mapping from query residues to template residues.
    查询序列中基于0的索引，定义从查询残基到模板残基的映射关系
*   `templateIndices: list[int]`: O-based indices in the template sequence,
    specifying the mapping from query residues to template residues defined in
    the mmCIF file. Note that unresolved mmCIF residues must be taken into
    account when specifying template indices.
    模板序列中基于0的索引，指定查询残基与mmCIF文件中模板残基的映射关系。注意：指定模板索引时必须考虑mmCIF中未解析的残基

A template is specified as an mmCIF string containing a single chain with the
structural template together with a 0-based mapping that maps query residue
indices to the template residue indices. The mapping is specified using two
lists of the same length. E.g. to express a mapping `{0: 0, 1: 2, 2: 5, 3: 6}`,
you would specify the two indices lists as:
模板被指定为一个 mmCIF 格式的字符串，其中包含一个带有结构模板的单链，以及一个从 0 开始的映射，该映射将查询序列的残基索引映射到模板序列的残基索引。该映射通过两个等长的列表来指定。例如，要表示映射 {0: 0, 1: 2, 2: 5, 3: 6}，您需要像下面这样指定两个索引列表：

```json
"queryIndices":    [0, 1, 2, 3],
"templateIndices": [0, 2, 5, 6]
```

Note that mmCIFs can have residues with missing atom coordinates (present in
residue tables but missing in the `_atom_site` table) – these must be taken into
account when specifying template indices. E.g. to align residues 4–7 in a
template with unresolved residues 1, 2, 3 and resolved residues 4, 5, 6, 7, you
need to set the template indices to 3, 4, 5, 6 (since 0-based indexing is used).
An example of a protein with unresolved residues 1–20 can be found here:
https://www.rcsb.org/structure/8UXY.
请注意，mmCIF 文件中可能包含缺少原子坐标的残基（即这些残基存在于残基表中，但缺失于 _atom_site 表中）——在指定模板索引时必须考虑到这一点。例如，要将一个模板中未解析的残基 1、2、3 和已解析的残基 4、5、6、7 与查询序列对齐，您需要将模板索引设置为 3, 4, 5, 6（因为使用的是从 0 开始的索引）。一个包含未解析残基 1-20 的蛋白质示例可以在这里找到：https://www.rcsb.org/structure/8UXY 。

You can provide multiple structural templates. Note that if an mmCIF containing
more than one chain is provided, you will get an error since it is not possible
to determine which of the chains should be used as the template.
您可以提供多个结构模板。请注意，如果提供的 mmCIF 文件包含不止一条链，您将会收到一个错误，因为程序无法确定应该使用哪条链作为模板。

You can run template-free (but still run genetic search and build MSA) by
setting templates to `[]` and either explicitly setting both `unpairedMsa` and
`pairedMsa` to `null`:
您也可以在无模板的情况下运行（但仍然执行遗传搜索和构建多序列比对 MSA），方法是将 templates 设置为 []，并明确地将 unpairedMsa 和 pairedMsa 都设置为 null：

```json
"protein": {
  "id": "A",
  "sequence": ...,
  "pairedMsa": null,
  "unpairedMsa": null,
  "templates": []
}
```

Or you can simply fully omit them:
或者，您可以直接完全省略它们：

```json
"protein": {
  "id": "A",
  "sequence": ...,
  "templates": []
}
```

You can also run with pre-computed MSA, but let AlphaFold 3 search for
templates. This can be achieved by setting `unpairedMsa` and `pairedMsa`, but
keeping templates unset (or set to `null`). The profile given as an input to
Hmmsearch when searching for templates will be built from the provided
`unpairedMsa`:
您还可以使用预先计算好的 MSA 运行，但让 AlphaFold 3 自己搜索模板。这可以通过设置 unpairedMsa 和 pairedMsa，同时保持 templates 未设置（或设为 null）来实现。在搜索模板时，输入到 Hmmsearch 的配置文件将从您提供的 unpairedMsa 构建而来：

```json
"protein": {
  "id": "A",
  "sequence": ...,
  "unpairedMsa": ...,
  "pairedMsa": ...,
  "templates": null
}
```

Or you can simply fully omit the `templates` field thus setting it implicitly to
`null`:
或者，您可以直接完全省略 templates 字段，从而隐式地将其设置为 null：

```json
"protein": {
  "id": "A",
  "sequence": ...,
  "unpairedMsa": ...,
  "pairedMsa": ...,
}
```

## Bonds 键

To manually specify covalent bonds, use the `bondedAtomPairs` field. This is
intended for modelling covalent ligands, and for defining multi-CCD ligands
(e.g. glycans). Defining covalent bonds between or within polymer entities is
not currently supported.
要手动指定共价键，请使用"bonded AtomPirs"字段。这是为了模拟共价配体，并定义多CCD配体（如聚糖）。目前尚不支持定义聚合物实体之间或内部的共价键。

Bonds are specified as pairs of (source atom, destination atom), with each atom
being uniquely addressed using 3 fields:
键被指定为成对（源原子、目标原子），每个原子使用3个字段进行唯一寻址：

*   **Entity ID** (`str`): this corresponds to the `id` field for that entity.
*   这对应于该实体的"id"字段。
*   **Residue ID** (`int`): this is 1-based residue index *within* the chain.
    For single-residue ligands, this is simply set to 1.
     残基ID (int): 这是链内从1开始计数的残基索引。对于单残基配体，直接设置为1即可。
*   **Atom name** (`str`): this is the unique atom name *within* the given
    residue. The atom name for protein/RNA/DNA residues or CCD ligands can be
    looked up in the CCD for the given chemical component. This also explains
    why SMILES ligands don't support bonds: there is no atom name that could be
    used to define the bond. This shortcoming can be addressed by using the
    user-provided CCD format (see below).
    原子名称 (str): 这是给定残基内唯一的原子名称。蛋白质/RNA/DNA残基或CCD配体的原子名称可在相应化学组分的CCD中查询。这也解释了为何SMILES配体不支持化学键定义：因为没有可用于定义化学键的原子名称。此缺陷可通过使用用户提供的CCD格式来解决（详见下文）。

The example below shows two bonds:
下面的例子显示了两个键：

```json
"bondedAtomPairs": [
  [["A", 145, "SG"], ["L", 1, "C04"]],
  [["J", 1, "O6"], ["J", 2, "C1"]]
]
```

The first bond is between chain A, residue 145, atom SG and chain L, residue 1,
atom C04. This is a typical example for a covalent ligand. The second bond is
between chain J, residue 1, atom O6 and chain J, residue 2, atom C1. This bond
is within the same entity and is a typical example when defining a glycan.
第一个键位于链A，残基145，原子SG和链L，残基1，原子C04之间。这是共价配体的一个典型例子。第二个键位于链J，残基1，原子O6和链J，残基2，原子C1之间。这种键位于同一实体内，是定义聚糖的典型例子。

All bonds are implicitly assumed to be covalent bonds. Other bond types are not
supported.
所有键都隐含地假设为共价键。不支持其他键类型。

### Defining Glycans 定义聚糖

Glycans are bound to a protein residue, and they are typically formed of
multiple chemical components. To define a glycan, define a new ligand with all
of the chemical components of the glycan. Then define a bond that links the
glycan to the protein residue, and all bonds that are within the glycan between
its individual chemical components.
聚糖通常与蛋白质残基结合，且由多个化学组分构成。要定义聚糖，需要创建一个包含该聚糖所有化学组分的新配体。然后定义一条将聚糖连接到蛋白质残基的化学键，以及聚糖内部各化学组分之间的所有化学键。

For example, to define the following glycan composed of 4 components (CMP1,
CMP2, CMP3, CMP4) bound to an asparagine in a protein chain A:
例如，要定义由4个组分（CMP1、CMP2、CMP3、CMP4）构成且与蛋白质链A的天冬酰胺残基连接的聚糖：

```
 ⋮
ALA            CMP4
 |              |
ASN ―― CMP1 ―― CMP2
 |              |
ALA            CMP3
 ⋮
```

You will need to specify:
您需要指定：

1.  Protein chain A.
2.  Ligand chain B with the 4 components.
3.  Bonds ASN-CMP1, CMP1-CMP2, CMP2-CMP3, CMP2-CMP4.

## User-provided CCD 用户提供的CCD

There are two approaches to model a custom ligand not defined in the CCD:
有两种方法可以建模CCD中未定义的自定义配体：

1.  If the ligand is not bonded to other entities, it can be defined using a
    [SMILES string](https://en.wikipedia.org/wiki/Simplified_Molecular_Input_Line_Entry_System).
    如果配体没有与其他实体结合，则可以使用[SMILES字符串]来定义(https://en.wikipedia.org/wiki/Simplified_Molecular_Input_Line_Entry_System).
2.  If it is bonded to other entities, or to be able to customise relevant
    features (such as bond orders, atom names and ideal coordinates used when
    conformer generation fails), it is necessary to define that particular
    ligand using the
    [CCD mmCIF format](https://www.wwpdb.org/data/ccd#mmcifFormat).
    如果它与其他实体结合，或者能够定制相关特征（如键序、原子名称和构象生成失败时使用的理想坐标），则有必要使用[CCD mmCIF格式]定义特定的配体(https://www.wwpdb.org/data/ccd#mmcifFormat).

Note that if a full CCD mmCIF is provided, any SMILES string input as part of
that mmCIF is ignored.
请注意，如果提供了完整的CCD mmCIF，则忽略作为该mmCIF一部分的任何SMILES字符串输入。

Once defined, this ligand needs to be assigned a name that doesn't clash with
existing CCD ligand names (e.g. `LIG-1`). Avoid underscores (`_`) in the name,
as it could cause issues in the mmCIF format.
一旦定义，该配体需要分配一个与现有CCD配体名称不冲突的名称（例如“LIG-1”）。避免在名称中使用下划线（“_”），因为这可能会导致mmCIF格式出现问题。

The newly defined ligand can then be used as a standard CCD ligand using its
custom name, and bonds can be linked to it using its named atom scheme.
然后，新定义的配体可以使用其自定义名称用作标准CCD配体，并且可以使用其命名的原子方案将键连接到它。

### Conformer Generation 构象生成

The data pipeline attempts to generate a conformer for ligands using RDKit. The
`Mol` used to generate the conformer is constructed either from the information
provided in the CCD mmCIF, or from the SMILES string if that is the only
information provided.
数据流程会尝试使用RDKit为配体生成构象异构体。用于生成构象的Mol对象要么根据CCD mmCIF文件中提供的信息构建，要么在仅提供SMILES字符串时根据SMILES信息构建。

If conformer generation fails, the model will fall back to using the ideal
coordinates in the CCD mmCIF if these are provided. If they are not provided,
the model will use the reference coordinates if the last modification date given
in the CCD mmCIF is prior to the training cutoff date. If no coordinates can be
found in this way, all conformer coordinates are set to zero and the model will
output `NaN` (`null` in the output JSON) confidences for the ligand.
如果构象生成失败，模型将回退使用CCD mmCIF中提供的理想坐标（若已提供）。如果未提供理想坐标，但CCD mmCIF中给出的最后修改日期早于训练截止日期，模型将使用参考坐标。如果通过以上方式均无法获取坐标，所有构象坐标将被设置为零，并且模型将为该配体输出NaN（在输出JSON中显示为null）置信度。

Note that sometimes conformer generation failures can be resolved by
increasinging the number of RDKit conformer iterations using the
`--conformer_max_iterations=...` flag.
请注意，有时通过使用--conformer_max_iterations=...参数增加RDKit构象迭代次数，可以解决构象生成失败的问题。

### User-provided CCD Format 用户提供的CCD格式

The user-provided CCD must be passed either:
用户提供的CCD必须通过以下任一方式：

*   In the `userCCD` field (in the root of the input JSON) as a string. Note
    that JSON doesn't allow newlines within strings, so newline characters
    (`\n`) must be used to delimit lines. Single rather than double quotes
    should also be used around strings like the chemical formula.
    在userCCD字段中（位于输入JSON的根级别）以字符串形式提供。注意JSON不允许字符串内包含换行符，因此必须使用换行符（\n）来分隔行。对于化学式等字符串内容，应使用单引号而非双引号。
*   In the `userCCDPath` field, as a path to a file that contains the
    user-provided chemical components dictionary. The path can be either
    absolute, or relative to the input JSON path. The file must be in the
    [CCD mmCIF format](https://www.wwpdb.org/data/ccd#mmcifFormat), and could be
    either plain text, or compressed using gzip, xz, or zstd.
    在userCCDPath字段中，以文件路径形式提供用户自定义化学组分词典。路径可以是绝对路径，或相对于输入JSON文件的路径。文件必须符合CCD mmCIF格式，可以是纯文本格式，或使用gzip、xz、zstd压缩。
    
The main pieces of information used are the atom names and elements, bonds, and
also the ideal coordinates (`pdbx_model_Cartn_{x,y,z}_ideal`) which essentially
serve as a structural template for the ligand if RDKit fails to generate
conformers for that ligand.
系统主要使用以下信息：原子名称与元素、化学键，以及理想坐标（pdbx_model_Cartn_{x,y,z}_ideal）。当RDKit无法为配体生成构象时，这些理想坐标将作为配体的结构模板。

The user-provided CCD can also be used to redefine standard chemical components
in the CCD. This can be useful if you need to redefine the ideal coordinates.
用户提供的CCD还可用于重新定义标准化学组分。当需要重新定义理想坐标时，这一功能非常有用。

Below is an example user-provided CCD redefining component X7F, which serves to
illustrate the required sections. For readability purposes, newlines have not
been replaced by `\n`.
以下是一个重新定义组分X7F的用户CCD示例，展示了必需的字段。为便于阅读，此处未将换行符替换为\n。

```
data_MY-X7F
#
_chem_comp.id MY-X7F
_chem_comp.name '5,8-bis(oxidanyl)naphthalene-1,4-dione'
_chem_comp.type non-polymer
_chem_comp.formula 'C10 H6 O4'
_chem_comp.mon_nstd_parent_comp_id ?
_chem_comp.pdbx_synonyms ?
_chem_comp.formula_weight 190.152
#
loop_
_chem_comp_atom.comp_id
_chem_comp_atom.atom_id
_chem_comp_atom.type_symbol
_chem_comp_atom.charge
_chem_comp_atom.pdbx_leaving_atom_flag
_chem_comp_atom.pdbx_model_Cartn_x_ideal
_chem_comp_atom.pdbx_model_Cartn_y_ideal
_chem_comp_atom.pdbx_model_Cartn_z_ideal
MY-X7F C02 C 0 N -1.418 -1.260 0.018
MY-X7F C03 C 0 N -0.665 -2.503 -0.247
MY-X7F C04 C 0 N 0.677 -2.501 -0.235
MY-X7F C05 C 0 N 1.421 -1.257 0.043
MY-X7F C06 C 0 N 0.706 0.032 0.008
MY-X7F C07 C 0 N -0.706 0.030 -0.004
MY-X7F C08 C 0 N -1.397 1.240 -0.037
MY-X7F C10 C 0 N -0.685 2.443 -0.057
MY-X7F C11 C 0 N 0.679 2.445 -0.045
MY-X7F C12 C 0 N 1.394 1.243 -0.013
MY-X7F O01 O 0 N -2.611 -1.301 0.247
MY-X7F O09 O 0 N -2.752 1.249 -0.049
MY-X7F O13 O 0 N 2.750 1.257 -0.001
MY-X7F O14 O 0 N 2.609 -1.294 0.298
MY-X7F H1 H 0 N -1.199 -3.419 -0.452
MY-X7F H2 H 0 N 1.216 -3.416 -0.429
MY-X7F H3 H 0 N -1.221 3.381 -0.082
MY-X7F H4 H 0 N 1.212 3.384 -0.062
MY-X7F H5 H 0 N -3.154 1.271 0.830
MY-X7F H6 H 0 N 3.151 1.241 -0.880
#
loop_
_chem_comp_bond.atom_id_1
_chem_comp_bond.atom_id_2
_chem_comp_bond.value_order
_chem_comp_bond.pdbx_aromatic_flag
O01 C02 DOUB N
O09 C08 SING N
C02 C03 SING N
C02 C07 SING N
C03 C04 DOUB N
C08 C07 DOUB Y
C08 C10 SING Y
C07 C06 SING Y
C10 C11 DOUB Y
C04 C05 SING N
C06 C05 SING N
C06 C12 DOUB Y
C11 C12 SING Y
C05 O14 DOUB N
C12 O13 SING N
C03 H1 SING N
C04 H2 SING N
C10 H3 SING N
C11 H4 SING N
O09 H5 SING N
O13 H6 SING N
#
```

### Mandatory fields 必填字段

Parsing the user-provided CCD needs only a subset of the fields that CCD uses.
The mandatory fields are described below. Refer to
[CCD documentation](https://www.wwpdb.org/data/ccd#mmcifFormat) for more
detailed explanation of each field. Note that not all of these fields are input
to the model, but they are necessary for the data pipeline to run – see the
[Model input fields](#model-input-fields) section below.
解析用户提供的CCD仅需使用CCD中的部分字段。必需字段如下所述。请参阅CCD文档获取各字段的详细说明。请注意，并非所有这些字段都会输入模型，但它们是数据流程运行所必需的——详见下文模型输入字段章节。

**Singular fields (containing just a single value)** 单数字段（仅包含单个值）

*   `_chem_comp.id`: The ID of the component. Must match the `_data` record and
    must not contain special CIF characters (like `_` or `#`).
*   `_chem_comp.name`: Optional full name of the component. If unknown, set to
    `?`.
*   `_chem_comp.type`: Type of the component, typically `non-polymer`.
*   `_chem_comp.formula`: Optional component formula. If unknown, set to `?`.
*   `_chem_comp.mon_nstd_parent_comp_id`: Optional parent component ID. If
    unknown, set to `?`.
*   `_chem_comp.pdbx_synonyms`: Optional synonym IDs. If unknown, set to `?`.
*   `_chem_comp.formula_weight`: Optional weight of the component. If unknown,
    set to `?`.

**Per-atom fields (containing one record per atom)** 每个原子字段（每个原子包含一条记录）

*   `_chem_comp_atom.comp_id`: Component ID.
*   `_chem_comp_atom.atom_id`: Atom ID.
*   `_chem_comp_atom.type_symbol`: Atom element type.
*   `_chem_comp_atom.charge`: Atom charge.
*   `_chem_comp_atom.pdbx_leaving_atom_flag`: Optional flag determining whether
    this is a leaving atom. If unset, assumed to be no (`N`) for all atoms.
*   `_chem_comp_atom.pdbx_model_Cartn_x_ideal`: Ideal x coordinate.
*   `_chem_comp_atom.pdbx_model_Cartn_y_ideal`: Ideal y coordinate.
*   `_chem_comp_atom.pdbx_model_Cartn_z_ideal`: Ideal z coordinate.

**Per-bond fields (containing one record per bond)** 每个键字段（每个键包含一条记录）

*   `_chem_comp_bond.atom_id_1`: The ID of the first of the two atoms that
    define the bond.
*   `_chem_comp_bond.atom_id_2`: The ID of the second of the two atoms that
    define the bond.
*   `_chem_comp_bond.value_order`: The bond order of the chemical bond
    associated with the specified atoms.
*   `_chem_comp_bond.pdbx_aromatic_flag`: Whether the bond is aromatic.

### Model input fields 模型输入字段

The following fields are used to generate input for the model:
以下字段用于为模型生成输入：

*   `_chem_comp_atom.atom_id`: Atom ID.
*   `_chem_comp_atom.type_symbol`: Atom element type.
*   `_chem_comp_atom.charge`: Atom charge.
*   `_chem_comp_atom.pdbx_model_Cartn_x_ideal`: Ideal x coordinate. Only used if
    conformer generation fails.
*   `_chem_comp_atom.pdbx_model_Cartn_y_ideal`: Ideal y coordinate. Only used if
    conformer generation fails.
*   `_chem_comp_atom.pdbx_model_Cartn_z_ideal`: Ideal z coordinate. Only used if
    conformer generation fails.
*   `_chem_comp_bond.atom_id_1`: The ID of the first of the two atoms that
    define the bond.
*   `_chem_comp_bond.atom_id_2`: The ID of the second of the two atoms that
    define the bond.

## Full Example

An example illustrating all the aspects of the input format is provided below.
Note that AlphaFold 3 won't run this input out of the box as it abbreviates
certain fields and the sequences are not biologically meaningful.
下面提供了一个示例，说明输入格式的所有方面。请注意，AlphaFold 3不会开箱即用地运行此输入，因为它缩写了某些字段，并且序列在生物学上没有意义。

```json
{
  "name": "Hello fold",
  "modelSeeds": [10, 42],
  "sequences": [
    {
      "protein": {
        "id": "A",
        "sequence": "PVLSCGEWQL",
        "modifications": [
          {"ptmType": "HY3", "ptmPosition": 1},
          {"ptmType": "P1L", "ptmPosition": 5}
        ],
        "unpairedMsa": ...,
        "pairedMsa": ""
      }
    },
    {
      "protein": {
        "id": "B",
        "sequence": "RPACQLW",
        "templates": [
          {
            "mmcif": ...,
            "queryIndices": [0, 1, 2, 4, 5, 6],
            "templateIndices": [0, 1, 2, 3, 4, 8]
          }
        ]
      }
    },
    {
      "dna": {
        "id": "C",
        "sequence": "GACCTCT",
        "modifications": [
          {"modificationType": "6OG", "basePosition": 1},
          {"modificationType": "6MA", "basePosition": 2}
        ]
      }
    },
    {
      "rna": {
        "id": "E",
        "sequence": "AGCU",
        "modifications": [
          {"modificationType": "2MG", "basePosition": 1},
          {"modificationType": "5MC", "basePosition": 4}
        ],
        "unpairedMsa": ...
      }
    },
    {
      "ligand": {
        "id": ["F", "G", "H"],
        "ccdCodes": ["ATP"]
      }
    },
    {
      "ligand": {
        "id": "I",
        "ccdCodes": ["NAG", "FUC"]
      }
    },
    {
      "ligand": {
        "id": "Z",
        "smiles": "CC(=O)OC1C[NH+]2CCC1CC2"
      }
    }
  ],
  "bondedAtomPairs": [
    [["A", 1, "CA"], ["G", 1, "CHA"]],
    [["I", 1, "O6"], ["I", 2, "C1"]]
  ],
  "userCCD": ...,
  "dialect": "alphafold3",
  "version": 3
}

```
