---
title: Picture Denoising with Log-Gabor Wavelet Transform
date: 2018-04-01 16:34:09
categories:
- Python
- Image Processing
---

### Concepts:
Wavelet transform is frequently used in picture denosing as it can cover broder length in frequency domain. Wavelet transforms are based on small wavelets with limited duration. The translated-version
wavelets locate where we concern. Whereas the scaled-version wavelets allow us to analyze the signal in different scale.

The log-Gabor filter is an advancement of Gabor filter. it better fits the statistics of natural images such as edges and corners. These features can be easily used as indicators in pattern recongnition. 

Definition of a 1D Gabor filter (from [wikipedia](https://en.wikipedia.org/wiki/Log_Gabor_filter)):

![](../images/log_gabor_filter)
___
### Algorithm Implementation:

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
___
### Outputs:

- 1D Gaussian noise filtering:
[]()

- 2D SEM image denosing:
[]()

___
### Code:
- Python version of 2D log-Gabor filter
~~~python
import math
import numpy as np
from PIL import Image
import matplotlib.pyplot as plt
~~~
