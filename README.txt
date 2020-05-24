# This file is part of custom scripts used in the article "Genotyping-free Parentage assignment
# using RAD-seq reads" published by Chen et al. in Ecology and Evolution.
#
#
# These scripts are distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
#

# STEPS
Genotyping-free Parentage assignment using RAD-seq reads consists of the following steps:

1. The first round of clustering on each sample for reconstructing all possible RAD loci.
	Recommend tool: USTACKS.	Note:Record the integer id used in offspring sample.

2. Extract allele consensus sequences of RAD loci from USTACKS clustering output.
	Recommend tool: extract_seq.

3. Merge the allele sequences of the offspriing with the allele sequences of each candidate parent.
	Recommend tool: fa_merge.

4. Second round of clustering on merged sequences.
	Recommend tool: USTACKS.

5. Remove redundant information and ambiguous RAD loci in .tags.tsv file output by <USTACKS>.
	Recommend tool: tags_filter.

6. Extract comparable intersected RAD loci from a list of files output by <tags_filter>.
	Recommend tool: extract_intersection.	Note: The integer id of offspring used in here.

7. Calculate LOD score from .tsv file output by <tags_filter> or <extract_interserction>
	Recommend tool: LOD.

# EXAMPLE:
One offspring C1 with three candidate parents P1, P2, P3.
Their relationship is recorded in the tsv file ./example/candidate_table.tsv:

C1	P1	P2	P3

Their RAD_seq reads in the following files:

./example/RAD/C1.1.fa.gz
./example/RAD/P1.1.fa.gz
./example/RAD/P2.1.fa.gz
./example/RAD/P3.1.fa.gz

Example pipeline in the file ./example/test.sh:

ustacks -f ./example/RAD/C1.1.fa.gz -o ./example/out_dir/ustacks -i 1 --name C1 -m 3 -M 2 --max-locus-stacks 10
ustacks -f ./example/RAD/P1.1.fa.gz -o ./example/out_dir/ustacks -i 2 --name P1 -m 3 -M 2 --max-locus-stacks 10
ustacks -f ./example/RAD/P2.1.fa.gz -o ./example/out_dir/ustacks -i 3 --name P2 -m 3 -M 2 --max-locus-stacks 10
ustacks -f ./example/RAD/P3.1.fa.gz -o ./example/out_dir/ustacks -i 4 --name P3 -m 3 -M 2 --max-locus-stacks 10
zcat ./example/out_dir/ustacks/C1.tags.tsv.gz > ./example/out_dir/ustacks/C1.tags.tsv
zcat ./example/out_dir/ustacks/P1.tags.tsv.gz > ./example/out_dir/ustacks/P1.tags.tsv
zcat ./example/out_dir/ustacks/P2.tags.tsv.gz > ./example/out_dir/ustacks/P2.tags.tsv
zcat ./example/out_dir/ustacks/P3.tags.tsv.gz > ./example/out_dir/ustacks/P3.tags.tsv

./extract_seq -i ./example/out_dir/ustacks/C1.tags.tsv -o ./example/out_dir/primary_fa/C1.primary.fasta
./extract_seq -i ./example/out_dir/ustacks/P1.tags.tsv -o ./example/out_dir/primary_fa/P1.primary.fasta
./extract_seq -i ./example/out_dir/ustacks/P2.tags.tsv -o ./example/out_dir/primary_fa/P2.primary.fasta
./extract_seq -i ./example/out_dir/ustacks/P3.tags.tsv -o ./example/out_dir/primary_fa/P3.primary.fasta

./fa_merge -i ./example/out_dir/primary_fa/C1.primary.fasta ./example/out_dir/primary_fa/P1.primary.fasta -o ./example/out_dir/merge_fa/C1P1.fasta
./fa_merge -i ./example/out_dir/primary_fa/C1.primary.fasta ./example/out_dir/primary_fa/P2.primary.fasta -o ./example/out_dir/merge_fa/C1P2.fasta
./fa_merge -i ./example/out_dir/primary_fa/C1.primary.fasta ./example/out_dir/primary_fa/P3.primary.fasta -o ./example/out_dir/merge_fa/C1P3.fasta

ustacks -f ./example/out_dir/merge_fa/C1P1.fasta -o ./example/out_dir/fa_ustacks -i 12 --name C1P1 -m 1 -M 2 --max-locus-stacks 10 -t fasta
ustacks -f ./example/out_dir/merge_fa/C1P2.fasta -o ./example/out_dir/fa_ustacks -i 13 --name C1P2 -m 1 -M 2 --max-locus-stacks 10 -t fasta
ustacks -f ./example/out_dir/merge_fa/C1P3.fasta -o ./example/out_dir/fa_ustacks -i 14 --name C1P3 -m 1 -M 2 --max-locus-stacks 10 -t fasta

./tags_filter -i ./example/out_dir/fa_ustacks/C1P1.tags.tsv -o ./example/out_dir/fa_ustacks/C1P1.tags.tsv.clean
./tags_filter -i ./example/out_dir/fa_ustacks/C1P2.tags.tsv -o ./example/out_dir/fa_ustacks/C1P2.tags.tsv.clean
./tags_filter -i ./example/out_dir/fa_ustacks/C1P3.tags.tsv -o ./example/out_dir/fa_ustacks/C1P3.tags.tsv.clean

./extract_intersection -i 1 -l ./example/out_dir/fa_ustacks/C1P1.tags.tsv.clean ./example/out_dir/fa_ustacks/C1P2.tags.tsv.clean ./example/out_dir/fa_ustacks/C1P3.tags.tsv.clean -s .intersection

./LOD -i ./example/out_dir/fa_ustacks/C1P1.tags.tsv.clean.intersection -l 145 -er 0.0024 -he 0.001
./LOD -i ./example/out_dir/fa_ustacks/C1P2.tags.tsv.clean.intersection -l 145 -er 0.0024 -he 0.001
./LOD -i ./example/out_dir/fa_ustacks/C1P3.tags.tsv.clean.intersection -l 145 -er 0.0024 -he 0.001
