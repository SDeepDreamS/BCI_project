# BCI_project
## Dataset

As we know, some sleep disorders, like obstructive sleep apnea (OSA), can lead to severe consequences. Our goal is to use EEG data to classify different apnea, and we are the first one who tries to use Nationwide Children’s Hospital (NCH) Sleep DatabBank to focus on this topic.

NCH Sleep DatabBank was built and published at Physionet and the National Sleep Research Resource(NSRR) to accelerate research on pediatric sleep and its connection field. This dataset consists of 3984 polysomnography studies and over 5.6 million clinical observations on 3,673 unique patients between 2017 and 2019 at NCH, which is a valuable resource for BCI investigations and advancing sleep studies.   

To further explain this dataset, the total length of the recording in the NCH Sleep DataBank amounts to 40,884 hours where the minimum length of study is 3 minutes, the maximum is 16.5 hours, and the mean is 10.3 hours. Majority of the recordings are collected with a sampling frequency of 256Hz, some of the studies were sampled at 400Hz and 512Hz.

This dataset also has some annotations, as the original paper described, each annotation has information including onset, duration, and description. The description of the event may be sleep stage labels or standard sleep event labels. Sleep stages are found in annotations with a duration of 30 seconds, where the descriptions include ‘Sleep stage W’, ‘Sleep stage N1’, ‘Sleep stage N2’, ‘Sleep stage N3’, or ‘Sleep stage R’. Besides sleep stage labels, some common events include: Obstructive Hypopnea, Obstructive Apnea, Hypopnea, Central Apnea, and Mixed Apnea. Our target is to detect these different apneas.

We access the dataset through [Physionet](https://physionet.org/) which contains enormous medical datasets. The [sleep dataset](https://physionet.org/content/nch-sleep/3.1.0/) is only avaliable for credential users who need to complete lessons regarding medical ethics. We have completed these lessons so that we can download the dataset. 

## Requirements 

The code has been tested with Tensorflow 2.8.2 and Python 3.7. To analyze EEG data, we need to install mne package to convert raw EEG data format to numpy format, Numpy for FFT, Sklearn package to perform ICA and random forest model, and Meegkit package for ASR.

## Related works

Before digging into this project, we do some paper surveys to follow up on the newest study in this field. Since there are no other researchers discussing on this topic, we don't compare our results with these papers, just focusing on the methods they use.

* **High-performance Diagnosis of Sleep Disorders: A Novel, Accurate and Fast Machine Learning Approach Using Electroencephalographic Data**

   This paper focus on Rapid Eye Movement sleep behavior disorders (RBD). The dataset was collected at the Sleep Disorders Center of the Ospedale Maggiore of Parma, Italy. It includes 22 patients with Rapid Eye Movement sleep behavior disorder and 16 healthy subjects. Before using machine learning methods to analyze EEG data, they use ICA as BSS method to filter out redundant noise.
    Later, they use FFT to get the power spectrum and use random forest to classify their data. The author mentioned that the reason they choose to use random forest is because other machine learning methods, like support vector machines, neural networks or k-nearest neighbors offer no indications about the most important predictors.


* **Machine Learning based Diagnosis of Diseases Using the Unfolded EEG Spectra: Towards an Intelligent Software Sensor**

    This paper presents the idea to unfold the EEG standard bandwidths in a more fine-graded equidistant 99-point spectrum to improve accuracy when diagnosing diseases. What's more, they also replace the hard-coded equidistant 99-point spectrum with a flexibly-grading spectrum to further improve the results. Although the machine learning method they use is still random forest, this novel pre-processing step enhances the accuracy.
<img src="https://i.imgur.com/gWrRJjf.png" height="300px" width="480px" />

* **Comparison of Motor Imagery EEG Classification using Feedforward and Convolutional Neural Network**

    In this paper, they used several data pre-processing methods and examined how they affect the classification performance of a feedforward neural network. After the preprocessing method, they train a convolutional neural network (CNN) to classify their data. 
    
## Preprocessing

### Remove eye-related artifacts
First, we use EOG signal to identify eye blink or movement artifacts.

<img src="https://i.imgur.com/4r0PA85.png" height="300px" width="480px" />

As we can see, eye-related artifacts happened around 2500 seconds. Based on the knowledge we learned from other papers and the class, we choose to use ICA to decompose the raw EEG signals here. ICA is a method for finding underlying factors from multivariate statistical data. Here we use it to decompose the artifact-related components, and then reconstruct the EEG signals with artifact-free components.

* Raw EEG signals

<img src="https://i.imgur.com/tPWwuqn.png" height="300px" width="480px" />

* Reconstructed EEG signals

<img src="https://i.imgur.com/tNmtoq2.png" height="300px" width="480px" />

### Artifact Subspace Reconstruction (ASR)

ASR is an adaptive method for the online or offline correction of artifacts comprising multichannel EEG recordings. We then use ASR to reconstruct our data. We used the first three seconds of the EEG wave as the resting state data. Then remove the artifacts in following 27 seconds. 
    
* Raw EEG wave

<img src="https://i.imgur.com/PmkjntV.png" height="300px" width="480px" />

* EEG wave applied ASR

<img src="https://i.imgur.com/cvaSbDu.png" height="300px" width="480px" />

### Fast Fourier transform (FFT)
FFT is an algorithm that calculates the discrete Fourier transform (DFT) of some sequence, and it can be used to transform the structure of the cycle of a waveform into sine components.

We use FFT to transform our data in this project. First, we concatenate 7 artifact free channels, and then apply fast fourier transformation to get the frequency domain data.

<img src="https://i.imgur.com/u7ykXRW.png" height="300px" width="480px" />

### Statistical features of EEG wave

Use wavedec function provided by pywt library to decode the coefficients of the EEG waves, and then get the statistical features such as mean, standard deviation, max, and min.

## Architecture (Methodology)

### Random Forest

Random forest is a Supervised Machine Learning Algorithm that is used widely in Classification and Regression problems. It builds decision trees on different samples and takes their majority vote for classification and average in case of regression.

## Validation methods

We use Area under the curve (AUC) as our evaluation metric. It is one of the most important evaluation metrics for checking any classification model’s performance. The higher the AUC, the better the model can predict 0 classes as 0 and 1 classes as 1.

## Results

* Overview



| Setting                                 | Label_0 AUC | Label_1 AUC | Label_2 AUC | Label_3 AUC | Label_4 AUC | Mean AUC |
| --------------------------------------- | ----------- | ----------- | ----------- | ----------- | ----------- | -------- |
| Raw EEG                                 | 0.613       | 0.644       | 0.581       | 0.593       | 0.564       | 0.599    |
| Raw EEG + FFT                           | 0.678       | 0.730       | 0.637       | 0.693       | 0.646       | 0.677    |
| ICA-processed EEG                       | 0.603       | 0.640       | 0.553       | 0.582       | 0.541       | 0.584    |
| ICA-processed EEG + FFT                 | 0.648       | 0.703       | 0.591       | 0.649       | 0.620       | 0.642    |
| ASR-processed EEG                       | 0.588       | 0.608       | 0.550       | 0.579       |   0.531       | 0.571 
| ASR-processed EEG + FFT                 | 0.655       | 0.701       | 0.602       | 0.662       |   0.631       | 0.650 
| Statistical features from raw EEG       | 0.827       | 0.882       | 0.803       | 0.819       | 0.724       | 0.811    |
| Statistical features from ICA-processed EEG | 0.736       | 0.794       | 0.694       | 0.725       | 0.645       | 0.719    |
| Statistical features from ASR-processed EEG | 0.748       | 0.809       | 0.717       | 0.740       | 0.666       | 0.772    |



* Visualization

   * Raw EEG wave as input
 
 <img src="https://i.imgur.com/QYOTR1i.png" height="300px" width="480px" />

   * Raw EEG wave with FFT as input 
 
 <img src="https://i.imgur.com/1vSAtAN.png" height="300px" width="480px" />

   * ICA-processed EEG wave as input 
 
 <img src="https://i.imgur.com/rYbHpqa.png" height="300px" width="480px" />

   * ICA-processed EEG wave with FFT as input
 
 <img src="https://i.imgur.com/E309f8v.png" height="300px" width="480px" />
    
   * ASR-processed EEG wave as input
 
 <img src="https://i.imgur.com/i0ojjUX.png" height="300px" width="480px" />

   * ASR-processed EEG wave with FFT as input
 
 <img src="https://i.imgur.com/LSEVyVg.png" height="300px" width="480px" />

   * Statistical features from raw EEG wave 
 
 <img src="https://i.imgur.com/Im6BMKL.png" height="300px" width="480px" />

   * Statistical features from ICA-processed EEG wave
 
 <img src="https://i.imgur.com/viu8Aqi.png" height="300px" width="480px" />
    
   * Statistical features from ASR-processed EEG wave
 
 <img src="https://i.imgur.com/aPmkTO3.png" height="300px" width="480px" />


## Discussion & Limitation

From the results, we can find that actually, the ICA-processed EEG wave can't outperform the original EEG wave. After we used ICA to identify and remove the eye-related artifact, the model can't predict better. Based on this result, we conjecture that for the sleep study, the eye movements are important features for the apnea classification. This scenario is not like the original BCI experiments, our subjects won't have a "wink" action because they are sleeping. Removing the eye-related information may lose some important features for distinguishing the sleep states, so it is better to keep the eye-related artifacts during the training stage. Similar scenario happended to the applicaiton of ASR. The ASR-processed wave perform worse than the original wave but better than the ICA-processed wave. Since the ASR techniques rely on the resting state data, we were not sure that the first three second is stable and enough for the algorithm. Hence, ASR may also remove some important features accidentlly. FFT and statistical process could enhance the performance. We consider that converting the time-domain data to frequency-domain data could let the model capture the features easier. The statistical features lower the dimension of the data and accentuate the features of each type of apnea.

Besides, we tried to use CNN to build our model. However, the results were not as expected. We guess that it is because our data quantity doesn't enough to build a CNN model, and it is also one of the limitations of our dataset.

## Acknowledgements

NCH Sleep DataBank was supported by the National Institute of Biomedical Imaging and Bioengineering of the National Institutes of Health under Award Number R01EB025018. The National Sleep Research Resource was supported by the U.S. National Institutes of Health, National Heart Lung and Blood Institute (R24 HL114473, 75N92019R002).
