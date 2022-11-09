# DPClust R package
MIAOMIAO
This R package contains methods for subclonal reconstruction through SNVs and/or CNAs from whole genome or whole exome sequencing data. The methods are originally described in [The Life History of 21 Breast Cancers](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3428864/), see [here](https://www.ncbi.nlm.nih.gov/pubmed/28270531) for a recent write up.

## Installation instructions

Install the latest release version from Github
```
R -q -e 'source("http://bioconductor.org/biocLite.R"); biocLite(c("optparse","KernSmooth","ks","lattice","ggplot2","gridExtra"))'
R -q -e 'devtools::install_github("Wedge-Oxford/dpclust")'
```

Alternatively, the software can be installed after downloading a release
```
R -q -e 'source("http://bioconductor.org/biocLite.R"); biocLite(c("optparse","KernSmooth","ks","lattice","ggplot2","gridExtra"))'
R -q -e 'install.packages([DPClust tarball], repos=NULL, type="source")'
```

## Running DPClust

The DPClust package comes with an example pipeline and some simulated data in `inst/example`. Run the examples as follows:
```
# check out the repository
git clone git@github.com:Wedge-Oxford/dpclust.git
cd dpclust/inst/example

# single sample case
./run.sh

# multi-sample case
./run_nd.sh 
```

## Docker

Run DPClust on provided example data. After checking out this repository, build the image:
```
docker build -t dpclust:2.2.7 .
```

Run DPClust as follows
```
docker run -it -v `pwd`:/mnt/output/ dpclust:2.2.7 /opt/dpclust/example/run_docker.sh
```

## Input description

DPClust takes as input a project description file and a file for each sample described in the project file. By providing the pipeline with an ID it can pick the case to run through. Since DPClust can take single and multi-sample cases it requires some encoding of matching multiple sample files to a single case. The project description is set up with that in mind. 

For multi-sample cases it is important that all DPClust input files for that case contain all mutations across all samples.

### Example data and pipelines
Example data is included with this package. See `inst/extdata/simulated_data/simulated_1d.txt` and `inst/extdata/simulated_data/simulated.txt` for examples of a single and a multi-sample case respectively.

The DPClust input file should be generated via the DPClust pre-processing package ([dpclust3p](https://github.com/Wedge-Oxford/dpclust3p)). Example pipelines to generate the DPClust input file ([here](https://github.com/Wedge-Oxford/dpclust3p/blob/master/inst/example/preproc_pipeline_simple.R) and [here](https://github.com/Wedge-Oxford/dpclust3p/blob/master/inst/example/preproc_pipeline_caveman.R)) and the DPClust project master file ([here](https://github.com/Wedge-Oxford/dpclust3p/blob/master/inst/example/preproc_dpclust_master_file.R)) are included in dpclust3p.

### DPClust input file

The DPClust input file requires these columns. See [here](https://www.ncbi.nlm.nih.gov/pubmed/28270531) for more details on the meaning and derivation of these values.

|Column|Description|
|---|---|
| chr	| Chromosome on which the mutation occurred |
| end	| Position at which the mutation occurred |
| WT.count	| The number of sequencing reads supporting the reference allele |
| mut.count	| The number of sequencing reads supporting the mutation allele |
| subclonal.CN	| The total copy number at the location of the mutation |
| mutation.copy.number |	The raw estimate of the average number of chromosome copies that carry the mutation |
| subclonal.fraction	| The estimate of the fraction of tumour cells (CCF) that carry the mutation |
| no.chrs.bearing.mut	| The mutation's multiplicity estimate |

### DPClust project master file contents

The DPClust project master file requires at least these columns

|Column|Description|
|---|---|
| sample	| Top level ID to use when naming output files across all samples of this case |
| subsample |	The ID to use on a per sample level (i.e. when comparing sample a with sample b) |
| datafile	 | The DPClust input filename (the path to these files is provided to the pipeline) |
| cellularity | The sample purity estimate (fraction of tumour cells in the sequencing sample) |

## Output description

### Single sample

DPClust creates the following output for a single sample case

|Column|Description|
|---|---|
|*_bestClusterInfo.txt			| Contains the mutation clusters found, for each cluster in each sample the proportion of tumour cells that the cluster represents (CCF) and the number of mutations assigned to the cluster |
|*_bestConsensusAssignments.bed		| Assignment of mutations to clusters, the cluster column refers to the cluster.no in the *_bestClusterInfo.txt file |
|*_mutationClusterLikelihoods.bed | Likelihoods of each mutation belonging to each cluster, this file may contain likelihoods of cluster assignments to clusters that were considered, but did not yield any mutations when performing a hard-assignment. These clusters will not show up in *_bestClusterInfo.txt |
|*_bestConsensusResults.RData		| R data file with all the output |
|*_DirichletProcessplot_with_cluster_locations_2.png  | Main output figure showing the raw input data in the background, the posterior density of mutation clusters in purple (with blue confidence interval) and the called mutation clusters and the number of assigned mutations in the foreground |
|*_mutation_assignments.png | Table figure showing the called mutation clusters |

### Multiple samples

DPClust creates the following output for a multi-sample case

|File|Description|
|---|---|
|*_bestClusterInfo.txt			| Contains the mutation clusters found, for each cluster in each sample the proportion of tumour cells that the cluster represents (CCF) and the number of mutations assigned to the cluster |
|*_bestConsensusAssignments.bed		| Assignment of mutations to clusters, the cluster column refers to the cluster.no in the *_bestClusterInfo.txt file |
|*_mutationClusterLikelihoods.bed | Likelihoods of each mutation belonging to each cluster, this file may contain likelihoods of cluster assignments to clusters that were considered, but did not yield any mutations when performing a hard-assignment. These clusters will not show up in *_bestClusterInfo.txt |
|*_bestConsensusResults.RData		| R data file with all the output |
|*_mutation_assignments.png | Table figure showing the called mutation clusters |
|*_most_likely_cluster_assignment_0.01.pdf  | Graphical depiction of the mutation assignments |

