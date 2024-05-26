# **EPI distortion correction using a GRE B<sub>0</sub> field map**

## **1. Introduction**
The aim of this page is to demonstrate how to use a gradient echo B<sub>0</sub> field map to perform distortion correction and registration of fMRI and structural images.

The required dataset include:

1. Original (non-brain-extracted) T<sub>1</sub> weighted image: _T1.nii.gz_
1. A single 3D volume from a functional EPI acquisition: _blip_down.nii.gz_
1. A field map acquisition (phase and magnitude): _FieldMap.nii.gz_ and _Mag_e2.nii.gz_

***


## **2. GRE B<sub>0</sub> field map preparation**
For the EPI distortion correction using a GRE field map, the first step is to acquire the required data with the Siemens _gre_field_mapping_ pulse sequence for the B<sub>0</sub> field map calculation using the available FSL tools. please check out [here](https://github.com/Lin-Brain-Lab/GRE-B0-field-mapping-for-Siemens-3T-Prisma-scanner/wiki).

***

## **3. Structural preparation**

The collected T<sub>1</sub> weighted structural image needs to be prepared: 

**Step 1:** Brain extract the T<sub>1</sub> weighted structural image using _bet_gui_. 

<img width="424" alt="image" src="https://github.com/Lin-Brain-Lab/EPI-distortion-correction-with-GRE-B0-field-map/assets/152513951/f51b2b84-ad5b-4f80-bda7-c2394c955f3d">

**Step 2:**
Segment the brain extracted T<sub>1</sub> weighted image as it will be needed later by the boundary-based registration in order to identify the desired boundaries (i.e. white matter boundaries):

    fast -B -I 10 -l 10 PATH/T1_brain.nii.gz

This will create several outputs in the PATH directory, of which the most important one for our purpose is the white matter partial volume estimate: _T1_brain_pve_2.nii.gz_. 

Load this, along with the _T1_brain.nii.gz_ into FSLeyes to assess the performance:

![screenshot](https://github.com/Lin-Brain-Lab/EPI-distortion-correction-with-GRE-B0-field-map/assets/152513951/82596baf-835f-4f99-a2d6-aab3e5e2dd9a)



**Step 3:** Make a binary segmentation from the white matter partial volume estimate _T1_brain_pve_2.nii.gz_ from step 2, which will be used to identify the boundary points:

    fslmaths PATH/T1_brain_pve_2.nii.gz -thr 0.5 -bin PATH/T1_wmseg.nii.gz

Where _-thr 0.5_ indicates that all voxels with more than 50% partial volume are selected as part of the binary mask. This will be used later on to assess the quality of the distortion correction.

***

## **4. Boundary-based (BBR) registration**
Results obtained using BBR are usually substantially better than alternative methods and this is now the strongly recommended way to register EPI and structural images. Viewing the transformed EPI with an overlay of the white-matter boundaries is a very good way to assess the accuracy of the registration, and distortion corretion.

Now that the field map and the structural image are processed, we can apply the BBR registration of the fMRI (EPI) to the structural, using the field map information from section 2 to correct for geometric distortion.

Two EPI parameters are needed here:
1. Effective echo spacing: in our case 0.51ms
1. Phase encoding direction: blip down (i.e. AP or -y), or blip up (i.e. PA or y)

Having all the required data in the FSl directory, for the EPI data collected with a blip down phase encoding scheme, run:

    epi_reg --echospacing=0.00051 --wmseg=T1_wmseg.nii.gz --fmap=FieldMap.nii.gz --fmapmag=Mag_e2.nii.gz --fmapmagbrain=Mag_e2_brain.nii.gz --pedir=-y  --epi=blip_down.nii.gz --t1=T1.nii.gz --t1brain=T1_brain.nii.gz --out=func2struct_negy

and for the EPI data collected with a blip up phase encoding scheme, run:

    epi_reg --echospacing=0.00051 --wmseg=T1_wmseg.nii.gz --fmap=FieldMap.nii.gz --fmapmag=Mag_e2.nii.gz --fmapmagbrain=Mag_e2_brain.nii.gz --pedir=y  --epi=blip_up.nii.gz --t1=T1.nii.gz --t1brain=T1_brain.nii.gz --out=func2struct_posy


The main outputs of interest are (for the negy version as an example):

1. _func2struct_negy.nii.gz_: the functional image resampled into structural space, and distortion corrected
1. _func2struct_negy_fast_wmedge.nii.gz_: binary image representing the white matter boundaries from the T<sub>1</sub> weighted structural image, which is very useful for viewing and assessing registration quality (normally shown in red on top of transformed functional image)

To assess the results, load the two main outputs _func2struct_negy.nii.gz_ and the _func2struct_negy_fast_wmedge.nii.gz_ into FSLeyes or any other viewer, give the latter a Red colormap, and put it on top of the overlay list.

***

## **5. In vivo example**

MRI scans were performed on a Siemens 3T Prisma scanner on a healthy female volunteer. The resting-state EPI data were acquired with the FOV = 210 * 210 mm<sup>2</sup>, matrix size = 64 * 64, spatial resolution of 3.3 * 3.3 * 2.5 mm<sup>3</sup>, TR/TE = 1500/30 ms, rBW = 2232 Hz/Px, echo spacing of 0.51ms, and total acquisition time of ~5 mins for 200 bold measurements. Scans were repeated for both blip down and blip up phase encoding directions. T<sub>1</sub> weighted structural image was collected using a high resolution MPRAGE protocol. The B<sub>0</sub> field map acquisition was performed as explained [here](https://github.com/Lin-Brain-Lab/GRE-B0-field-mapping-for-Siemens-3T-Prisma-scanner/wiki).

Sample axial EPI brain slices registered to the structural image, (left) distortion corrected, and (right) uncorrected:

<img width="899" alt="image" src="https://github.com/Lin-Brain-Lab/EPI-distortion-correction-with-GRE-B0-field-map/assets/152513951/b1de2b2a-b9b8-45b9-857f-10a2881122a0">

<img width="896" alt="image" src="https://github.com/Lin-Brain-Lab/EPI-distortion-correction-with-GRE-B0-field-map/assets/152513951/34c1515e-4c38-453a-b389-a72fb732b930">

<img width="897" alt="image" src="https://github.com/Lin-Brain-Lab/EPI-distortion-correction-with-GRE-B0-field-map/assets/152513951/224abd20-0f39-4042-b35d-fbd86ec72e78">

***

## **6. Input data**

### **6.1. B<sub>0</sub> field map data:**
1. [Mag_e1.nii.gz](https://github.com/Lin-Brain-Lab/EPI-distortion-correction-with-GRE-B0-field-map/files/13982739/Mag_e1.nii.gz)
1. [Mag_e2.nii.gz](https://github.com/Lin-Brain-Lab/EPI-distortion-correction-with-GRE-B0-field-map/files/13982700/Mag_e2.nii.gz)
1. [ph.nii.gz](https://github.com/Lin-Brain-Lab/EPI-distortion-correction-with-GRE-B0-field-map/files/13982743/ph.nii.gz)

### **6.2. fMRI data:**
1. [blip_down.nii.gz](https://github.com/Lin-Brain-Lab/EPI-distortion-correction-with-GRE-B0-field-map/files/13982702/blip_down.nii.gz)
1. [blip_up.nii.gz](https://github.com/Lin-Brain-Lab/EPI-distortion-correction-with-GRE-B0-field-map/files/14043005/blip_up.nii.gz)

### **6.3. Structural data:**
1. [T1.nii.gz](https://github.com/Lin-Brain-Lab/EPI-distortion-correction-with-GRE-B0-field-map/files/13982711/T1.nii.gz)

***

## **7. References**

1. https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/FUGUE/Guide
2. https://www.fmrib.ox.ac.uk/primers/intro_primer/ExBox19/IntroBox19.html
3. https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/FLIRT_BBR
4. https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/FLIRT/UserGuide#epi_reg
