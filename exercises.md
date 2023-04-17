# Exercises

1. [Setting up the cloud computing](#setting-up-the-cloud-computing)
   1. [Setting up VS Code](#setting-up-vs-code)
   2. [Connecting to the remote machine with VS Code](#connecting-to-the-remote-machine-with-vs-code)
   3. [Cloning the course's GitHub repository](#cloning-the-courses-github-repository)
2. [Getting the raw data](#getting-the-raw-data)
3. [QC and trimming](#qc-and-trimming)
   1. [QC of the raw data](#qc-of-the-raw-data)
   2. [Read trimming](#read-trimming)
   3. [QC of the trimmed data](#qc-of-the-trimmed-data)
4. [Read-based taxonomic profiling](#read-based-taxonomic-profiling)
   1. [singleM](#singlem)
   2. [sourmash](#sourmash)
   3. [Visualizing the taxonomic profiles](#visualizing-the-taxonomic-profiles)
5. [Metagenome assembly](#metagenome-assembly)
   1. [Assembly QC](#assembly-qc)
6. [Genome-resolved metagenomics with anvi'o](#genome-resolved-metagenomics-with-anvio)

## Setting up the cloud computing

We will use the [Amazon Cloud](https://aws.amazon.com/ec2/) (AWS EC2) services for most of the analyses.  
The IP address of the remote machine will change every day, so a new IP adress will be posted in Slack each morning.  
Your username - that you have received by e-mail/Slack - will be the same for the whole course.  
We will use `ssh` to connect to the remote machine.  
We encourage the use of [VS Code](https://code.visualstudio.com/Download), but you are welcome to use any IDE or terminal emulator that you are comfortable with.  

### Setting up VS Code

1. Download `VS Code` and set it up as shown [here](Lectures/course-outline-and-practical-info.pdf)  
2. Save the `.pem` file that you have received by e-mail somewhere in your computer  
3. **Linux/MacOS users only:**  
   1. Launch the `Terminal` app  
   2. `cd` to the directory where you saved the `.pem` file
   3. run `chmod 600 userXX.pem` (remember to change `userXX.pem` by the name of your own file)
4. Back to `VS Code`, go to `View -> Command Palette`  
5. Search for `ssh config`  
6. Select `Remote-SHH: Open SSH Configuration File...`  
7. In the next dialogue box, hit `Enter/Return`
8. Copy and paste the following text:

```
Host physalia
  HostName 54.245.21.143
  User user1
  IdentityFile ~/Desktop/user1.pem
```

9. In the 2nd, 3rd, and 4th lines:  
   1. HostName: change to the IP adress of the day
   2. User: change to your own username
   3. IdentityFile: change to the location and name of the `.pem` file that you have saved in your computer 
10. Save and close the `config` file

### Connecting to the remote machine with VS Code

11. Go to `View -> Command Palette`
12. Search for `ssh connect`
13. Select `Remote-SHH: Connect to Host...`
14. Select `physalia` (a new window will open)
15. If a dialogue box opens asking the server type, select `Linux`
16. If a dialogue box opens asking if you are sure, select `Continue`
17. If a terminal does not open by default, go to `Terminal -> New Terminal`

That's it, you should now be connected to the remote machine and ready to go!  
**Remember:** every day you should redo steps 4-7 and update `HostName` to match the IP adress of the day.  

### Cloning the course's GitHub repository

Once you have connected to the remote machine, you will be in your home folder (`/users/userXX`, also represented by `~` or `$HOME`).  
**Remember:** You can check where you are with the command `pwd`.  

To have access to the course's content, let's copy the GitHub repository to your `home` folder using `git clone`:

**Do this on the first day:** 

```bash
cd ~
git clone https://github.com/karkman/Physalia_EnvMetagenomics_2023
```

**Do this every once in a while, at least each day before starting the activities:**  

```bash
cd ~/Physalia_EnvMetagenomics_2023
git pull
```

**Note:** All exercises will be executed inside the `Physalia_EnvMetagenomics_2023` folder that you cloned inside your own `home` folder.  
So remember to `cd ~/Physalia_EnvMetagenomics_2023` every time you connect to the remote machine.  

## Getting the raw data

Choose which data set you would like to analyse (`Tundra` or `WWTP`).  
In the commands below, first uncomment the study you chose (remove the `#`) and run the `export` command.  
Then, copy the raw sequencing data to your own `01_DATA` folder.  
Also copy the file `SAMPLES.txt`, which will be useful for running `for loop` and etc.  

```bash
# export STUDY="WWTP"
# export STUDY="Tundra"

cd ~/Physalia_EnvMetagenomics_2023
mkdir 01_DATA

cp ~/Share/Data/${STUDY}/raw/*.fastq.gz 01_DATA/
cp ~/Share/Data/${STUDY}/SAMPLES.txt ./
```

## QC and trimming

Now that you have copied the raw data to your working directory, let's do some quality control.  
The sequencing process is subject to several types of problems that can introduce errors and artifacts in the sequences.  
Because of this, bioinformatics analyses usually start with the quality control of raw sequences.  
He we will use [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc) and [MultiQC](https://multiqc.info/) to obtain quality reports, and [Cutadapt](https://cutadapt.readthedocs.io/en/stable/) and [chopper](https://github.com/wdecoster/chopper) for trimming the Illumina and Nanopore data, respectively.  

### QC of the raw data

Go to your `Physalia_EnvMetagenomics_2023` folder, create a folder for the QC files, and activate the `conda` environment:  

```bash
cd ~/Physalia_EnvMetagenomics_2023
mkdir 02_QC_RAW
conda activate QC
```

And now you're ready to run the QC on the raw data:

```bash
fastqc 01_DATA/*.fastq.gz -o 02_QC_RAW -t 4
multiqc 02_QC_RAW -o 02_QC_RAW --interactive
```

After the QC is finished, copy the `MultiQC` report (`02_QC_RAW/multiqc_report.html`) to your local machine and open it with your favourite browser.  
We will go through the report together before continuing with the pre-processing.  

**NOTE:** to move files to and from local and remote machines, you can use: 
- The command-line tool [scp](https://kb.iu.edu/d/agye)  
- A file transfer software such as [FileZilla](https://filezilla-project.org)  
- The `VS Code` built-in `Explorer` tool (`View -> Explorer`)  

### Read trimming

Our QC reports tell us that a significant percentage of the raw sequences contain some isses such as the presence of adapters.  
Before proceeding, it is necessary to clean up/trim the raw sequences.  
Before start trimming the data, let's create a folder for the processed data and activate the `conda` environment:  

```bash
cd ~/Physalia_EnvMetagenomics_2023
mkdir 03_TRIMMED
conda activate QC
```

For the Illumina data, we will use a `for loop` to process each of the samples one after the other:  

```bash
for sample in $(cat SAMPLES.txt); do
  cutadapt 01_DATA/${sample}.illumina.R1.fastq.gz \
           01_DATA/${sample}.illumina.R2.fastq.gz \
           -o 03_TRIMMED/${sample}.illumina.R1.fastq.gz \
           -p 03_TRIMMED/${sample}.illumina.R2.fastq.gz \
           -a CTGTCTCTTATACACATCTCCGAGCCCACGAGAC \
           -A CTGTCTCTTATACACATCTGACGCTGCCGACGA \
           -m 50 \
           -q 20 \
           -j 4 > 03_TRIMMED/${sample}.illumina.log
done
```

While `Cutadapt` is running: looking at the [online manual](https://cutadapt.readthedocs.io/en/stable/index.html) or running `cutadapt --help`, answer:  

- What do the `-o`, `-p`, `-a`, `-A`, `m`, `-q`, and `-j` flags mean?  
- How did we choose the values for `-m` and `-q`?  
- What is the purpose of the redirection (`> 03_TRIMMED/${sample}.illumina.log`)?  

And now we trim the Nanopore data:  

```bash
gunzip -c 01_DATA/nanopore.fastq.gz | chopper -q 10 -l 1000 -t 4 | gzip > 03_TRIMMED/nanopore.fastq.gz
```

### QC of the trimmed data

Now the data has been trimmed, it would be a good idea to run `FastQC` and `MultiQC` again.  
Modify the [commands used for the raw data](#qc-of-the-raw-data) to match the trimmed data and run the two QC softwares.  

While you wait, take a look at the `Cutadapt` logs.  
When `Cutadapt` runs, it prints lots of interesting information to the screen, which we lose once we logout of the remote machine.  
Because we used redirection (`>`) to capture the standard output (`stdout`) of `Cutadapt`, this information is now stored in a file (`03_TRIMMED/${sample}.illumina.log`).  
Take a look at the log file for one of the samples using the program `less`:  

**NOTE:** You can scroll up and down using the arrow keys on your keyboard, or move one "page" at a time using the spacebar.  
**NOTE:** To quit `less`, hit the `q` key.  
**NOTE:** If you have set it up, you can also access the files using the `Explorer` tab on `VS Code` (`View -> Explorer`).  

By looking at the `Cutadapt` log, can you answer:  
- How many read pairs we had originally?  
- How many reads contained adapters?  
- How many read pairs were removed because they were too short?  
- How many base calls were quality-trimmed?  
- Overall, what is the percentage of base pairs that were kept?  

When `FastQC` and `MultiQC` have finished, copy the `MultiQC` report to your local machine and open it with a browser.  
Compare this with the report obtained earlier for the raw data.  
Do the data look better now?  

## Read-based taxonomic profiling

There are many different tools and approaches for obtaining taxonomic profiles from metagenomes.  
Here we will use two read-based tools: [singleM](https://wwood.github.io/singlem/) and [sourmash](https://sourmash.readthedocs.io/en/latest/).  
What is the basic approach that each of these tools use and how they can impact the results?  
And how similar are taxonomic profiles obtained from Illumina and Nanopore data?  
Well, let's find out!  

First let's create a folder to store the results:  

```bash
cd ~/Physalia_EnvMetagenomics_2023
mkdir 05_TAXONOMIC_PROFILE
```

### singleM

And now let's run `singleM`:  

```bash
conda activate singleM

# Illumina data
singlem pipe --forward 03_TRIMMED/*.R1.fastq.gz \
             --reverse 03_TRIMMED/*.R2.fastq.gz \
             --otu_table 05_TAXONOMIC_PROFILE/singleM.Illumina.tsv \
             --singlem_packages ~/Share/Databases/singlem_pkgs_r95/S2.8.ribosomal_protein_S2_rpsB.gpkg.spkg \
             --threads 4

# Nanopore data
singlem pipe --sequences 03_TRIMMED/nanopore.fastq.gz \
             --otu_table 05_TAXONOMIC_PROFILE/singleM.nanopore.tsv \
             --singlem_packages ~/Share/Databases/singlem_pkgs_r95/S2.8.ribosomal_protein_S2_rpsB.gpkg.spkg \
             --threads 4
```

### sourmash

And now `sourmash`:  

```bash
conda activate sourmash

# Illumina data
for sample in $(cat SAMPLES.txt); do
  sourmash sketch dna 03_TRIMMED/${sample}.illumina.R?.fastq.gz \
                      -p k=31,scaled=1000,abund \
                      -o 05_TAXONOMIC_PROFILE/${sample}.sig.zip \
                      --merge ${sample}

  sourmash gather 05_TAXONOMIC_PROFILE/${sample}.sig.zip \
                  ~/Share/Databases/gtdb-rs207.genomic-reps.dna.k31.zip \
                  -k 31 \
                  -o 05_TAXONOMIC_PROFILE/${sample}.gather.csv
done

# Nanopore data
sourmash sketch dna 03_TRIMMED/nanopore.fastq.gz \
                    -p k=31,scaled=1000,abund \
                    -o 05_TAXONOMIC_PROFILE/nanopore.sig.zip

sourmash gather 05_TAXONOMIC_PROFILE/nanopore.sig.zip \
                ~/Share/Databases/gtdb-rs207.genomic-reps.dna.k31.zip \
                -k 31 \
                -o 05_TAXONOMIC_PROFILE/nanopore.gather.csv

# Gather results
sourmash tax metagenome -g 05_TAXONOMIC_PROFILE/*.gather.csv \
                        -t ~/Share/Databases/gtdb-rs207.genomic-reps.dna.k31.zip \
                        --output-dir 05_TAXONOMIC_PROFILE \
                        --output-base sourmash \
                        --output-form lineage_summary \
                        --rank genus
```

### Visualizing the taxonomic profiles  

Now that we have got our hands into some tables describing the abundance of the different taxa in our metagenome, it is time to make sense of the data.  
One way to do this is making summaries, plots, statistical tests, etc, as you would normally do for any kind of species distribution data.  
Here you are free to use whichever tool you are most familiar with (but we all know that there is only one co`R`rect tool for this).  

The idea here is to: 
- Compare the taxonomic profiles from each different tools  
- Compare the taxonomic profiles obtainted from Illumina and Nanopore data  
- Learn what are the main (most abundant) taxa in our samples  
- Learn about potential differences in community composition between the samples  
- Learn what fraction of the community we were actually able to identify at, let's say, the genus level  

Hopefully you will be able to learn a bit about these metagenomic datasets.  
And realise that there is so much that still remains unknown...  

If you don't have R installed or can't install packages yourself, we have prepared a virtual Rstudio for you with example data.  
Just click this: [![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/karkman/binder_rstudio/main?urlpath=rstudio)

## Metagenome assembly

```bash
cd ~/Physalia_EnvMetagenomics_2023
mkdir 06_ASSEMBLY
```

For metagenomic assembly of our Nanopore data we will use [Flye](https://github.com/fenderglass/Flye). Flye is a long-read de novo assembler that handles also metagenomic data.  

Before you start the assembly, have a look at the [Flye manual](https://github.com/fenderglass/Flye/blob/flye/docs/USAGE.md), escpecially the parts about Nanopore data and metagenome assembly.

__What options do we need?__  
We have only given the output directory in the script below.

```bash 
conda activate flye

flye  ... \
      --out-dir 06_ASSEMBLY

conda deactivate 
```

### Assembly QC

The metagenomic assembly was done with heavily downsampled sequence data. But we have prepared a metagenomic assembly from a bigger set for you.  
Copy the bigger assembly from the `Share` folder to the same output folder as metaflye output. We will run QC for both assemblies and compare the outputs.  

```bash
# export STUDY="WWTP"
# export STUDY="Tundra"

cp ~/Share/${STUDY}/aassembly/* 06_ASSEMBLY/

mkdir 07_ASSEMBLY_QC
```

For assembly QC we will use the metagenomic version of Quality Assessment Tool for Genome Assemblies, [Quast](http://quast.sourceforge.net/) for evaluating (and comparing) our assemblies.

```bash
conda activate quast
metaquast.py 06_ASSEMBLY/*.fasta \
      --output-dir 07_ASSEMBLY_QC \
      --max-ref-number 0 \
      --fast \
      --threads 4
```

## Genome-resolved metagenomics with anvi'o

```bash
mkdir 08_ANVIO
conda activate anvio
```

### Contigs database and annotations

### Mapping Illumina reads back to assembly

### Profiling 

### Visualization

### Metagenomic binninng
