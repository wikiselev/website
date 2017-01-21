---
date: 2012-10-20T21:28:49Z
title: "Automated snapshotting with IGV genomic browser"
---

Working on my current project I ran into a problem of taking snapshots of protein-RNA binding sites (CLIP data) of proteins from a given __protein_list__ at specific genomic locations (a given __gene_list__). To take a snapshot of any genomic region I usually use [IGV genomic browser](http://www.broadinstitute.org/igv/home) (in this post version 2.1.24 (2563) was used). When __protein_list__ (<10 data files) and __gene_list__ (<10 regions) are small snapshotting can be easily done manually. However, during the development of the project these lists can become quite big and unfeasible for manual processing. And that’s what happened with me. By accident, at the same time I found out that IGV browser is able to execute batch files of its own format. It didn’t take me long to figure out how it all works and in about half an hour I wrote a simple batch script which automated the whole process of taking my snapshots in IGV browser. I won’t go into the description how to write such a script as you can find all necessary information at the link above. However, I’ll make some useful comments about the batch file.

Let’s define the __gene_list__ as a simple text file with the following structure:

```
                              # Examples of the values:
gene1                         # PTGS2
gene1_full_name               # Prostaglandin-endoperoxide synthase 2 (COX-2)
gene1_url                     # http://www.ncbi.nlm.nih.gov/gene/5743
gene1_location                # chr1:186,640,920-186,641,060

gene2
gene2_full_name
gene2_url
gene2_location

# all other genomic locations
```

And shall the __protein_list__ be just a folder (/Users/Vladimir/IGV/protein_list/) with CLIP data files, let’s say of .BED format (can be other formats, e.g. WIG):

protein1.BED
protein2.BED
…

Having defined __gene_list__ and __protein_list__ as shown above my IGV batch file will look like:

```
new

load /Users/Vladimir/IGV/protein_list/protein1.BED,/Users/Vladimir/IGV/protein_list/protein2.BED # all other protein data files

snapshotDirectory /Users/Vladimir/IGV/snapshots/

goto gene1_location
snapshot gene1.png

goto gene2_location
snapshot gene2.png

# all other genomic locations
```

Now my comments about the batch file and protein data files:
1. If your BED (or other format) data files are large it’s better to use [IGV tools](http://www.broadinstitute.org/igv/igvtools) to process them into a more convenient format to improve the performance of IGV browser (.tdf).
2. IGV browser is very sensitive to a file path definition – I always had to define the full path to a file or a folder to make it work (e.g. /Users/Vladimir/IGV/protein_list/protein1.BED or /Users/Vladimir/IGV/snapshots/ instead of just protein_list/protein1.BED or snapshots/). Also please separate files in the _load_ line strictly by a comma and don’t add any space before and after it! Otherwise, the script won’t work!

To run the batch file you need to go to the File menu tab of the IGV browser and click on _Run Batch Script..._. Then choose your batch file (in my case ‘IGV_script.txt’) in the windows dialog and wait until the browser takes all the snapshots for you automatically and then voila! All my snapshots are now in /Users/Vladimir/IGV/snapshots/ folder. If I run my example script ‘IGV_script.txt’ I will get 2 snapshots.

But, unfortunately, that was not the end of the problem. My ultimate __gene_list__ contained more than 50 genomic regions providing me with more than 50 snapshots (I imagine, you could have more than that). It was not easy at all to view, share and compare those snapshots. I felt that I needed another automated script that would integrate all my snapshots into one file with a basic annotation to make my life easy. I tried several options but ended up with the old and reliable [LaTex](http://en.wikipedia.org/wiki/LaTeX). Here is an example of a .tex script which can be used to combine into one file snapshots generated by my example batch file ‘IGV_script.txt’:

```tex
\documentclass[landscape,letterpaper]{article}
\renewcommand{\rmdefault}{phv}      % Arial
\setlength{\oddsidemargin}{-0.9in}    
\setlength{\textwidth}{10.7in}         
\setlength{\textheight}{7in}         
\setlength{\topmargin}{-1.0in}         
\setlength{\headsep}{0.25in}         
\setlength{\parskip}{1.2ex}
\setlength{\parindent}{0mm}
\usepackage{graphicx}
\usepackage{hyperref}
\hypersetup{colorlinks=true}
\usepackage[dvipsnames]{xcolor}

\begin{document}
{\bf gene1_full_name}
\url{gene1_url}
gene1_location
Color abbreviations: {\bf {\color{green} A}, {\color{blue} C}, {\color{red} T}, {\color{Orange} G}}
\begin{figure}[!ht]
\begin{center}
    \includegraphics[scale = X]{snapshots/gene1.png}     % change X to a value between 0 and 1 to fit a snapshot to the file page size
\end{center}
\end{figure}
\cleardoublepage
{\bf gene2_full_name}
\url{gene2_url}
gene2_location
Color abbreviations: {\bf {\color{green} A}, {\color{blue} C}, {\color{red} T}, {\color{Orange} G}}
\begin{figure}[!ht]
\begin{center}
    \includegraphics[scale = X]{snapshots/gene2.png}     % change X to a value between 0 and 1 to fit a snapshot to the file page size
\end{center}
\end{figure}
% all other snapshots
\end{document}
```

The outcome of this script is a .pdf document on each page of which is a single IGV snapshot with its basic description.

So, now when you have all your snapshots combined in one file it’s much easier to deal with your results!

Hope this post was helpful. Please let me know if you’ve had any problems with it.

Note: Everywhere throughout this post I used IGV genomic browser of version 2.1.24 (2563)

P.S. Of course one could automate the process even more, so that the only initial income parameters required are __gene_list__ and __protein_list__. Then it is possible to create (e.g. using a python script) and execute (e.g. using a shell script) all necessary scripts automatically. I’ll probably do this later on and update you about it.

__UPD__. If your __gene_list__ is an automatically generated BED file then you can use a utility, called [bedToIgv](https://groups.google.com/forum/#!msg/bedtools-discuss/lkrMv5J_LJI/Q38uzhI6l0MJ), provided by BEDtools to generate an IGV batch file. Note that you should install the newest version of BEDtools to be able to use this utility (v. 2.17.0 on 13/11/12). If you __gene_list__ is not an automatically generated BED file then the utility won’t be very useful for you.