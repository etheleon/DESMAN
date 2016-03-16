# DESMAN _De novo_ Extraction of Strains from MetAgeNomes

![alt tag](desmans.jpg)

##Installation

To install simply type:
    
    sudo python ./setup.py install
    
These items are prerequisities for the installation of desman:

*python v2.7.*
*gcc
*gsl

The installation procedure varies on different systems, 
and described in this README is only how to proceed with a linux (ubuntu) distribution.

The first item, python v2.7.*, should be installed on a modern Ubuntu distribution. 
A c-compiler, e.g. gcc, is needed to compile the c parts of concoct that uses the 
GNU Scientific Library gsl. For linux (ubuntu) this is installed through:

    sudo apt-get install build-essential libgsl0-dev

##Simple example

To illustrate the actual strain inference algorithm we will start with a simple example using base frequencies 
that have been pre-prepared. Below we also give [a complete example](#complete_example) including 
pre-processing. The starting point for a Desman analysis is a csv file with base frequencies e.g.: 

[Strain mock community frequencies for COG0015](data/contig_6or16_genesL_scgCOG0015.freq)

This has the following format:

    Contig,Position,SampleName1-A,SampleName1-C,SampleName1-G,SampleName1-T,...,SampleNameN-A,SampleNameN-C,SampleNameN-G,SampleNameN-T

where SampleName1,...,SampleNameN gives the names of the different samples in the analysis. Followed 
by one line for each position with format:

    gene name, position, freq. of A in sample 1, freq. of C in 1,freq. of G in 1,freq. of T in 1,..., freq. of A in sample N, freq. of C in N,freq. of G in N,freq. of T in N 


###Finding variant positions for the test data set

The first step is to identify variant positions. This is performed by the desman script Variant_Filter.py. 
Start assuming you are in the DESMAN repo directory by making a test folder.

    mkdir test
    cd test

Then run the example data file which corresponds to a single COG from the mock community data set 
described in the manuscript. This COG0015 has 933 variant positions. The input file is in the data 
folder. We run the variant filtering as follows:

    python ../desman/Variant_Filter.py ../data/contig_6or16_genesL_scgCOG0015.freq -o COG0015_out -p

The variant filtering has a number of optional parameters to see them run:

    python ../desman/Variant_Filter.py -h
    
They should all be fairly self explanatory. We recommend always using the 
the '-p' flag for one dimenisonal optimisition of individual base frequencies if it is not 
too time consuming. The '-o' option is a file stub all output files will be generated with this prefix.
A log file will be generated 'COG0015_out_log.txt' and output files: 

1. COG0015_outp_df.csv: This gives p-values for each position.

2. COG0015_outq_df.csv: This gives q-values for each position.

3. COG0015_outr_df.csv: This gives log-ratio statistics for each position.

4. COG0015_outsel_var.csv: This is the file of selected variants.

5. COG0015_outtran_df.csv: A matrix of estimated error rates.

###Inferring haplotypes and abundances for the test data set

Having found the variant positions we will now the run the program for inferring haplotypes and their abundance:

    desman COG0015_outsel_var.csv -g 5 -e COG0015_outtran_df.csv -o COG0015_out_g5 -i 50 

These parameters specify the variants file. Then number of haplotypes as five '-g 5', an initial 
estimate for the error transition matrix taken from the variant detection '-e COG0015_outtran_df.csv', 
an output directory '-o COG0015_out_g5' and the number of iterations, '-i 50'.
The program takes the selected variants and infers haplotypes and their abundances using the Gibbs sampler given 
the assumption that five strains are present. All output files will be generated in the directory COG0015_out_g5. 
Once the program has finished running a few minutes on a typical computer it will generate the following 
files inside the output directory:

1. log_file.txt: This logs the progress of the program through the three stages: 
NTF initialisation, 'burn-in' Gibbs sampler and the sampling itself. 

2. Eta_star.csv: Prediction for error transition matrix (rows true bases, columns observed probabilities) 						

3. Filtered_Tau_star.csv: Prediction for strain haplotypes. Each row of comma separated file contains:

    gene name, position, haplotype1-A,  haplotype1-C,  haplotype1-G,  haplotype1-T,..., haplotypeG-A,  haplotypeG-C,  haplotypeG-G,  haplotypeG-T  

where 1 indicates the base present in that haplotype at that position.

4. Gamma_star.csv: This gives the relative frequency of each haplotye in each sample. One row for each sample.

5. Probabilistic_Tau.csv: Prediction for strain haplotypes but now an estimate of the poseterior probability of asignments.

6. Selected_variants.csv: Variants used for strain calling if filtering applied.

7. fit.txt: Statistics evaluating fit as number of haplotypes, number of non-degenerate haplotypes inferred, log-Likelihood, Aikake information criterion

#Complete example of _de novo_ strain level analysis from metagenome data
<a name="complete_example"></a>

