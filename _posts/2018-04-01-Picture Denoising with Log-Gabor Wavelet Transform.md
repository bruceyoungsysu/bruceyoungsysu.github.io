---
title: Picture Denoising with Log-Gabor Wavelet Transform
date: 2018-04-01 16:34:09
categories:
- Python
- Image Processing
---

### Concepts
Wavelet transform is frequently used in picture denosing as it can cover broder length in frequency domain. Wavelet transforms are based on small wavelets with limited duration. The translated-version
wavelets locate where we concern. Whereas the scaled-version wavelets allow us to analyze the signal in different scale.

The log-Gabor filter is an advancement of Gabor filter. it better fits the statistics of natural images such as edges and corners. These features can be easily used as indicators in pattern recongnition. 

Definition of a 1D Gabor filter (from [wikipedia](https://en.wikipedia.org/wiki/Log_Gabor_filter)):

![](https://wikimedia.org/api/rest_v1/media/math/render/svg/f1aa5bade456f80d1adfccd89acd28e230e99180)

___
### Algorithm Implementation

```graphTB
    A[Input Array] --FFT--> B(Freq Array)
    B --> C((multiply))
    D> For each freq scale--] --> E(Freq filter)
    E --> C
    F> For each orientation sampling] --> G(Orientation filter)
    G --> C
    C -->|iFFT| A1((add))
    B1(total energy) --> A1
    A1 --> B1
    B1 --real part--> C1[clean image]
```
![](https://github.com/bruceyoungsysu/bruceyoungsysu.github.io/blob/master/_posts/log_gabor/Log_gabor_wavelet_algo.PNG?raw=true)

___
### Outputs

- 1D Gaussian noise filtering:
![](https://github.com/bruceyoungsysu/bruceyoungsysu.github.io/blob/master/_posts/log_gabor/1D_log_gabor_test.PNG?raw=true)

- 2D SEM image denosing:
![](#https://github.com/bruceyoungsysu/bruceyoungsysu.github.io/blob/master/_posts/log_gabor/2d_log_gabor_test.PNG?raw=true)

___
### Code
Python version of 2D log-Gabor filter:
- Libraries
~~~python
import math
import numpy as np
from PIL import Image
import matplotlib.pyplot as plt
~~~
- Function definition
~~~python
def denoise_pp_2d(arr, k, nscale, mult, norient, softness):
    """
    Denoising pictures with log-Garbor wavelet transformation.
    ------
    Attributes:
        arr: numpy array like. The input signal need to denoise.
        k: Standard deviation of noise to reject. Affect threshold.
        nscale: Scale of wavelet. Affect frequency coverd by wavelets.
        mult: Affect threshold.
        norient: number of angle sampling of log-Gabor filter. Bigger norient brings higher accuracy with longer run time.
        softness: Method to calculate threshold. 0 - hard threshold, 1 - soft threshold.
    ------
    Returns:
        cleanimaghe: numpy array. Denoised signal
    """
    
    'installation parameters'
    minWaveLength = 2.  # wavelength. Affects wavelet scale in freq domain
    sigmaOnf = 0.55     # bandwith of frequency
    dThetaOnSigma = 1.  #
    epsilon = .00001

    # get the radius of theta of each pixel. For each pixel the filter varies.
    thetaSigma = math.pi/norient/dThetaOnSigma  # bandwith of angular
    imagefft = np.fft.fft2(arr)
    (rows, cols) = np.shape(imagefft)

    x = np.dot(np.reshape(np.ones(rows), (rows, 1)),
               np.reshape(np.array([x for x in range(round(-cols / 2), round(cols / 2)+(cols%2)*1)]) / (cols / 2), (1, cols)))
    y = np.dot(np.reshape(np.array([x for x in range(round(-rows / 2), round(rows / 2)+(rows%2*1))]) / (rows / 2), (rows, 1)),
               np.reshape(np.ones(cols), (1, cols)))

    radius = np.sqrt(x ** 2 + y ** 2)
    radius[round(rows/2), round(cols/2)] = 1
    theta = np.arctan2(-y, x)
    total_energy = np.zeros((rows, cols))

    sig = []
    asig = []
    aMean = []
    estMeanEO = []

    for o in range(1, norient+1):

        # get the filter for each orientation sampling
        print("processing orientation"+str(o))
        angl = (o - 1) * math.pi / norient
        wavelength = minWaveLength
        ds = np.sin(theta) * math.cos(angl) - np.cos(theta) * math.sin(angl)  # projecttion of angl to theta
        dc = np.cos(theta) * math.cos(angl) + np.sin(theta) * math.sin(angl)  # projecttion of angl to theta+pi/2
        dtheta = abs(np.arctan2(ds, dc))  # theta-angl
        spread = np.exp((-dtheta ** 2) / (2 * thetaSigma ** 2))  # exponential

        for i in range(1, nscale+1):

            # for each filter scale, scale is applied to threshold T
            freq = 1./wavelength
            freqLen = freq / 0.5

            # labor filter matrix at each pixel, freq domain should be similar to time domain
            logGabor = np.exp((-(np.log(radius / freqLen)) ** 2) / (2 * math.log(sigmaOnf) ** 2))
            logGabor[round(rows / 2), round(cols / 2)] = 0  # set the center value of filer
            fil = logGabor * spread  # log-gabor filter of each orientation
            fil = np.fft.fftshift(fil)  # shift 0 freq to corner
            EOfft = imagefft * fil  # apply the log-Gabor filter
            EO = np.fft.ifft2(EOfft)
            aEO = abs(EO)

            if i == 1:
                # Estimate the Rayleigh distribution of noise, get s == 1 scale threshold

                medianEO = np.median(aEO) # get the median value
                meanEO = medianEO * 0.5 * math.sqrt(-math.pi / math.log(0.5, math.e))  # get the mean value

                RayVar = (4 - math.pi) * (meanEO ** 2) / math.pi
                RayMean = meanEO

                estMeanEO = np.dot(estMeanEO, meanEO)
                sig = np.dot(sig, math.sqrt(RayVar))

            T = (RayMean + k * math.sqrt(RayVar)) / (mult ** (i - 1))  # threshold
            validEO = aEO > T
            V = softness * T * EO / (aEO + epsilon)
            V = ~validEO * EO + validEO * V  # noise part
            # V = 0
            EO = EO - V  # signal part
            total_energy = total_energy + EO
            wavelength = wavelength * mult  # the next wavelet

    cleanimage = np.real(total_energy)
    return cleanimage
~~~
