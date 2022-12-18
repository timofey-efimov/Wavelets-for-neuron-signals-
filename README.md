# Wavelets-for-neuron-signals-

<!-- ABOUT THE PROJECT -->
## About The Project

This repository contains implementation of the algorithm, described in “Wavelet transform for real-time detection of action potentials in neural signals.” by Quotb et al (2011), available at the following link: [Paper Link](https://www.frontiersin.org/articles/10.3389/fneng.2011.00007/full). The premise of the paper is to use wavelet detection methods to detect the action potential in the neural data. The reason to use wavelets instead of the normal thresholding is higher accuracy and stability of results as the noise increases; however, it might come at computational cost. Wavelet transform provides a balance between those two, and the paper suggests an overview of various techniques. Due to complexity and volume of the computational aspect of the project done by Quotb et al., we decided to replicate the major results– Figure 7 and Figure 8, which plots the correct spike detection percent for SNRs between 0 and 10 dB using wavelet detection with detail levels 1 through 4. This figure shows how detection accuracy changes with respect to SNR, and confirms the stability of the system in comparison with the more commonly-used thresholding.


<!-- * [![tch][TCH]][TCH-url] -->
 <img src="https://i.ibb.co/TT1X1hz/Neural-Signal-Processing.jpg" alt="drawing" height="300"/>

 

### Data Description

The paper used an entirely synthetic dataset, to which they added Gaussian noise. However, we had raw neural data from rats' brains provided by our professor Shaoyk Dutta from Rice University, which we then filtered out using smoothing and thresholding. We used the data from one of the tetrodes, not all of them, because using all of them would have extremely high computational cost and we could proof the effectiveness of our implementation with much less data as it is not a machine learning model and in fact the detection algorithm is real-time, it does not need the entire dataset to work efficiently. 

More specifically, the raw data we used is spikes_ep2 from tetrode12.mat. The shape of this data is 22730202 by 4, because there are 4 channels and the number of waveform samples is 22730202. 
    

### Data Preprocessing 

The raw neural data from the tetrode is noisy because it was obtained in a real-world setting. The first thing is to filter it using the 4-th order Butterworth bandpass filter. The sampling frequency for the raw neural data signal is 30kHz, so the critical frequency for the Butterworth filter ( where it drops to -3db) is therefore at 0.04. 

This is how the raw data looks and the the data filtered with low-pass filter:

 <img src="https://i.ibb.co/4jDW6fk/raw.png" alt="drawing" width="600"/>
 
 <img src="https://i.ibb.co/9bfyHQ9/filtered.png" alt="drawing" width="600"/>

As can be seen from the plots above, the 4-th order Butterworth filter smoothes the raw signal a lot and reduces the amplitude significantly. If we did not filter and left the threshold at 60 uV, then the scheme for ground truth would detect a lot of “fake” action potentials.

Later we obtain the ground-truth for the wavelet-based model using thresholding. Given the sampling frequency of 30kHz and the duration of action potential of around 1 ms, the reasonable window for the action potential is 30-40 samples; however, we did include 40 to have a “safe zone” around the actual potential due to some noise. From the spike itself, we take 10 samples to the left and 30 samples to the right, and the spike is thresholded at 60.

A sample action potential is visualized below:

 <img src="https://i.ibb.co/2vmQvZq/SampleAP.png" alt="drawing" width="600"/>

Below you can see the demonstration of the performance of the thresholding. The idea is that we detect an action potential if it exceeds the threshold in at least one of four channels.

 <img src="https://i.ibb.co/6wVt5Nj/thresholding.png" alt="drawing" width="600"/>

From the plot, you can see how different channels are in fact very similar, which confirms the hypothesis that it is reasonable to detect action potential in all channels once it is detected in one of them. 


## Project Authors: 

Alexa Thomases,Timofey Efimov

### Built With

<!-- * [![py][Python]][Python-url]
* [![sp][Spacy]][Spacy-url] -->

 ![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54)


<!-- SOFTWARE INSTALLATION -->
### Getting Started

## Prerequisites: wavelets and filtering

Specifically, in the section of the paper we replicated, the authors used a digital implementation method called the Stationary Wavelet Transform (SWT). The SWT is similar to the better-known Discrete Wavelet Transform (DWT); however, unlike the DWT, the SWT does not downsample the original signal. The SWT can be understood as a signal decomposition.The scheme for the decomposition is the following(taken from the aforementioned paper):

 <img src="https://i.ibb.co/r3bgWR7/a1.png" alt="drawing" width="600"/>

Given some level K, the SWT will provide the approximation and detail coefficients for all levels after that, because the wavelet is “recursive” in this sense as can be seen from the figure above. It is impossible to calculate K-th approximation or detailed coefficients without finding K-1 th one and down to level 0.

In order to remain consistent with the paper, we chose the Haar signal as our mother wavelet. Then, we implemented the SWT signal processing steps in accordance with Figures 3 and 4 of the paper in order to detect action potentials. We evaluated our correct detection rate for SNRs between 0 and 10dB and for SWT detail levels 1 through 4. Our results, which are summarized in our figure below, are a replication of Figure 8 from the paper. 


## Figure 7&8 Replication

In order to replicate the figure, we had to follow the procedure outlined in the paper. On the higher level, the procedure looks the following:

 <img src="https://i.ibb.co/dGGg5RQ/Wavelet-detection-module.png" alt="drawing" width="600"/>

For the sake of reproducing paper results, we used levels from 1 up to 4, and for each of them applied the algorithm in the above Figure. The idea is that we take wavelet detail coefficients for level 1 and level K (for K from 1 to 4). Then the absolute value is taken for the entire array of numbers (length of wavelet detail coefficient array equals the length of the input array). Then we go into the upper loop, where the first thing we do is to compare the values in the array to the current estimation of the standard deviation. The first estimation of the standard deviation for Gaussian noise equals  = median(filtered neural signal)0.6795, and the value is taken from the paper, where this standard deviation is defined for the offline case. The paper had many parts which could be interpreted in multiple ways, and the initial standard deviation. There also have been certain misinterpretations with how exactly the algorithm works after this point. For example, the paper said that the low-pass filter F1 should have gain of G1; however, the figure suggests that the gain G1 is applied after P is subtracted from the output of the low-pass filter. Our final solution implemented the first case, and repeated the upper loop for 100 times. Low-pass filter is a 4-th order Butterworth filter with a cut-off frequency of 10 Hz, and sampling rate of 10 kHz. Both lowpass filters F1 and F2 have the same structure. One of the observations is that as we increased the number of iterations, it did not change a lot, so it converged relatively fast.

Once we have obtained the estimation for the standard deviation, we again lowpass the signal and amplify it by G2, which is a tunable parameter. Then the estimation for the threshold is compared to the wavelet detail coefficient representation of the signal. 

In short, the idea that for each SNR we have the same threshold because threshold is based the Level-1 wavelet detail coefficients, and the results across level differ because the higher the level is, the better it is( it amplifies the action potential peaks better while smoothing out the rest of the signal). 

For example, in the figures below you can see the action potentials, the part of the signal in its original form, its level-1 wavelet detail coefficient representation, and its levels 2 to 4 wavelet detail coefficient representation. 


 <font size="5s"> Figure 7 results reproduced: </font>
 <br>
 <img src="https://i.ibb.co/N6z0pQF/six-pictures.png" alt="drawing" width="600"/>


 <font size="5s"> Figure 7 results original: </font>
 <br>
  <img src="https://i.ibb.co/xqfLd1z/figure7-result.png" alt="drawing" width="600"/>

From the values on the vertical axis and general waveform you can see that the higher level detail coefficient amplifies the peaks and attenuates the rest of the signal, making action potentials easier to detect. You can also see how higher level representations give better estimation of the signal and look cleaner, which make sense for why they provide higher accuracy.

The figure we created, which replicates Figure 8 using our own data and results, is below alongside the reference figure: 

  <img src="https://i.ibb.co/VqQhfHG/best-result-307.png" alt="drawing" height="300"/>  <img src="https://i.ibb.co/ssrN96Q/expected-figure.png" alt="drawing" height="300"/>


<!-- ACKNOWLEDGMENTS -->
## Acknowledgments

* The authors of the paper “Wavelet transform for real-time detection of action potentials in neural signals.” by Quotb et al (2011), available at the following link: [Paper Link](https://www.frontiersin.org/articles/10.3389/fneng.2011.00007/full).

