# CNRein

[![PyPI version](https://badge.fury.io/py/CNRein.svg?cacheSeconds=60)](https://badge.fury.io/py/CNRein)

CNRein (formerly known as DeepCopy) is a deep reinforcement learning based evolution-aware algorithm for haplotype-specific copy number calling on single cell DNA sequencing data. 

<p align="center">
  <img width="1000" height="220" src="./overview.png">
</p>

> **_NOTE:_**  This is an alternative version of CNRein adapted for usage at the Computational Oncology group. The original version of CNRein can be found [here](https://github.com/elkebir-group/CNRein).

## Installation

### Manual

Manual installation is currently available and can be achieved by cloning this GitHub repository and installing the below requirements all available on conda:
- Python 3
- numpy
- pandas
- samtools
- bcftools
- pysam
- statsmodels
- pytorch
- shapeit4
- tqdm
- matplotlib
- dendropy
- scikit-bio

## Usage

### With manual installation

If installed manually, the default usage is 

```bash
python script.py -input <BAM files folder>/*.bam \
    -seperateBAMs \
    -ref <reference folder location> \
    -output <location to store results> \
    -refGenome <either "hg19" or "hg38"> \
    -cellConfigFile <bestfitting file location>
```

An example usage in the CNIO cluster could be as below

```bash
sbatch -o CNRein_DLP47-51_log.txt  -p long -t10000 --mem=100G -J CNRein --wrap "python script.py -input /storage/scratch01/groups/co/scdnaseq_processing/DLP47-51/alignment/markedbams/*.bam -seperateBAMs -ref /home/_groups/co/reference/CNRein-ref-renamed -output /storage/scratch01/groups/co/pipeline_testing/CNRein_DLP47-51 -refGenome hg19 -cellConfigFile /storage/scratch01/groups/co/scdnaseq_processing/DLP47-51/abscn/absoluteCN_bestfitting.txt"
```

Remember to edit the absoluteCN_bestfitting.txt file to indicate which samples you want to run CNRein on, along with their ploidy values. This file should be in the format:

```
sampleID	use	ploidy	purity	clonality	powered	LOHok	essentialok	essential_hloss
DLP47_A1.markdup	TRUE	2.1	1	0.00945457055904532	TRUE	TRUE	TRUE
DLP47_A12.markdup	FALSE	3.1	1	0.0622335579809648	TRUE	TRUE	FALSE	CWF19L2
DLP47_B1.markdup	FALSE	6.3	1	0.0662268847835926	TRUE	TRUE	FALSE	CPSF2,RINT1,SNW1
DLP47_G1.markdup	FALSE	6.8	1	0.065422716283008	TRUE	TRUE	TRUE
DLP48_A1.markdup	TRUE	2.1	1	0.0194238916537555	TRUE	TRUE	TRUE
DLP48_A11.markdup	TRUE	2.2	1	0.0540640563468282	TRUE	TRUE	TRUE
DLP48_B1.markdup	TRUE	2.1	1	0.0142949794564785	TRUE	TRUE	TRUE
...
```

Additionally, one can run only parts of the CNRein pipeline by commenting some lines in the DeepSomaticCopy/pipeline.py file:
```python
def runEverything(bamLoc, refLoc, outLoc, refGenome, doCB=False, maxPloidy=10, cellConfigFile= None):

    runAllSteps(bamLoc, refLoc, outLoc, refGenome, useCB=doCB, cellConfigFile=cellConfigFile)
    runProcessFull(outLoc, refLoc, refGenome, cellConfigFile=None)
    scalorRunAll(outLoc, maxPloidy=maxPloidy, cellConfigFile=cellConfigFile)
    easyRunRL(outLoc)
    saveReformatCSV(outLoc, isNaive=False)
    findTreeFromFile(outLoc)
```
- The "runAllSteps" processes the BAMs. Desired cells are selected and merged into one single bam to extract the total reads per cell per chromosome.
- The "runProcessFull" step utilizes BAM data as inputs, and produces intermediary files (stored in "initial", "counts", "info", "phased", "phasedCounts" and "readCounts").
- The "scalorRunAll" step produces segments with haplotype specific read counts and GC bias corrected read depths (stored in "binScale") and generates NaiveCopy's predictions stored in "finalPrediction".
- The "easyRunRL" step utilizes the outputs of both "runProcessFull" and "scalorRunAll", and produces predictions in "finalPrediction", as well as the neural network model stored in "model".
- The "saveReformatCSV" step transforms the .npz files into a more interpretable .csv for subsequent analyses.
- The "findTreeFromFile" step computes the phylogenetic tree using the native CNRein algorithm.

In terms of the precise files, we have the following. 

#### runAllSteps step
Inputs: "-input" BAM location, "-cellConfigFile" (optional) file location.

Outputs: All files in ./counts, ./info, ./phased, ./phasedCounts and ./readCounts. 

#### runProcessFull step
Inputs: All files in ./counts, ./info, ./phased, ./phasedCounts and ./readCounts.

Outputs: All files in ./initial.

#### scalorRunAll step
Inputs: All files in ./initial.

Outputs: All files in ./binScale. Additionally, "./finalPrediciton/CNNaivePrediction.csv". 

#### easyRunRL step
Inputs: In ./binScale the files "BAF_noise.npz", "bins.npz", "chr_avg.npz", "filtered_HAP_avg.npz", "filtered_RDR_avg.npz", "filtered_RDR_noise.npz", and "initialUniqueCNA.npz". 

Outputs: In ./model the files "model_now.pt", and "pred_now.npz". Additionally, "./finalPrediciton/CNReinPrediction.csv".

## Input requirements

The default reference files are publically available at https://zenodo.org/records/10076403. 
The final output in the form of an easily interpretable CSV file is produced in the folder "finalPrediction" within the user provided "-output" folder. 





