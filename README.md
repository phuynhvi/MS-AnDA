
# DESCRIPTION
MS-AnDA is a Python library for Multi-scale Analog Data Assimilation, applications to ocean remote sensing. We presented a novel data-driven model for the spatio-temporal interpolation of satellite-derived SST (sea surface temperature) & SLA (sea level anomaly) fields, an extension of analog data assimilation framework (https://github.com/ptandeo/AnDA) to high-dimensional satellite-derived geophysical fields.
 
This Python library is an additional material of the publication "Data-driven Models for the Spatio-Temporal Interpolation of satellite-derived SST Fields", from **R. Fablet, P. Huynh Viet, R. Lguensat**, accepted to *IEEE Transactions on Computational Imaging*

# How to use
-The toolbox includes 3 main modules:
-1. Module **Parameters** (AnDA_variables.py): 
   * Class **PR**: to specify general parameters
      * Use multi-scale or single-scale (global-scale) assimilation ?
