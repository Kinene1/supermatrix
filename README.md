# supermatrix #
scripts to add new proteins to alignment using hmms, and simple diagnostics/visualizations

## add_taxa_to_align.py ##
Script to add new taxa to an existing protein supermatrix alignment. Proteins from new taxa are untrimmed, though a trimming step may be implemented. Input alignments could be in a number of formats, as a supermatrix with a separate partition file, or individual alignments as separate files.
* Multiple new taxa can be added with `-t`, as space-separate protein files (could be gene models or translated from transcriptomes). 
* By default, only the single best hit is taken (changed with `-m`), and is renamed to the corresponding species given in `-T`. 
* Species names from `-T` and files from `-t` must be in the same order. 
* Several folders with many intermediate files are generated (`-d`, `-I`, and `-S`), in case it is needed to re-examine later.
* For alignment format (`-f`), most cases *phylip* format is actually *phylip-relaxed*.
* By default, the e-value cutoff is determined uniquely for each gene based on the lower limit of that hmm against the original gene set. This is to reduce the chance of finding out-paralogs. However, a static e-value cutoff for `hmmsearch` can be given using `-e`.
* To generate the new supermatrix with the added taxa, specify the name of the new file with `-U`. A new partition file will be automatically generated if `-U` is specified.

`add_taxa_to_align.py -a philippe2009_FullAlignment.phy -i philippe2009_partitions.txt -t ~/genomes/apis_mellifera/amel_OGSv3.2_pep.fa -T Apis_mellifera -f phylip-relaxed -U philippe2009_w_amel.aln`

Requires [BioPython](http://biopython.org/wiki/Download), [hmmsearch and hmmbuild](http://hmmer.org/), and [mafft v7.3.10](http://mafft.cbrc.jp/alignment/software/source.html), though could be modified to use any aligner. Older versions fo mafft are compatible if using the `-r` option  (current script requires an option in v7.3), which skips the mafft alignment-trimming step.

Binaries are assumed to be in the user's PATH. This can be changed with the options `--mafft`, `--hmmbin`, and `--fasttree`. Both `--mafft` and `--fasttree` should point to binaries, but `--hmmbin` points to a folder containing both hmmsearch and hmmbuild.

`add_taxa_to_align.py -a philippe2009_FullAlignment.phy -i philippe2009_partitions.txt -t ~/genomes/apis_mellifera/amel_OGSv3.2_pep.fa -T Apis_mellifera -f phylip-relaxed -U philippe2009_w_amel.aln --mafft ~/programs/mafft --hmmbin ~/programs/`

## join_alignments.py ##
Join multiple individual alignment files into a supermatrix, allowing for only one occurence of any taxa in each alignment. Names must be the same, though can have unique identifiers (like gene names or numbers) as long as they can be systematically split from the taxon names (using `-d`).

`join_alignments.py -a hehenberger2017_alignments/* -d "@" -u hehenberger2017_supermatrix.fasta`

## split_supermatrix_to_taxa.py ##
Split a supermatrix into fasta files, one for each taxa where individual proteins are defined by the partition file. Empty proteins are ignored, but gaps are retained. This is NOT the reverse operation of `join_alignments.py`, which joins multiple alignment files into a supermatrix.

`./split_supermatrix_to_taxa.py -a simion2017_97sp_401632pos_1719genes.fasta.gz -d simion_taxa -p simion2017_partitions.txt`

## check_supermatrix_alignments.py ##
Quick diagnostic script to check matrix occupancy. Adjust format accordingly based on the alignment using the `-f` option. As above, in most cases *phylip* format is probably *phylip-relaxed*.

`check_supermatrix_alignments.py -a philippe2009_FullAlignment.phy -p philippe2009_partitions.txt -f phylip-relaxed`

Requires [BioPython](http://biopython.org/wiki/Download)

To generate a chart of matrix occupancy, add the `-m` option with the name of the new output file. This matrix can be visualized using the R script `draw_matrix_occupancy.R`.

`check_supermatrix_alignments.py -a philippe2009_FullAlignment.phy -p philippe2009_partitions.txt -f phylip-relaxed -m philippe2009_occupancy_matrix.tab`

To reorder the matrix based on taxa in a rooted tree, use the `-T` option with a nexus-format tree. For instance, open any tree in figtree, rotate branches, etc, then copy and paste the tree to a text file, and this is in nexus format. Then run `draw_matrix_occupancy.R` below.

`check_supermatrix_alignments.py -p Metazoa_full_Models_short.txt -a Metazoa_full-fix.phy -f phylip-relaxed -m Metazoa_full_matrix.tab -T Metazoa_full.nex`

## filter_supermatrix.py ##
Filter supermatrices based on coverage for each gene. Minimum coverage is given by `-c` for values between 0 and 1. A new partition file is automatically generated based on the output name `-o`. 

`filter_supermatrix.py -a simion2017_97sp_401632pos_1719genes.fasta -c 0.75 -p simion2017_partitions.txt -o simion2017_97sp_75cov.fasta`

Requires [BioPython](http://biopython.org/wiki/Download)

## draw_matrix_occupancy.R ##
A graph of matrix occupancy can be generated with the accompanied R script. Genes that are present are colored blue, partial genes are red, and absent genes are white, for each taxa. Taxa with 100% occupancy are colored green, while those with under 50% are colored purple. By default, the taxon order in the graph is the same order as given in the supermatrix alignment, which is then preserved in the matrix occupancy chart above. Taxa can be reordered arbitrarily in either file.

The script can be run in the terminal as:

`Rscript draw_matrix_occupancy.R philippe2009_occupancy_matrix.tab`

![philippe2009_occupancy_matrix.png](https://github.com/wrf/supermatrix/blob/master/philippe2009_occupancy_matrix.png)

## test alignments and occupancy matrices ##
* Philippe et al (2009) dataset of 128 genes and 30k positions, from [Phylogenomics Revives Traditional Views on Deep Animal Relationships](https://www.sciencedirect.com/science/article/pii/S0960982209008057)
* [Hejnol 2009](https://bitbucket.org/caseywdunn/hejnol_etal_2009) dataset of 1486 genes for 270k positions, from [Assessing the root of bilaterian animals with scalable phylogenomic methods](http://rspb.royalsocietypublishing.org/content/276/1677/4261)
* [Nosenko 2013](https://data.ub.uni-muenchen.de/55/) datasets of 35 genes/9k sites and 87 genes/14k sites from 71 taxa, from [Deep metazoan phylogeny: When different genes tell different stories](http://www.sciencedirect.com/science/article/pii/S1055790313000298)
* [Zapata 2015](https://bitbucket.org/caseywdunn/cnidaria2014) dataset of 1262 genes for 365k positions, from [Phylogenomic Analyses Support Traditional Relationships within Cnidaria](http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0139068)
* Cannon et al (2016) dataset of 212 genes, from [Xenacoelomorpha is the sister group to Nephrozoa](http://www.nature.com/nature/journal/v530/n7588/full/nature16520.html) 
* [Simion et al 2017](https://github.com/psimion/SuppData_Metazoa_2017) dataset of 1719 genes, where partition file has been reduced to only the numbers and ? in the supermatrix are replaced with gaps, from [A Large and Consistent Phylogenomic Dataset Supports Sponges as the Sister Group to All Other Animals](http://www.sciencedirect.com/science/article/pii/S0960982217301999)
* [Hehenberger 2017](http://datadryad.org/resource/doi:10.5061/dryad.26bv4) dataset of 255 genes from 38 taxa, from [Novel Predators Reshape Holozoan Phylogeny and Reveal the Presence of a Two-Component Signaling System in the Ancestor of Animals](http://www.sciencedirect.com/science/article/pii/S0960982217307078)
* [Whelan 2017](https://figshare.com/articles/Ctenophora_Phylogeny_Datasets_and_Core_Orthologs/4484138) datasets of 350, 212, and 117 genes, from [Ctenophore relationships and their placement as the sister group to all other animals](https://www.nature.com/articles/s41559-017-0331-3)
