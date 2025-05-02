# sars-cov2-ab
Scripts to prepare database of SARS-Cov2 antibodies.
This repos contains software to prepare antibody database.
The database was used in development of RBD-AIM software described in
Terekhov et al. Mining antibody functionality via AI-guided structural landscape profiling
(Nature Comm. 2025, submittted for publication)

## Requirements 
Conda environment file (environment.yaml or environment.txt) included in the repo.
1) Install conda
2) Install mamba https://anaconda.org/conda-forge/mamba 
3) Create environment:
    mamba env create -n seqvec -f environment.yml
    conda activate seqvec
    Check that SeqVec works properly, there might be issues https://github.com/rostlab/SeqVec
    and data files uniref50_v2 are downloaded

4) Start Jupiter:
   jupyter-notebook
Open one of the scripts
Be careful about paths - most of them are hardcoded and will require fixing

## Input files
1) CoV-AbDab database (last version https://opig.stats.ox.ac.uk/webapps/covabdab/static/downloads/CoV-AbDab_080224.csv)
2) File containing list of pdbids of Ab-RBD complexes (pdbids listed in the CoV-AbDab database may be omitted). This file
   can be manually extracted from rcsb pdb server (https://www.rcsb.org/) using advanced search with the following pattern:
    Scientific Name of the Source Organism is Severe acute respiratory syndrome coronavirus 2 AND
    Scientific Name of the Source Organism is Homo Sapiens AND
    Title has any of words (antibody, antibodies, Fab, Fabs)
3) Excel file of Abs with epitope class from supplement of https://doi.org/10.1038/s41586-022-04980-y

## Scripts
1. proc_covadab_pdb_p1.ipynb
   Load and check CoV-AbDab database, calculates sequqences of CDR1-3 of both chains.
2. proc_covadab_pdb_p2.ipynb
   1) Read covadab with CDRs and select only human Abs
   2) Read list with additional pdbids
   3) Download pdb structures from the rcsb sever
   4) For structures of human antibodies in the covadab database process structure file to 
      find Abs H and L chain and RBD chain, split pdb file to H+L+RBD chains and H+L chains only
      fitted to reference RBD structure
   5) For pdb files which are not in the covadb database, extract Ab name, sequence, CDRs, and
      process structure file by similar way
   6) Fit Ab-RBD to ref complex and calculates decart coordinates of 3 points for Ab
          1. center of mass of RBD heavy atoms interacting with Ab
          2. center of mass of Ab heavy atoms interacting with RBD
          3. center of CA atoms of Av variable domain
   7) Save unfiltered database of Abs
   8) Drop Abs not interacting with RBD (residue 319-541) and write next database version
   9) Calculate Seqvec embedded vectors for CDRs and plot distance between embedded vectors vs distance between epitopes map. Delete outliers based on plotted map and save next version of database
   10) Drop ECM structures which sugnificantly differ from X-Ray
   11) Drop duplicates based on sequence of 6 CDRs and keep the last entries. Save database of Abs with unique CDRs
3. proc_covadab_pdb_p1.ipynb
   1) Read database from part2
   2) Read excel file with Abs epitope
   3) Calculate CDRs of Abs from excel file
   4) Filter excel file by unique CDRs
   5) Remove entries with _omi suffix in epitope
   6) Find intersection of pdb database and filtered excel file
   7) Compare list of residues contacted with RBD and contacts specific for epitope
   8) Excluding Abs with <2 epitope-specific contacts
   9) Create pymol file for Abs epitope visualization
   10) Manually reassign Abs epitope class based on pymol visualization
   11) Create pymol file for Abs epitope visualization (modified)
   12) Calculate centers (3 points for each epitope class) for epitope class and save to npy file
   13) Assign class to Ab from covadab database
   14) output database ready to RBD-AIM software
   15) Create pse file of visalization Abs with pdb structure

