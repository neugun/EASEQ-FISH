## 3D-SeqFISH analysis toolbox

See [here](https://github.com/JaneliaSciComp/multifish) for active EASI-FISH pipeline updates. 

## Table of Contents #
   * [Description](#description)
   * [Installation](#Installation)
   * [Modules](#modules)
      * [Preprocessing with EASIFISH Pipeline](#EASIFISH Pipeline)
      * [Warp Mask and Spot assignment](#Warp Mask and Spot assignment)
      * [Classic Point cloud registration](#Classic Point cloud registration)
      * [Deep-learning based Point cloud registration](#Deep-learning based Point cloud registration)
      * [Barcoding and decoding analysis](#Barcoding and decoding analysis)
   * [Pipeline](#pipeline)
      * [Example data](#Example data)
      * [Jupyter notebooks](#Jupyter notebooks)
      * [Bash command](#Bash command)
   * [Additional information](#additional-information)
      * [Visualization](#visualization)
      * [Post processing](#post-processing)  
      * [Reference](#Reference)

## Description #

This 3D-SeqFISH analysis toolbox is a integrated 3d_SeqFISH analysis method, aiming to provide the general solution for 3D registration and decoding analysis for 3D spatial transcriptome. This toolbox handles large-scale, multi-round, high-resolution image data acquired using EASI-FISH (Expansion-Assisted Iterative Fluorescence *In Situ* Hybridization). It takes advantage of the [n5](https://github.com/saalfeldlab/n5) filesystem to allow for rapid and parallel data reading and writing. It performs automated image stitching, distributed and highly accurate multi-round image registration, 3D cell segmentation, and distributed spot detection. We also envision this workflow being adapted for analysis of other image-based spatial transcriptomic data. 
![](/Diagrams/3DseqFISH_diagram_v1.png). 

## Installation #
It is largely based on EASIFISH, MULTIFISH, bigstream, napari, Point cloud registration methods, and SeqFISH, etc. 
`3D-SeqFISH` can be installed with `pip`:
```
   pip install 3D-SeqFISH
```
`bigstream`  can be installed with  `pip`:
```
   pip install bigstream
```
`napari`  can be installed with  `pip`:
```
   pip install napari
```
`Starfinity`  can be installed with  `pip`:
```
   pip install git+https://github.com/mpicbg-csbd/stardist@refinement
```

## Modules #

### Preprocessing with EASIFISH Pipeline #

For imaging large volumes, multiple sub-volumes (tiles) are sequentially acquired, followed by computational stitching into a single large image. We used an Apache Spark-based high-performance stitching pipeline ([Gao et al., 2019](https://science.sciencemag.org/content/363/6424/eaau8302.long)). The pipeline automatically performs a flat-field correction for each tile to account for intensity variations across the lightsheet. It then derives the globally optimal translation for each tile that minimizes the sum of square distances to competing optimal pairwise translations estimated by phase-correlation ([Preibisch et al., 2009](https://academic.oup.com/bioinformatics/article/25/11/1463/332497)).The stitching module can be executed with the EASI-FISH pipeline (see above). For additional details, please see [stitching-spark](https://github.com/saalfeldlab/stitching-spark).
For coarsely register images, We developed [BigStream](https://github.com/GFleishman/bigstream) for robust and fully automated non-rigid registration of multi-round FISH data. BigStream first performs fast global affine transformation using a feature-based random sample consensus (RANSAC) algorithm. The image volume is then divided into overlapping blocks and another round of feature-based affine transformation is performed, followed by a fast 3D deformable registration [greedypy](https://github.com/GFleishman/greedypy) ([Yushkevich, 2016](https://github.com/pyushkevich/greedy)) on each block. Bigstream can be executed as part of the EASI-FISH pipeline. It also can be installed and used seperately. 
For 3D segmentation, [Starfinity](https://github.com/mpicbg-csbd/stardist/tree/refinement) is a deep learning-based automatic 3D segmentation software. Starfinity is an extension of [Stardist](https://github.com/mpicbg-csbd/stardist), an earlier cell detection approach (Schmidt et al., 2018; Weigert et al., 2020) and is based on the dense prediction of cell border distances and their subsequent aggregation into pixel affinities. A starfinity [model](https://doi.org/10.25378/janelia.13624268) was trained to predict cell body shapes from DAPI-stained RNA images and is provided for testing. Starfinity can be executed as part of the EASI-FISH pipeline. It can also be installed and used independently. 
For distributed spot detection, we used [RS-FISH](https://github.com/PreibischLab/RS-FISH) and our developed hAirlocalize, methods based on the JAVA and MATLAB spot detection algorithm. [Airlocalize](https://github.com/timotheelionnet/AIRLOCALIZE)([Lionnet et al., 2011](https://www.nature.com/articles/nmeth.1551)) to allow rapid spot detection on full-resolution large image datasets. hAirlocalize can be executed independently or as part of the EASI-FISH pipeline (see above). For independent execution, we recommend working with the n5 filesystem due to large file size.
![](/Diagrams/Pipeline.gif). 

### Warp Mask and Spot assignment #
For extracting the point clouds to do the 3D registrataion, a necessary step is to assign the spots for cell masks of the fixed and moving rounds. We used the inverse tranformation exported from the above bigstream registration and warp the cell mask of the fixed rounds to get the mask of moving rounds images.The spots will be assigned to these new cell masks.

### Classic Point cloud registration #
For accurately register the point clouds of multiple rounds, we apply methods for register the FISH spots with the classic linear registration methods. Ransac is a powerful tool for coarsely registered the spots, ICP then will be used for a fine registration.
For registering the sequentially 3-channel acquired FISH images, we first correct the chromatic abberation, registered the DAPI channel, apply the DAPI transformation to each FISH channel, and finally do the point cloud registration for all FISH channels of all rounds.

### Deep-learning based Point cloud registration #
For more accurately register the point clouds of multiple rounds. We also apply deep-learning based registeration method here, RPMnet and PCRnet appeal to be the ideal options for a better registration.

### Barcoding and decoding analysis #
We adapted the idea of [seqFISH][seqFISH-PLUS](https://github.com/CaiGroup/seqFISH-PLUS) and [HCR 3.0] (https://www.molecularinstruments.com/hcr-rnafish-products) for our 3D-seqFISH experiments. For barcoding the genes, we used 3 different channel/colors will be used and 4 rounds will be run. 1-3 extra rounds will be conducted for error corrections.
For encoding the genes, we modified SeqFISH design by using a best threshold radius for determine the colocalized spots of each rounds (see Youden J's metrics). For increase the detection efficency, the nearest 3 spots for a specific bit will be used for find the most correlated code for a specific gene.

## Pipeline #
We build a self-contained, highly flexible, and platform agnostic computational [pipeline](https://github.com/JaneliaSciComp/multifish), which supports turnkey EASI-FISH data analysis on local machines and the LSF compute cluster. The pipeline is freely available, open source, and modular. It can rapidly process large datasets greater than 10 TB in size with minimal manual intervention. The pipeline can be used to analyze EASI-FISH dataset end-to-end. It takes `czi` image files acquired from Zeiss Z.1 lightsheet microscope as input and outputs 1) processed image data at different scales and 2) transcript counts that can be readily used for cell type identification. The pipeline also provides options to run individual analysis modules, such as image stitching or registration. 

### Example data #
3D-SeqFISH [example datasets] We provide the related images and point clouds of a example Cell 5# for testing the 3D registration.See the description later.
EASI-FISH [example datasets](https://doi.org/10.25378/janelia.c.5276708.v1) are provided for software testing. For instructions on performing a demo run with the example data using the end-to-end analysis pipeline, see [here](https://github.com/JaneliaSciComp/multifish). 

### Jupyter notebooks #
Now, ROI_ransac_napari_fixmovROI.ipynb is the major python script for processing the 3d registration of FISH point clouds.

### Bash command #
Pipeline was executed with Bash command.
We used the Slurm high-performance cluster for computing, see the description for the Slurm workload manager [Slurm workload manager](https://slurm.schedmd.com/documentation.html).

## Additional information #

### Visualization #
Fiji-based [n5-viewer](https://github.com/saalfeldlab/n5-viewer) can be used for large image dataset visualization on local machines. The workflow also outputs processed intermediate image data in the stitching (`n5`), registration (`n5`) and segmentation (`tif`) steps. For inspection of spot extracted with hAirlocalize, we recommend the python-based multi-dimensional image viewer, [napari](https://napari.org/). Example [notebooks](https://github.com/multiFISH/EASI-FISH/tree/master/data_visualization) are provided. 

### Post processing #
Code used for [Post processing](https://github.com/multiFISH/EASI-FISH/tree/master/data_processing), such as assign spots, cell morphological measurements, dense spot analysis, FISH signal intensity measurements, lipofuscin subtraction are included in this repository. 

### Reference #

**Expansion-Assisted Iterative-FISH defines lateral hypothalamus spatio-molecular organization** <br/>
*Yuhan Wang, Mark Eddison, Greg Fleishman, Martin Weigert, Shengjin Xu, Fredrick E. Henry, Tim Wang, Andrew L. Lemire, Uwe Schmidt, Hui Yang, Konrad Rokicki, Cristian Goina, Karel Svoboda, Eugene W. Myers, Stephan Saalfeld, Wyatt Korff, Scott M. Sternson, Paul W. Tillberg* <br/>
bioRxiv 2021.03.08.434304; doi: https://doi.org/10.1101/2021.03.08.434304