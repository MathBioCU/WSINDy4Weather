# WSINDy4Weather
Weak SINDy model discovery for weather data

![changing_scales_fine](https://github.com/user-attachments/assets/ae67286a-1e1b-443a-b71c-bfdc63f2483c)

Code accompanying pre-print paper "Learning Weather Models from Data with Weak SINDy" (currently under review), available at: https://arxiv.org/abs/2501.00738

ArXiv citation:
```
@misc{minor2025learningweathermodelsdata,
      title={Learning Weather Models from Data with WSINDy}, 
      author={Seth Minor and Daniel A. Messenger and Vanja Dukic and David M. Bortz},
      year={2025},
      eprint={2501.00738},
      archivePrefix={arXiv},
      primaryClass={physics.geo-ph},
      url={https://arxiv.org/abs/2501.00738}, 
}
```

See the tutorial here: https://github.com/SethMinor/PyWSINDy-for-PDEs/blob/main/WSINDy_Tutorial.ipynb

Find the example data at:
- `barotropic_wsindy` https://pyqg.readthedocs.io/en/latest/examples/barotropic.html
- `full_globe_wsindy` https://cds.climate.copernicus.eu/datasets/reanalysis-era5-pressure-levels?tab=overview
- `geophysical_wsindy` https://turbulence.idies.jhu.edu/datasets/geophysicalTurbulence/sabl
- `swe_wsindy` https://dedalus-project.readthedocs.io/en/latest/pages/examples/ivp_sphere_shallow_water.html

###### This algorithm uses the following dependencies:
```python
import torch
import numpy as np
import matplotlib.pyplot as plt
import scipy
import symengine as sp
import itertools
import re
```
###### Optional dependencies based upon GPU/parallelism features are as follows:
```python
import torch.nn.functional as nnF # GPU
from concurrent.futures import ProcessPoolExecutor, as_completed # Parallelism
```
## Usage
###### For a dataset `U` (tensor), function library `fj` (dictionary), and derivative library `alpha` (tuple), the syntax is as follows:
```python
w = wsindy(U, fj, alpha, **params)
```
###### Example algorithm hyperparameter specification:
```python
# Grid parameters (should match dimension of dataset)
(Lx, Ly, T) = (30*np.pi, 30*np.pi, 20)
(dx, dy, dt) = (Lx/U.shape[0], Ly/U.shape[1], T/U.shape[-1])

# Function library
fields = 1 # Number of scalar fields
powers = 4 # Maximum monomial power
poly = get_poly(powers, fields)
trig = () # (Frequency, phase) pairs
fj = {'poly': poly, 'trig': trig}

# Derivative library
lhs = ((0,0,1),) # Evolution operator D^0
dimension = 2 # Spatial dimensions
pure_derivs = 4 # Include up to fourth order
cross_derivs = 2 # Include up to second order
rhs = get_alpha(dimension, pure_derivs, cross_derivs)
alpha = lhs + rhs

params = {
    # x = spatial domain(s)
    # dx = spatial discretization(s)
    # t = temporal domain
    # dt = temporal discretization
    # aux_fields = extra library variables
    # aux_scales = scaling factors for aux fields
    #--------------------------------------------
    'x' : [(0, Lx), (0, Ly)],
    'dx' : [dx, dy],
    't' : (0, T),
    'dt' : dt,

    # m = explicit (mx,...,mt) values
    # s = explicit (sx,...,st) values
    # lambdas = MSTLS threshold search space
    # threshold = known optimal threshold
    # p = explicit (px,...,pt) values
    # tau = test function tolerance
    # tau_hat = Fourier test function tolerance
    # scales = explicit (yu,yx,yt) scaling factors
    # M = explicit scaling matrix
    #---------------------------------------------

    # verbosity = report info and create plots? (0 or 1)
    # init_guess = [x0, y0, m1, m2], for (kx,kt) curve fit
    # max_its = specify maximum number of MSTLS iterations
    # sigma_NR = noise ratio of artifical gaussian noise
    # sparsify = use 'original' or 'scaled' data in MSTLS
    #-----------------------------------------------------
    'verbosity' : 1,
    'sigma_NR' : 0.0,
    'sparsify' : 'original'}
```
