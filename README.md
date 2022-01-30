# Antibody Modelling Benchmark

This repository contains the raw, intermediate and output data involved in the comparison of three tools of protein structure modelling from plain amino acid sequence. The evaluated tools are:

* [RoseTTAFold](/RosettaCommons/RoseTTAFold)
* [SWISS-MODEL](https://swissmodel.expasy.org/)
* [ABodyBuilder](http://opig.stats.ox.ac.uk/webapps/abodybuilder)

Each folder in this repository is named after an antibody PDB entry whose crystal structure was used as the baseline in the calculation of RMSD values.


Overall Procedure
---

1. Antibody amino acid sequences were obtained from [IMGT](https://www.imgt.org/).
2. Antibody crystal structures were obtained from [RCSB](https://www.rcsb.org/).
3. The three tools were used to generate 3D structures for each antibody sequence.
4. [ProFit](http://www.bioinf.org.uk/software/profit/) was used to calculate RMSD of the modelled structure and the crystal structure.
5. [GraphPad](https://www.graphpad.com/) was used for statistical analysis.


RoseTTAFold
---

* **Input**: Antibody heavy and light chain sequences in **FASTA** format, i.e., `input_h.fa` and `input_l.fa`
* **Output**: `complex_0001.pdb`

Run the following scripts at a command line. It may take hours to days to run depending on user's computer performance.

```bash
# Generating MSA
run_e2e_ver.sh input_h.fa .
run_e2e_ver.sh input_l.fa .

# Generating pMSA
python make_joint_MSA_bacterial.py H.a3m L.a3m

# Filtering paired alignment
hhfilter -i paired.a3m -o filtered.a3m -id 90 -cov 75

# Modeling 3D structure
complex.sh H.fa .
complex.sh L.fa .

# Relaxing complex structure
relax.static.linuxgccrelease \
-database rosetta_bin_linux_2021.16.61629_bundle/main/database \
-s complex.pdb \
-out:pdb \
-relax:fast \
-relax:constrain_relax_to_start_coords \
-relax:ramp_constraints false \
-ex1 \
-ex2 \
-use_input_sc \
-flip_HNQ \
-no_optH false \
-nstruct 1
```


SWISS-MODEL Modelling
---

* **Input**: Plain antibody heavy and light chain sequences
* **Output**: `<PDB_ID>_GMQE.pdb`


ABodyBuilder Modelling
---

* **Input**: Plain antibody heavy and light chain sequences
* **Output**: `<PDB_ID>_ab.pdb`


RMSD Calculation
---

A documentation of **ProFit** can be found [here](https://structbio.vanderbilt.edu/xray/manuals/ProFit.pdf).

To calculate the RMSD, first launch **ProFit** at a command line:
```bash
profit
```

Then run the following scripts under the **ProFit** prompt, replacing with actual file names and parameters:
```bash
reference 1fl5_heavy.pdb # Heavy chain of 1fl5 crystal structure
mobile 1fl5_090_heavy.pdb # Heavy chain of structure which chose template with GMQE score 0.9
atoms N,CA,C,O # Backbone atoms
zone 1-25 # The framework regions of the first fit
zone 34-50
zone 61-98
zone 109-119
fit
rzone 26-33 # Fit CDR regions
delrzone * # Must clear the rzone before testing another CDR region
rzone 51-60
delrzone *
rzone 99-108
```
