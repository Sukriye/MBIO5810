### ASSESSING THE FREQUNCY OF A FUNCTIONAL GENE IN A METAGENOME

- Download the docker bwawrik/bioinformatics:latest

```sh
docker pull bwawrik/biorinformatics:latest
```

- Make a data directory. If you are working locally on a mac using boot2docker you will need to work in ~/data. Please replace where appropriate

```sh
mkdir /data
(* on  your mac: mkdir ~/data)
```

- Start the docker and mount /data

```sh
docker run -t -i -v /data:/data bwawrik/bioinformatics:latest
(* locally on mac: docker run -t -i -v ~/data:/data bwawrik/bioinformatics:latest)
```

- Install usearch8.0.1517 (note: the following will not work with older usearch versions)

```sh
mkdir -p /opt/local/software/usearch
cd /opt/local/software/usearch
wget http://mgmic.oscer.ou.edu/sequence_data/tutorials/usearch8.0.1517_i86linux32 
chmod 777 *
cd /usr/local/bin
ln -s /opt/local/software/usearch/usearch8.0.1517_i86linux32 ./usearch8
```

- Download the dsrA fasta file and make a usearchable database

```sh
wget https://github.com/bwawrik/MBIO5810/raw/master/sequence_data/dsrA_database.fas.gz
gunzip dsrA_database.fas.gz
usearch8 -makeudb_usearch dsrA_database.fas -output dsrA_database.udb
```

- Get your data files

```sh
wget https://github.com/bwawrik/MBIO5810/raw/master/sequence_data/TORDIS3_forward_paired.50k.fq.gz
wget https://github.com/bwawrik/MBIO5810/raw/master/sequence_data/TORDIS3_reverse_paired.50k.fq.gz
```

- Convert fastq files to fasta

```sh
read_fastq -i TORDIS3_reverse_paired.50k.fq | write_fasta -o TORDIS3_reverse_paired.50K.fasta -x
read_fastq -i TORDIS3_forward_paired.50k.fq | write_fasta -o TORDIS3_forward_paired.50K.fasta -x
```

- Search database with data files and

```sh
usearch8 -usearch_global TORDIS3_reverse_paired.50k.fasta -db dsrA_database.udb -id 0.7 -strand both -mincols 50 -maxhits 1 -qsegout T3_dsrAhits_r.fas -blast6out T3dsrAhits_r.tab

usearch8 -usearch_global TORDIS3_forward_paired.50k.fasta -db dsrA_database.udb -id 0.7 -strand both -mincols 50 -maxhits 1 -qsegout T3_dsrAhits_f.fas -blast6out T3dsrAhits_f.tab
```

- Prep your output for further analysis

```sh
sed -i 's/>/>F_/g' T3_dsrAhits_f.fas
sed -i 's/>/>R_/g' T3_dsrAhits_r.fas
cat T3_dsrAhits_r.fas T3_dsrAhits_r.fas > T3dsrAhits_FR.fas
cat T3dsrAhits_f.tab T3dsrAhits_r.tab >T3dsrAhits_FR.tab
```

- Determine how many hits you found

```sh
fgrep -o ">" T3dsrAhits_FR.fas | wc -l
```

- Lets make a log scale bar graph of the percent frequency of hits

```sh
wget https://github.com/bwawrik/MBIO5810/raw/master/R_scripts/bargraph_redgreen_scale.r

Rscript bargraph_redgreen_scale.r $(calc 100*$(fgrep -o ">" T3dsrAhits_FR.fas | wc -l)/$(fgrep -o "+" TORDIS3_reverse_paired.50k.fq | wc -l)*2) out.png
```

The 'calc' function is coded into my .bashrc in the docker. It is not part of linux and looks like this:
```sh
calc () {
    bc -l <<< "$@"
}
```

#### Are drsA genes abundant in this sample ? 
