# Statistical-Machine-Translation

For the  installation , do visit statistical documentation

MOSES INSTALLATION

INITIAL SETUP
	
sudo apt update
Assumes installation in a folder called translation located in the home folder.

mkdir ~/translation; cd ~/translation

Install dependencies using apt

sudo apt install g++ git subversion automake libtool zlib1g-dev libicu-dev libboost-all-dev libbz2-dev liblzma-dev python-dev graphviz imagemagick make cmake libgoogle-perftools-dev autoconf doxygen

If package already present, apt will ignore.

Clone from git: git clone https://github.com/moses-smt/mosesdecoder.git

BOOST

Download from sourceforge: wget http://downloads.sourceforge.net/project/boost/boost/1.80.0/boost_1_80_0.tar.gz
Extract: tar zxvf boost_1_80_0.tar.gz

Go to directory: cd boost_1_80_0/

Run: ./bootstrap.sh

Compile and
echo FAILURE if unsuccessful: ./b2 -j4 --prefix=$PWD --libdir=$PWD/lib64 --layout=system link=static install || echo FAILURE
Gcc upgrade and downgrade 
If successful, there should be no failed targets else it will be boost compatibility issue. 

	If boost is not compatible run the script to download all the versions of boost and compile.
 
#!/bin/bash
for ver in $(seq $1 -1 $2); do
  echo $ver
  wget http://downloads.sourceforge.net/project/boost/boost/1.${ver}.0/boost_1_${ver}_0.tar.gz
  tar zxvf boost_1_${ver}_0.tar.gz
  cd boost_1_${ver}_0/
  ./bootstrap.sh
  ./b2 -j4 --prefix=$PWD --libdir=$PWD/lib64 --layout=system link=static install || echo FAILURE > install_log.txt
  cd ../
done

# wget http://downloads.sourceforge.net/project/boost/boost/1.60.0/boost_1_60_0.tar.gz
# tar zxvf boost_1_60_0.tar.gz
# cd boost_1_60_0/
# ./bootstrap.sh
# ./b2 -j4 --prefix=$PWD --libdir=$PWD/lib64 --layout=system link=static install || echo FAILURE


GIZA

Clone from github: git clone https://github.com/lngvietthang/giza-pp.git
Go to the directory: cd giza-pp
Compile: make

SRILM
Download and extract SRILM tar file
Create a directory called srilm (foldername is important): mkdir srilm
Extract contents of tar file to the new folder: tar -xzvf srilm-1.7.3.tar.gz -C srilm
Move this folder inside mosesdecoder: mv srilm mosesdecoder/. 
Some prerequisites
TCSH.
GNU awk (gawk), to interpret many of the utility scripts;
gzip, to read/write compressed files
bzip2, to read/write .bz2 compressed files (optional)
p7zip, to read/write .7z compressed files (optional)
xz, to read/write .xz compressed files (optional)
Make changes to the Makefile.
Add the full path of the folder where SRILM is extracted, in the SRILM field.

Compile: make NO_TCL=1 MACHINE_TYPE=i686-m64 World

Also runs properly without NO_TCL=1. That is, with TCL. This maybe better.


CMPH

Clone from git: git clone https://github.com/zvelo/cmph.git

cd cmph
Install dependency: sudo apt install libcmph-dev

Configure and compile:
./configure
make
sudo make install

CMPH is installed globally.



XML-RPC

Get package. 

Clone from git: git clone https://github.com/ensc/xmlrpc-c.git
cd xmlrpc-c/
Configure and compile
./configure
make
sudo make install

COMPILE MOSES


Create a folder called tools inside mosesdecoder, and copy some giza executables into it.
mkdir tools

cp ../giza-pp/GIZA++-v2/GIZA++ ../giza-pp/GIZA++-v2/snt2cooc.out ../giza-pp/mkcls-v2/mkcls tools
Compile Moses: ./bjam -j8 --with-boost=/home/<user>/translation/boost_1_80_0 --with-srilm=/home/<user>/translation/mosesdecoder/srilm
				OR
./bjam -a --with-boost=/home/<user>/translation/boost_1_80_0 --with-srilm=/home/<user>/translation/mosesdecoder/srilm -j8




LANGUAGE MODEL

mkdir lang_model
cd lang_model
 /home/<user>/SMT/mosesdecoder/srilm/bin/i686-m64/ngram-count -text /home/<user>/SMT/mosesdecoder/corpus/enta_clean.en -order 3 -lm output_langmodel
  /home/<user>/SMT/mosesdecoder/srilm/bin/i686-m64/ngram-count -text /home/<user>/SMT/mosesdecoder/corpus/enta_clean.ta -order 3 -lm output_langmodel
  To Binarize the model
     /home/<user>/SMT/mosesdecoder/srilm/bin/i686-m64/ngram-count -text /home/<user>/SMT/mosesdecoder/corpus/enta_clean.en  -order 3 -lm en_bin.lm -write-binary-lm
     /home/<user>/SMT/mosesdecoder/srilm/bin/i686-m64/ngram-count -text /home/<user>/SMT/mosesdecoder/corpus/enta_clean.ta  -order 3 -lm ta_bin.lm -write-binary-lm
Exit the folder

TRAINING 
Create folder to save MT models
mkdir working

cd working/

    nohup nice /home/<user>/SMT/mosesdecoder/scripts/training/train-model.perl -root-dir train -corpus /home/<user>/SMT/mosesdecoder/corpus/enta -f en -e ta -alignment grow-diag-final-and -    reordering msd-bidirectional-fe -lm 0:3:/home/<user>/SMT/mosesdecoder/model/ta_bin.lm:8 -external-bin-dir /home/<user>/SMT/mosesdecoder/tools >& training.out &
    
To monitor the process in terminal 
	tail -f training.out	 
To binarize phrase table.gz

/home/<user>/SMT/mosesdecoder/bin/processPhraseTableMin -in /home/<user>/SMT/mosesdecoder/working/train/model/phrase-table.gz -out /home/<user>/SMT/mosesdecoder/working/train/model/phrase-table -nscores 4 -threads 4 
 To copy and rename to phrase-table
 cp /home/<user>/SMT/mosesdecoder/working/train/model/phrase-table.minphr /home/<user>/SMT/mosesdecoder/working/train/model/phrase-table
To binarize reordering-table.wbe-msd-bidirectional-fe.gz
 /home/<user>/SMT/mosesdecoder/bin/processLexicalTableMin -in /home/<user>/SMT/mosesdecoder/working/train/model/reordering-table.wbe-msd-bidirectional-fe.gz -out /home/<user>/SMT/mosesdecoder/working/train/model/reordering-table -threads 4


 To copy and rename to reordering-table
 cp /home/<user>/SMT/mosesdecoder/working/train/model/reordering-table.minlexr /home/<user>/SMT/mosesdecoder/working/train/model/reordering-table

The following changes have to be made in the moses.ini file

Change KENLM to SRILM
Change PhraseDictionaryMemory to PhraseDictionaryCompact
Change phrase-table.gz to phrase-table (i.e. path to the binarized phrase table)
Change reordering-table.wbe-msd-bidirectional-fe.gz to reordering-table (i.e., path to binarized reordering model)

TESTING
/home/<user>/SMT/mosesdecoder/bin/moses -f /home/<user>/SMT/mosesdecoder/working/train/model/moses.ini < /home/<user>/SMT/mosesdecoder/working/train/test/src.txt > /home/<user>/SMT/mosesdecoder/working/train/test/output.txt


SCORING 
1)For corpus-level scoring, where test.ta is the reference file and out.ta is the output file. It is advisable to use multi-bleu for scoring.

../scripts/generic/multi-bleu.perl -lc ../corpus/test.ta < out.ta

/home/<user>/SMT/mosesdecoder/scripts/generic/multi-bleu.perl -lc /home/<user>/SMT/mosesdecoder/large_working_model/train/test/reference.txt < /home/<user>/SMT/mosesdecoder/large_working_model/train/test/output.txt
2) For sentence-level scoring, where test.ta is the reference file and out.ta is the output file.

../bin/sentence-bleu ../corpus/test.ta < out.ta > sent_bleu_scores


	/home/<user>/SMT/mosesdecoder/bin/sentence-bleu /home/<user>/SMT/mosesdecoder/large_working_model/train/test/reference.txt </home/<user>/SMT/mosesdecoder/large_working_model/train/test/output.txt > sent_bleu_scores








  
