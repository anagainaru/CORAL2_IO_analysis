# Analyzing the IO logs

Two step analysis:
  - Extract the list of features from the raw Darshan logs
  - Create clusters with applications with similar patterns

**Required packages to test the scripts in this repo**

 - For plotting: python package `seaborn`
 - For feature extraction: python package `lightgbm`
 - For extracting the distance matrix: python packages `sklearn`, `dataset`, `hdbscan`
 - For the feature importance list: `libomp` and python package `xgboost`
 
All the dependencies with their versions are written in `requirements.txt`.
To install them, you will need `pip`. Run `pip install -r requirements.txt`.

## Extract the feature list

### 1. Extract the list of applications

Scripts available in folder `extract_logs`.

List of applications that generated the Darshan logs we will analyze. The list of application will be annotated by the field it is part of (example XGC is fusion, CANDLE is AI etc).

For Summit application, the `extract_job_info.sh` script is used to extract the list of applications.

### 2. Parse all the darshan logs

Scripts available in folder `extract_logs`.

And create a log that follows the format needed by the python scripts used to extract features.
For Summit, we use the `extract_logs.sh` script to extract all the darshan logs in a given timeframe and containing a keyword (either application name or user or none).
In the current configuration, the script extracts all the logs between September and December for user againaru.

The darshan files need to be parsed with the `--file` option followed by the default option in order to be usable by the python scripts:

```bash
darshan-parser --file ${darshan_log} > logs/${filename}.log
darshan-parser ${darshan_log} >> logs/${filename}.log
```

### 3. Extract the list features

Scripts available in folder `features`.

Extract features from each darshan log and store them in the same file.

```bash
for i in logs/*.log; do
    python extract_feature_list.py $i summit_againaru_features.csv; 
done
```

There are currently 161 features (including the user submitting the application, the name of the application and of the executable).
List of all the features and explanation of what each represents can be dound in the file `feature.description`.

The output file is a CSV file containing one entry for each darshan log.

There are 13 applications whose behavior on Summit 2020 was extracted into a separate csv file:
Castro (1.3k runs), RiemannProblem (100), cp2k (200), dirac (200), gronor (800), gtc (10k), meshfem3D (150), namd (3.5k),
nwchem (200), qmcpack (3k), stemdl (100), vasp (16.5k), xgc (800). Details about their runs can be found in: [Summit application behavior](https://github.com/anagainaru/HPC_IOpatterns/tree/main/aggregated-patterns/summit)

![feature_correlation](https://user-images.githubusercontent.com/16229479/200451969-016cd903-8ee2-42e3-ab52-193b8eb44495.png)


## Create clusters of similar data

Scripts available in folder `clustering`.

Applications are clustered based on their features (except the app name and user name features) using hdbscan.HDBSCAN ([Link](https://hdbscan.readthedocs.io/)).
HDBSCAN is a clustering algorithm that extends DBSCAN by converting it into a hierarchical clustering algorithm, and then using a technique to extract a flat clustering based in the stability of clusters. A detailed explanation of how clustered are being formed can be found [here](https://hdbscan.readthedocs.io/en/latest/how_hdbscan_works.html).

```bash
$ python extract_clusters.py filename.csv
Before filter: 121
After filter: 2
Total columns 198, clustering columns 159
Optimal number of clusters: 0
Number of noisy points: 2
Start classification from 2.0
Clusters successfully saved in folder todel.clusters/
$ cat filename.clusters/cluster.0.000.csv | rev | cut -d',' -f 1 | rev | sort -u
-1
```
The previous example shows the output for an application that has consistent behavior throughout all its runs.

Example results for the clustering method for the list of application runs (and darshan logs) from this repo in: [Summit application behavior](https://github.com/anagainaru/HPC_IOpatterns/tree/main/aggregated-patterns/summit).

![Hierarchical clustering](docs/cluster_tree.png)


### 1. Extract the list of applications in each clusters

The script in `extract_clusters.py` creates the figure on the left. It uses hdbscan to create the cluster hierarchy then it identifies each entry in the features file that corresponds to each cluster and saves it in a file. The script saves the main clusters from the level with the stable clusters from the condensed tree (dashed line in the figure) in `*.0.csv`. The file contains one line for each entry in the features file with an additional column (`Cluster`) with labels from 0 to n-1, where n is the number of identified clusters.

The script also saves information about the two clusters created at each split, one file for each split distance (binary labels only for the dataset involved in the split). For example for the first split the file will contain all the application runs with their features and an additional column (`Cluster`) that takes values 0 (for the apps that are classified on the left side) and 1 for the right side. 

### 2. Create the distance matrix

The script in `extract_clusters.py` creates the figure on the right. The code is based on the one implemented [here](https://github.com/MihailoIsakov/SC2020).
It creates the clusters using hdbscan, identifies the 6 more frequently ran applications and computes the manhattan and euclidean distances between all these application runs. The script output the figure.

### 3. Feature importance
