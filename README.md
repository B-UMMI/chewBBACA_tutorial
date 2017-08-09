
## Objective
The objective of this tutorial is to illustrate the complete workflow of a chewBBACA pipeline for creating a wgMLST and a cgMLST schema for a colection of 714 _Streptococcus agalactiae_ genomes (32 complete genomes and 682 assemblies deposited on the Sequence Read Archive) by providing step-by-step instructions and displaying the obtained outputs.

All information about NCBI genomes used in this example is on the [.tsv file](https://github.com/B-UMMI/chewBBACA_tutorial/tree/master/genomes/NCBI_genomes_proks.Sagalactiae_allGenomes.2016_08_03.tsv).
 inside the `genomes` folder. 

The setup is done by the following steps   
1. do a `git clone https://github.com/B-UMMI/chewBBACA_tutorial` to download the entire repository in the folder of your choice 
2. `cd chewBBACA_tutorial/` to enter the dir
3. `unzip genomes/complete_genomes.zip` to extract all the complete genomes files. a directory named complete_genomes will be available at `chewBBACA tutorial/`

The reported times using the 32 complete genomes were calculated for a Dual QuadCore laptop with  intel i7 2.4GHz using 6 cores. Using slower machines or using less number of CPUs can greatly increase the duration of the analyses.
 

## Schema creation 

The wgMLST schema was created using the **32** _Streptococcus agalactiae_ complete genomes (32 genomes with a level of assembly classified as complete genome or chromossome)  available at NCBI. 
The sequences are present in the `complete_genomes/` folder. The command is the following:  

`chewBBACA.py CreateSchema -i complete_genomes/ --cpu 6 -o schema_seed -t "Streptococcus agalactiae"`

The command uses 6 CPU and outputs the schema to `schema_seed` folder using the `prodigal` training set for _Streptococcus agalactiae_ and took around 16 minutes to complete resulting on a wgMLST schema wiht 3129 loci. 
At this point the schema is defined as a set of loci each with a single allele. 

## Allele calling 
The next step was to perform allele calling with the created wgMLST schema for the **32** complete genomes. 

```chewBBACA.py Allelecall -i complete_genomes/ -g schema_seed/ -o results_cg --cpu 6 -t "Streptococcus agalactiae"```

[comment]: <> (JAC IS here in review)

The allele call used the default BSR threshold of 0.6 (more information on the thresold [here](https://github.com/mickaelsilva/chewBBACA/wiki/AlleleCalling)) and took approximately 22 mins to complete (an average of 41 secs per/for each genome)  

## Paralog detection 

The next step on the analysis is to determine if some of the loci can be considered paralogous, based on the result of the wgMLST allele calling. The Allele call returns a list of Paralogous genes in the `RepeatedLoci.txt` file that can be found on the `results_cg` folder. 
The output example is present in `chewBBACA_tutorial/results_cg/results_20170809T100937/`. In `chewBBACA_tutorial/results_cg/results_20170809T100937/RepeatedLoci.txt`, a set of 24 loci were identified as possible paralogs 
that were removed from further analysis. For a more detailed description see the [Alelle Calling](https://github.com/B-UMMI/chewBBACA/wiki/2.-Allele-Calling) entry on the wiki. 


`chewBBACA.py RemoveGenes -i results_alleles.txt -g RepeatedLoci.txt -o alleleCallMatrix_cg`

A set of **1133** loci were found to be present in all the analyzed complete genomes, while **1264** loci were present in at least 95%.

`chewBBACA.py TestGenomeQuality -i alleleCallMatrix_cg.tsv -n 13 -t 200 -s 5`

![Genome quality testing of complete genomes](http://i.imgur.com/Zh6GRk9.png)
[larger image fig 1](http://i.imgur.com/Zh6GRk9.png) or [see interactive plot online](http://im.fm.ul.pt/chewBBACA/GenomeQual/GenomeQualityPlot_complete_genomes.html)

For further analysis only the **1264** loci that represent the cgMLST schema will be used. The list can be retrieved from the `analysis_cg/Genes_95%.txt` that TestGenomeQuality creates. 
There you can find the list of genes present in 95% of the strains per threshold, in this case we will use any from 60 to 195. You can see the list file with **1264** loci at `analysis_cg/listgenes_core.txt` and for future use you should add for each loci the full path for each loci fasta file.

## Allele call for 682 Streptococci agalactiae assemblies 

683 assemblies of _Streptococcus agalactiae_ available on NCBI were downloaded ( 03-08-2016, downloadable zip file [here](https://drive.google.com/file/d/0Bw6VuoagsdhmaWEtR25fODlJTEk/view?usp=sharing)) 
and analyzed with [MLST](https://github.com/tseemann/mlst) in order to exclude possible mislabeled 
samples as _Streptococcus agalactiae_. 

A total of **682 genomes** were downloaded being 2 (GCA_000323065.2_ASM32306v2 and GCA_001017915.1_ASM101791v1) detected as being of a different species/contamination and removed from the analysis. 
Allele call was performed on the remaining 680 genomes using the **1264 loci** for schema validation. Paralog detection found no paralog loci.


```chewBBACA.py Allelecall -i .genomes/ -g listgenes_core.txt -o results --cpu 20 -t "Streptococcus agalactiae"```

Run on a slurm based HPC with 20 cpu took approximately 41 mins to complete (an average of 3.5 secs per/for each genome) 

Since our previous complete genomes allele call was performed with the wgMLST schema we need to remove the loci that constitute the auxiliary genome, in order to be able to compare with the cgMLST allele call we performed for the 680 genomes.  
We select the alleleCallMatrix_cg.tsv file we previously created (already paralog free but still a wgMLST profile) and we extracted only the locus present in 95% of the matrix.

```ExtractCgMLST -i alleleCallMatrix_cg.tsv -o cgMLST_completegenomes -p 0.95```

Now the file `cgMLST_completegenomes/cgMLST.tsv` can be concatenated with the allele call result from the 680 genomes `results_all/results_20170809T110653/results_alleles.tsv`. The concatenated file can be found at `analysis_all/cgMLST_all.tsv`.

The new concatenated file was analyzed in order to assess the cgMLST allele quality attribution for all the genomes.

`chewBBACA.py TestGenomeQuality -i results_alleles.tsv -n 13 -t 300 -s 5`

![Genome quality testing of all genomes](http://i.imgur.com/j4u22ZE.png)
[larger image here fig 2](http://i.imgur.com/j4u22ZE.png) or [see interactive plot online](http://im.fm.ul.pt/chewBBACA/GenomeQual/GenomeQualityPlot_all_genomes.html)

While the presence of locus in 95% of genomes remains constant at around **1200** loci, considering all 
or most of the genomes (90%<x<100%) the number of loci present in all genomes is very low (34) and presents some variation when some specific genomes are removed from the analysis.

We selected the results at the threshold of 25 for further analysis as it presented a significant
 loci presence in 95% of genomes change (+50 loci) and an acceptable loci 
 presence in all considered genomes (650 genomes/437 loci). 
 
The genomes that were detected for each threshold are presented on the file `analysis_all/removedGenomes.txt` and a file `analysis_all/removedGenomes_25.txt` was created with only the genomes detected for the threshold at 25.

`chewBBACA.py ExtractCgMLST -i cgMLST_all.tsv -o cgMLST_25 -g removedGenomes_25.txt`

## Minimum Spanning Tree
`analysis_all/cgMLST_25/cgMLST.tsv` was uploaded to Phyloviz online and can be accessed [here](https://online.phyloviz.net/main/dataset/share/cfab1610a3ca3a80cf9c139e436ce741fc5fa29dcc5aeb3988025491d4434143fc72f6284aaff7d60c6a2ae5e19f57d6be3e5e0baf679a7e37d4ecb96f1d746b8a7cee5882f4a65f586967bd0143)


## Genome Quality analysis
Since no quality control was previously performed, it is possible that some assemblies 
here included were of lower quality. A general analysis of the assemblies show a N50 
variation that ranges from 8055 to over 2.2M, while the number of contigs ranges between 
1 and 553 per sample. These results made us question how the quality of the 
genomes might had affected the allele call results and consequently provoked severe drop on 
locus present in all genomes.  

As stated previously, in order to obtain a considerable comparable cgMLST profile, 
some genomes had to be removed (62) since they presented more missing data than others. 
In order to assess the possible reason of their poor allele call performance, two plots 
were built. The removed genomes were then highlighted and dashed lines linking the same 
genomes annotations. 

The first plot representing the total number of bp on contigs with a size over 
10k bp and the N50 of all assemblies, sorted by decreasing values

![Genome Analysis](http://i.imgur.com/I0fNqtd.png)
[larger image fig 3](http://i.imgur.com/I0fNqtd.png)

The second plot representing the total number of contigs and the number of 
contigs over 10k bp

![Genome Analysis 2](http://i.imgur.com/fabxi0Z.png)
[larger image fig 4](http://i.imgur.com/fabxi0Z.png)
  
[See interactive plot online](http://im.fm.ul.pt/chewBBACA/GenomeQual/AssemblyStatsStack.html)

At first sight, most of the removed genomes (57) were located on the lower number of 
bp/N50 (fig.3) and the higher number of contigs (fig.4)

The remaining 5 genomes were individually checked :
    
1. **GCA_000186445.1** [here](https://www.ncbi.nlm.nih.gov/assembly/GCA_000186445.1) - 21 contigs but only 1 is above 10k (Scaffold with lot of N ,134 real contigs)
2. **GCA_000221325.2** [here](https://www.ncbi.nlm.nih.gov/assembly/GCA_000221325.2)- NCBI curated it out of RefSeq because it had a genome length too large
3. **GCA_000427055.1** [here](https://www.ncbi.nlm.nih.gov/assembly/GCA_000427055.1)- NCBI curated it out of RefSeq because it had many frameshifted proteins
4. **GCA_000289455.1** [here](https://www.ncbi.nlm.nih.gov/assembly/GCA_000289455.1)- No ST found. Assembly has a problem not yet detected?
5. **GCA_000288835.1** [here](https://www.ncbi.nlm.nih.gov/assembly/GCA_000288835.1)- NCBI curated it out of RefSeq because it had many frameshifted proteins


## Schema Evaluation 
Schema Evaluator was run on the cgMLST schema :

`chewBBACA.py SchemaEvaluator -i schema_seed/ -l rms/RmS.html -ta 11 --title "cgMLST schema GBS tutorial schema evaluator" --cpu 6`

[See the schema evaluator page here](http://im.fm.ul.pt/chewBBACA/SchemaEval/rms/RmS.html)
