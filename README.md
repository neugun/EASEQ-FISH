## 3D-SeqFISH analysis toolbox

## Table of Contents #
   * [Description](#description)
   * [Installation](#Installation)
   * [Modules](#modules)
      * [Preprocessing with EASIFISH Pipeline](#preprocessing-with-EASIFISH-Pipeline)
      * [Spot assignment](#spot-assignment)
      * [Classic Point cloud registration](#classic-point-cloud-registration)
      * [Deep learning](#deep-learning)
      * [Barcoding and decoding analysis](#barcoding-and-decoding-analysis)
   * [Pipeline](#pipeline)
      * [Example data](#example-data)
      * [Jupyter notebook](#jupyter-notebook)
      * [Bash command](#bash-command)
   * [Additional information](#additional-information)
      * [Visualization](#visualization)
      * [Post processing](#post-processing)  
      * [Reference](#reference)
      * [Contributors](#contributors)

## Description #

The 3D-SeqFISH analysis toolbox integrate EASIFISH (Expansion-Assisted Iterative Fluorescence *In Situ* Hybridization) pipeline and 3D registration method for analyzing image-based 3D spatial transcriptomic data. We aim to provide the general solution for 3D spatial transcriptome, including automated image stitching, 3D cell segmentation, distributed spot detection, and distributed and highly accurate multi-round 3D FISH spot registration (nm-level accuracy), and decoding analysis.  <br/>
![](/Diagrams/3DseqFISH_diagram_v1.png)

## Installation #

`3D-SeqFISH` analysis are inspired by Point cloud registration and seqFISH decoding methods, and it can be installed with `pip`:
```
   pip install 3D-SeqFISH
```

Preprocessing for 3D-SeqFISH is based on EASIFISH (Expansion-Assisted Iterative Fluorescence *In Situ* Hybridization), MULTIFISH, bigstream, napari, and stardist, etc. <br/> 

`bigstream`  can be installed with  `pip`:
```
   pip install bigstream
```
`napari`  can be installed with  `pip`:
```
   python -m pip install "napari[all]"
```
`Starfinity`  can be installed with  `pip`:
```
   pip install git+https://github.com/mpicbg-csbd/stardist@refinement
```

## Modules #

### Preprocessing with EASIFISH Pipeline #
![](/Diagrams/Pipeline.gif)
See [updates](https://github.com/multiFISH/EASI-FISH) for EASI-FISH pipeline, and [MULTIFISH pipeline](https://github.com/JaneliaSciComp/multifish). <br/>
For imaging large volumes, multiple sub-volumes (tiles) are sequentially acquired with 3X Expansion microscopy, followed by [computational stitching](https://science.sciencemag.org/content/363/6424/eaau8302.long) into a single large image. For independent execution, we recommend working with the n5 filesystem due to large file size. <br/>
The stitching module can be executed with the EASI-FISH pipeline (see above). For coarsely registering images, we used [BigStream](https://github.com/GFleishman/bigstream) for robust and fully automated non-rigid registration of multi-round FISH data. For 3D segmentation, [Starfinity](https://github.com/mpicbg-csbd/stardist/tree/refinement) is a deep learning-based automatic 3D segmentation software. Starfinity is an extension of [Stardist](https://github.com/mpicbg-csbd/stardist), an earlier cell detection approach (Schmidt et al., 2018; Weigert et al., 2020) and is based on the dense prediction of cell border distances and their subsequent aggregation into pixel affinities. A starfinity [model](https://doi.org/10.25378/janelia.13624268) was trained to predict cell body shapes from DAPI-stained RNA images and is provided for testing. <br/>
For distributed spot detection, we used [RS-FISH](https://github.com/PreibischLab/RS-FISH) or hAirlocalize [Airlocalize](https://github.com/timotheelionnet/AIRLOCALIZE) to allow rapid spot detection on full-resolution large image datasets. <br/>
Assign spots, cell morphological measurements, dense spot analysis, FISH signal intensity measurements, lipofuscin subtraction are included in the EASIFISH Pipeline.

### Spot assignment #
To extract the point clouds for later 3D registrataion, we use the inverse tranformation exported from the above bigstream registration to warp the cell mask of the fixed rounds. Then, we assign the spots for cell masks of the fixed and moving rounds. The spots will be assigned to these new cell masks.

### Classic Point cloud registration #
To accurately register the point clouds of multiple rounds, we apply classic linear registration methods for registering the FISH spots.For registering the sequentially 3-channel acquired FISH images, we first correct the chromatic abberation, registered the DAPI channel, apply the DAPI transformation to each FISH channel, and finally do the point cloud registration for all FISH channels of all rounds. <br/>
Random sample consensus (RANSAC) is a powerful tool for coarsely registered the spots, and ICP then provides a solution for fine registration. Point cloud registration method first performs fast global affine transformation using a feature-based RANSAC algorithm. ICP then derives the globally optimal transformation for each cell that minimizes the sum of square distances to competing optimal affine matrix. <br/>
Point cloud registration can be executed as part of the 3D-SeqFISH pipeline. It also can be installed and used seperately. 
![](/Diagrams/3DseqFISH_diagram_v1_DAPI.png)

### Deep-learning based Point cloud registration #
For more accurately register the point clouds of multiple rounds, we also apply deep-learning based [registeration method](https://github.com/vinits5/learning3d#use-of-registration-networks).[RPMnet](https://github.com/yewzijian/RPMNet) and [PCRnet](https://github.com/vinits5/pcrnet) appeal to be the ideal options for a better registration.

### Barcoding and decoding analysis #
We borrow the idea of [seqFISH](https://github.com/CaiGroup/seqFISH-PLUS) and [HCR 3.0](https://www.molecularinstruments.com/hcr-rnafish-products) for our 3D-seqFISH experiments. For barcoding the genes, 3 different channels will be used and 4-5 rounds of FISH will be run. 1-3 extra rounds will be conducted for error corrections. <br/>
For decoding the genes, we modify the SeqFISH design by using a best threshold radius for determine the colocalized spots of each rounds (see [Youden's J Statistic](https://www.kaggle.com/code/willstone98/youden-s-j-statistic-for-threshold-determination/notebook)). <br/>
For increase the detection efficency, the nearest 3 spots for a specific barcode bit will be used for searching the most correlated code of a specific gene. <br/>

## Pipeline #
We build a self-contained, highly flexible, and platform agnostic computational [pipeline](https://github.com/JaneliaSciComp/multifish), which supports a turnkey 3D-SeqFISH analysis on local machines and the High performance compute cluster (such as Slurm or LSF). The pipeline is freely available, open source, and modular. It can rapidly process large datasets greater than 10 TB in size with minimal manual intervention. The pipeline can be used to analyze EASI-FISH dataset end-to-end. It takes `czi` image files acquired from Zeiss Z7 lightsheet microscope as input and outputs 1) processed image data at different scales and 2) transcript counts that can be readily used for cell type identification. The pipeline also provides options to run individual analysis modules, such as point cloud registration and gene decoding. 

### Example data #
This toolbox handles large-scale, multi-round, high-resolution image data acquired using EASI-FISH. It takes advantage of the [n5](https://github.com/saalfeldlab/n5) filesystem to allow for rapid and parallel data reading and writing.  <br/>
[Example images](https://doi.org/10.25378/janelia.c.5276708.v1) are provided for testing EASI-FISH pipeline. <br/>
We provide the related images and point clouds of a example cell (5#) for testing the 3D registration. Stay tuned to the description later. <br/>

### Jupyter notebook #
ROI_ransac_napari_fixmovROI.ipynb now is the only python script for processing the 3d registration of FISH point clouds. Stay tuned for individual components.

### Bash command #
Pipeline was executed with Bash command. <br/>
We used the Slurm high-performance cluster for computing, see the description for the Slurm workload manager [Slurm workload manager](https://slurm.schedmd.com/documentation.html).

## Additional information #

### Visualization #
Fiji-based [n5-viewer](https://github.com/saalfeldlab/n5-viewer) can be used for large image dataset visualization on local machines. The workflow also outputs processed intermediate image data in the stitching (`n5`), registration (`n5`) and segmentation (`tif`) steps. For inspection of spot extracted with hAirlocalize, we recommend the python-based multi-dimensional image viewer, [napari](https://napari.org/). Example [notebooks](https://github.com/multiFISH/EASI-FISH/tree/master/data_visualization) are provided. 

### Post processing #
Code used for [Post processing](https://github.com/multiFISH/EASI-FISH/tree/master/data_processing), such as assign spots, cell morphological measurements, dense spot analysis, FISH signal intensity measurements, lipofuscin subtraction are included in this repository. 

### Reference #

**EASI-FISH for thick tissue defines lateral hypothalamus spatio-molecular organization** <br/>
*Yuhan Wang, Mark Eddison, Greg Fleishman, Martin Weigert, Shengjin Xu, Fredrick E. Henry, Tim Wang, Andrew L. Lemire, Uwe Schmidt, Hui Yang, Konrad Rokicki, Cristian Goina, Karel Svoboda, Eugene W. Myers, Stephan Saalfeld, Wyatt Korff, Scott M. Sternson, Paul W. Tillberg* <br/>
https://www.sciencedirect.com/science/article/pii/S0092867421013398

### Contributors#
Zhenggang Zhu <br/>
Charlotte Wang <br/>