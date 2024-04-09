# Description
A web app for gene expression and alternative polyadenylation analysis of TCGA RNA-Seq data.

# Workflow
![image](https://github.com/brss12/APAtizer/assets/121204829/ea7c789d-907a-42bc-b331-94387a9b4325)

## 1. Pre-processing BAM files
The script [bam_analysis.smk](bam_analysis.smk) employs the snakemake workflow to create fastqc reports, sort and remove the duplicates from the raw TCGA BAM files. The script requires the raw BAM files to be placed in a folder called **RAW_BAM** and, upon running, creates two new folders called **SORTED_BAM** and **TRIMMED_READS**. In the **SORTED_BAM** folder, is where the sorted bam files along with fastqc reports are placed and in the **TRIMMED_READS** is where the de-duplicated bam files along with fastqc reports are placed.

Depending on the size and ammount of BAM files, we recommend running the script in a HPC environment. Run the following script on the same directory of the **RAW_BAM** folder. 
```shell
snakemake -s bam_analysis.smk --cores [number_of_cores] --wait-for-files
```
Indexing the de-duplicated BAM files from the previous step is a good practice. The script [index_script.sh](index_script.sh) can be executed by running the following command and specifying as argument the number of threads to use.
```shell
./index_script.sh [number_of_threads]
```
## 2. Creating the DaPars2 txt files
Alternative polyadenylation analysis in APAtizer can be done by using one of two algorithms, APAlyzer and DaPars2. To use the APAlyzer algorithm, the de-duplicated bam files obtained in the previous step are used as input. On the other hand, for the DaPars2 algorithm, the user needs to run the first part of the DaPars2 analysis in the terminal and APAtizer only deals with the final manipulations and calculations to obtain the final gene lists.

The script [dapars2.smk](dapars2_files/dapars2.smk) runs the first part of the DaPars2 analysis. The script must be executed in the same directory of the **TRIMMED_READS** folder. The following command runs the script.
```shell
snakemake -s dapars2.smk --cores [number_of_cores] --wait-for-files
```
After this, run the following command to make a folder called **DaPars_data_final** and move all of the output files to it.
```shell
mkdir DaPars_data_final/ && mv DaPars_data_chr*/*temp* DaPars_data_final/
```
## 3. Creating htseq files for DGE analysis
The htseq files are used in the differential gene expression (DGE) analysis in APAtizer. These files are created via htseq-count by running the script [htseq_script.sh](htseq_script.sh). Also, those files are filtered and manipulated by some extra commands as listed below.
```shell
./htseq_script.sh && mkdir --parents TRIMMED_htseq/FILTERED && mv TRIMMED_READS/*htseq* TRIMMED_htseq/ && mv filter_folder/ TRIMMED_htseq/ && Rscript protein_coding_filter.R
```
All done! Now you are ready to use APAtizer!
# APAtizer walkthrough
## DaPars2 interface
<img src="https://github.com/brss12/APAtizer/assets/121204829/de5fe1fe-6f0c-4167-a109-55dac866f430" alt="DAPARS_data"> 

In this section, in the input space the user can select the 24 output files originated by the DaPars2 analysis that are located in the folder **DaPars_data_final** and the TCGA sample sheet.

In the output space, we can observe the lists of genes that go through 3'UTR APA lengthening events (*Len genes*) and 3'UTR APA shortening events (*Short genes*). The user can also search the lists for a specific gene of interest and download the lists using the download button below the search box.

## APAlyzer interface
<img src="https://github.com/brss12/APAtizer/assets/121204829/f96b5fde-b5c6-4827-9a4d-542281aeca4f" alt="APA_List">
<img src="https://github.com/brss12/APAtizer/assets/121204829/a9ed2b83-dc63-462a-9bd2-43a32168f60f" alt="APA_Volcano">

Here, in the input space the user may paste the full path of the folder **TRIMMED_READS** that contains all the de-duplicated BAM files and the index files, select the TCGA sample sheet. The user may also select the output types of the analysis such as the lists that are displayed and the plots. For the lists, the user can choose between 3'UTR APA lengthening (*NvsT_APA_UP*), 3'UTR APA shortening (*NvsT_APA_DN*) and non-significant (*NvsT_APA_NC*). In the case of the plots, the choice is between a Volcano plot with the top 40 significant genes highlighted (*APA Volcano Plot (top40)*), the same plot but with no highlights (*APA Volcano Plot*) and a box plot (APA Box).

In the output space, in the tab called *Number of APA events* one can see a small table where the number of non significant, lengthening and shortening genes is present. In *NvsT_APA* the full lists chosen in the input space are presented to the user and he can search the list and download it. Finally, in *Plots* the user can visualize the plots selected in the input space and download them as well.

## DGE interface
<img src="https://github.com/brss12/APAtizer/assets/121204829/d69edb1f-7d95-4e78-9da4-9fecd09b7c44" alt="DGE_List">
<img src="https://github.com/brss12/APAtizer/assets/121204829/3ae16b84-87f0-494d-9c08-31b9c048f346" alt="DGE_Pca">
<img src="https://github.com/brss12/APAtizer/assets/121204829/2dd18423-06f7-4285-a528-8dc7b24b1ce2" alt="DGE_Volcano">
<img src="https://github.com/brss12/APAtizer/assets/121204829/4f76e1e6-ff5a-4c8c-8688-f3e9c1e386f9" alt="DGE_Heatmap">

## GO TERMS interface
<img src="https://github.com/brss12/APAtizer/assets/121204829/9c48d65a-e3d9-4c5d-8b4f-c4d9a6f2fdee" alt="GO_BP">
<img src="https://github.com/brss12/APAtizer/assets/121204829/45e43e24-c6c1-48a1-8244-ffb25a837756" alt="GO_MF">

## VENN DIAGRAMS interface
<img src="https://github.com/brss12/APAtizer/assets/121204829/0faebdf0-1baa-4be4-9d0b-75aab8cf99d3" alt="Venn_2">

## SURVIVAL ANALYSIS interface
<img src="https://github.com/brss12/APAtizer/assets/121204829/e661e322-0d8a-4914-8222-404d7d7f7e30" alt="Survival">
