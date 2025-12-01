Naja_synteny_VenomGenes
================
Alex
2025-11-28

# Workflow to compare venom coding regions of naja naja

We start with two prerequesites:

1.  Annotated reference genome of Naja naja from Suryamohan et al. from
    an dindian locality
2.  A long scaffold query genome from a sri lankan individual of Naja
    naja

<br>

## Mapping the query genome to the refernece genome to attain a mapped genome with the same chromosome names

I used ragtag for this (Alonge, Michael, et al. “Automated assembly
scaffolding elevates a new tomato system for high-throughput genome
editing.” Genome Biology (2022).
<https://doi.org/10.1186/s13059-022-02823-7>)

All you need to do is install a “ragtag” environment the same way we
would do with conda (other neccessary modules are python and minimap,
but these are on scw), then use the following code:

``` bash
#!/bin/bash --login
#SBATCH --job-name=ragdag
#SBATCH --output=/scratch/b.lxn25yng/slurmscripts/logs/fst_ang_%J.out
#job stderr file
#SBATCH --error=/scratch/b.lxn25yng/slurmscripts/logs/fst_ang_%J.err
#SBATCH --partition=htc
#SBATCH --time=0-12:00
#SBATCH --nodes=1
#SBATCH --ntasks=10
#SBATCH --mem=64G
#SBATCH --account=scw2119
#SBATCH --chdir=/scratch/b.lxn25yng/slurmscripts/

module purge

# Load modules
module load python/3.10.4
module load minimap2/2.24

source /scratch/b.lxn25yng/ragtag_env/bin/activate

cd /scratch/b.lxn25yng/naja/

ragtag.py scaffold GCA_009733165.1_Nana_v5_genomic.fna naja_sl.fasta -o ragtag_out
```

<br>

Now we have a query genome mapped to the original fasta reference
genome(to visualize in igv i cut out all the unasigned scaffolds, too
many to visualize in igv): show dotplot of total alignments

## Toxin Annotation using ToxCodAn

(Nachtigall et al. (2024) ToxCodAn-Genome: an automated pipeline for
toxin-gene annotation in genome assembly of venomous lineages. Giga
Science. DOI: <https://doi.org/10.1093/gigascience/giad116>):

The following script had the following prerequesits to be installed in
this case on my local machine:

python, biopython and pandas

NCBI-BLAST (v2.9.0 or above)

Exonerate

Miniprot

GffRead

R

to run toxcodan, i downloaded the elapidae toxin database from toxcodan
github containing 1150 toxic CDS from 76 species.

``` bash
wget https://raw.githubusercontent.com/pedronachtigall/ToxCodAn-Genome/main/Databases/Elapidae_db.fasta
```

<br>

To run toxcodan i installed it as a conda envirmonment onmy laptop,
workflow c0pied from toxcodan github:

<br>

### If the user wants to install ToxCodAn-Genome and all dependencies using Conda environment, follow the steps below:

Tip: First, ensure that you have added all conda channels properly in
the following order:
```bash
conda config –add channels defaults conda config –add channels bioconda
conda config –add channels conda-forge
```
Create the environment:
```bash
conda create -n ToxcodanGenome -c bioconda python biopython pandas blast
exonerate miniprot gffread hisat2 samtools stringtie trinity spades

Git clone the ToxCodAn-Genome repository and add the bin to your PATH:

git clone <https://github.com/pedronachtigall/ToxCodAn-Genome.git> echo
“export PATH=$PATH:$PWD/ToxCodAn-Genome/bin/” \>\> ~/.bashrc Replace
~/.bashrc to ~/.bash_profile if needed.
```
It may be needed to apply “execution permission” to all bin executables
in “ToxCodAn-Genome/bin/”:

chmod +x ToxCodAn-Genome/bin/\* Then, run ToxCodAn-Genome as described
in the “Usage” section.

To activate the environment to run ToxCodAn-Genome just use the command:
```bash
conda activate ToxcodanGenome
```
To deactivate the environment just use the command:
```bash
conda deactivate
```
<br>

Once all works:

``` bash
conda activate ToxcodanGenome
```

run toxcodan, twice once for each fasta, i used standard filters of 80%
identity match, 400 min bp and 50000 max bp as recommended by pedro.
changing these filters had minimal impact on results, non perfectly
matched toxins were numerous mainly due to unexpected start or stop
codons, this could very likely be due to the reference genome containing
many unasigned scaffold:

<br>

``` bash
 /home/alexn/Annotating/ToxCodAn-Genome/bin/toxcodan-genome.py -g /home/alexn/Annotating/GCA_009733165.1_Nana_v5_genomic.fasta -d /home/alexn/Annotating/Elapidae_db.fasta -c 6
```

this spits out ToxCodAnGenome-output into the working directory.
remember to change this to e.g. ToxCodAnGenome-outputNajaS1 then:

``` bash
 /home/alexn/Annotating/ToxCodAn-Genome/bin/toxcodan-genome.py -g /home/alexn/Annotating/ragtag_najaS2-najaS1.fasta -d /home/alexn/Annotating/Elapidae_db.fasta -c 6
```

<br>

You end up with multiple files, toxin annotation and matched getion gtfs
and their according cds, these can be used with IGV or other
applications. I went on to make a tsv of the toxin annotaion file to
then visualize in karyoplot, i manually filtered the tsv for 3Ftx, CRISP
and SVMP toxins, again twice:

``` bash
python ToxCodAn-Genome/bin/fromCDStoGENE.py   Naja/ToxCodAnGenome_outputNajaS1/toxin_annotation.gtf   Naja/ToxCodAnGenome_outputNajaS1/toxin_annotation_GENE_NajaS1.tsv

python ToxCodAn-Genome/bin/fromCDStoGENE.py   ToxCodAnGenome_outputNajaS2/toxin_annotation.gtf   ToxCodAnGenome_outputNanaS1/toxin_annotation_GENE_NajaS2.tsv
```

``` bash
Rscript ToxCodAn-Genome/b
in/PlotToxinLoci.R   -i ToxCodAnGenome_outputNajaS1/toxin_annotation_GENE_NajaS1.tsv   -o output


Rscript ToxCodAn-Genome/b
in/PlotToxinLoci.R   -i ToxCodAnGenome_outputNajaS2/toxin_annotation_GENE_NajaS2.tsv   -o output
```

<br>

<figure>
<img src="najas1.png" alt="Naja Reference Karyoplot" />
<figcaption aria-hidden="true">Naja Reference Karyoplot</figcaption>
</figure>

<figure>
<img src="najas2.png" alt="Naja Query Karyoplot" />
<figcaption aria-hidden="true">Naja Query Karyoplot</figcaption>
</figure>

