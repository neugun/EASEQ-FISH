## 3D-SeqFISH analysis toolbox

## Table of Contents #
   * [Description](#description)
      * [News and plans](#news-and-plans)
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

The 3D-SeqFISH analysis toolbox integrate EASIFISH (Expansion-Assisted Iterative Fluorescence *In Situ* Hybridization) pipeline and 3D registration method for analyzing image-based 3D spatial transcriptomic data. We aim to provide the general solution for 3D spatial transcriptome, including automated image stitching, 3D cell segmentation, distributed spot detection, highly accurate multi-round 3D FISH spot registration (nm-level accuracy), and efficiently decoding hundreds of genes from sample images.  <br/><br/>
![](/Diagrams/3DseqFISH_diagram_v1.png)

### News and plans #
- Released the 3D-SeqFISH github page on 7/2/22
- Add 3D-SeqFISH individual scripts on 7/3/22
- Test PFH and FPFH on 7/3/22
- Grouped registration scripts and evaluation on 7/3/22
- Test RANSAC-ICP on 7/4/22
- Generate shuffle point clouds on 7/6/22
- Determine the threshold for radius on 7/9/22
- Add 3D-SeqFISH decoding results for a single tile image on 7/11/22
- Test deep learning methods for registration by 8/1/22
- Write decoding analysis by 10/1/22
- Add 3D-SeqFISH bash scripts by 9/1/22

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

![](/Diagrams/3DseqFISH_diagram_v1_EASIFISH.png)
- See updates for [EASI-FISH pipeline](https://github.com/multiFISH/EASI-FISH), and [MULTIFISH pipeline](https://github.com/JaneliaSciComp/multifish). <br/>
- For imaging large volumes, multiple sub-volumes (tiles) are sequentially acquired with 3X Expansion microscopy, followed by [computational stitching](https://science.sciencemag.org/content/363/6424/eaau8302.long) into a single large image. For independent execution, we recommend working with the n5 filesystem due to large file size. <br/>
- For coarsely registering images, we used [BigStream](https://github.com/GFleishman/bigstream) for robust and fully automated non-rigid registration of multi-round FISH data. For 3D segmentation, [Starfinity](https://github.com/mpicbg-csbd/stardist/tree/refinement) is a deep learning-based automatic 3D segmentation software. Starfinity is an extension of [Stardist](https://github.com/mpicbg-csbd/stardist). <br/>
- For distributed spot detection, we used [RS-FISH](https://github.com/PreibischLab/RS-FISH) or [hAirlocalize](https://github.com/timotheelionnet/AIRLOCALIZE) to allow rapid spot detection on full-resolution large image datasets. <br/>
- Assign spots, cell morphological measurements, dense spot analysis, FISH signal intensity measurements, lipofuscin subtraction are included in the EASIFISH Pipeline.

### Spot assignment #
![](/Diagrams/3DseqFISH_diagram_v1_WARPMASK.png)
- To extract the point clouds for later 3D registrataion, we use the inverse tranformation exported from the above bigstream registration to warp the cell mask of the fixed rounds. Then, we adapt filter to remove the noise on the cell edges of the moving masks. After we adjust the image size of moving masks, we assign the spots for cell masks of the fixed and moving rounds.

### Classic Point cloud registration #
![](/Diagrams/3DseqFISH_diagram_v1_DAPI.png)
- To accurately register the point clouds of multiple rounds, we apply classic linear registration methods for registering the FISH spots. For registering the sequentially 3-channel acquired FISH images, we first correct the chromatic abberation, registered the DAPI channel, apply the DAPI transformation to each FISH channel, and finally do the point cloud registration for all FISH channels of all rounds. <br/>
- Random sample consensus (RANSAC) is a powerful tool for coarsely registered the spots, and ICP then provides a solution for fine registration. Point cloud registration method first performs fast global affine transformation using a feature-based RANSAC algorithm. ICP then derives the globally optimal transformation for each cell that minimizes the sum of square distances to competing optimal affine matrix. <br/>
- Point cloud registration can be executed as part of the 3D-SeqFISH pipeline. It also can be installed and used seperately. <br/>
- ROI_ransac_napari_fixmovROI.ipynb now is the only python script for processing the 3d registration of FISH point clouds.

### Deep-learning based Point cloud registration #
- For more accurately register the point clouds of multiple rounds, we also apply deep-learning based [registeration method](https://github.com/vinits5/learning3d#use-of-registration-networks). [RPMnet](https://github.com/yewzijian/RPMNet) and [PCRnet](https://github.com/vinits5/pcrnet) appeal to be the ideal options for a better registration.

### Barcoding and decoding analysis #
- We benefit from the idea of [seqFISH](https://github.com/CaiGroup/seqFISH-PLUS) and [HCR 3.0](https://www.molecularinstruments.com/hcr-rnafish-products) for our 3D-seqFISH experiments. For barcoding the genes, 3 different channels of lightsheet microscope will be used and 4-5 rounds of FISH will be run for encoding 81-243 genes. 2-6 extra rounds will be conducted for error corrections, examine the decoding efficiency and probe the dense expressed genes. <br/>
- For decoding the genes, we modify the SeqFISH design by using a best threshold radius for determine the colocalized spots of each rounds (see [Youden's J Statistic](https://www.kaggle.com/code/willstone98/youden-s-j-statistic-for-threshold-determination/notebook)). <br/>
- For increase the detection efficency, the nearest 3 spots for a specific barcode bit will be used for searching the most correlated code of a specific gene. <br/>

## Pipeline #
- We build a self-contained, highly flexible, and platform agnostic computational [pipeline](https://github.com/JaneliaSciComp/multifish), which supports a turnkey 3D-SeqFISH analysis on local machines and the High performance compute cluster (such as Slurm or LSF). It can rapidly process large datasets greater than 10 TB in size with minimal manual intervention. The pipeline can be used to analyze EASI-FISH dataset end-to-end. It takes `czi` image files acquired from Zeiss Z7 lightsheet microscope as input and outputs 1) processed image data at different scales and 2) transcript counts that can be readily used for cell type identification. The pipeline also provides options to run individual analysis modules, such as point cloud registration and gene decoding. 

### Example data #
- This toolbox handles large-scale, multi-round, high-resolution image data acquired using EASI-FISH. It takes advantage of the [n5](https://github.com/saalfeldlab/n5) filesystem to allow for rapid and parallel data reading and writing. [Example images](https://doi.org/10.25378/janelia.c.5276708.v1) are provided for testing EASI-FISH pipeline. <br/>
- We provide the related images and point clouds of a example cell (5#) for testing the 3D registration. Stay tuned to the description later. <br/>

### Jupyter notebook #
- ROI_ransac_napari_fixmovROI.ipynb showes the registation steps and visualization of registered point clouds and images.

### Bash command #
- Pipeline was executed with Bash command. <br/>
- We used the Slurm high-performance cluster for computing, see the description for the Slurm workload manager [Slurm workload manager](https://slurm.schedmd.com/documentation.html).

## Additional information #

### Visualization #
- Fiji-based [n5-viewer](https://github.com/saalfeldlab/n5-viewer) can be used for large image dataset visualization on local machines. The workflow also outputs processed intermediate image data in the stitching (`n5`), registration (`n5`) and segmentation (`tif`) steps. For inspection of spot extracted with hAirlocalize, we recommend the python-based multi-dimensional image viewer, [napari](https://napari.org/). Example [notebooks](https://github.com/multiFISH/EASI-FISH/tree/master/data_visualization) are provided. 

### Post processing #
- Code used for [Post processing](https://github.com/multiFISH/EASI-FISH/tree/master/data_processing), such as assign spots, cell morphological measurements, dense spot analysis, FISH signal intensity measurements, lipofuscin subtraction are included in this repository. 

### Reference #

**EASI-FISH for thick tissue defines lateral hypothalamus spatio-molecular organization** <br/>
*Yuhan Wang, Mark Eddison, Greg Fleishman, Martin Weigert, Shengjin Xu, Fredrick E. Henry, Tim Wang, Andrew L. Lemire, Uwe Schmidt, Hui Yang, Konrad Rokicki, Cristian Goina, Karel Svoboda, Eugene W. Myers, Stephan Saalfeld, Wyatt Korff, Scott M. Sternson, Paul W. Tillberg* <br/>
https://www.sciencedirect.com/science/article/pii/S0092867421013398

### Contributors#
Zhenggang Zhu <br/>
Shiqi Wang (Charlotte) <br/>
