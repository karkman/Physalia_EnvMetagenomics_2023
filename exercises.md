# Exercises

1. [Exercises](#exercises)
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

mkdir 01_DATA
cp ~/Share/Data/$STUDY/raw/* 01_DATA
cp ~/Share/Data/$STUDAY/SAMPLES.txt .
```

## QC and trimming

Now that you have copied the raw data to your working directory, let's do some quality control.  
We will use [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc) and [MultiQC](https://multiqc.info/) for quality control, and [Cutadapt](https://cutadapt.readthedocs.io/en/stable/) and [chopper](https://github.com/wdecoster/chopper) for trimming the Illumina and Nanopore data, respectively.  

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

After the QC is finished, copy the MultiQC report (`02_QC_RAW/multiqc_report.html`) to your local machine and open it with your favourite browser.  
We will go through the report together before continuing with the pre-processing.

### Read trimming

```bash
mkdir 02_TRIMMED
conda activate QC
```

**Illumina data:**  

__ANTTI__: ARE THE ADAPTERS THE SAME FOR YOUR SAMPLES? ARE YOUR SAMPLES NEXTSEQ TOO?

```bash
for sample in ${cat SAMPLES.txt}; do
  cutadapt 01_DATA/${sample}.illumina.R1.fastq.gz \
           01_DATA/${sample}.illumina.R2.fastq.gz \
           -o 02_TRIMMED/${sample}.illumina.R1.fastq.gz \
           -p 02_TRIMMED/${sample}.illumina.R2.fastq.gz \
           -a CTGTCTCTTATACACATCTCCGAGCCCACGAGAC \
           -A CTGTCTCTTATACACATCTGACGCTGCCGACGA \
           -m 50 \
           -j 4 \
           --nextseq-trim 20 &> 02_TRIMMED/${sample}.illumina.log
done
```

**Nanopore data:**  

```bash
gunzip -c 01_DATA/nanopore.fastq.gz | chopper -q 10 -l 1000 -t 4 | gzip > 02_TRIMMED/nanopore.fastq.gz
```

### QC of the trimmed data

## Read-based taxonomic profiling