## 3D-SeqFISH analysis toolbox

## Table of Contents #
   * [Description](#description)
      * [News and plans](#news-and-plans)
   * [Installation](#Installation)
   * [Modules](#modules)
      * [Preprocessing with EASIFISH Pipeline](#preprocessing-with-EASIFISH-Pipeline)
      * [Spot assignment](#spot-assignment)
      * [Traditional point cloud registration](#traditional-point-cloud-registration)
      * [Deep-learning based point cloud registration](#deep-learning-based-point-cloud-registration)
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

The 3D-SeqFISH analysis toolbox integrates EASIFISH (Expansion-Assisted Iterative Fluorescence *In Situ* Hybridization) pipeline and 3D registration method for analyzing image-based 3D spatial transcriptomic data. We aim to provide the general solution for 3D spatial transcriptome, including automated image stitching, 3D cell segmentation, distributed spot detection, highly accurate multi-round 3D FISH spot registration (nm-level accuracy), and efficiently decoding hundreds of genes for thick tissues.  <br/><br/>
![](/Diagrams/3DseqFISH_diagram_v1.png)

### News and plans #
- Released the 3D-SeqFISH github page on 7/2/22
- Tested FPFH on 7/4/22
- Determined the threshold for radius on 7/5/22
- Grouping registration scripts and evaluation on 7/4/22
- Generated shuffled point clouds on 7/7/22
- Test RANSAC-ICP on 7/8/22
- Test PFH on 7/8/22
- Add 3D-SeqFISH individual scripts on 7/10/22
- Summary of 3D-SeqFISH decoding results for a single tile image on 7/11/22
- Test deep learning methods for registration by 8/1/22
- Add decoding analysis by 9/1/22
- Test 3D-SeqFISH bash scripts by 9/1/22

## Installation #

`3D-SeqFISH` analysis are inspired by point cloud registration and seqFISH decoding methods. Python >= 3.6 required,jupyter notebook,matplotlib,numpy,math,cv2,skimage,scipy,pandas,seaborn,zarr,z5py,qt5 packages are installed inside a anaconda envioronment (python 3.8.12), and it can be installed with `pip`:
```
   pip install 3D-SeqFISH
```
   pip install pclpy==0.11.0 # Python == 3.6 required
```
   pip install python-pcl
```
```
   pip install open3D
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
- For imaging large volumes, multiple sub-volumes (tiles) are sequentially acquired with 3X Expansion microscopy, followed by [computational stitching](https://science.sciencemag.org/content/363/6424/eaau8302.long) into a single large image with the n5 filesystem due to large file size. <br/>
- For coarsely registering images, we used [BigStream](https://github.com/GFleishman/bigstream) for rigid and non-rigid register multi-round FISH data. For 3D segmentation, [Starfinity](https://github.com/mpicbg-csbd/stardist/tree/refinement) is a deep learning-based automatic 3D segmentation software. Starfinity is an extension of [Stardist](https://github.com/mpicbg-csbd/stardist). <br/>
- For distributed spot detection, we used [RS-FISH](https://github.com/PreibischLab/RS-FISH) or [hAirlocalize](https://github.com/timotheelionnet/AIRLOCALIZE) to allow rapid spot detection on full-resolution large image datasets. <br/>
- Other processsing steps, such as assigning spots, cell morphological measurements, dense spot analysis, FISH signal intensity measurements, lipofuscin subtraction have been included in the EASIFISH Pipeline.

### Spot assignment #
![](/Diagrams/3DseqFISH_diagram_v1_WARPMASK.png)
- To extract the point clouds for later 3D registrataion, we use the inverse tranformation exported from the above bigstream registration to warp the cell mask of the fixed rounds. Then, we adapt filter to remove the noise on the cell edges of the moving masks. After we adjust the image size of moving masks, we assign the spots for cell masks of the fixed and moving rounds.

### Traditional point cloud registration #
- Traditional point cloud registration methods are mostly used for registering the FISH images and RANSAC + ICP has been used as the SOTA.
- ROI_ransac_napari_fixmovROI.ipynb (/3d_SeqFISH/) now is the only python script for processing the 3d registration of FISH point clouds.<br/>
- Feature-based Random sample consensus (RANSAC) algorithm is a powerful tool to coarsely registered the spots for each cell by estimating the fast global affine transformation. Iterative closest point (ICP) derives the globally optimal transformation for each cell that minimizes the sum of square distances to competing optimal affine matrix, which is provides a solution for fine registration. <br/>
- Correct the chromatic abberation or the misalignment of image tracks will always be the initial step. Image-assisted or point feature based point cloud registratio with RANSAC + ICP has been used for registering green and red channel spots. Spots here can be lipofuscins, dual color probes, or multicolor beads.
- Point cloud registration can be executed as part of the 3D-SeqFISH pipeline. It also can be installed and used seperately. <br/>
- To accurately register the point clouds of multiple rounds and find the best way for it, we have applied 3 strageties to register the FISH spots: 1) image-assisted point cloud registratio with RANSAC + ICP; 2) point cloud registratio with FPFH-RANSAC + ICP; 3) point cloud registratio with PFH-RANSAC + RANSAC-ICP (to be tested). <br/>
1) Image-assisted point cloud registratio with RANSAC + ICP: we use ransac-affine methods (can be also found in the bigstream registration) for global registering the point clouds by correlating the DAPI/FISH image context. Specficially, we first use the correlation of the DAPI context for DAPI blobs to do the ransac-affine, and use ICP-affine to finely register the DAPI channel. Next, we apply the DAPI transformation to spots of each FISH channel and the track transformation to spots of green and magenta channel. Then, we correlated the FISH image context for FISH spots of each cell to perform later ransac-affine and do ICP-affine to register the point clouds for all FISH channels of all rounds. <br/>
![](/Diagrams/3DseqFISH_diagram_v1_DAPI.png)
2) Point cloud registration with FPFH-RANSAC + ICP <br/>
- The step of this method does not extract features from the image (no image will be generated and used here), but only use features from the point clouds (here is the fast point feature histogram, FPFH). The detailed steps are: a) we use ransac-affine for global registering the point clouds with DAPI blobs, and do ICP-affine to finely register the DAPI channel. But this step can be omitted. b) we apply the DAPI transformation to spots of each FISH channel and the track transformation to spots of green and magenta channels. Then, we correlated the point feature for FISH spots of each cell to perform local ransac-affine and do ICP-affine to register the point clouds for all FISH channels of all rounds. <br/>
- FPFH is the fast way to extract features, but it not has comparable features as the images or other points feature extraction method (e.g., PFH or SHOT). Our results with this method shows point clouds of some cells have not been well-registered or the distance of corresponding spots > 1 um, indicating it is worse than the above method . <br/>
3) Point cloud registration with PFH-RANSAC + RANSAC-ICP (to be tested) <br/>
- PFH will include more features than FPFH, and RANSAC-ICP is a method that removing outliers of each iterative step during the fine ICP registration. <br/>
<br/>

### Deep-learning based point cloud registration #
- Image-assisted or points feature based RANSAC affine method is not excel at extracting rich and important features, and these showed results are still not ideal in accuracy and efficiency.
- To more accurately register the point clouds of multiple rounds, we also apply [deep-learning based registeration method](https://github.com/vinits5/learning3d#use-of-registration-networks). [RPMnet](https://github.com/yewzijian/RPMNet) and [PCRnet](https://github.com/vinits5/pcrnet) appeal to be the ideal options for getting a better registration.

### Barcoding and decoding analysis #
- We benefit from the idea of [seqFISH](https://github.com/CaiGroup/seqFISH-PLUS) and [HCR 3.0](https://www.molecularinstruments.com/hcr-rnafish-products) for our 3D-seqFISH barcoding and decoding. For barcoding the genes, 3 different channels of FISH images acquired with lightsheet microscope will be used and 4 rounds of EASIFISH will be run for encoding 81 genes. 3-6 extra rounds will be conducted for error correction, increasing the decoding efficiency and probing dense expressed genes with non-combinatorial FISH. <br/>
- For decoding the genes for our seqFISH images and spots, we choose 1 μm as the best threshold radius to determine the colocalized spots of each rounds, which is validated by [Youden's J Statistic](https://www.kaggle.com/code/willstone98/youden-s-j-statistic-for-threshold-determination/notebook)). <br/>
- For increase the detection efficency, the nearest 3 spots for a specific barcode bit will be used for searching the most correlated code of a specific gene. <br/>

## Pipeline #
- We will build a highly flexible computational [pipeline](https://github.com/JaneliaSciComp/multifish), which supports a turnkey 3D-SeqFISH analysis on local machines and the High performance compute cluster (such as Slurm or AWS). It can rapidly process large datasets greater than 10 TB in size with minimal manual intervention. The pipeline also provides options to run individual analysis modules, such as point cloud registration and gene decoding. 

### Example data #
- This toolbox handles large-scale, multi-round, high-resolution image data acquired using EASI-FISH. It takes advantage of the [n5](https://github.com/saalfeldlab/n5) filesystem to allow for rapid and parallel data reading and writing. [Example images](https://doi.org/10.25378/janelia.c.5276708.v1) are provided for testing EASI-FISH pipeline. <br/>
- We provide the related images and point clouds of a example cell (5#) for testing the 3D registration. Stay tuned to the description of the data later. <br/>

### Jupyter notebook #
- ROI_ransac_napari_fixmovROI.ipynb has each registation step and can do the visualization of registered point clouds and images.

### Bash command #
- Pipeline is to be executed with Bash command. <br/>
- We use the Slurm high-performance cluster for computing, see the description for the Slurm workload manager [Slurm workload manager](https://slurm.schedmd.com/documentation.html).

## Additional information #

### Visualization #
- Fiji-based [n5-viewer](https://github.com/saalfeldlab/n5-viewer) can be used for large image dataset visualization. For inspection of spot registration, we recommend the python-based multi-dimensional image viewer, [napari](https://napari.org/). Example [notebooks](https://github.com/multiFISH/EASI-FISH/tree/master/data_visualization) are provided. 

### Post processing #
- Code used for [Post processing](https://github.com/multiFISH/EASI-FISH/tree/master/data_processing), such as assigning spots, cell morphological measurements, dense spot analysis, FISH signal intensity measurements, lipofuscin subtraction are included in this repository. 

### Reference #

**EASI-FISH for thick tissue defines lateral hypothalamus spatio-molecular organization** <br/>
*Yuhan Wang, Mark Eddison, Greg Fleishman, Martin Weigert, Shengjin Xu, Fredrick E. Henry, Tim Wang, Andrew L. Lemire, Uwe Schmidt, Hui Yang, Konrad Rokicki, Cristian Goina, Karel Svoboda, Eugene W. Myers, Stephan Saalfeld, Wyatt Korff, Scott M. Sternson, Paul W. Tillberg* <br/>
https://www.sciencedirect.com/science/article/pii/S0092867421013398

### Contributors#
Zhenggang Zhu <br/>
Shiqi Wang (Charlotte) <br/>
