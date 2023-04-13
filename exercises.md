# Exercises

1. [Exercises](#exercises)
   1. [Setting up the cloud computing](#setting-up-the-cloud-computing)
      1. [Setting up VS Code](#setting-up-vs-code)
      2. [Cloning the course's GitHub repository](#cloning-the-courses-github-repository)
   2. [Getting the raw data](#getting-the-raw-data)
   3. [Basic QC of sequencing data](#basic-qc-of-sequencing-data)
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

**First day:** 

```bash
cd ~
git clone https://github.com/karkman/Physalia_EnvMetagenomics_2023
```

**Every once in a while, at least each day before starting the activities:**  

```bash
cd ~/Physalia_EnvMetagenomics_2023
git pull
```

All exercises will be executed inside the course repository folder (`Physalia_EnvMetagenomics_2023`) that you cloned to your own home folder.

## Getting the raw data

Choose which data set you would like to analyse on the course (`Tundra` or `WWTP`).  
Copy the raw sequence data to your own `01_DATA` folder.  
Before copying, uncomment the study you chose (remove `#`).

```bash
#STUDY="WWTP"
#STUDY="Tundra"

cp ~/Share/Data/$STUDY/raw/* 01_DATA/
```

## Basic QC of sequencing data

## Read-based taxonomic profiling