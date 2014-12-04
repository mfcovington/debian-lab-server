# dalliance configuration

I am trying out [dalliance](http://www.bioalliance.org) as a genome browser for our server

## prerequisites

node.js and bootstrap; see main installs.md file for instructions

## Dalliance

Clone into /mnt/data/www

     cd /mnt/data/www
     git clone https://github.com/dasmoth/dalliance.git
     cd dalliance
     more README.md
     npm install -g gulp
     npm install # Install dependencies
     gulp        # Build Dalliance
     
## organization

within the dalliance folder there is `data` where I will put each genomes info and `pages` where I will put the script for each browser.

# Brapa 1.2

Setting up a browser for Brapa 1.2

## File conversions

### convert genome files to twobit
Dalliance requires sequence files to be in [2bit sequence format](http://genome.ucsc.edu/goldenpath/help/twoBit.html).

Fasta files can be converted to 2bit using UCSC [command line utilities `faToTwoBit`, `twoBitInfo`, and `twoBitToFa`](http://hgdownload.soe.ucsc.edu/admin/exe/)

Using them is simple:
	
	faToTwoBit B.rapa0830Chromosomes.fas B.rapa0830Chromosomes.2bit
	twoBitInfo B.rapa0830Chromosomes.2bit
	twoBitInfo B.rapa0830Chromosomes.2bit stdout | more
	
### Convert gff to bed
Dalliance requires that the genome description file be in bigBed format.

install gff2bed as part of [bedops](http://bedops.readthedocs.org/en/latest/content/reference/file-management/conversion/gff2bed.html)

    gff2bed < ~/Sequences/ref_genomes/B_rapa/annotation/Brapa_gene_v1.2.gff > Brapa_gene_1.2.bed

The results file needs to be converted to BED12 format, with one entry per gene and exons defined in columns 10 (number of exons) 11 (exon starts) and 12 (exon ends) (BED12 format).  I am doing this in R.

__Rscript__

	setwd("~/git/dalliance/Brapa0830/")
	
	bed <- read.delim("Brapa_gene_1.2.bed",header=F,as.is=T)
	
	head(bed)
	
	summary(bed)
	
	bed.new <- bed[bed$V8=="mRNA",] #the correct feature to use may depend on the GFF
	
	bed.cds <- bed[bed$V8=="CDS",] #often the correct feature here would be "EXON"
	bed.cds$V4 <- sub("Parent=(.*)$","\\1",bed.cds$V10)
	bed.new$V5 <- 1000
	  
	starts <- tapply(bed.cds$V2,bed.cds$V4,function(x) paste(x-min(x),collapse=","))
	sizes <- tapply(bed.cds$V3-bed.cds$V2,bed.cds$V4,function(x) paste(x,collapse=","))
		
	n.exons <- tapply(bed.cds$V2,bed.cds$V4,length)
	head(n.exons)
	
	bed.new <- cbind(bed.new[,1:6],
	                 bed.new[,2],
	                 bed.new[,2],
	                 "255,0,0",
	                 n.exons[bed.new$V4],
	                 sizes[bed.new$V4],
	                 starts[bed.new$V4]
	)
	
	head(bed.new)
	
	write.table(bed.new,"Brapa_gene_1.2_BED12.bed",col.names=F,row.names=F,sep="\t",quote=F)


### bed to bigbed

Now the bed file needs to be converted to bigBed

    twoBitInfo B.rapa_genome0830.2bit stdout | sort -k2rn > B.rapa_genome0830.chrom.sizes
    bedToBigBed -extraIndex=name Brapa_gene_1.2_BED12.bed B.rapa_genome0830.chrom.sizes Brapa_gene_1.2_BED12.bb
    

### vcf to tabix vcf

First we need to compress the vcf files with bgz (even if they were previously compressed with gz).  Then run tabix.

	brew install tabix
	gunzip R500_IMB211.2014-03-25.vcf.gz
	bgzip R500_IMB211.2014-03-25.vcf
	gunzip IMB211_Chiifu_Snps.gz
	bgzip IMB211_Chiifu_Snps
	tabix R500_IMB211.2014-03-25.vcf.gz
	mv IMB211_Chiifu_Snps.gz IMB211_Chiifu_Snps.vcf.gz
	tabix IMB211_Chiifu_Snps.vcf.gz
	
To display these in Dalliance use the following:

                    {name: 'IMB211',
                      desc: 'IMB211 vs R500 SNPs',
                      tier_type: 'tabix',
                      payload: 'vcf',
                      uri: 'http://localhost/data/dalliance/IMB211_Chiifu_Snps.vcf.gz',
                    }
    
### other bed files

These are from Upendra and were in Julin's dropbox folder

    sort -k1,1 -k2,2n sample_align_mydb_pasa.gene_structures_post_PASA_updates.21776.no.UTR.no.novel.gff3_bed3.RSEM.isoforms.filtered.dplyr.isoforms.renamed.bed > sample_align_mydb_pasa.gene_structures_post_PASA_updates.21776.no.UTR.no.novel.gff3_bed3.RSEM.isoforms.filtered.dplyr.isoforms.renamed.sorted.bed
    bedToBigBed -extraIndex=name sample_align_mydb_pasa.gene_structures_post_PASA_updates.21776.no.UTR.no.novel.gff3_bed3.RSEM.isoforms.filtered.dplyr.isoforms.renamed.sorted.bed B.rapa_genome0830.chrom.sizes PASA.bb
    
# B. rapa 1.5

## file conversion

    faToTwoBit ~/Sequences/ref_genomes/B_rapa/genome/V1.5/Brapa_sequence_v1.5.fa Brapa_genome_V1.5.2bit
    twoBitInfo Brapa_genome_V1.5.2bit stdout | sort -k2rn > B.rapa_genome_V1.5.chrom.sizes
    
    gff2bed < ~/Sequences/ref_genomes/B_rapa/annotation/Brapa_gene_v1.5.gff > Brapa_gene_v1.5.bed

Then use my bed2Bed12.R script

    bedToBigBed -extraIndex=name Brapa_gene_V1.5_BED12.bed B.rapa_genome_V1.5.chrom.sizes Brapa_gene_V1.5_BED12.bb

## for de novo transcripts

File was emailed by Upendra.  hand editted by me to convert spaces to tabs where necessary.

	bedsort velvet_trinity_tophat_comb_novel_final_transcripts.fasta.cap.contigs.singlets.transdecoder.filtered.blast.filtered.ucsc_v1.5_filtered.sorted.merged.filtered.renamed.sorted.renamed.final.final.renamed.bed velvet_trinity_tophat_comb_novel_final_transcripts.fasta.cap.contigs.singlets.transdecoder.filtered.blast.filtered.ucsc_v1.5_filtered.sorted.merged.filtered.renamed.sorted.renamed.final.final.renamed.bed
	bedToBigBed -extraIndex=name velvet_trinity_tophat_comb_novel_final_transcripts.fasta.cap.contigs.singlets.transdecoder.filtered.blast.filtered.ucsc_v1.5_filtered.sorted.merged.filtered.renamed.sorted.renamed.final.final.renamed.bed B.rapa_genome_V1.5.chrom.sizes denovo.bb

    
    

