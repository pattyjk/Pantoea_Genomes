## Anvi'o pangenome workflow (v. 5)
```
#load anvio with Anaconda
source activate anvio5

#fix fasta files to be compatible with Anvio
mkdir fixed_reps
mkdir fixed_fasta

#run for loop
for i in *.fna
do
    anvi-script-reformat-fasta $i -o fixed_fasta/$i --simplify-names --report-file=fixed_reps/$i
done

#make contigs database for each genome
mkdir contigs_db
cd fixed_fasta

for i in *.fna
do
    anvi-gen-contigs-database -f $i -o /home/pattyjk/Desktop/pantoea_ncbi/contigs_db/$i.db -n pantoea
done

#generate COGs
cd ..
cd contigs_db

#make a contigis database if not already done (anvi-setup-ncbi-cogs --num-threads 6)
for i in *.db
do
anvi-run-ncbi-cogs -c $i --num-threads 8
done

#run HMMs for single copy genes
#cd contigs_db 

for i in *.db
do
anvi-run-hmms -c $i -T 8
done

cd ..
```

## Make a Anvio genomes database file with bash and R
```
#change back to root
cd /

#write absoulte files paths to a file
ls -R1 ./home/pattyjk/Desktop/pantoea_ncbi/contigs_db |    while read l; do case $l in *:) d=${l%:};; "") d=;; *) echo "$d/$l";; esac; done > /home/pattyjk/Desktop/pantoea_ncbi/db_files.txt

#fix file name
sed -i 's/\.\//\//g' /home/pattyjk/Desktop/pantoea_ncbi/db_files.txt

#change back to folder 
cd /home/pattyjk/Desktop/pantoea_ncbi

#make a genomes name file and fix it up
sed 's/\///g' db_files.txt > gen_names.txt
sed -i 's/homepattyjkDesktoppantoea_ncbicontigs_db//g' gen_names.txt
sed -i 's/\.//g' gen_names.txt
sed -i 's/fnadb//g' gen_names.txt

#open R to generate database
R

#read in file locations
anvi_gen<-read.delim('db_files.txt', header=F)
names(anvi_gen)<-'contigs_db_path'

#read in genome names
anvi_names<-read.delim('gen_names.txt', header=F)
names(anvi_names)<-'name'

#catenate
anvi_gen<-cbind(anvi_names, anvi_gen)

#write file
write.table(anvi_gen, 'anvi_gen.txt', quote=F, row.names=F, sep='\t')

#exit R
quit()
n
```


## Catenate genomes into a single database
```
anvi-gen-genomes-storage -e anvi_gen.txt -o pantoea-GENOMES.db 

#pangenome analysis
anvi-pan-genome -g pantoea-GENOMES.db -n pantoea --enforce-hierarchical-clustering --min-occurrence 2

#view analysis
anvi-display-pan -g pantoea-GENOMES.db -p pantoea/pantoea-PAN.db
```

## Calculate ANI between genomes
```
anvi-compute-ani -o ANI -e anvi_gen.txt -T 8 -p pantoea/pantoea-PAN.db
```

## Make genome tree
```
#generate genome tree
anvi-get-sequences-for-hmm-hits --external-genomes anvi_gen.txt -o concatenated-proteins.fa --hmm-source Campbell_et_al --gene-names Ribosomal_L1,Ribosomal_L2,Ribosomal_L3,Ribosomal_L4,Ribosomal_L5,Ribosomal_L6 --return-best-hit --get-aa-sequence --concatenate

anvi-gen-phylogenomic-tree -f concatenated-proteins.fa -o pantoea_tree.tree
```

## Extract 16S rRNA gene sequences
```
anvi-get-sequences-for-hmm-hits -e anvi_gen.txt --gene-names Bacterial_16S_rRNA -o pantoea_16s.fna --return-best-hit
```

## Annotate biosynthetic clusters with AntiSmash
```
source activate antismash
cd raw_fasta

mkdir anti_out
for i in *.fna
do
antismash --input-type nucl --outputfolder anti_out/out_$i $i 
   
done
```
