

# Machine-Learning-for-Asset-Managers - Oslo Børs
Lets apply the lessons from the book to real world data using the data from the last 936 trading days from Oslo Børs. That means we have 183 instruments

Note: building mlfinlab
```
$ pip install -r requirements.txt
$ pylint mlfinlab --rcfile=.pylintrc -f text
$ bash coverage     
```
Runs the unit-tests. Alternative:
```
pip install pytest
pip install coverage
pip install pytest-cov
pytest --cov=foo --cov=bar
```

## Chapter 2 Denoising and Detoning

S_value contains the time-series of instruments. One instrument per column.
```
>>> S_value
array([[  7.3311,  94.83  ,   3.4349, ...,  90.03  , 234.85  ,  27.6   ],
       [  7.1928,  94.83  ,   3.4139, ...,  90.43  , 238.69  ,  29.15  ],
       [  7.1005,  94.83  ,   3.4069, ...,  89.63  , 239.58  ,  29.15  ],
       ...,
       [  1.36  , 112.    ,   4.22  , ...,  21.6   , 337.7   ,  34.    ],
       [  1.465 , 112.    ,   4.315 , ...,  21.62  , 334.    ,  33.7   ],
       [  1.38  , 112.    ,   4.33  , ...,  22.48  , 331.2   ,  33.5   ]])
>>> S_value.shape
(936L, 183L)
```
There is a corresponding list of portfolio names:
```
>>> portfolio_name[0:3]
['FIVEPG.ol', 'AASB-ME.ol', 'ASC.ol']
>>> portfolio_name[-3:]
['XXL.ol', 'YAR.ol', 'ZAL.ol']
```
The time-series of values must be converted to returns.
```
>>> S, instrument_returns = calculate_returns(S_value)
>>> S
array([[-0.01886484,  0.        , -0.00611372, ...,  0.00444296,
         0.01635086,  0.05615942],
       [-0.01283228,  0.        , -0.00205044, ..., -0.00884662,
         0.00372869,  0.        ],
       [ 0.        ,  0.        ,  0.00616396, ...,  0.00892558,
         0.00784707, -0.01989708],
       ...,
       [-0.13375796, -0.00884956,  0.0047619 , ..., -0.02262443,
        -0.0088054 ,  0.        ],
       [ 0.07720588,  0.        ,  0.02251185, ...,  0.00092593,
        -0.01095647, -0.00882353],
       [-0.05802048,  0.        ,  0.00347625, ...,  0.03977798,
        -0.00838323, -0.00593472]])
```
Fitting the covariance matrix to the marcenko-pasture distribution assumes the elements of the matrix is normal distribution with mean 0 and variance:
```
>>>eVal0, eVec0, denoised_eVal, denoised_eVec, denoised_corr, var0 = denoise_OL(S)
>>> var0
0.8858105886313286
```
Fitting denoised_corr to the marcenko-pasture pdf:
| ![marcenko-pastur of oslo børs last 936 trading days.png](https://github.com/emoen/Machine-Learning-for-Asset-Managers-Oslo-Bors/blob/main/img/ol_n_183_T_936.png) | 
|:--:| 
| *Marcenko-Pasture theoretical probability density function, and empirical density function from 183 instruments the last 936 trading days:* |

The largest eigenvalues of 24.02 is the market and can be de-toned.

The eigenvalues before and after denoising:
| ![eigenvalues before and after denoising](https://github.com/emoen/Machine-Learning-for-Asset-Managers-Oslo-Bors/blob/main/img/ol_eigenvalue_method_w_osebx.png) | 
|:--:| 
| *A comparison of eigenvalues before and after applying the residual eigenvalue method:* |


The minimum-variance-portfolio with a long/short strategy gives VVL.OL as biggest long, and STB.OL as biggest short:
```
>>> np.argsort(w)
array([158, 106, 151,  38,  23,  22,  77,  10,   6,  54,  47, 111,   7,
        46, 137,   8, 175, 148, 181,   5, 157, 165, 150,  62,  13, 119,
       161,  39, 169, 114,   2,  18,  96,  30, 103, 172, 108,  16, 115,
         4,  92,   9,  83,  97,  28,  29, 112, 166, 155, 102,  33,  40,
        79, 156,  66,  94,  24, 100,  81,  76, 176,  31,  48, 153,  95,
       122,  88,  87, 125,  80, 141,  43,  68,  72,  20, 160,  57,  73,
       104, 127,  74,  85,  36, 180,  99,  55, 130, 121,  15,  98, 152,
       154, 162, 138,  86, 147, 168,  49,  41, 136,  93, 135, 116,  91,
       105, 134, 113,  14, 146, 145, 129,  19, 139, 118,  61,  82, 164,
        12, 143, 133,  17,  32, 124,  56, 182,  35, 173, 132, 101,  42,
        89,  53,  52,  44,  25,   1,  84,  90, 177,  21,  34, 171, 126,
       117,  11, 140, 123, 128, 170,   0,  71,  58, 107,  60,  26, 120,
       110,  51, 109,  45, 163,  63,  78,  75,  64, 149,  59,  50,  70,
       159, 167, 179, 178, 142,  67,  65, 144,   3,  37,  27,  69, 131,
       174], dtype=int64)
>>> portfolio_name[174]
'VVL.ol'
>>> portfolio_name[158]
'STB.ol'
```

## Chapter 4 Optimal Clustering


Running the ONC algorith on Oslo Børs with 183 instruments and time frame 934:
| ![fig_4_1_ol_block_correlation_matrix.png](https://github.com/emoen/Machine-Learning-for-Asset-Managers-Oslo-Bors/tree/main/img/fig_4_1_ol_block_correlation_matrix.png) | 
|:--:| 
| * There are 2 clusters. A small cluster of 8 instruments which are the salmon farming companies like AUSS.ol, BAKKA.ol, GSF.ol, LSG.ol and SALM.ol: * |

## Chapter 7 Portfolio Construction

Modern portfolio theory, or mean-variance analysis, is a mathematical framework for assembling a portfolio of assets such that the expected return is maximized for a given level of risk. It uses the variance of asset prices as a proxy for risk.

### Markowitz Portfolio and NCO Example

Lets compare markowitz minimum variance portfolio with minimum variance portfolio from NCO algorithm on a small worked example of 5 stock time series.
| time-step/stock | 0   | 1 | 2  | 3 | 4   | 
|-----------------|-----|---|----|---|-----|
| t.1             | 1   | 1 | 3  | 4 | 5   | 
| t.2             | 1.1 | 1 | 2  | 3 | 5   | 
| t.3             | 1.2 | 1 | 1.3| 4 | 5   | 
| t.4             | 1.3 | 1 | 1  | 3 | 5   | 
| t.5             | 1.4 | 1 | 1  | 4 | 5.5 | 
| t.6             | 1.5 | 1.1 | 1 | 3 | 5.5 | 

Covariance matrixes are generally evalueted on returns and not price as - even random walks has spurious correlations. While returns are stationary (mean 0). Therefore correlation is calculated on returns:
| time-step/stock | 0   | 1 | 2  |   3   | 4  | 
|-----------------|-----|---|----|-------|----|
| t.1             | -   | - |   -   | -   | -  | 
| t.2             | .1  | 0 | -.33 | -.25| 0  | 
| t.3             | .09 | 0 | -.35 | .33 | 0  | 
| t.4             | .08 | 0 | 0    | -.25| 0  | 
| t.5             | .08 | 0 | 0    | .33 | 0.1| 
| t.6             | .07 | .1 | 0   | -.25| 0  | 

Return Covariance matrix:
| stock/stock | 0   | 1  | 2  | 3 | 4   | 
|-------------|-----|----|----|---|-----|
| 0 | 0.00012774 |-0.00032726| -0.00178085 | -0.0001758 | -0.00018989 |
| 1 | -0.00032726 |  0.002   |  0.00457051 | -0.00583333 | -0.0005    |
| 2 |-0.00178085 |  0.00457051 | 0.02993721 | 0.00228098 | 0.00457051|
| 3 |-0.0001758 | -0.00583333 |  0.00228098 |  0.10208333 |  0.00875|
| 4 |-0.00018989 | -0.0005 | 0.00457051 | 0.00875 | 0.002|

Return on investment and volatility (std)
| function/stock | 0   | 1  | 2  | 3 | 4   | 
|----------------|-----|----|----|---|-----|
| ROI            | .5  | .1 | -.67 | -.25 | .1  | 
| Volatility     | .17 | .04| .74  | .5  | .24 | 
| NCO allocations| 0.92342949|  0.04993295|  0.04785576|  0.00165416| -0.02287237 |
| Markowitz alloc| 2.00849581| -0.51407928|  0.32638057|  0.04062639| -0.86142349 | 
| M Expected ret |-0.0250346| 0.111758771| 0.009465804| -0.01020999| -0.009415984|
| M Expected sum | 0.0765640| -| -| -| -|
| M volatility   |0.020303852| -0.02056317| 0.050509720| 0.011609958| -0.034456939|
| M sum vol      | 0.02740342|   |  |  |  |
| NCO Expected ret|0.00173010| 0.124913494| -0.03988751| -0.14265565| 0.0003460207|
| NCO Expected sum|-0.05555356| -| -| -| -|
| NCO volatility |0.009334934| 0.001997318| 0.007406020| 0.000472716| -0.000914895|
| NCO sum vol    |0.018296094|   |  |  |  |

Expected return of NCO (-0.06) is lower than markowitz (0.08), as expected. Minimum variance Markowitz volatility (0.02) is higher than NCO volatility(0.01), which is unexpected.



