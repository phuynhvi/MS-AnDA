
# DESCRIPTION
MS-AnDA is a Python library for Multi-scale Analog Data Assimilation, applications to ocean remote sensing. We presented a novel data-driven model for the spatio-temporal interpolation of satellite-derived SST (sea surface temperature) & SLA (sea level anomaly) fields, an extension of analog data assimilation framework (https://github.com/ptandeo/AnDA) to high-dimensional satellite-derived geophysical fields.
 
This Python library is an additional material of the publication "Data-driven Models for the Spatio-Temporal Interpolation of satellite-derived SST Fields", from **R. Fablet, P. Huynh Viet, R. Lguensat**, accepted to *IEEE Transactions on Computational Imaging*

# Basic Overview
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
   * Class **General_AF**: to specify parameters for Analog Forecasting
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
      ```bash
      # All results will be computed and stored in this class.
      class AnDA_result:
           itrp_AnDA = [] # AnDA interpolation
           itrp_OI = []   # OI product, for comparison
           itrp_postAnDA = []  # Post_processing AnDA interpolation (removing block artifacts)
           GT = []   # groundtruth
           Obs = []  # Observation
           LR = []   # Low resolution product
           # stats: rmse & correlation of interpolation to the groundtruth
           rmse_AnDA = [] 
           corr_AnDA = []
           rmse_OI = []
           corr_OI = []
           rmse_postAnDA = []
           corr_postAnDA = []
      ```
2. Module **Transform functions** (*AnDa_transform_functions.py*): 
    * Perform Global PCA (to find LR), patch-based PCA for multi-scale assimilation
    * Post-processing to remove block artifact due to overlapping patches
    * Perform VE-DINEOF
    * Find gradient, Fourier power spectrum
    * Loading and preprocessing data according to the parameters described in **PR**
3. Module **Multi-scale Assimilation** (*Multiscale_Assimilation.py*): based on informations from PR, VAR, AF, defining a specific kind of assimilation
    * Class **Single_patch_assimilation**:
       * Processing on one single patch.
       * Input: position of patch (rows, columns) over initial image.
    * Class **Multi_patch_assimilation**:
       * Processing on a zone of image (defined by its size and coordinates of top-left point), by dividing into multiples patches, then plugging them into **Single_patch_assimilation**
       * Input: number of parallel jobs, or number of patches are executed simultaneously.
       
# Test
Specify all necessary parameters described in class **PR**, and **General_AF**
Load data into class **VAR**:
```bash
VAR_ = VAR()
VAR_ = Load_data(PR_sst) 
```
Visualize an example of reference Groundtruth, Observation and Optimal Interpolation product
```bash
day = 50
colormap='nipy_spectral'
plt.clf()
gt = VAR_.dX_GT_test[day,:,:]   
obs = VAR_.Obs_test[day,:,:]    
itrp = VAR_.Optimal_itrp[day,:,:]   
vmin = np.nanmin(gt)
vmax = np.nanmax(gt)
plt.subplot(1,3,1)
plt.imshow(gt,aspect='auto',cmap=colormap,vmin=vmin,vmax=vmax)
plt.colorbar()
plt.title('GT')
plt.subplot(1,3,2)
plt.imshow(obs,aspect='auto',cmap=colormap,vmin=vmin,vmax=vmax)
plt.colorbar() 
plt.title('Obs')
plt.subplot(1,3,3)
plt.imshow(itrp,aspect='auto',cmap=colormap,vmin=vmin,vmax=vmax)
plt.colorbar()  
plt.title('OI')
plt.draw()
```
![](../master/Image/dis.png)

Define test zone (top-left point and size of zone): 
```bash
r_start = 0 
c_start = 0 
r_length = 150 
c_length = 300 
```
Define multiprocessing level:
```bash
level = 22 # 22 patches executed simultaneously
```
Run Assimilation:
```bash
saved_path =  './data/AMSRE/AnDA/AnDA_local_cond.pickle'
AnDA_sst_1 = AnDA_result()
MS_AnDA_sst = MS_AnDA(VAR_sst, PR_sst, AF_sst)
AnDA_sst_1 = MS_AnDA_sst.multi_patches_assimilation(level, r_start, r_length, c_start, c_length)
```
Save result:
```bash
with open(saved_path, 'wb') as handle:
    pickle.dump(AnDA_sst_1, handle)
```
Reload result:
Save result:
```bash
with open(saved_path, 'rb') as handle:
    AnDA_sst_1 = pickle.load(handle) 
```
Run VE-DINEOF algorithms to compare with AnDA interpolation.
```bash
sla_dineof = VE_Dineof(PR_sla, VAR_sla.dX_orig+VAR_sla.X_lr, VAR_sla.Optimal_itrp+VAR_sla.X_lr[PR_sla.training_days:], VAR_sla.Obs_test, 100, 50)
```
