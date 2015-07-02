#Installing Kraken on EC2

##Prepare the EC2 computer
Given the large memory requirements of the database construction of Kraken, I used the r3.8xlarge instance (32 core, 244 GB ram).

A spot instance was requested, with a EBS mounted disk of 600Gb. A little exagerated, but just to be sure :)

Once the request for instance was fulfilled, I logged in to the instance, and mounted the EBS disk using this procedure (http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-using-volumes.html). In my case, the device was located in /dev/xvdb, so:

	sudo mkfs -t ext4 /dev/xvdb
	sudo mkdir kraken
	sudo mount /dev/xvdb kraken
	sudo chgrp -R ubuntu kraken/
	sudo chown -R ubuntu kraken/

#####Install build tools

	sudo apt-get update
	sudo apt-get install g++ gcc make jellyfish

##Download Kraken and reference databases
Part of this procedure is based in this post (https://groups.google.com/forum/#!msg/kraken-users/wL-8GvGeLls/e-N3PJxYqbMJ) in the Kraken users group.

#####Download kraken

	wget http://ccb.jhu.edu/software/kraken/dl/kraken-0.10.5-beta.tgz
	tar -xvf kraken-0.10.5-beta.tgz


#####Install kraken

	cd kraken-0.10.5-beta/
	./install_kraken.sh /home/ubuntu/kraken

#####Download databases
Our database name will be `full_062715`

First we get the taxonomy

	./kraken-build --download-taxonomy --db full_062715

Then we download the databases to include:

	./kraken-build --download-library bacteria -db full_062715
	./kraken-build --download-library viruses -db full_062715
	./kraken-build --download-library plasmids -db full_062715

Some genomes were added manually, and can be found in the `genomes_add` folder. These were added to the database using the command:

	./kraken-build --add-to-library genomes_add/GENOMENAME --db full_062715

TODO: Script to download genomes from NCBI with the appropiate fasta header for kraken

##### Building the database

	./kraken-build --build -db full_062715 --threads 32

Process took around 1 hour 30 minutes

##Running kraken

Following the manual, I created a ramdisk to load the database

	sudo mkdir /ramdisk
	sudo mount -t ramfs none /ramdisk
	sudo chgrp -R ubuntu /ramdisk
	sudo chown -R ubuntu /ramdisk
	cp -a full_062715 /ramdisk

This took around 10 minutes

To run Kraken for a test dataset:

	../../kraken --db /ramdisk/full_062715 --threads 32 --output Apis_Kraken --fasta-input ame_ref_Amel_4.5_chrUn.fa
	../../kraken-translate --db /ramdisk/full_062715 Apis_Kraken > Apis_Krkaen.labels

The first command runs the kraken classifier, while the second one generates the labels with the lineage of each sequence.

Finally, we can generate a report with the `kraken-report` command:

	../../kraken-report --db /ramdisk/full_062715 Apis_Kraken > Apis_Kraken.report

######Running paired reads

For paired reads in fast format:

	../../kraken --db /ramdisk/full_062715 --threads 32 -fastq-input --paired --output kraken_output --unclassified-out unclassified_reads b0200_1.fastq b0200_2.fastq

This is running the anaysis on a fastq input, paired reads, and saving the output of the unclassified reads
