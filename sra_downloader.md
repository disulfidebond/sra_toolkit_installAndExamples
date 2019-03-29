## Overview
If you wish to extract fastq or BAM files from an SRA file, there are two main ways to do this.  You can [download and install the sra-toolkit](), note that there are versions available for Centos, Ubuntu, MacOSX, and Windows.  Alternatively, you can use a docker image.  Both will be explained in this writeup.

### Install and Use SRA-toolkit
1) Download the appropriate binary from the above link
2) If you do not have sudo/admin permission, then download it to a local folder, uncompress it, and then add that folder to your PATH.  Here is an example:

         tar -zxvf sratoolkit.2.9.6-mac64.tar.gz
        # this will uncompress the file in the current directory, $PWD in the next command
        export "PATH=$PATH:${PWD}/sratoolkit.2.96-mac64/bin"
        # this will add the sra toolkit to your PATH until you close the current Terminal Window.
        # to make it more permanent, copy the above export command and paste it to your .bash_profile or .bashrc file

3) If you have sudo/admin in access and you want to install it for all users on the computer, do the following:

         tar -zxvf sratoolkit.2.9.6-mac64.tar.gz
        # this will uncompress the file in the current directory
        sudo cp -r sratoolkit.2.9.6-mac64/bin /usr/local/bin/
        # this will copy the sratoolkit binaries to /usr/local/bin

4) To use the sra toolkit, do something similar to:

          fasterq-dump -p -S SRAARCHIVE
         # this will run the fasterq command, it will show progress as it downloads, and it will Split fastq files into R1 and R2
         # here is an example for multiple SRA files:
         for i in {49..54} ; do 
           FQDL=$(echo "SRR17130${i}") ; 
           fasterq-dump -p -S $FQDL ; 
           echo "$FQDL download completed" ; 
           sleep 2 ; 
          done

### Use Docker Image
1) You must have Docker installed, and you must be logged in to your DockerHub account
2) Download a docker image that has the sra toolkit installed, such as:

         docker pull inutano/sra-toolkit

3) Then run this command:

         mkdir sra_download_directory # create new directory, this will be $PWD in the docker command
        cd sra_download_directory
        # the next command does the following, with the following options:
        # # run docker, 
        # # # remove the container after the command is finished
        # # # mount the $PWD as a volume in the Docker container
        # # # then run the command faster-dump to extract fastq files into the $PWD from the SRA archive SRAARCHIVE
        docker run --rm -v "${PWD}":/data -w /data inutano/sra-toolkit fasterq-dump --split-files SRAARCHIVE

### Comments
* Do **NOT** run _fastq-dump_, because this command is older, slower, and will be discontinued soon
* Only use the sratoolkit on a drive with plenty (more than 200GB free) of storage space
