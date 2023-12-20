# Smart Energy Asset Management Intelligence - Fault Detection and Diagnosis

Documentation part of the NSSN Grand Challenge - Smart Energy Asset Management Intelligence Project

This repository contains the following projects:
1. **Performance to Peers (P2P) Alogrithm**: Contains Jupyter Notebooks, Python Project, and data to perform the P2P algorithm
2. **Fault Detection and Diagnosis at AC side**: Contains Jupyter Notebooks and data to perform the labelling process at AC side
3. **Fault Detection and Diagnosis at DC side**: Contains Jupyter Notebooks and data to perform the labelling process at DC side

## Table of Contents

- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
- [Data Prepare](#1-data-prepare)
- [Data Preprocessing](#2-data-preprocessing)
- [P2P Algorithm](#3-performance-to-peer-p2p-algorithm)
  - [P2P Algorithm for Minute Data](#31-p2p-algorithm-for-minitues-data-monitor-level)
  - [P2P Algorithm for Daily Data](#32-p2p-algorithm-for-daily-data-system-level)
- [Fault Detection and Diagnosis at AC Side](#4-fault-detection-and-diagnosis-at-ac-side)
  - [Recurring Underperformance Detection](#41-recurring-underperformance-detection)
  - [Clustering Algorithm](#42-clustering-algorithm)
- [Fault Detection and Diagnosis at DC Side](#5-fault-detection-and-diagnosis-at-dc-side)
  - [Zero Generation](#51-zero-generation)
  - [Shading](#52-shading-fault-detection)
  - [Clipping](#53-clipping-fault-detection)
  - [Significant Difference Between Different Phases/MPPTs](#54-significant-difference-between-difference-phasesmppts)
  - [String Fault Detection](#55-string-fault-detection)
  - [Python Project for DC-side Fault Diagnosis](#56-python-project-fault-diagnosis)
  - [Statistics](#57-statistics)
- [Acknowledgements](#acknowledgements)

## Prerequisites

What things you need to install and run this code:

- Python 3.9
- Jupyter Notebook
- Required Python libraries

## 1. Data Prepare

This project contains almost 1200 PV monitors across Australia

Jupyter Notebooks can be found under the folder `1-Data_Analysis`
![`PV systems across Australia`](1-Data_Analysis/iShot_2023-10-17_09.20.47.png)

## 2. Data Preprocessing
Generally, we need to process the outlier (too large or negative), missing data, and data with time zone issues (generate power at night)
Jupyter Notebooks can be found under the folder `2-Preprocess_Data`

- `20-Preprocess_data_for_SingleMonitor.ipynb`: Elaborate Procedure of Data Preprocessing from a Single Monitor. Monitor ID should be provided
- `21-Preprocess_data_for_allmonitors.ipynb`: RElaborate Procedure of Data Preprocessing for multiple monitors
- `22-ClearSky_Day.ipynb`: Calculate the clearsky day based on the expected generation and the clear-sky model


## 3. Performance to Peer (P2P) Algorithm
This work is based on the reference "J. Leloux, L. Narvarte, A. Desportes, and D. Trebosc, “Performance to peers (p2p): A benchmark approach to fault detections applied to photovoltaic
system fleets,” Solar Energy, vol. 202, pp. 522–539, 2020."

Proposed P2P algorithm with our own dataset
![`Proposed P2P algorithm`](3-Performance2Peers_Alagorithm/P2P.png)
<span style="color: red;">**There are some challenges when implementing P2P algorithms for real-world industrial applications.**</span>
- **Peak Power Selection for Capacity Utilisation Factor (CUF) Calculation**:
DC-rated PV size is more stable and reliable for the CUF calculation than the maximum value of measured data and is recommended as the peak power
- **Traditional P2P Calculation Method Suffers from Large Variations with Different Reference CUF**
- **Cloudy Weather Introduces Noise to the P2P Calculation**
- **Insufficient Number of Peers Available for Comparison within the Recommended Distance**:
If only clear-sky days are used for underperformance detection, a larger distance, for example, 30 km, can be adopted without compromising on accuracy.
- **Orientation Concerns for the P2P Calculation:** The results of our industry dataset suggest that if the orientations of
the PV systems do not vary significantly, the orientation effect can be ignored for shading faults. Further studies are required.
![`Orientation`](3-Performance2Peers_Alagorithm/22135464_peers_image.png)


### 3.1. P2P algorithm for minitues data (Monitor Level)
Jupyter Notebooks can be found under the folder `3-Performance2Peers_Alagorithm/31-P2P_ShortTerm_Monitor.ipynb` with 5-minute power data
The resulst of P2P algorithms with a focus PV and it peers in 10 km are as followings:
![`CUF of the Focus PV and its Peers in 10 km`](3-Performance2Peers_Alagorithm/P2P_CUF.png)
![`P2P fault detection results`](3-Performance2Peers_Alagorithm/P2P_detection.png)

### 3.2. P2P algorithm for daily data (System Level)
Jupyter Notebooks can be found under the folder `3-Performance2Peers_Alagorithm/32-P2P_LongTerm_PVSite.ipynb` with daily energy data
The process is same as the 5-minute P2P algorithm and the results and discussions are as followings:
- **Daily Actual, Expected, and Clear-Sky Energy**

![`Daily Actual, Expected, Clear-Sky Energy`](3-Performance2Peers_Alagorithm/daily_energy.png)

- **P2P calculation with division method $P2P=\frac{CUF_{focu}}{CUF_{ref}}$ and delta method $P2P = 1 - (CUF_{ref} - CUF_{focus})$**

![`P2P calculation with division method`](3-Performance2Peers_Alagorithm/daily_p2p_division.png)
![`P2P calculation with delta method`](3-Performance2Peers_Alagorithm/daily_p2p_delta.png)

It can be seen that the division method suffers some noise when the $CUF_{ref}$ is small (generally at the begining/end of a day)
So, we proposed a mixture algorithm for P2P calculation. If the $CUF_{ref}$ less than a threshold, delta method is adopted. Otherwise, division method is used for P2P calcualtion.

![`P2P calculation with mixture method`](3-Performance2Peers_Alagorithm/daily_p2p_mix.png)

<span style="color:blue">Conclusion: the delta method is more reasonable than the division method for this PV system </span>

- **P2P compared with Performance Ratio (PR) indicator**

The underperformance detection based P2P and PR are very similar.

![`P2P calculation with mixture method`](3-Performance2Peers_Alagorithm/daily_P2P_vs_PR.png)

## 4. Fault Detection and Diagnosis at AC Side
Only AC power generation is available for fault detection and diagnosis of PV systems at AC side

### 4.1. Recurring Underperformance Detection
Example of recurring underperformance that might be caused by:
- Dust and dirt accumulation on solar panels.
- Shading from nearby objects 

![`Recurring Underperformance`](4-FaultDetection_ACLevel/figures/recurring_underperformance.png)

The jupyter code can be found `4-FaultDetection_ACLevel/40-AC_Recurring_Underperformance.ipynb`. The flowchart of the method is as following,

![`Recurring Underperformance flowchart`](4-FaultDetection_ACLevel/figures/flowchat_recurring.png)

### 4.2. Clustering algorithm

- Clustering for different PV systems: the jupyter code can be found at: `4-FaultDetection_ACLevel/42-Clustering_ShortTerm.ipynb`

 ![`binary clustering for different PV systems`](4-FaultDetection_ACLevel/figures/multiple_clustering.png)

 For different PV systems, it might be too messy for clustering
 
- Clustering for different days of one PV system: the jupyter code can be found at 
`4-FaultDetection_ACLevel/41-Clustering_ShortTerm_singleMNTR.ipynb`

  - Clustering with raw data

  ![`binary clustering with raw data`](4-FaultDetection_ACLevel/figures/raw_clustering.png)

  - Clustering with decomposition: Hodrick-Prescott (HP) filter (HPFilter) 

  ![`hpl results`](4-FaultDetection_ACLevel/figures/hpfilter.png)

  ![`hpl clustering`](4-FaultDetection_ACLevel/figures/hpl_clustering.png)

As there is no data to quantify the performance of clustering, it is difficult to tell which method is best.

### 4.3. Machine Learning-based Recurring Underperformance Detection
- To generate samples based on the labelled file

Jupyter code can be found at `4-FaultDetection_ACLevel/machine_learning/43-0-Generate_Dailysamples.ipynb`

**There are some systems sufffer from time-zone issue**

![`Time zone issue`](4-FaultDetection_ACLevel/machine_learning/timezone_sample.png)

- **Fault detection based on machine learning, such as Random Forest**

Jupyter code can be found at `4-FaultDetection_ACLevel/machine_learning/43-1-Recurring_Binary_ML.ipynb`

Faults mainly include recurring underperformance, clipping, and tripping

More specific codes can be found at `4-FaultDetection_ACLevel/machine_learning/43-4-Level1_Multiclass.ipynb`

**The number of labeled samples available for training models for different types of faults is significantly imbalanced.**

With limited test dataset, the $f_1$ score can reach to 94%

![`roc`](4-FaultDetection_ACLevel/machine_learning/roc.png)

- **Fault detection based on deep learning, such as LSTM**

Jupyter code can be found at `4-FaultDetection_ACLevel/machine_learning/43-2-Recurring_Binary_NN.ipynb`

**Before implementing any machine learning algorithm, we should make sure that the labelled data are all corrrect.**

## 5. Fault Detection and Diagnosis at DC Side

### 5.1. Zero Generation
Jupyter code can be found at `5-Fault_Detection_DCLevel/50-FIMER_Shading_Clipping_0Gen.ipynb`

The conditions for zero generation are as followings:

- DC generation == 0 
- AC generation == 0 
- day time : Based on the sunrise and sunset times calculated by PVlib, we can narrow it down 2-hour as a daytime window.


### 5.2. Shading Fault Detection

Jupyter code can be found at `5-Fault_Detection_DCLevel/50-FIMER_Shading_Clipping_0Gen.ipynb`

Shading detection is based on the comparison results between the AC/DC power generation and theoretical generation.

![`shading`](5-Fault_Detection_DCLevel/shading_detection.png)

### 5.3. Clipping Fault Detection

Jupyter code can be found at `5-Fault_Detection_DCLevel/50-FIMER_Shading_Clipping_0Gen.ipynb`

![`clipping`](5-Fault_Detection_DCLevel/clipping.png)

### 5.4. Significant Difference between difference phases/MPPTs

Jupyter code can be found at `5-Fault_Detection_DCLevel/51-SMA_MPPTs_signficant_difference.ipynb` for all the monitors and `5-Fault_Detection_DCLevel/51-SMA_Sigdiff_Single_monitor.ipynb` for a single system as an example.

Significant difference are between different phases or MPPTs

![`significant difference`](5-Fault_Detection_DCLevel/significant_difference.png)

### 5.5. String Fault Detection

This is based on historical time-series data for a single MPPT to identify consecutive string fault causes relative fixed generation reduction. 

![`string_fault`](5-Fault_Detection_DCLevel/String_fault.png)

The baseline for detecting generation reduction should also account for the seasonal effects.

![`string_fault_baseline`](5-Fault_Detection_DCLevel/string_fault_baseline.png)

### 5.6 Python Project Fault Diagnosis

This is fault detection and diagnosis of PV systems based on the following flowchart:

![`dc_fault`](5-Fault_Detection_DCLevel/54-DC_Labelling_Python/TrippingAndClippingFaults.png)

### 5.7 Statistics

![`statistics`](5-Fault_Detection_DCLevel/statistics.png)

#### Input Data

- `MNTR_ddb_20230630.csv`: List of monitors. Exported from DynamoDB.
- `SITE_nodeType_20230630.csv`: List of sites. Exported from DynamoDB.
- `Diagno Labelling.xlsx`: List of labelled data.

## 6. Acknowledgements

- NSSN Grand Challenge
- GSES
- UNSW
- UTS

