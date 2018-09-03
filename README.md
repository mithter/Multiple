Multiple is a program that searches for Transcription factor binding sites.
It combines the sequence data, the position data and the expression data for
finding the statistically significant over represented de novo sequence motifs.
It requires as input the sequence data (DNA promoter regions around the Transcription Start Sites (TSSs))
and the expression data (the expression profiles across the conditions).
The user have the choice to input the expression data either in the its 
original form or by first summarizing it into covariates. Each covariate 
may correspond to the coordinate of the gene on an axis (e.g. obtained by PCA or ICA) 
or to its position in a tree (e.g. obtained by hierarchical clustering).



##########################
#Command line and options#
##########################


Usage : multiple [-seq <file>] [-xvectors <file>] [-xtrees <file>]
        [-prefix <string>] [-nmotif <#(default 2)>] [-wmin <#(default 3)>] 
        [-wmax <#(default 25)>] [-breakpmax <#(default 20)>]
        [-Nsweep <#(default 10000)>] [-rngseed <#(default 2)>] 
        [-ICbg <bool(default false)>] [-widthbg <#(default wmin+3)> <double(default 1)>] 
        [-validate <bool(default false)>] [-TSS <bool(default true)>] [-geo_prior_breakpoints <double(default 0.25)>] 
        [-z <double(default 1)> <double(default 1)> <double(default 1)>] 
        [-probit <double(default 1)> <double(default 0.2)> <double(default -2.3)>]
        [-palprior <double(default 0)> <double(default 0.5)> <#(default 1)> <double(default 0.5)>]


Description of the command line arguments
-------------------------------------------

-seq <file> : the name of the sequence file, formatted as seen below.

Optional
--------

-xvectors <file> : the name of the expression file that contains the PCA/ICA covariates.

-xtree <file> : the name of the expression file that contain the clustering tree, see format below.

-prefix <string> : the prefix to be added before every file name, the idea of the prefix is that in case 
that the user want to run many jobs with different seeds or with different set of parameters it will not be 
necessary to change the directory but just change the prefix while starting the job.

-nmotif <#(default 2)> :  the number of motif that the user is expecting/wanting to find in the input data.

-wmin <#(default 3)> : the minimal width that the motif will not be allowed to be reduce after while searching.

-wmax <#(default 25)> : the maximum width of the motifs to be discovered.

-breakpmax <#(default 20)> : we divide the promoter regions into subregions with different probability of finding the
algorithm, this parameter will set an upper bound for the sequence division.

-Nsweep <#(default 10000)> : the number of MCMC iteration you want the algorithm to run for.

-rngseed <#(default 2)> : this parameter gives the freedom to change the seed which may serve in case of
performing stability analysis.

-ICbg <bool(default false)> : In the case of overlap between two or more motifs, there is a parameter
that assign a membership of this nucleotide to one of the intersecting motifs. By adjusting this parameter
the user indicates whether the algorithm will use the information content of the intersecting motif for this
assignment, or just assume a uniform distribution that it could have been generated by any of the motifs.

-widthbg <#(default wmin+3)> <double(default 1)> : the prior for the width of the motif, the motif 
will have piece-wise distribution the first part of the motif will have a uniform distribution and the 
second part will have a geometric distribution. The user will enter two values, the first will corresponds
the width of the first part with uniform distribution and the second value will be the parameter of the geometric 
distribution. For example, if the user entered 8 and 0.9 this will mean the motif will have a uniform probability
to have a width between wmin and 8, and will be less likely to extend after 8 and this likelihood for extension will
be controlled by the 0.9 value.

-validate <bool(default false)>] : this parameter may not be useful for the user, it was used in the building of the
code to validate that it is working as supposed to.

-TSS <bool(default true)> : this parameter if deactivated, the algorithm will not divide the sequence into
sub-sequences with different probability of motif occurrence, but rather will treat the whole sequence as one
piece with uniform probability of seeing a motif.

-geo_prior_breakpoints <double(default 0.25)> : the parameter of the geometric distribution to be put over
the number of break points that divide the promoter region between 0 (the sequence as one piece) and breakpmax.
You can adjust this value to 1 to have a uniform distribution.

-z <double(default 1)> <double(default 1)> <double(default 1)> : This is to adjust the pseudo count of the of three
Dirichlet distribution used in the algorithm. The first is for the motif nucleotides' count, the second is for the background
nucletides' count, and the third is for motif counts in the subregions.

-probit <double(default 1)> <double(default 0.2)> <double(default -2.3)> : adjusting the values for the regression
model (propit) used to summarize the expression values into probabilities of occurrence. The first is for the probit model
variance, the second and the third are for for the variance and the mean values of the intercept respectively. 
The mean of the intercept may thought of as the prior value on the number of motifs without looking at the expression data, 
while the variance of the intercept may thought of as how much we are expecting this value to vary from the initial value. 
We can think of the variance of the probit model as how much we want the shape of the model to change.

-palprior <double(default 0)> <double(default 0.5)> <#(default 1)> <double(default 0.5)> : In the case that the user is
interested specifically in the palindromic motif, where the four parameters are respectively: 
probability for a motif to be "palindrome", probability for a pair of columns in a palindrome to be coupled, 
minimum length for a half palindrome, a parameter that account for the centering of the palindrome 
(1 means no centering, all positions are equally likely for the center).


The sequence file should be formatted like

1512 121
>lmo0001
TTAACTGGCTGTGGACAACCGTTTTTCACATCTGGACAGTTTTGTGGATAGATTTGGTAAGTCCTTGCTATCAGAGTGATTTTCTGATATTATAATTCCTGTCGAATAGAAATATAGCTGG
>lmo0002
GCAGCATGGCTTGTAACCTACTTATCCACAAATCCACAGCGCCTATTACTATTACTACGATTTTTTATTAATTAATTAAAGGTTTTTGAGGTTTTACAAAATATAAAAAAATGGGGGATAC
...

where 1512 specifies the number of sequences to be analyzed and 121
indicates the nucleotide length of each sequence. The subsequent lines
(here 3242) provide the sequence information: they are composed of a
sequence identifier separated from the sequence itself by a tabulation
character.

Notice: all the sequences need to have the same length and that
missing data is not allowed.


The topology and branch length information about the tree should
provided in a (three * number of trees) columns format as shown below

1
-20     -1231   0.0025
...
-982    6       0.0038
...
353     434     0.0155
...
1509	  1510	  0.5587

The first row has only the number of the inputted trees. The rows afterwards corresponds
to nodes, such as, the second row corresponds to the first node and so on.
The first two columns indicate the two children of this node and the last column
gives the height of the node. The n leaves (here 1512) are numbered
negatively from -1 to -n, the n-1 internal nodes are numbered
positively from 1 to n-1. The nodes need to be ordered by increasing
height such that all the descendants of a node are found above the
line that describes the node.

Notice: this input format supposes an ultra-metric binary tree as all
the leaves are implicitly assumed to have height 0. These requirements
are naturally fulfilled if the tree was obtained by hierarchical
clustering of the matrix of pair-wise correlation coefficients using
the average linkage algorithm. Modifying the program to allow other
types of tree should not be too difficult.


#############
#Output files#
##############

The standard output provide you the following useful information

* The user-specified parameters that were used.


The parameters file 
-------------------------------
The parameter file is edited every ten sweeps by adding the new values for
these parameters respectively:

sweep: the number of the sweep in which the file is being edited.

motif: the index of the motif whose values are written in the row.

nb_regions: how many subregions correspond to the given motif.

phi: in the case of not inputting expression data phi will indicate the
probability of seeing the motif in the input data.

refpos: the reference position, since the position of the motif inside
the sequence does not refer to the first position but rather to a reference
position and by turn the reference position is need in order to align the motif occurrence.
This kind of referring to the motif gives the freedom to the motif to go outside the sequence.

width: the width of the motif at the given sweep.

bg_order: the order Markov model describing the background

track_left: this value is increased or decreased by 1 according to the change
happening to the most left nucleotide of the motif, this value combined with the
width allows us to imagine how the motif moved over the sequence across the iterations
which is good if we want to align the PWMs and also to access the convergence of the motif.

prob_x: This value is basically the multiplication of the probabilities of the nucleotides
in the data. If we initialize all the PWM for all the motifs and the parameters for the background to 0.25 
which means equal probabilities to see the different nucleotides (A,C,G,T) inside the motifs as well as
in the background. The prob_x value will equal to (log(0.25)* number of sequences * number of nucleotides in the sequence)
This value should improve across the iterations since the estimation of the PWMs is improving and by turn the information
content is improving. This value will give us an indication on how fast this improvement is happening across the iterations.

T_probit: The next K columns in the parameter files are titles T_probit corresponding to K-1 covariate and the intercept 
of the regression mode. The cells of the T_probit columns take values between 0 and 3, where 0 means that the corresponding 
covariate is inactive, 1 is given if the corresponding covariate represent a PCA/ICA axis and the inputs in this axis
should be treat as they are without binarizing them to only 2 values (having a motif or not) instead of degrees of believe.
2 As well is for the PCA/ICA axis but in case that the corresponding component will be binarized. 3 is given if the corresponding component is representing the height of the gene on a tree.

beta: After the K T_probit column, the file contains K beta columns where the cells of these columns contain the value that
the covariate is influencing the regression model final value (the value that tells if the gene has the motif or not).


The PWM file 
-------------------------------
The PWM file is edited every ten sweeps by adding the new values for the position weight Matrices 
describing the motifs and some other parameters related to the motif shape,
these parameters are respectively:

sweep: the number of the sweep in which the file is being edited.

motif: the index of the motif whose values are written in the row.

width: the width of the motif in concern.

refpos: the reference position (explained above).

centerpal: tells which nucleotide within the motif represents the center of the palindrome (if there is any).

npairedcolpal: the number of paired columns for the current motif in the current iteration (even number).

prob: after that, there are (maxw * 4) columns describing the probability of seeing (A,C,G,T) respectively. For example, the first four columns are the probability to see (A,C,G,T) in the first position and so on.


The position pdf file 
-------------------------------
The position pdf file is edited every 100 sweeps and it contains a column for every nucleotide in the sequence, this column represents the probability of seeing the motif (the reference position) at this nucleotide. There are four other columns in the file, these columns represent respectively:

sweep: the number of the sweep in which the file is being edited.

motif: the index of the motif whose values are written in the row.

phi: explained above.

nb_regions: represent the amount of sub regions that the sequence has been divided into, each sub region has a different probability of seeing the motif.

pos_prob: after that, there are L columns corresponding to L nucleotides forming the sequence. The values in the cells of these columns are the probability of seeing the motif (the reference position) at this nucleotide.



The motif sequence file 
-------------------------------
The motif sequence file is edited every 100 sweeps and it contains the positions of the motif occurrences in all the sequences. There are another two columns describing the sweep and the motif as before. The columns in the file are ordered in the following manner:

sweep: the number of the sweep in which the file is being edited.

motif: the index of the motif whose values are written in the row.

position: After that, there are N columns called position for the N sequences in the input data. The values in these columns are the the position of the motif in concern in the corresponding sequence. When the motif is absent, the value (-1) is given.

