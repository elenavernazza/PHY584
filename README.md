# The High Granularity Calorimeter (HGCAL) Beam Test Analysis
## Mini-Stage for the PHY584 course 

We are going to use this repository as starting point for the PHY584 course.
In this mini-stage you are going to work on beam test data for the High Granularity
Calorimeter for the CMS experiment upgrade.

Content of this README:

* [The High Granularity Calorimeter (HGCAL)](#S-introduction)
	* [Material on HGCAL beam tests](#S-material)
* [Setting up the working environment ](#S-environment)
* [Learning the data analysis tools](#S-analysis-tools)
* [First look into the data](#S-first-look)
* [Description of the data content](#S-data-content)
* [References](#S-references)

To have a better understanding of calorimetry, I highly suggest [this set of slides as an entry point](https://github.com/elenavernazza/PHY584/blob/master/slides/L10_Calorimetry.pdf).

**Remember:**  Always feel free to ask questions, either for curiosity or for a better understanding of what you are doing!

> *There are naive questions, tedious questions, ill-phrased questions, questions put after inadequate self-criticism. But every question is a cry to understand the world. **There is no such thing as a dumb question.***

### <a name="S-introduction"></a>The High Granularity Calorimeter (HGCAL)

The Large Hadron Collider (LHC) is intended to accumulate 300 fb^{-1} by the end of Run 3 (2023). After this data acquisition phase the third long shutdown (LS3) is planned to allow the upgrade operations necessary for the High Luminosity phase (HL-LHC). The HL-LHC is expected to reach an integrated luminosity of some 3000 fb^{-1} by the end of mid-2030s. In addition to this, the HL-LHC is expected to have a larger pileup rate up to 140-200 events per bunch crossing. The detectors at the current stage will not manage to cope with such a harsh environment and are undergoing an upgrade program.

As part of this upgrade program, the CMS Collaboration is proposing to build a high granularity calorimeter (HGCAL) to replace the current endcap calorimeters, which were designed to cope with an integrated luminosity of 500 fb^{-1}.

The HGCAL will be a sampling calorimeter with a silicon-based electromagnetic compartment and a mixture of silicon-based and scintillator tiles for the hadronic compartment. It will be the first large scale calorimeter where silicon is used as active material, hence beam tests are fundamental as a proof of concept of the HGCAL design.

In October 2018 the first large scale prototype of HGCAL was conducted at CERN, using electrons and pions over a wide energy range (20-300 GeV) to fully asssess the detector's performance. In this mini-stage you are going to use these data to learn how electromagnetic showers appear in such a calorimeter, to understand the main differences between electrons and pions showers and to make comparisons with theoretical models and Monte Carlo simulations.

More information about HGCAL can be found in the Technical Design Report (TDR) [[1](#R-tdr)]. For this mini-stage it might be useful to have a look to the "Introduction and overview", the "Reconstruction and detector performance" chapters.

#### <a name="S-material"></a>Material on HGCAL beam tests

In this section you can find some useful material to start getting familiar with HGCAL and the structure of the data we are going to analyze in this mini-stage.

* Set of slides by Artur: introduction to HGCAL

	https://github.com/elenavernazza/PHY584/blob/master/slides/ALobanov_HGCAL_ICHEP2018.pdf

* Set of slides by Artur: HGCAL beam test summary:

	https://github.com/elenavernazza/PHY584/blob/master/slides/HGC_TBOct2018_Summary.pdf


* Set of slides by Thorben, describing the structure of the TTrees we are going to use in this analysis:

	https://github.com/elenavernazza/PHY584/blob/master/slides/Reco_ntuples_06Nov2018.pdf


* Set of slides on 2018 beam tests results:

	https://github.com/elenavernazza/PHY584/blob/master/slides/171009_HGCAL_Bonanomi_IPRD19.pdf
	
Do not hesitate to ask me any question or for additional material.

### <a name="S-environment"></a>Setting up the working environment [^a]

You should first set up your laptop in a way that you can connect to the LLR servers and use the Jupyter notebook from the outside. For this, you need to create a SSH key and upload it to any LLR server.
If you are using an operating system of the Microsoft Windows family, you can install Ubuntu via the Linux Subsystem for Windows and start a terminal emulator this way.

Once you are in your terminal, create the key with the following program (just hit enter all the time to accept the defaults):

	ssh-keygen

Next, you copy the key over to an LLR server:

	ssh-copy-id -i ~/.ssh/mykey appro2@polui04.in2p3.fr

This one time you'll need a password, just ask me about it. Now it's time to configure your ssh client to connect to the LLR severs via the correct proxy server from the outside. Just download the following config file to set it up and move it to the right directory:

	mv ~/.ssh/config ~/.ssh/config.bak
	wget -O ~/.ssh/config http://evernazz.web.cern.ch/evernazz/PHY584/ssh.config

You should now be able to conect to any of the LLR interactive servers as follows, even from the outside:

	ssh polui06

If this particular machine is not available, try any of `polui01`, `polui03`, `polui04`, `polui06` or `polui07`.


#### What you need to do every time you want to work with the notebook
To analyze the test beam data, we use the [Jupyter Notebook](https://jupyter.org/).

From within LLR, which includes the **LLR-WIFI**, it is enough to connect to an interactive machine, preferentially `polui04`:

	ssh polui04

Start the notebook with:
	
	notebook

You should now see a link that you can paste in your browser to access the Jupyter notebook.
From outside LLR, one first has to make a SSH tunnel to the LLR network:
	
	ssh -N -L 8080:localhost:8080 polui04

You can now start the notebook from another terminal while keeping the proxy open.

	jupyter notebook --no-browser --port=8080

You should now see two links, pick the one of the form `http://127.0.0.1:8080/?token=...` and paste in your browser to access the Jupyter notebook.

[^a]: From [Jonas' ministage repository](https://llrgit.in2p3.fr/rembser/hgc-testbeam-mini-stage#how-to-get-into-the-analysis-environment).

### <a name="S-tools"></a>Analysis tools
In this ministage we are going to use the `python` environment for our analysis. Have a look at the [tutorials](https://github.com/elenavernazza/PHY584/tree/master/tutorials) folder to find some notebooks with quick examples on [`numpy`](https://numpy.org/) and [`matplotlib`](https://matplotlib.org/). The data we are going to use come in `.root` format, a very common one in HEP as it was born together with the ROOT framework, one of the most used frameworks for data analysis in high energy physics. Here we are going to use [`uproot`](https://github.com/scikit-hep/uproot), a reader and a writer of the ROOT file format using only python and numpy, to open our ROOT files as [`pandas` data frames](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html). 

The entry point for the work we are going to do is the `tutorials` folder. The best strategy for you to work is to clone this repository, add a new folder (e.g. `work`) and develop your notebooks starting from the questions and the exercises you have under the `tutorials` folder. 

Do not forget your best friend when working with the code: the internet! The internet is an extremely powerful source for python-related questions. Check references such as [`stack overflow`](https://stackoverflow.com/), [Jake VanderPlas website](http://vanderplas.com/)
and his [open-source book](https://jakevdp.github.io/PythonDataScienceHandbook/). 

Have a look at this article: ["Everything you wanted to know about Data Analysis and Fittingbut were afraid to ask"](https://arxiv.org/pdf/1210.3781.pdf).

### <a name="S-references"></a>References

[1]<a name="R-tdr"></a> The Phase-2 Upgrade of the CMS Endcap Calorimeter: <https://cds.cern.ch/record/2293646>
