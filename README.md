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

Run the following scripts at a command line. We used `input.fa` as the input and `complex_0001.pdb` as output here.

```bash
# Generating MSA
run_e2e_ver.sh input.fa .

# Generating pMSA
python make_joint_MSA_bacterial.py H.a3m L.a3m

# Filtering paired alignment
hhfilter -i paired.a3m -o filtered.a3m -id 90 -cov 75

# Modeling 3D structure
complex.sh H.fa .

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

* **Input**: Antibody heavy and light chain sequences
* **Output**: `<PDB_ID>_GMQE.pdb`


ABodyBuilder Modelling
---

* **Input**: Antibody heavy and light chain sequences
* **Output**: `<PDB_ID>_ab.pdb` or `<PDB_ID>A.pdb`


RMSD Calculation
---

A documentation of **ProFit** can be found [here](https://structbio.vanderbilt.edu/xray/manuals/ProFit.pdf).

To calculate the RMSD, first launch **ProFit** at a command line:
```bash
profit
```

Then run the following scripts under the **ProFit** prompt, replacing with actual file names and parameters:
```bash
reference PDB_ID.pdb
mobile MODEL_NAME.pdb
atoms N,CA,C,O # Backbone atoms
zone 24-34 # The framework region of the first fit
fit
rzone 30-40 # CDR region
delrzone * # Must clear the rzone before testing another CDR region
rzone 40-50
```
