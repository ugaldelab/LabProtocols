#EGGnog annotation of a genome

This is the procedure used to annotate a genome using the EggNOG database.

##Download and prepapre EggNOG database

The complete hmm models were obtained from the eggnog website (eggnogdb.embl.de), from this link: http://eggnogdb.embl.de/download/eggnog_4.1/data/NOG/NOG.hmm.tar.gz

  wget http://eggnogdb.embl.de/download/eggnog_4.1/data/NOG/NOG.hmm.tar.gz
  tar -xvf NOG.hmm.tar.gz

To build the database for searches, we have to concatenate all the models into a single file. In our machine this was a problem, due to the large number of files (a limitation on the version of the linux kernel that we have), so this was the used command:

  echo NOG_hmm/* | xargs cat > AllEggNOG.hmm

To make the database:

  hmmpress AllEggNOG.hmm

To run the comparison of a multi-fasta file with aminoacids, this can be done using the following command,
in this case using the Lkun protein file:

  hmmscan --cpu 15 --tblout test.txt -E 0.00001 -o run.out ~/research/DBs/EggNOG/AllEggNOG.hmm ../Lkun_03042015.faa

Here, we are running hmmscan with 15 cpus, creating a tabular output to test.txt, using an Evalue cutoff of 1e-5
