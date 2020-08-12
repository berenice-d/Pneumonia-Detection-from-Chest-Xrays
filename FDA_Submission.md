# FDA  Submission

**Your Name:** Berenice Dethier

**Name of your Device:** pneuMODEL

## Algorithm Description 

### 1. General Information

**Intended Use Statement:** 
Assisting the radiologist in the detection of pneumonia from chest X-ray

**Indications for Use:**  
This algorithm is intended for use on men and women from the ages of 2-90 who have been administered a chest XRay (PA or AP view). The algorithm should be used for screening of chest X-rays from patients with suspected Pneumonia. After inference, the image and result should be assessed by a radiologist.

**Device Limitations:** 
Not recommended for patients who also have Edema, Effusion, Hernia, and/or Inflitration, as these comorbidities have a similar pixel profile. 

**Clinical Impact of Performance:** 
This algorithm was created with a threshold favoring a high recall, meaning that the number of false negatives is very low. The algorithm is not likely to mislabel a patient who is actually sick. On the other hand, there are a lot of false positives: the algorithm is likely to flag patients who don't have the coniditon as sick. The impact of false positive is limited, since a radiologist is still looking at the X-ray to establish the diagnostic.


### 2. Algorithm Design and Function

![Flowchart](https://github.com/berenice-d/Pneumonia-Detection-from-Chest-Xrays/blob/master/fig/flow.png)

**DICOM Checking Steps:**
Confirm:
*Patient is between ages 2-90
*Patient is Male or Female
*X-ray is of bodypart "chest"
*Patient position is 'PA' (Posterior/Anterior) or 'AP' (Anterior/Posterior)
*Modality is "DX" (Digital X-ray)

**Preprocessing Steps:**
The algorithm performs the following preprocessing steps on an image data:
* Converts RGB to Grayscale (if needed)
* Resizes the image to 244 x 244 (input shape of the CNN)
* Normalizes the intensity (between 0 and 1 from original range of 0-255)

**CNN Architecture:**
CNN based on VGG16, with additional fully-connected layers in the end:
![CNN](https://github.com/berenice-d/Pneumonia-Detection-from-Chest-Xrays/blob/master/fig/CNN.png)


### 3. Algorithm Training

**Parameters:**
* Types of augmentation used during training: 
    * Horizontal flip, but no vertical flip
    * Height shift range of 0.1
    * Width shift range of 0.1
    * Rotation range of 20 
    * Shear range of 0.1 
    * Zoom range of 0.1
    
* Batch size:
    * Train batch size: 32
    * Validation batch size: one large batch of 1024
    
* Optimizer learning rate: 0.0001

* Layers of pre-existing architecture that were frozen: layers 0 to 16 (incl.) of VGG16 network

* Layers of pre-existing architecture that were fine-tuned: last 2 layers of VGG16 network (block5_conv3 and block5_pool)

* Layers added to pre-existing architecture:
    * Flatten
    * Dropout 0.5
    * Dense layer, 1024 nodes, relu activation
    * Dropout 0.5
    * Dense layer, 512 nodes, relu activation
    * Dropout 0.5
    * Dense layer, 256 nodes, relu activation
    * Dropout 0.5
    * Dense layer, 128 nodes, relu activation
    * Dense layer, 1 node, sigmoid activation 


Training history: the model of epoch 25 was saved.
![loss](https://github.com/berenice-d/Pneumonia-Detection-from-Chest-Xrays/blob/master/fig/history_loss.png)

![accuracy](https://github.com/berenice-d/Pneumonia-Detection-from-Chest-Xrays/blob/master/fig/history_acc.png)

Curves:
![loss](https://github.com/berenice-d/Pneumonia-Detection-from-Chest-Xrays/blob/master/fig/ROC.png)

![accuracy](https://github.com/berenice-d/Pneumonia-Detection-from-Chest-Xrays/blob/master/fig/PR.png)

All metrics:
![metrics](https://github.com/berenice-d/Pneumonia-Detection-from-Chest-Xrays/blob/master/fig/all_indic.png)

**Final Threshold and Explanation:**

The model does not have a high precision with any meaningful recall value, which is why we favor a high recall (to the detriment of precision). 
The threshold is set at 0.36, to ensure a recall above 0.8.


Alternatively, we could have used the threshold at the highest F1 score. The maximum F1 score for the model is 0.427 and it is achieved with threshold value of 0.4.
Below is the comparison of F1 score with those given in 
[CheXNet: Radiologist-Level Pneumonia Detection on Chest X-Rayswith Deep Learning](
https://arxiv.org/pdf/1711.05225.pdf):

| Person or Device | F1    | 
|------------------|-------|
| Radiologist 1    | 0.383 |
| Radiologist 2    | 0.356 |
| Radiologist 3    | 0.365 |
| Radiologist 4    | 0.442 |
| Radiologist Avg. | 0.387 |
| CheXNet          | 0.435 |
| My Model         | 0.427 |


Comparing the F1 scores themselves, this model achieves higher 
maximum F1 score than an average radiologist 
in the study. State of the art neural network, as well as one radiologist from the study,
do achieve higher F1 score, but the model's performance is comparable and in many cases 
exceeds the performance of human radiologists (in terms of F1 score).

If we compare the highest F1 to the high recall performances, we obtain:
| Threshold          | F1    | Precision   | Recall |  
|--------------------|-------|-------------|--------|
| 0.4    Max F1      | 0.427 |  0.304      |  0.720 |
| 0.36   Recall>0.8  | 0.420 |  0.283      |  0.815 |


From the confusion matrix, we also know that if the model predicts negative, it is correct 91.3% of the time. If the model 
predicts positive, it is correct 28.3% of the time.
![confusion_matrix](https://github.com/berenice-d/Pneumonia-Detection-from-Chest-Xrays/blob/master/fig/conf.png)


### 4. Databases

**Description of Training Dataset:** 
Training dataset consisted of 2290 chest X-ray images, with a 50/50 split between positive and negative cases. Patient ages ranged between 2-90 years old. Fifty-seven percent (57%) of patients were male and 43% were female. The images were taken in PA and AP positions with roughly 50% of images in each position.
In the population, the three most prevalent conditions were Infiltration (50% of the patients with a condition), Effusion (45%) and Atelectasis (35%). Among patients with Pneumonia, the most prevalent comorbidities were Infiltration (42% of Pneumonia patients), Edema (24%), Effusion (19%) and Atelectasis (18%). Pneumonia diagnostic was slightly correlated with Edema.

Example images:
![example](https://github.com/berenice-d/Pneumonia-Detection-from-Chest-Xrays/blob/master/fig/example_img.png)

**Description of Validation Dataset:** 
Validation dataset consisted of 1430 chest X-ray images, with a 20/80 split between positive and negative cases to reflect the actual prevalence of pneumonia. The population distribution was the same as for the training set.


### 5. Ground Truth
The dataset was curated by the NIH to address the problem of a lack of large x-ray datasets with ground truth labels to be used in the creation of disease detection algorithms. 

There are 112,120 X-ray images with disease labels from 30,805 unique patients in this dataset.  The disease labels were created using Natural Language Processing (NLP) to mine the associated radiological reports. The labels include 14 common thoracic pathologies: 
- Atelectasis 
- Consolidation
- Infiltration
- Pneumothorax
- Edema
- Emphysema
- Fibrosis
- Effusion
- Pneumonia
- Pleural thickening
- Cardiomegaly
- Nodule
- Mass
- Hernia 

The biggest limitation of this dataset is that image labels were NLP-extracted so there could be some erroneous labels but the NLP labeling accuracy is estimated to be >90%.

The original radiology reports are not publicly available but you can find more details on the labeling process [here.](https://arxiv.org/abs/1705.02315) 


### 6. FDA Validation Plan

**Patient Population Description for FDA Validation Dataset:**
Age range: 2-90 years old
Sex: Male and female
Type of imaging modality: DX (Digital X-rays)
Body part imaged: Chest
Prevalence of disease of interest: 20%
Other diseases that should be excluded as comorbidities in the population: Edema, Effusion, Hernia, and Inflitration

**Ground Truth Acquisition Methodology:**
The golden standard for obtaining ground truth would be to perform one of these tests:
* Sputum test
* Pleural fluid culture

The tests are quite expensive, and in real life diagnosis is usually established by a physician based on radiologist's analysis/description.
The device intends to assist radiologists, not replace them, therefore the ground truth for the FDA Validation Dataset can be obtained by the silver standard: an average of labels by three radiologists.
The same method is used in the [CheXNet paper](https://arxiv.org/pdf/1711.05225.pdf).  

**Algorithm Performance Standard:**
In terms of Clinical performance, the algorithm's performance can be measured by calculating F1 score against 'silver standard' ground truth as described above. 
The algorithm's F1 score should exceed **0.387** which is an average F1 score taken over
three human radiologists, as given in [CheXNet: Radiologist-Level Pneumonia Detection on Chest X-Rayswith Deep Learning](
https://arxiv.org/pdf/1711.05225.pdf), where a similar method is used
to compare device's F1 score to average F1 score over three radiologists.

A 95% confidence interval given in the paper for average F1 score of 0.387 
is (0.330, 0.442), so algorithm's 2.5% and 97.5% precentiles should 
also be calculated to get 95% confidence interval. This interval, 
when subtracted the interval above for the average, should not contain 0,
which will indicate statistical significance of its improvement 
of the average F1 score. The same method for assessing statistical 
significance is presented in the above paper. 
