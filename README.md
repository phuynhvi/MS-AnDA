
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
   * Class **VAR**: to store all necessary datasets
      * Training and testing catalog for detail fields in both original and EOF space
      * Observation
      * LR product
      * Condition dataset used in AF (if exists)
      * Indexing set that points out the position of a patch over original image
      ```bash
      # Program will automatically load all data into this variable according the parameters described in class **PR**
      class VAR:
           X_lr = []
           dX_orig = []
           Optimal_itrp = []    
           dX_train = [] # training catalogs  for dX in EOF space
           dX_eof_coeff = [] # EOF base vector
           dX_eof_mu = [] # EOF mean vector    
           dX_GT_test = [] # dX GT in test year
           Obs_test = [] # Observation in test year, by applying mask to dX GT    
           dX_cond = [] # condition used for AF
           gr_vl_train = [] # gradient, velocity used as physical condition
           gr_vl_test = {}  
           gr_vl_coeff = {}        
           index_patch = [] # store order of every image patch: 0, 1,..total_patchs
           neighbor_patchs = [] # store order of neighbors of every image patch
      ```
   * Class **AF**: to specify parameters for Analog Forecasting
      * Use condition for analog forecasting ?. If using condition, specify where is the condition
      * Use clusterized version ?. If using, specify number of *k* clusters
      * Use global or local analog by specifying form of neighborhood
      * Select three forecasting strategies: locally constant, increment, local linear
      * Variance of initial error, observation error
      * Pre-trained nearest neighbor searchers ( FLANN )
      ```bash
      # Example of Analog Forecasting for SST
      AF_sst = General_AF()
      AF_sst.flag_reduced  = True # True: Clusterized version of Local Linear AF
      AF_sst.flag_cond = False # True: use Obs at t+lag as condition to select successors
                        # False: no condition in analog forecasting
      AF_sst.flag_model = False # True: Use gradient, velocity as additional regressors in AF
      AF_sst.flag_catalog = True # True: Use each catalog for each patch position
                          # False: Use only one big catalog for all positions 
      AF_sst.cluster = 1     # number of cluster for clusterized ver.
      AF_sst.k = 200  # number of analogs
      AF_sst.k_initial = 200 # retrieving k_initial nearest neighbors, then using condition to retrieve k analogs, k_initial must >= k
      AF_sst.neighborhood = np.ones([PR_sst.n,PR_sst.n]) # global analogs
      AF_sst.neighborhood = np.eye(PR_sst.n)+np.diag(np.ones(PR_sst.n-1),1)+ np.diag(np.ones(PR_sst.n-1),-1)+ \
                             np.diag(np.ones(PR_sst.n-2),2)+np.diag(np.ones(PR_sst.n-2),-2)
      AF_sst.neighborhood[0:2,:5] = 1
      AF_sst.neighborhood[PR_sst.n-2:,PR_sst.n-5:] = 1 # local analogs
      AF_sst.neighborhood[PR_sst.n-2:,PR_sst.n-5:] = 1 # local analogs
      AF_sst.regression = 'local_linear' # forecasting strategies. select among: locally_constant, increment, local_linear 
      AF_sst.sampling = 'gaussian' 
      AF_sst.B = 0.05 # variance of initial state error
      AF_sst.R = 0.1  # variance of observation error
      ```
   * Class **AnDA_result**: store AnDA’s results, such as GT, Observation, Optimal Interpolation, AnDA Interpolations and statistical errors (rmse, correlation)
      * sdsds
      ```bash
      ```
      
