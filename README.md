#### Download the latest release: mutree 2.7182 ( [zip](../../archive/v2.7182.zip) | [tar.gz](../../archive/v2.7182.tar.gz) ).
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.583634.svg)](https://doi.org/10.5281/zenodo.583634)


---


mutree
=======

### A pipeline for phylogenetic tree inference and mutation recurrence discovery

__Adrian Baez-Ortega  
Transmissible Cancer Group, University of Cambridge__

mutree is a generalization and extension of [Asif Tamuri's treesub](https://github.com/tamuri/treesub) pipeline. It makes use of [RAxML](http://sco.h-its.org/exelixis/web/software/raxml/index.html) [1] and parts of treesub itself (which in turns uses the Java libraries [PAL](http://iubio.bio.indiana.edu/soft/molbio/evolve/pal/pal.html) [2] and [BioJava](http://biojava.org/) [3]) in order to infer a phylogenetic tree and identify candidate recurrent coding-affecting mutations in it, from a coding DNA sequence alignment.

The pipeline generates:

* A maximum likelihood phylogenetic tree including bootstrap values in its branches (Newick format).

* A version of the ML tree showing all the annotated mutations in the branches where they occur (Nexus format).

* A version of the ML tree showing only the recurrent mutations in the branches where they occur (Nexus format). A nonsynonymous mutation in a branch of the tree is considered to be recurrent if another nonsynonymous mutation in the same gene has been found in a different branch.

* A text table with all the single-nucleotide substitutions found in the alignments, indicating whether they are nonsynonymous and recurrent. 

mutree has been tested on an Ubuntu 14.04.4 system, and it should behave well in any Linux distribution. It should also work well on Mac OS X.


---


## Installation

mutree depends on the installation of the following software:

* [__RAxML__](http://sco.h-its.org/exelixis/web/software/raxml/index.html) version 8.2.9 or later. mutree requires compiling the `raxmlHPC-SSE3` and `raxmlHPC-PTHREADS-SSE3` RAxML executables, which should work well in processors up to 5 years old.

* A recent [__Java__](https://www.java.com) runtime (1.6+), which might be already installed in your system.

* Although it is not required in order to run the pipeline, some visualisation tool is needed to open the output tree files. [__FigTree__](http://tree.bio.ed.ac.uk/software/figtree) can read the Nexus format in which the substitution trees are output. The tree showing the bootstrap support values (in Newick format) can be opened using e.g. [__Dendroscope__](http://dendroscope.org/), or converted to a different format.

mutree already includes its own (slightly customized) version of the treesub pipeline, named 'treesub-TCG'. Therefore, installing treesub is not necessary, although in some cases it may have to be re-compiled (see NOTE below).

The following instructions describe the steps for installing mutree and all its components in an Ubuntu 14.04.4 system; they should be valid for any Ubuntu or Debian Linux distribution. The tools employed have available Mac and Windows versions (please consult their respective websites). mutree itself has not been tested on Mac or Windows systems, but it might work with an appropriate Bash shell.

 1. __Install RAxML__

    You only need to install RAxML if the commands `which raxmlHPC-PTHREADS-SSE3` or `which raxmlHPC-SSE3` do not print anything in the terminal.

    Go to the desired installation folder (in this example, the Software folder inside your home directory, or `~/Software`):

        cd ~/Software

    Download and compile RAxML:

        wget https://github.com/stamatak/standard-RAxML/archive/v8.2.9.tar.gz
        tar zxvf v8.2.9.tar.gz
        rm v8.2.9.tar.gz

        cd standard-RAxML-8.2.9/
        make -f Makefile.SSE3.gcc
        rm *.o
        make -f Makefile.SSE3.PTHREADS.gcc
        rm *.o

    Then, edit your `~/.bashrc` file using:

        nano ~/.bashrc

    and append the `standard-RAxML-8.2.9` directory at the end of your PATH variable. If the PATH variable is not defined, you can define it by adding the following line at the end of the `~/.bashrc` file:

        export PATH=~/Software/standard-RAxML-8.2.9:$PATH

    Then save and close the file (Ctrl-X).

 2. __Install the Java Runtime Environment__

    You only need to install Java if the command `which java` does not print anything in the terminal.

        sudo apt-get install default-jre

    The system will ask for your password; you need to have administrator permissions in your system in order to use `sudo apt-get install`.

 3. __Install mutree__

    Go to the desired installation folder, and download and uncompress mutree (replace `2.xx` with the latest version):

        cd ~/Software

        wget https://github.com/adrianbaezortega/mutree/archive/v2.xx.tar.gz
        tar zxvf v2.xx.tar.gz
        rm v2.xx.tar.gz

    Then, edit your `~/.bashrc` file using:

        nano ~/.bashrc

    and append the `mutree-2.xx/src` directory at the end of your PATH variable. If the PATH variable was not defined, not its line should look like:

        export PATH=~/Software/standard-RAxML-8.2.9:~/Software/mutree-2.xx/src:$PATH

    Then save and close the file (Ctrl-X).

    Either close the terminal and open a new one, or source the `~/.bashrc` file in order to apply the changes:

        source ~/.bashrc

    Then you should be able to run the following commands, which should print something like this:

        which raxmlHPC-PTHREADS-SSE3  # prints: [...]/standard-RAxML-8.2.9/raxmlHPC-PTHREADS-SSE3
        which raxmlHPC-SSE3           # prints: [...]/standard-RAxML-8.2.9/which raxmlHPC-SSE3
        which java                    # prints: /usr/bin/java
        which mutree                 # prints: [...]/mutree-2.xx/src/mutree

__And now you can have fun!__

__NOTE__: If you encounter problems while using mutree and they seem to be related to the treesub pipeline, you can try re-compiling it. You need to go to the `treesub-TCG` folder within the mutree installation directory, and re-compile treesub using [Ant](http://ant.apache.org/):

    cd ~/Software/mutree-2.xx/treesub-TCG
    export ANT_OPTS="-Xmx256m"
    ant compile jar

---


## Running mutree

The pipeline __requires__ the following input:

* __*Absolute path* to a coding sequence (CDS) alignment file, in FASTA format (`-i` option).__ Each sequence in the file should be composed of a concatenation of multiple gene CDS sequences, all of which __must be in frame__ (i.e. the concatenated sequence must contain codon bases only, and its length must be a multiple of 3). If the length of a CDS is not a multiple of 3, any trailing bases after the last codon have to be removed before adding the CDS to the concatenated sequence. Each sequence in the FASTA file represents a sample (taxon), and must be labeled with a unique sample name. __Sample names cannot include any blank spaces, tabulators, carriage returns, colons, commas, parentheses or square brackets. Each sequence must be on a single line, so that odd lines in the file contain the sample names, while even lines contain the sequences. The first sequence in the file will be used as an outgroup to root the tree, so this should be the reference sequence or a suitable outgroup sample.__ An example can be found in the file [mutree-2.xx/examples/Alignment_H3HASO.fna](examples/Alignment_H3HASO.fna) (this has been adapted from one of treesub's example files).

* __*Absolute path* to a "gene table" (`-g` option).__ This is mandatory unless the `-f` option is used. The gene table must be a tab-delimited file with no header and two columns: gene symbol and CDS start position (position of the first nucleotide in the concatenated sequence). This allows mapping each mutation to the gene where it occurs and finding recurrent mutations. An example can be found in the file [mutree-2.xx/examples/GeneTable_H3HASO.txt](examples/GeneTable_H3HASO.txt) (the gene symbols and positions have been defined arbitrarily for this example).

* __*Absolute path* to an output directory (`-o` option).__ The directory will be created if necessary. The pipeline implements a checkpoint logging system, so in the event that the execution is interrupted before finishing, re-running mutree with the same output directory will resume the execution after the last successfully completed step.

mutree also accepts other __optional__ input:

* __Number of RAxML threads (`-t` option).__ This allows using the multi-threaded version of RAxML to substantially speed up the tree inference and the ancestral sequence reconstruction. This value can be any positive integer, and cannot be higher than the available number of processors. The default value is 1.

* __Custom RAxML options for tree inference (`-r` option).__ This allows personalizing the RAxML routine, which uses rapid bootstrapping followed by maximum likelihood search by default (see pipeline description below). Custom options must be specified as a single string within quotes, and must include all the required options for running RAxML, __except__ for the options `-s`, `-n`, `-w` and `-T`, which cannot be used.

* __Custom RAxML options for ancestral sequence reconstruction (`-a` option).__ This allows personalizing the ASR settings, which consist of a GTR substitution model plus a Gamma model of rate heterogeneity by default (see pipeline description below). Custom options must be specified as a single string within quotes, and must include all the required options for running RAxML, __except__ for the options `-f`, `-s`, `-n`, `-w` and `-T`, which cannot be used.

* __Perform tree inference and rooting only (`-f` option).__ If this option is specified, only the first three steps of the pipeline will be run. Thus, in this case, it is not necessary to provide a gene table via `-g`, and there is also no need for the input alignment (`-i`) to be composed of coding sequences (unless the rest of the pipeline is to be run afterwards). This option does not require any arguments.

Following from this, the mutree command should look similar to the example below:

    mutree -i /path/to/alignment.fna -o /path/to/out_dir -g /path/to/gene_table.txt -t 8 -r "-m GTRGAMMA -# 10 -p 12345" -a "-m GTRGAMMA --HKY85 -M"

Most users should not need to use options `-r` and `-a`. The example input files can be used for a quick test run (without bootstrapping):

    mutree -i /path/to/mutree-2.xx/examples/Alignment_H3HASO.fna -g /path/to/mutree-2.xx/examples/GeneTable_H3HASO.txt -o /path/to/out_dir -r "-m GTRGAMMA -p 12345"

(Because the sequences in this arbitrary example have a high mutation density, there will be more than one nonsynonymous substitution in every gene, and therefore all the nonsynonymous substitutions will appear as recurrent. However, it is useful as a model of how the input and output should look like.)

__Running `mutree` without any arguments or with the `-h` option will print the usage information__; the `-v` option will print the program version only.


---


## Pipeline description

The pipeline is composed of six steps:

 1. __Input processing__
 
    The input FASTA alignment is transformed to PHYLIP format and the sequences are relabelled so that they are compatible with the tools employed. If the alignment contains sites composed only of undetermined characters ('N's) in all the sequences, a version without such sites will be generated as an input for step 2. The codons containing variable sites will be concatenated and written to a different file (in which 'N' characters will be replaced by 'A's), which will be used in step 4.

 2. __Maximum likelihood tree inference__
 
    RAxML is used to build a maximum likelihood (ML) phylogenetic tree from the input alignment. This can be a very expensive process. By default, rapid bootstrapping (with an extended majority­rule consensus tree stop criterion) is performed prior to a thorough ML tree search, which employs a GTR substitution model plus a Gamma model of rate heterogeneity (`-f a -m GTRGAMMA -# autoMRE -x 931078 -p 272730` configuration; see the [RAxML manual](http://sco.h-its.org/exelixis/resource/download/NewManual.pdf)). However, custom RAxML options can be specified via mutree's `-r` option. Custom options must be specified between quotes (e.g. `-r "-m GTRGAMMA -# 10 -p 12345"`), and must include all the options required for running RAxML, __except__ for the options `-s`, `-n`, `-w` and `-T`, which cannot be used.

 3. __Tree rooting__
 
    Treesub is used to root the ML tree by the outgroup sequence, which should be the first sequence in the input alignment FASTA file.

 4. __Ancestral sequence reconstruction__
 
    RAxML is used to perform marginal reconstruction of ancestral sequences from the input alignment and the ML tree. To allow the processing of very long coding sequences, this step runs on the set of codons that contain a mutation in any of the sequences. Here, RAxML employs a GTR substitution model plus a Gamma model of rate heterogeneity by default (`-f A -m GTRGAMMA` configuration; see the [RAxML manual](http://sco.h-its.org/exelixis/resource/download/NewManual.pdf)). However, custom RAxML options for ancestral sequence reconstruction can be specified via mutree's `-a` option. Custom options must be specified between quotes (e.g. `-a "-m GTRGAMMA --HKY85 -M"`), and must include all the options required for running RAxML, __except__ for the options `-f`, `-s`, `-n`, `-w` and `-T`, which cannot be used. 

 5. __Tree annotation__
 
    Treesub is used to annotate the mutations occurring in each branch of the tree, based on the reconstructed ancestral sequences. Mutations are assessed for their amino acid change. As part of this process, each branch receives a unique label.

 6. __Recurrent mutation identification__
 
    Finally, mutated codons' positions are translated into their actual positions in the original sequence, and the input gene table is then used to map each mutation to the gene (CDS) where it occurs. Any group of nonsynonymous mutations affecting the same gene are marked as recurrent. A new tree is produced which shows only the identified recurrent mutations in each branch.

Each one of the pipeline steps will generate an intermediate folder within the specified output directory. The 'logs' folder contains the global execution log, as well as the checkpoint file, which is used to record the current stage of the pipeline and can be modified in order to restart the execution in any given step: when re-running mutree with the same output folder as before, execution will be resumed at the step that follows the last step recorded in the checkpoint file.

The pipeline's final output will be stored in a folder named 'Output', and will consist of:

* A tab-delimited text file containing the information for all the identified mutations in the tree (branch/node, gene, position, codon and amino acid changes, and whether the mutation is nonsynonymous/recurrent).

* Five versions of the same phylogenetic tree:

    - Standard ML tree as produced by RAxML (Newick format).
    
    - Tree showing the bootstrap support values in its branch bifurcations (Newick format, unrooted; only if bootstrapping is performed).
    
    - Tree showing the branch and node labels employed in the output table of mutations (Newick format).

    - Tree showing all the mutations identified in each branch (Nexus format).
    
    - Tree showing the candidate recurrent mutations identified in each branch (Nexus format).


---

## Citation

Please cite mutree as:

Adrian Baez-Ortega. mutree: A pipeline for phylogenetic tree inference and recurrent mutation discovery. Zenodo (2017). [doi:10.5281/zenodo.583634](https://doi.org/10.5281/zenodo.583634).


---


## License

Copyright © 2016–2017 Transmissible Cancer Group, University of Cambridge  
Author: Adrian Baez-Ortega ([ORCID 0000-0002-9201-4420](http://orcid.org/0000-0002-9201-4420); ab2324@cam.ac.uk)

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation; either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program. If not, see http://www.gnu.org/licenses.


---


## References

1. Stamatakis, A. 2006. RAxML-VI-HPC: Maximum Likelihood-based Phylogenetic Analyses with Thousands of Taxa and Mixed Models. _Bioinformatics_ 22(21):2688–2690.

2. Drummond, A., Strimmer, K. 2001. PAL: An object-oriented programming library for molecular evolution and phylogenetics. _Bioinformatics_ 17: 662-663.

3. Holland, R.C.G., Down, T., Pocock, M., Prlić, A., Huen, D., James, K., Foisy, S., Dräger, A., Yates, A., Heuer, M., Schreiber, M.J. 2008. BioJava: an Open-Source Framework for Bioinformatics. _Bioinformatics_ 24(18): 2096-2097.