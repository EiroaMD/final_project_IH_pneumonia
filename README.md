# :hospital: Pneumonia.
**Final Project** | Dani Eiroa | **IH BCN Data Analytics PT 2019**

## Overview
<p align="center">
  <img src="https://prod-images-static.radiopaedia.org/images/1371188/24a32cc68436686e6b1852a00b57d4_jumbo.jpg" width="300">
</p>

A pneumonia is an acute infection of the lung and is characterized by the appearance of fever and respiratory symptoms, together with the presence of lung opacities on Chest-X-Rays (CXR).

According to the Spanish Society or Radiology (SERAM), there are no reliable data on the number of (CXR) not reported in Spain, although there is a widespread conviction that, with exceptions, radiology services have never reported 100% of them. There are hospitals that, in fact, have stopped reporting CXR, as the workload has been inclined towards the reporting of more complicated techniques, such as CT and MRI.

Throughout the process described below, we aim to develop a tool that classifies a given CXR in one of this two classes: Normal and Pneumonia.

## Data Preparation
The dataset is comprised of a total 26684 Chest X-Rays (CXR), some of them of patients affected by pneumonia and others with no pneumonia.

The dataset also contains two .csv files:
  - One with the patient Id (same as original image file name), a Target column (0 for normal and 1 for pneumonia) and the coordinates for the opacities consistent with pneumonia, as the original challenge included a segmentation task.
  - Another containing the class of the image (Opacity, Normal, Not-normal/No opacity) as well as patient Id.
  
The dataset was obtained from Kaggle and provided and labeled by the Radiological Society of North America (RSNA). [Link](https://www.kaggle.com/c/rsna-pneumonia-detection-challenge/data)


#### General description of the dataset such as the size, complexity, data types, etc.
26684 [DICOM](https://es.wikipedia.org/wiki/DICOM) files. (3.53 GB)

     - stage_2_train_labels.csv (26684, 6) (1.49 MB)
        - Patient Id: nominal variable. File names correspond to patient Id.
        - X and Y: the center of the boxes for the segmentation part.
        - height and width: the dimension of the box for the segmentation part.
        - target: binary. 0-normal and 1-pneumonia.
        
    - stage_2_detailed_class_info.csv (26684, 2) (1.69 MB)
        - Patient Id: nominal variable. File names correspond to patient Id.
        - Class: nominal variable with three possible values:
            - 'Normal'
            - 'Not normal/Not pneumonia'
            - 'Pneumonia'

### **Data Ingestion:**
Dataset was downloaded and unzziped using python functions defined in all three files of [step 2](https://github.com/EiroaMD/final_project_IH_pneumonia/tree/master/2%20Image%20Management), as different strategies were carried out to try to solve the problem.

CSVs were loaded using pandas `.read_csv()` method and explored accordingly (see 'Data Wrangling and Cleaning').

### **Data Wrangling and Cleaning:**
The original kaggle challenge included a segmentation task. For this reason, there were columns showing the coordinates of the pneumonic opacities and patient Id's that were repeated (some X-rays contained more than one pneumonic opacities). Those columns and duplicate rows were dropped. The two `.csv` files were joined by patiend Id.

Using specific python libraries (see 'Tools' below), the following metadata from DICOM files were extracted and added to the dataframe: age, sex and X-ray [projection/view](https://en.wikipedia.org/wiki/Chest_radiograph#Views) (antero-posterior or postero-anterior).

The target column was already coded:
    - 0 - normal.
    - 1 - pneumonia.

The type column values were strings, so two strategies were carried out:
    - Numerical coding:
        - 0 - Normal
        - 1 - Not normal/Not pneumonia
        - 2 - Neumonia
    - Dummy creation:
        - Just in case the information had to be fed to the algorithm in that format.

As the dataset initial purpose was a challenge, the test subset is obviously not labeled. So the first thing that was carried out was creating three stratified subsets from the train folder, which was already labeled:
  - Training subset (80%)
  - Validation subset (10%)
  - Test subset (10%)

That process was done twice, stratifying both by [target (binary)](https://github.com/EiroaMD/final_project_IH_pneumonia/blob/master/2%20Image%20Management/2_file_classifier_t_v_t.ipynb) and [class (three classes)](https://github.com/EiroaMD/final_project_IH_pneumonia/blob/master/2%20Image%20Management/2_file_classifier_t_v_t_3_classes.ipynb). 

Also, after some setbacks described below, a third notebook doing the same process after [balancing](https://github.com/EiroaMD/final_project_IH_pneumonia/blob/master/2%20Image%20Management/2_file_classifier_t_v_t_balanced.ipynb) data was created.  Next, the images were copied into train/validation/test folders by creating as many folders as classes.

**In brief**, we ended this step with a clean dataframe shaped (26684, 12) and 26884 images divided 80-10-10% between train, validation, and testing, each folder containing either two (normal|pneumonia) or three (normal|not-normal/not-pneumonia|pneumonia) subfolders.

### **Data Storage:**
Throughout the process, a few `.csv`files were generated, to keep record of the files belonging to the different subsets. They are stored in the [`/data`](https://github.com/EiroaMD/final_project_IH_pneumonia/tree/master/data/csv) folder of the github repository.

The heft of the files was stored on a Google Cloud virtual machine with GPUs, where the  model training was carried out.

<a href="https://www.codecogs.com/eqnedit.php?latex=\large&space;H0:&space;\bar{x}&space;^{normal}&space;=&space;\bar{x}&space;^{pneumonia}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\large&space;H0:&space;\bar{x}&space;^{normal}&space;=&space;\bar{x}&space;^{pneumonia}" title="\large H0: \bar{x} ^{normal} = \bar{x} ^{pneumonia}" /></a>

<a href="https://www.codecogs.com/eqnedit.php?latex=\large&space;H1:&space;\bar{x}&space;^{normal}&space;\neq&space;\bar{x}&space;^{pneumonia}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\large&space;H1:&space;\bar{x}&space;^{normal}&space;\neq&space;\bar{x}&space;^{pneumonia}" title="\large H1: \bar{x} ^{normal} \neq \bar{x} ^{pneumonia}" /></a>

<a href="https://www.codecogs.com/eqnedit.php?latex=\large&space;H0:&space;\bar{x}&space;^{normal}&space;=&space;\bar{x}&space;^{nnnp}&space;=&space;\bar{x}&space;^{pneumonia}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\large&space;H0:&space;\bar{x}&space;^{normal}&space;=&space;\bar{x}&space;^{nnnp}&space;=&space;\bar{x}&space;^{pneumonia}" title="\large H0: \bar{x} ^{normal} = \bar{x} ^{nnnp} = \bar{x} ^{pneumonia}" /></a>

<a href="https://www.codecogs.com/eqnedit.php?latex=\large&space;H1:&space;Means\hspace{2mm}&space;are\hspace{2mm}&space;not\hspace{2mm}&space;all\hspace{2mm}&space;equal." target="_blank"><img src="https://latex.codecogs.com/gif.latex?\large&space;H1:&space;Means\hspace{2mm}&space;are\hspace{2mm}&space;not\hspace{2mm}&space;all\hspace{2mm}&space;equal." title="\large H1: Means\hspace{2mm} are\hspace{2mm} not\hspace{2mm} all\hspace{2mm} equal." /></a>

## Data Analysis
Far far away, behind the word mountains, far from the countries Vokalia and Consonantia, there live the blind texts. Separated they live in Bookmarksgrove right at the coast of the Semantics, a large language ocean. A small river named Duden flows by their place and supplies it with the necessary regelialia. It is a paradisematic country, in which roasted parts of sentences fly into your mouth. Even the all-powerful Pointing has no control about the blind texts it is an almost unorthographic life One day however a small line of blind text by the name of Lorem Ipsum decided to
## Conclusions
Far far away, behind the word mountains, far from the countries Vokalia and Consonantia, there live the blind texts. Separated they live in Bookmarksgrove right at the coast of the Semantics, a large language ocean. A small river named Duden flows by their place and supplies it with the necessary regelialia. It is a paradisematic country, in which roasted parts of sentences fly into your mouth. Even the all-powerful Pointing has no control about the blind texts it is an almost unorthographic life One day however a small line of blind text by the name of Lorem Ipsum decided to
## Problems - Setbacks
Far far away, behind the word mountains, far from the countries Vokalia and Consonantia, there live the blind texts. Separated they live in Bookmarksgrove right at the coast of the Semantics, a large language ocean. A small river named Duden flows by their place and supplies it with the necessary regelialia. It is a paradisematic country, in which roasted parts of sentences fly into your mouth. Even the all-powerful Pointing has no control about the blind texts it is an almost unorthographic life One day however a small line of blind text by the name of Lorem Ipsum decided to
## References
