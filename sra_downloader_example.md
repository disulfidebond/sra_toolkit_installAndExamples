## Overview
This writeup provides an example of using basic scripting with the SRA downloader to download and cnvert SRA files from NCBI.

### Prerequisites
* The sra toolkit must be installed.  [There is a description of how to do this here](https://github.com/disulfidebond/sra_toolkit_installAndExamples/blob/version1/sra_downloader.md).
* Basic knowledge of Bash or Python scripting.  A Bash example is provided here.

### Steps to retrieve all Runs associated with a given BioProject
1) Search for an SRA term to be downloaded, then select one of the SRA entries. Below is an example of searching for an SRA dataset from the University of Oregon, then selecting that SRA Experiment entry to get more information.
![](https://github.com/disulfidebond/sra_toolkit_installAndExamples/blob/version1/images/sra_img1.png)

2) Next, select the BioProject to retrieve a listing of that BioProject
![](https://github.com/disulfidebond/sra_toolkit_installAndExamples/blob/version1/images/sra_img2.png)

3) Then, click the URL that shows all SRA Experiments
![](https://github.com/disulfidebond/sra_toolkit_installAndExamples/blob/version1/images/sra_img3.png)

4) To retrieve all SRA Runs for this BioProject, in the window that opens, click 'Send to', then click 'File'. Finally, select the 'RunInfo' format under the Format dropdown menu.  Save this file as 'sra_runInfo.csv'.
![](https://github.com/disulfidebond/sra_toolkit_installAndExamples/blob/version1/images/sra_img4.png)

5) Parse out the SRA run ID's only from this file.  A simple example of this would be:

        cat sra_runInfo.csv | cut -d, -f1 > sra_runInfo.parsed.csv
        
6) Then, run the fasterq-dump tool for each of the identifiers:

        declare -a ARR
        ARR=($(<sra_runInfo.parsed.csv))
        for i in "${ARR[@]}" ; do
          fasterq-dump -p -S $i # -p shows a progress bar -S splits the reads into different files
          echo "Downloaded $i"
          sleep 1
        done
        echo 'SRA download complete'

### Comments
* Do not run AsperaConnect when running the sra-toolkit.  If you do so, you will get unhelpful network socket errors at random intervals.

* The default behavior for sra-toolkit is to place the temporary directory within your $HOME directory.  If you are downloading lots of files, make a symbolic link to an external directory:

        pwd
        # outputs /home
        ls
        # outputs Documents Desktop ncbi
        mv ncbi /Volumes/ExternalDrive/
        ln -s /Volumes/ExternalDrive/ncbi ncbi
        
* It also can be helpful to set up a loop, similar to this:

        let j=1 ; 
        for i in "${ARR[@]}" ; do 
          fasterq-dump -p -S $i ; 
          echo "download complete for $i" ; 
          if [ "$j" -ne 1 ] ; then 
            GZNAME=$(echo "${i}_${j}.fastq") ; 
            gzip $GZNAME ; 
            let j=1 ; 
            echo "gzip complete for $GZNAME" ; 
          else 
            GZNAME=$(echo "${i}_${j}.fastq") ;  
            gzip $GZNAME ; 
            let j=$j+1 ; 
            echo "gzip complete for $GZNAME" ; 
          fi ; 
          sleep 1 ; 
        done
