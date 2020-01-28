# HASLR: fast hybrid assembly of long reads

- [Introduction](#about)
- [Installation](#installation)
  - [Requirements](#requirements)
  - [Dependencies](#dependencies)
  - [Using conda](#bioconda)
  - [Building from source](#building)
- [Command line](#command)
  - [Options](#options)
- [Quick start](#quickstart)
  - [Output files](#output)
- [Preprint](#publication)
- [Bug report](#bugs)
- [Copyright and License](#license)
- [Author](#author)

## <a name="about"></a>Introduction
HASLR is a tool for rapid genome assembly of long sequencing reads. HASLR is a hybrid tool which means it requires long reads generated by Third Generation Sequencing technologies (such as PacBio or Oxford Nanopore) together with Next Generation Sequencing reads (such as Illumina) from the same sample. HASLR is capable of assembling large genomes on a single computing node. Our experiments show that it can assemble a CHM1 human dataset in less than 10 hours using 64 CPU threads.

## <a name="installation"></a>Installation

### <a name="requirements"></a>Requirements
- GCC ≥ 4.8.5
- Python3
- zlib

### <a name="dependencies"></a>Dependencies
HASLR depends on the following tools which will be installed automatically:
- [SPOA](https://github.com/rvaser/spoa) - For consensus calling of long reads
- [Minia](https://github.com/GATB/minia) - For assembling short reads
- [minimap2](https://github.com/lh3/minimap2) - For aligning short read contigs onto long reads
- [fastutils](https://github.com/haghshenas/fastutils) - For FASTA/Q manipulation

### <a name="bioconda"></a>Using conda
HASLR can be installed using conda package manager via bioconda channel:

```bash
conda install -c bioconda haslr
```

### <a name="building"></a>Building from source
```
git clone https://github.com/vpc-ccg/haslr.git
cd haslr
make
```
After a successful build, the content of `bin` directory should be as the following:
```
bin/fastutils
bin/haslr_assemble
bin/haslr.py
bin/minia
bin/minimap2
bin/nooverlap
```
Note that `bin/haslr.py` is the main python wrapper of HASLR.

## <a name="command"></a>Command line
```
haslr.py [-t THREADS] -o OUT_DIR -g GENOME_SIZE -l LONG -x LONG_TYPE -s SHORT [SHORT ...]
```

### <a name="options"></a>Options
```
required arguments:
  -o, --out OUT_DIR              output directory
  -g, --genome GENOME_SIZE       estimated genome size; accepted suffixes are k,m,g
  -l, --long LONG                long read file
  -x, --type LONG_TYPE           type of long reads chosen from {pacbio,nanopore}
  -s, --short SHORT [SHORT ...]  short read file

optional arguments:
  -t, --threads THREADS          number of CPU threads to use [1]
  --cov-lr COV_LR                amount of long read coverage to use for assembly [25]
  --aln-block ALN_BLOCK          minimum length of alignment block [500]
  --aln-sim ALN_SIM              minimum alignment similarity [0.85]
  --edge-sup EDGE_SUP            minimum number of long read supporting each edge [3]
  --minia-kmer MINIA_KMER        kmer size used by minia [49]
  --minia-solid MINIA_SOLID      minimum kmer abundance used by minia [3]
  --minia-asm MINIA_ASM          type of minia assembly chosen from {contigs,unitigs} [contigs]
  -v, --version                  print version
  -h, --help                     show this help message and exit
```

## <a name="quickstart"></a>Quick start
Here we assemble a sample *E. coli* dataset of PacBio and Illumina reads:
```bash
# download PacBio reads
wget http://gembox.cbcb.umd.edu/mhap/raw/ecoli_filtered.fastq.gz
# download Illumina reads
wget http://gembox.cbcb.umd.edu/mhap/raw/ecoli_miseq.1.fastq.gz
wget http://gembox.cbcb.umd.edu/mhap/raw/ecoli_miseq.2.fastq.gz
# run HASLR using 8 threads
haslr.py -t 8 -o ecoli -g 4.6m -l ecoli_filtered.fastq.gz -x pacbio -s ecoli_miseq.1.fastq.gz ecoli_miseq.2.fastq.gz
```

### <a name="output"></a>Output files
With a successful run, the structure of the output directory will be as follows:
```
ecoli                                               # output directory
├── asm_contigs_k49_a3_lr25x_b500_s3_sim0.85        # output directory containing long read assembly files
│   ├── asm.final.ann                               # annotation of the final assembly
│   ├── asm.final.fa                                # finall assembly in FASTA format
│   ├── backbone.01.init.gfa                        # initial backbone graph
│   ├── backbone.01.init.stat                       # statistics of graph stored in backbone.01.init.gfa
│   ├── backbone.02.weakEdge.gfa                    # backbone graph after removing weak edges
│   ├── backbone.02.weakEdge.stat                   # statistics of graph stored in backbone.02.weakEdge.gfa
│   ├── backbone.03.tip.gfa                         # backbone graph after tip removal
│   ├── backbone.03.tip.log                         # log of HASLR (step: tip removal)
│   ├── backbone.03.tip.stat                        # statistics of graph stored in backbone.03.tip.gfa
│   ├── backbone.04.simplebubble.gfa                # backbone graph after simple bubble removal
│   ├── backbone.04.simplebubble.log                # log of HASLR (step: simple bubble removal)
│   ├── backbone.04.simplebubble.stat               # statistics of graph stored in backbone.04.simplebubble.gfa
│   ├── backbone.05.superbubble.gfa                 # backbone graph after super bubble removal
│   ├── backbone.05.superbubble.log                 # log of HASLR (step: super bubble removal)
│   ├── backbone.05.superbubble.stat                # statistics of graph stored in backbone.05.superbubble.gfa
│   ├── backbone.06.smallbubble.gfa                 # backbone graph after small bubble removal 
│   ├── backbone.06.smallbubble.log                 # log of HASLR (step: small bubble removal)
│   ├── backbone.06.smallbubble.stat                # statistics of graph stored in backbone.06.smallbubble.gfa
│   ├── backbone.branching.log                      # list of branching nodes in backbone graph
│   ├── compact_uniq.txt                            # compact representation of long reads
│   ├── index.contig                                # contig index generated by HASLR
│   ├── index.longread                              # long read index generated by HASLR
│   ├── log_asmfinal.txt                            # log of HASLR (step: generating the assembly from the cleaned backbone graph)
│   ├── log_consensus.txt                           # log of HASLR (step: calling consensus sequence between anchors)
│   └── log_coordinate.txt                          # log of HASLR (step: calculating long read coordinates between anchors)
├── asm_contigs_k49_a3_lr25x_b500_s3_sim0.85.err    # log of HASLR
├── asm_contigs_k49_a3_lr25x_b500_s3_sim0.85.out    # 
├── lr25x.fasta                                     # longest 25x coverage of long reads
├── map_contigs_k49_a3_lr25x.log                    # log of minimap2
├── map_contigs_k49_a3_lr25x.paf                    # alignments of short read contigs onto long reads
├── sr.fofn                                         # list of short read files; Minia's input
├── sr_k49_a3.contigs.fa                            # short read contigs generated by Minia
├── sr_k49_a3.contigs.nooverlap.250.fa              # short read contigs at least 250 bp in length without overlaps
├── sr_k49_a3.contigs.nooverlap.fa                  # short read contigs without overlaps
├── sr_k49_a3.h5                                    # 
├── sr_k49_a3.log                                   # log of Minia
└── sr_k49_a3.unitigs.fa                            # short read unitigs generated by Minia
```
Note that the name of output files and folders with be changed depending on the parameters passed to `haslr.py`.

## <a name="publication"></a>Preprint

> Haghshenas E., Asghari H., Stoye J., Chauve C., and Hach F. (2020)
> *bioRxiv*. [doi:10.1101/2020.01.27.921817][doi]

## <a name="bugs"></a>Bug report
Please report the bugs through HASLR's issue tracker at [https://github.com/vpc-ccg/haslr/issues](https://github.com/vpc-ccg/haslr/issues).

## <a name="license"></a>Copyright and License
This software is released under GNU General Public License (v3.0)
- SPOA is released under MIT license
- Minia is released under AGPL license 
- minimap2 is released under MIT license
- fastutils is released under GPL license 

## <a name="author"></a>Author
[Ehsan Haghshenas](https://github.com/haghshenas) (ehaghshe AT sfu DOT ca)
