# DeepAutoCov
Code to reproduce the simulation in the following publication:

Forecasting dominance of SARS-CoV-2 lineages by anomaly detection using deep AutoEncoders Simone Rancati, Giovanna Nicora, Mattia Prosperi, Riccardo Bellazzi, Simone Marini, Marco Salemi bioRxiv 2023.10.24.563721; doi: https://doi.org/10.1101/2023.10.24.563721

Scripts to predict anomalies, i.e., Future Dominant Lineages (FDLs) with the Deep Learning AutoEncoder and to perform the simulation are located in the model folder. Scripts to generate the dataset and the feature representations are within the Feature_Extraction folder.

## Feature Extraction
The files to create the dataset is <code>First_Filtration.R</code> and <code>Data_filtration_kmers.py</code>.

I)<code>First_Filtration.R</code> serves as the primary tool for processing the <code>metadataset.tsv</code> file obtained from the GISAID website (https://gisaid.org/) along with its corresponding <code>Spikes.fasta</code> file. It filters and restructures the data, outputting two refined files: <code>metadata.csv</code> and <code>spikes.fasta</code>. Example: <code>Rscript First_Filtration.R metadata.tsv spikes.fasta output_sequences.fasta output_metadata.csv</code>


Mandatory:


-spikes.fasta: path where the input fasta file is stored (Example file: <code>data_github/Spikes_prova.fasta</code>);

-metadata.tsv: path where the input metadata (tsv) is stored (Example file:<code>data_github/pseudodataset.tsv</code>): Sequences and metadata should be in the same order. All columns are necessary and must be in the same order as in the example file, i.e.: <code> Virus name, Last vaccinated, Passage details/history, Type, Accession ID, Collection date, Location, Additional location information, Sequence length, Host, Patient age, Gender, Clade, Pango lineage, Pango version, Variant, AA Substitutions, Submission date, Is reference?, Is complete?, Is high coverage?, Is low coverage?, N-Content, GC-Content</code>

-output_sequences.fasta: path where the output fasta file is saved

-output_metadata.csv: path where the output csv file is saved


-Output:


1) Metadata: Metadata of filtered sequences (<code>metadata.csv</code>);
2) Spikes: File fasta of filtered sequence (<code>spikes.fasta</code>).



II)<code>Data_filtration_kmers.py</code>acts the second phase in the data processing workflow, building upon the outputs generated by the <code>First_Filtration.R</code> script. It takes the metadataset.csv and spikes.fasta files as inputs, which are the refined versions of the original datasets.Example: <code>python Data_Filtration_kmers.py -f Spikes_prova.fasta -c pseudodataset.csv -m 1000 -l 30 -p /path/to/save/dataset_interest_2023 </code>


Mandatory:

-f: path where the input fasta file is stored (Example file: <code>data_github/Spikes_prova.fasta</code>);

-c: path where the input metadata (csv) is stored (Example file: <code>data_github/pseudodataset.csv</code>). Sequences and metadata should be in the same order. All columns are necessary and must be in the same order as in the example file, i.e.: <code> Virus name, Last vaccinated, Passage details/history, Type, Accession ID, Collection date, Location, Additional location information, Sequence length, Host, Patient age, Gender, Clade, Pango lineage, Pango version, Variant, AA Substitutions, Submission date, Is reference?, Is complete?, Is high coverage?, Is low coverage?, N-Content, GC-Content</code>


Optional:
-n: nation (e.g., "France") (if not specified, all sequences are used) (<code>default: ['/']</code>);

-m: Filter: minimum lenght of the spike sequences (<code>default value: 1000</code>); 

-l: Filter: accepted amino acid distant from lineage median (<code>default value: 30</code>); as in: for each lineage, how the protein length can vary to be accepted?

-p: path to save the outputs.


-Output:

1) Metadata: Metadata of filtered sequences (<code>filtered_metadatataset</code>);
2) Dataset: It creates a folder (<code>dataset</code>) that contains subfolders (numbered by weeks <code>i.e; 1</code>). Each subfolder has:
  a) csv file for each sequence (<code>EPI_ISL_6331230.csv</code>).In the first raw contains all the k-mers possible.    The second, instead, contains a sequence of 0/1 that indicates the presence or absence of k-mers.
  b) txt File (<code>week_dataset.txt</code>) that contains the identificators and the sequence of 0/1


III)<code>Supplementary Information Preprocessing.md</code> contains additional information about the pre-processing performed on the data. 


IV)

## Simulation
This repository hosts the code required to replicate simulations for various datasets, including Global, United States of America, United Kingdom, Denmark, and France. Within each dataset's folder, you will find two scripts:

1) <code>DeepAutoCov.py</code>: This is the primary script where the simulation is executed.
2) <code>utils.py</code>: This script contains all the functions utilized in the main script, DeepAutoCov.py.

To run the simulation :
<code>python DeepAutoCov.py -p /path/to/dataset/ -c /path/to/metadata.csv -k /path/to/kmers_file.csv -s /path/where/to/save/output -m 0.1 -e 300 -b 256 -d 1024 -r 1e-7 </code>

Mandatory:

-p path of dataset created during the feature extraction (<code>Exemple: /path/to/save/dataset/</code>);

-c path where <code>filtered_metadatataset</code> is stored (<code>Exemple: /path/to/metadata.csv </code>);

-k path where kmers are stored (example: first line of csv file created in subfolders <code>/path/to/EPI_ISL_6331230.csv</code>).

Optional:

-s path to save the outputs (<code>/path/to/save/drive_save</code>);

-m fraction of kmers that are different from 0 to mantain during the simulation (<code>default value: 0.01</code>);

-e number of epochs (<code>default value: 300</code>);

-b batch size for the first week (<code>default value: 256</code>);

-d Sets the encoding dimension (<code>default value: 1024</code>);

-r learning rate (<code>default value: 1e-7</code>).

-n Simulation Weeks (<code>default value: 2 </code>).


-Output:
1) Precision-graph of the top 100 sequences with higer mean square error (mse) considereted as anomalies by DeepAutoCov model (<code>precision_top100.png</code>);
2) Graph of the precision considering all the sequences considerated anomalies by DeepAutoCov model (<code>precision_overall.png</code>);
3) File txt that contain the features of Autoencoder Model (<code>Features.txt.png</code>);
4) File.h5 which contains the information (weights) of the trained AutoEncoder (<code>Autoencoder_models.h5</code>);
5) Graph of number of features (k-mers) during simulation (<code>number_of_features.png</code>);
6) file CSV that contains for each simulation week and for each sequence the k-mers not reproduced correctly by DeepAutoCov (The first column contain the id_sequence and other columns contain the k-mers not reproduced correctly) (<code>summary_kmers_week.csv</code>).
7) File txt containing for each FDLs the weeks in advance when DeepAutoCov flags them as anomalies (<code>Distance_prediction.txt</code>);
8) File txt containing the median frequency of FDLs when DeepAutoCov flags them as anomalies (<code>median_area.txt</code>);
9) File txt containing precision of DeepAutoCov (<code>precision_top100.txt</code>);
10) File txt that contains for each week of simulation the lineages defined as anomalies by DeepAutoCov (<code>TOP_100.txt</code>). 
