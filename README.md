
# DESCRIPTION
MS-AnDA is a Python library for Multi-scale Analog Data Assimilation, applications to ocean remote sensing. We presented a novel data-driven model for the spatio-temporal interpolation of satellite-derived SST (sea surface temperature) & SLA (sea level anomaly) fields, an extension of analog data assimilation framework (https://github.com/ptandeo/AnDA) to high-dimensional satellite-derived geophysical fields.
 
This Python library is an additional material of the publication "Data-driven Models for the Spatio-Temporal Interpolation of satellite-derived SST Fields", from **R. Fablet, P. Huynh Viet, R. Lguensat**, accepted to *IEEE Transactions on Computational Imaging*

# How to use
The toolbox includes 3 main modules:
1. Module **Parameters** (*AnDA_variables.py*): 
   * Class **PR**: to specify general parameters
      * Use multi-scale or single-scale (global-scale) assimilation ?
      * Dimension of state vector (or reduced dimensionality in PCA space)
      * Size of patch (eg. 20 × 20)
      * Size of training dataset, testing dataset (number of images)
      * Directories of datasets: sst (sla), observation, OI product (ostia)...
 ```bash
 # Example of setting parameter for SST
  PR_sst = PR() 
  PR_sst.flag_scale = True  # True: multi scale AnDA, False: global scale AnDA                 
  PR_sst.n = 50 # dimension state vector
  PR_sst.patch_r = 20 # r_size of patch 
  PR_sst.patch_c = 20 # c_size of patch
  PR_sst.training_days = 2558 # num of training images: 2008-2014 
  PR_sst.test_days = 364 # num of test images: 2015
  PR_sst.lag = 1 # lag of time series: t -> t+lag
  PR_sst.G_PCA = 20 # N_eof for global PCA
  # Input dataset
  PR_sst.path_X = './data/AMSRE/sst.npz' # directory of sst data
  PR_sst.path_OI = './data/AMSRE/OI.npz' # directory of OI product (ostia sst, in this case)
  PR_sst.path_mask = './AMSRE/metop_mask.npz' # directory of observation mask
  # Dataset automatically created during execution
  PR_sst.path_X_lr = './data/AMSRE/sst_lr_30.npz' # directory of LR product
  PR_sst.path_dX_PCA = './data/AMSRE/dX_pca.npz' # directory of PCA transformation of detail fields
  PR_sst.path_index_patches = './data/AMSRE/list_pos.pickle' # directory to store all position of each patch over image
  PR_sst.path_neighbor_patches = './data/AMSRE/pair_pos.pickle' # directory to store position of each path's neighbors 
 ```
      
