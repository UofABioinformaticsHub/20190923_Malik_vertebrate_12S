# Setup Entrez Direct (EDirect)

```bash
cd ~
CWD=$(pwd)
perl -MNet::FTP -e \
  '$ftp = new Net::FTP("ftp.ncbi.nlm.nih.gov", Passive => 1);
   $ftp->login; $ftp->binary;
   $ftp->get("/entrez/entrezdirect/edirect.tar.gz");'
gunzip -c edirect.tar.gz | tar xf -
rm edirect.tar.gz
export PATH=${PATH}:$CWD/edirect >& /dev/null || setenv PATH "${PATH}:$CWD/edirect"
./edirect/setup.sh

# Setup path permamantly
echo "export PATH=\${PATH}:$CWD/edirect" >> $HOME/.bashrc
```

# Search for all Vertebrate 12S Sequences

```bash
PROJECT_WORKING_DIRECTORY='Documents/research/support/Arif_Malik/'

mkdir -p "${PROJECT_WORKING_DIRECTORY}"
cd "${PROJECT_WORKING_DIRECTORY}"

# Taxonomy
time esearch -db nuccore -query 'vertebrates[porgn] AND 12S ribosomal rna[Title] OR 12S rrna[Title] NOT partial [Title]' \
  | efilter -location mitochondrion \
  | efetch -format xml \
  | xtract -pattern Seq-entry -element Textseq-id_accession -element Textseq-id_version -element OrgName_lineage \
  | sed 's/\t/./' \
  | awk '$1~/\./' \
  > 12S_vertebrates.taxonomy

# Sequences
time esearch -db nuccore -query 'vertebrates[porgn] AND 12S ribosomal rna[Title] OR 12S rrna[Title] NOT partial [Title]' \
  | efilter -location mitochondrion \
  | efetch -format fasta \
  | bgzip \
  > 12S_vertebrates_all.fasta.gz
```

# Extract Sequence Subset Based on Taxonomy

Ensure we get the subset of sequences for which we have taxonomic information:

```bash
cut -f1 12S_vertebrates.taxonomy \
  | xargs samtools faidx 12S_vertebrates_all.fasta.gz \
  | bgzip \
  > 12S_vertebrates.fasta.gz
```

# Resources

 * https://www.ncbi.nlm.nih.gov/books/NBK179288/
 * http://home.cc.umanitoba.ca/~psgendb/birchhomedir/doc/NCBI/edirect/chapter6.pdf
 * https://ncbi-hackathons.github.io/EDirectCookbook/
 * http://bioinformatics.cvr.ac.uk/blog/ncbi-entrez-direct-unix-e-utilities/

