# Enceladus_Medium_Pangenome
Pangenome analysis of 3 pure cultures and 5 MAGs from microbes cultivated in Enceladus Medium 042a.

## Step 1: Install Anvi’o
```
# Installation: Follow the directions here: 
https://anvio.org/install/linux/stable/
```

## Step 2: Build the contig_db for each genome, run the hmms, and annotate the COGs and KEGGs at once for your genomes. 

### On your local computer: 
```
# cd into the directory containing your genomes
cd /vortexfs1/omics/huber/selkassas/Enceladus_pangenome/Enceladus_Genomes

# For loop to build all of the contig_db, run the hmms, and annotate the COGs and KEGGs at once for your genomes: 
# Make sure all of your files have the same extension, or alter the code to account for differences in extensions (ex. *.fa, *.fasta, *.fna):
for file in *.fasta
do
SAMPLE=$(basename ${file} | cut -d '.' -f 1)
anvi-gen-contigs-database -f "${file}" --project-name ${SAMPLE} -o ${SAMPLE}_contigs-db.db 
anvi-run-hmms -c "${SAMPLE}_contigs-db.db"
anvi-run-ncbi-cogs -c "${SAMPLE}_contigs-db.db" --num-threads 4
anvi-run-kegg-kofams -c "${SAMPLE}_contigs-db.db" --num-threads 4
done 
```


### On HPC: 
```
#enter scavenger

srun -p scavenger --time=04:00:00 --ntasks-per-node 1 --mem=5gb --pty bash

#Slurm Script: 

#!/bin/bash
#SBATCH --partition=compute                          # Queue selection
#SBATCH --job-name=generate_pangenome                # Job name
#SBATCH --mail-type=ALL                              # Mail events (BEGIN, END, FAIL, ALL)
#SBATCH --mail-user=selkassas@whoi.edu               # Where to send mail
#SBATCH --ntasks=1                                   # Run a single task
#SBATCH --cpus-per-task=4                            # Number of CPU cores per task
#SBATCH --mem=100gb                                  # Job memory request
#SBATCH --time=24:00:00								   # Time limit hrs:min:sec
#SBATCH --output=generate_pangenome.log     		   # Job log name
export OMP_NUM_THREADS=4

#enter conda and activate environment
eval "$(/vortexfs1/home/selkassas/miniforge3/bin/conda shell.bash hook)" 
conda activate /vortexfs1/home/selkassas/miniforge3/envs/anvio-8

#cd into the directory containing your genomes
cd /vortexfs1/omics/huber/selkassas/Enceladus_pangenome/Enceladus_Genomes

# For loop to build all of the contig_db, run the hmms, and annotate the COGs and KEGGs at once for your genomes: 
# Make sure all of your files have the same extension, or alter the code to account for differences in extensions (ex. *.fa, *.fasta, *.fna):
for file in *.fasta
do
SAMPLE=$(basename ${file} | cut -d '.' -f 1)
anvi-gen-contigs-database -f "${file}" --project-name ${SAMPLE} -o ${SAMPLE}_contigs-db.db 
anvi-run-hmms -c "${SAMPLE}_contigs-db.db"
anvi-run-ncbi-cogs -c "${SAMPLE}_contigs-db.db" --num-threads 4
anvi-run-kegg-kofams -c "${SAMPLE}_contigs-db.db" --num-threads 4
done 
```


## Step 3: Generate genome file and genomes storage (same code for both local and HPC)
```
# Generate genome file
anvi-script-gen-genomes-file --input-dir /vortexfs1/omics/huber/selkassas/Enceladus_pangenome/Enceladus_Genomes \
                             --output-file Enceladus_genomes.txt

# Generate genomes storage
anvi-gen-genomes-storage -e Enceladus_genomes.txt \
                         -o ENCELADUS_GENOMES.db

```

## Step 4: Generate Pangenome

### On your local computer: 
```
# Generate Pangenome
anvi-pan-genome -g ENCELADUS_GENOMES.db \
                --project-name "ENCELADUS_PANGENOME" \
                --output-dir ENCELADUS_PANGENOME \
                --num-threads 4 \
                --minbit 0.5 \
                --min-occurrence 2 #we need this because the genomes are so disparate
```

### On HPC: 
```
#!/bin/bash
#SBATCH --partition=compute                          # Queue selection
#SBATCH --job-name=generate_pangenome                # Job name
#SBATCH --mail-type=ALL                              # Mail events (BEGIN, END, FAIL, ALL)
#SBATCH --mail-user=selkassas@whoi.edu               # Where to send mail
#SBATCH --ntasks=1                                   # Run a single task
#SBATCH --cpus-per-task=4                            # Number of CPU cores per task
#SBATCH --mem=40 gb                                  # Job memory request
#SBATCH --time=24:00:00								   # Time limit hrs:min:sec
#SBATCH --output=generate_pangenome.log     		   # Job log name
export OMP_NUM_THREADS=4

# enter conda and activate environment
eval "$(/vortexfs1/home/selkassas/miniforge3/bin/conda shell.bash hook)" 
conda activate /vortexfs1/home/selkassas/miniforge3/envs/anvio-8

# cd into the directory containing your genomes
cd /vortexfs1/omics/huber/selkassas/Enceladus_pangenome/Enceladus_Genomes

# Generate Pangenome
anvi-pan-genome -g ENCELADUS_GENOMES.db \
                --project-name "ENCELADUS_PANGENOME" \
                --output-dir ENCELADUS_PANGENOME \
                --num-threads 4 \
                --minbit 0.5 \
                --min-occurrence 2 #we need this because the genomes are so disparate
```

## Step 5: Display Pangenome Results:

### On your local computer:

```
# This will launch an interactive browser
anvi-display-pan -p /vortexfs1/omics/huber/selkassas/Enceladus_pangenome/Enceladus_Genomes/ENCELADUS_PANGENOME/ENCELADUS_PANGENOME-PAN.db \
-g /vortexfs1/omics/huber/selkassas/Enceladus_pangenome/Enceladus_Genomes/ENCELADUS_GENOMES.db
```

### On HPC:
```
# Check out Dr. Emilie Skoog’s guide to SSH tunneling. 
https://github.com/emilieskoog/SSH-tunneling/blob/main/SSH%20tunneling%20(specific%20example%20for%20anvi%E2%80%99o).md

# Enter a Local SSH tunnel
local ~ $ ssh -L 8090:localhost:8090 selkassas@poseidon-l2.whoi.edu
# enter your password

# activate your environment
conda activate anvio-8

# display pangenome 
anvi-display-pan -p /vortexfs1/omics/huber/selkassas/Enceladus_pangenome/Enceladus_Genomes/ENCELADUS_PANGENOME/ENCELADUS_PANGENOME-PAN.db \
-g /vortexfs1/omics/huber/selkassas/Enceladus_pangenome/Enceladus_Genomes/ENCELADUS_GENOMES.db \
--server-only -P 8090

Go to a webbrowser and type http://localhost:8090
```

## Step 6: Generate a table showing which gene clusters are present in which genomes
```
# Generate default collection
anvi-script-add-default-collection -p ENCELADUS_PANGENOME-PAN.db

# Generate summary file
anvi-summarize -p /vortexfs1/omics/huber/selkassas/Enceladus_pangenome/Enceladus_Genomes/ENCELADUS_PANGENOME/ENCELADUS_PANGENOME-PAN.db -g /vortexfs1/omics/huber/selkassas/Enceladus_pangenome/Enceladus_Genomes/ENCELADUS_GENOMES.db -C DEFAULT -o PAN_SUMMARY

#check the collection exists
anvi-summarize -p /vortexfs1/omics/huber/selkassas/Enceladus_pangenome/Enceladus_Genomes/ENCELADUS_GENOMES.db --list-collections

# optional: check for shared gene clusters among the genomes
awk 'BEGIN{FS="\t"} NR==1{print; next} {sum=0; for(i=2;i<=NF;i++) sum+=$i; if(sum==(NF-1)) print}' ENCELADUS_PANGENOME/PAN_SUMMARY/ENCELADUS_PANGENOME_gene_clusters_summary.txt > shared_gene_clusters.txt
```


