# exo-datacards

Combine datacards for CMS exotica analyses.

## Steps to upload datacards

An example datacard submission is in the `EXO-16-056` directory. 

1. Fork this repository under your name space
2. Clone your fork of this repository
3. Copy your datacards into a folder named after your CADI line (e.g. `EXO-16-056`) with a `README.md` containing (a) special instructions or tags to set up combine (if necessary), (b) example combine command (with special options if any), and (c) expected output
4. Create a folder named "Checks" and copy there the following four files: fitResults_t0, fitResults_t1, impacts_t0.pdf, impacts_t1.pdf

First, install the Combine CC7 release CMSSW_10_2_X - recommended version from [http://cms-analysis.github.io/HiggsAnalysis-CombinedLimit/#setting-up-the-environment-and-installation](http://cms-analysis.github.io/HiggsAnalysis-CombinedLimit/#setting-up-the-environment-and-installation)

Install the Combine Harvester package:

	cd $CMSSW_BASE/src
	git clone git@github.com:cms-analysis/CombineHarvester.git
	scram b -j 8

Set the environment and authenticate your grid certificate:

	cmsenv
	voms-proxy-init -voms cms -rfc --valid 168:0


* How to get fitResults_t0 and fitResults_t1:

      text2workspace.py datacard.txt  # Load your physics model with the -P option in case you do not use the standard one

      combine -M FitDiagnostics -d datacard.root -t -1 --expectSignal 0 --rMin -10 --forceRecreateNLL -n _t0  # Increase the rMin value if (rMin * Nsig + Nbackground) < 0 for any channel
      python $CMSSW_BASE/src/HiggsAnalysis/CombinedLimit/test/diffNuisances.py  -a fitDiagnostics_t0.root -g plots_t0.root >> ./fitResults_t0 

      combine -M FitDiagnostics -d datacard.root -t -1 --expectSignal 1  --forceRecreateNLL -n _t1
      python $CMSSW_BASE/src/HiggsAnalysis/CombinedLimit/test/diffNuisances.py  -a fitDiagnostics_t1.root -g plots_t1.root >> ./fitResults_t1 

* How to get impacts_t0.pdf and impacts_t1.pdf:

      combineTool.py -M Impacts -d datacard.root -t -1 --expectSignal 0 --rMin -10 --doInitialFit --allPars -m 1 -n t0
      combineTool.py -M Impacts -d datacard.root -t -1 --expectSignal 1 --rMin -10 --doInitialFit --allPars -m 1 -n t1

      combineTool.py -M Impacts -d datacard.root -o impacts_t0.json -t -1 --expectSignal 0 --rMin -10 --doFits -m 1 -n t0 --job-mode condor --task-name t0 --sub-opts '+JobFlavour = "espresso"\nrequirements = (OpSysAndVer =?= "CentOS7")'
      combineTool.py -M Impacts -d datacard.root -o impacts_t1.json -t -1 --expectSignal 1 --rMin -10 --doFits -m 1 -n t1 --job-mode condor --task-name t1 --sub-opts '+JobFlavour = "espresso"\nrequirements = (OpSysAndVer =?= "CentOS7")'

...wait until the condor jobs finish...

	combineTool.py -M Impacts -d datacard.root -m 1 -n t0 -o impacts_t0.json
	combineTool.py -M Impacts -d datacard.root -m 1 -n t1 -o impacts_t1.json

	plotImpacts.py -i  impacts_t0.json -o  impacts_t0
	plotImpacts.py -i  impacts_t1.json -o  impacts_t1

  
5. Commit your changes to your fork
6. Go to Settings/Members on your fork's webpage and add the group `cms-exo-mci` with Max access level Reporter, 
7. Submit a merge request with your updates to this repository

## Additional resources 
The exotica combine contact will check datacards submitted to this repository as part of the approval process: 
- https://twiki.cern.ch/twiki/bin/view/CMS/ExoApprovalChecklist

Some good references from other PAGs for simple checks your datacards should pass are: 
- https://twiki.cern.ch/twiki/bin/view/CMS/HiggsWG/HiggsPAGPreapprovalChecks
- https://twiki.cern.ch/twiki/bin/viewauth/CMS/SUSPAGPreapprovalChecks

There are also some nice scripts put together by Carlos Erice:
- https://github.com/cericeci/combineScripts