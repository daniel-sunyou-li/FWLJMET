# CRAB3 Set-up
Note that for `bash`, use `.sh` extensions instead of `.csh`:

	source /cvmfs/cms.cern.ch/cmsset_default.csh
	cmsenv
	source /cvmfs/cms.cern.ch/crab3/crab.csh
	
If you haven't initialized your grid proxy yet:

	voms-proxy-init --voms cms

## Running CRAB3 Jobs
Before running your CRAB3 jobs, check the `sample_list_*UL.py` files to make sure that the samples are correct. Samples are grouped together based on EDAnalyzer run conditions as well as their physics. If you want to test individual samples, use the `TEST` group. In the case that a sample is incorrectly named or unavailable, you can check its status using [`GrASP`](https://cms-pdmv.cern.ch/grasp/), and, in particular, use the `MiniAODv2` filter and follow the "DAS" link to obtain the full correct path. 

__Important Note:__ be mindful of how many jobs you submit at once as your CRAB3 priority will suffer after too long. Also, be diligent about monitoring space usage on either the LPC or BRUX to be sure that there is available space for further sample production. A full `LJMet` production can often take up multiple TBs of space, while we only have 60 TB available on the LPC and 200 TB in BRUX. To check available space on the LPC EOS group area:

	eosgrpquota lpcljm
	
And for more individualized information:

	eosls /store/group/lpcljm/ -lh

__Quick Submission:__  
Be sure to check the possible groups available for submission in `sample_list_*UL.py` before submitting, and you are able to submit multiple groups at once. As an example for the `singleLep` final state for the year `2017` including multiple groups:

	python create_configs.py -f singleLep -y 2017 -g DATA TTTX --shifts 
	python submit_crab_jobs.py -f crab_configs_singleLep2017UL/ -g DATA TTTX --shifts
	python check_crab_jobs.py -l logs_singleLep2017UL/ -g DATA TTTX --report --verbose
	
__Submitting a test job:__  
If you are unfamiliar with using CRAB3, it is recommended to try submitting a test job before running all your samples. After setting up your environment and choosing a test submission, first create the relevant configuration files:

	python create_configs.py -f [FINALSTATE] -y [YEAR] -g TEST --shifts 
	
The `--shift` argument includes the `JEC` and `JER` shifts resulting in multiple trees to be added to the `ntuple` in addition to the `nominal` tree. You can run the same command to produce configuration files that only run on the `nominal` tree. To see available options for `-f` and `-y`, you can run the `-h` option to view possible inputs:

	python create_configs.py -h
	
which applies to any of the following python scripts. After running `create_configs.py`, you should see a directory `crab_configs_[FINALSTATE][YEAR]UL/` with two files inside, one is the EDAnalyzer configuration file beginning with "runFWLJMET" and the other is the CRAB3 configuration file. You can run the CRAB3 test job using:

	python submit_crab_jobs.py -f [FOLDER] -g TEST --shifts
	
where you reference the directory you just created-it will automatically parse through all the subdirectories available. Running `submit_crab_jobs.py` will create a log directory named `logs_[FINALSTATE][YEAR]UL/`. While your job is running, you can check the status using:

	python check_crab_jobs.py -l logs_[FINALSTATE][YEAR]UL/ -g TEST --report
	
to see the state of your CRAB3 jobs. Adding `--verbose` will show you any errors in the CRAB job in greater detail. In case you need to resubmit your jobs, you can replace `--report` with `--resubmit`, and similarly, if you need to kill the job, you can replace with `--kill`.

__Checking the completed jobs:__  
When your jobs finish, they should be stored on the LPC EOS group area by default-the other option is on BRUX. If stored on EOS, you can view them using:

	eosls /store/group/lpcljm/FWLJMET106XUL_[FINALSTATE][YEAR]UL_RunIISummer20/ -lh
