# SIGMOD 2021 submission
EIRES: Efficient Integration of Remote Data in Event Stream Processing
---

## Prerequisites
* The compiler needs to support C++11 or higher.
* Bushfire detection code requires `boost` library, especially `geometry`, to compute intersections of polygons among others. That is to compute overlap of geography boundaries over satellite event streams.  To run bushfire detection code, please configure a boost lib, https://www.boost.org/users/history/version_1_72_0.html
* All running/configuration scripts are written for linux OS and Mac OS. Windows OS users need to change the paths accordingly (replace "/" with "\\").
* We build parsers to parse query workloads from files. We define query workloads in files ending with `.eql`.  `run/sythetic.eql`, `run/bf-7.7_14.16.eql` and `google_cluster.eql` are query workloads for sythetic, day-time bushfire detection and google cluster monitoring respectively. 


## Code
Source code is in `src`. Separate directories are built for synthetic data, bushfire detection and google cluster monitoring.

Parameters meaning are listed as below:
| parameter      | semantic        |
| :------------- | :--------------:|
| -c | query workload file |
| -q | specific query name |
| -F | stream source file |
| -g | set greedy selection |
| -b | configure CEP engine as baseline 1 |
| -A | configure perfetch (PFetch) to CEP engine |
| -B | configure lazy evlaution (LzEval) to CEP engine |
| -D | the number of events to process |
| -C | cache capacity |
| -f | number of fetch worker threads |
| -L | transmission latency |
| -u | utility updating frequency |
| -X | utility estimation noise |
| -Y | partial match relax ratio for LzEval |
| -Z | prefetching probability |
| -p | throughput dumping file name |
| -n | latency dumping file name |
| -s | appending timestamps for discarded matches |

#### Synthetic
Directories,  `EIRES_cost_cache` and `EIRES_LRU_cache` are EIRES codebase combine with cost-based cache and LRU cache respectively.
They have similar code structures. Entry points, `main` functions, are in defined in `EIRES_cost_cache/cep_match/cep_match.cpp` and `EIRES_LRU_cache/cep_match/cep_match.cpp`.

#### Bushfire detection
Bushfire detection code is in `EIRES_bushfire`. The entry piont, `main` function is defined in `EIRES_bushfire/cep/cep_match.cpp`.

#### Cluster monitoring
Google cluster monitoring code is in `EIRES_google_cluster_monitoring`. The `main` function is defined in `EIRES_google_cluster_monitoring/cep_match/cep_match.cpp`




##
## Data
All datasets are in `data` directory. We build separate directories for synthetic datasets, bushfire detection datasets and google cluster monitoring datasets.

#### Synthetic datasets
They are in `data/sythetic_datasets/` with two synthetic data generators implemented by `Uniform_generator.cpp` and `Zipf_generator.cpp`.  As their names suggest, they generate payload value of event streams based on uniform and Zipf disributions respectively. The number of events is configurable. Due to limited capacity, we pushed two sample stream files composed of 500K events, `data/sythetic_datasets/Stream_uniform_500K.csv` and `data/sythetic_datasets/Stream_Zipf_500K.csv`.

#### Bushfire detection datasets
They are in `data/bushfire_datasets/`.
##### Raw satellite data
Imagery information is obtained from the satellite data streams available on Amazon AWS. http://tiny.cc/drt2oz 

The data samples are generated by the Advanced Baseline Imager (ABI) of GOES-16 satellite, which captures Earth’s radiance in 16 spectral bands via a variety of radiance detectors. Basically, they are digital maps of outgoing radiance values at the top of Earth’s atmosphere at visible, infrared, and near-infrared wavelengths. Then, the samples are compressed, packetized, and sent to the ground station, in which they are converted to geo-located and calibrated pixels, covering the whole America continent. The raw image pixels are kept in Network Common Data Form (netCDF) format, which is descriptive, flexible, standardized among large research projects. Each band/channel of an image sample is kept in a separate netCDF file for each 15-minute interval.

![alt text][band]

[band]: figs/Bands.png "" 

##### weather data
Weather datasets are crawled from https://www.wunderground.com

##### Data pre-processing
To leverage GOES-16 satellite, we need pre-processing and generate event streams. In a nutshell, we cluster GOES-16 radiation levels per channel using kMeans and represent each cluster as a Polygon data type (using boost library). Following figures show the process.

![alt text][a]

[a]: figs/1.png ""

![alt text][b]

[b]: figs/2.png ""

![alt text][c]

[c]: figs/3.png ""

![alt text][d]

[d]: figs/4.png ""

![alt text][e]

[e]: figs/5.png ""

![alt text][f]

[f]: figs/6.png ""

Following figures are some visualized results. 

![alt text][h]

[h]: figs/8.png ""

![alt text][i]

[i]: figs/9.png ""

There are four generated stream files.
```
california.csv
woolsey.csv
county.csv
kincade.csv
```
#### Google cluster monitoring datasets
Google cluster monitoring traces are very well defined.
Full datasets and descriptions are publicly available at https://github.com/google/cluster-data. Due to limited capacity, we pushed a small sample, `data/google_cluster_monitoring_datasets/sample_event_stream.dat.gz`.

---

## Running scripts
#### Compile
```
cd EIRES
sh compile.sh
```
#### Running
we prepared scripts to run experiment of sythetic setting, bushfire detection and google cluster monitoring respectively.
Each script runs Baseline1, Baseline2, PFetch, FzEval and Hybrid for related quries and streams for 20 times.
Latency and throughput measurement are monitored and dumped to files for later post analysis.

```
cd run
```
##### run sythetic code and datasets
```
sh run_sythetic.sh
```
##### run bushfire detection code and datasets
```
sh run_bushfire.sh
```
##### run google cluster monitoring code and datasets
```
sh run_google_cluster.sh
```

## Post analysis scripts
We analyse 5th, 25th, 50th, 75th, 95th percentiles latency and throughput. They are realized by `run/process-latency.py` and `run/process-throughput.py`
We prepare scripts to perform all the post analysis.
After running the evalutions. 

```
cd run
sh analyse_sythetic.sh
sh analyse_bushfire.sh
sh analyse_google_cluster.sh
```
Following files will be generated. File names are self-explained. Thery cover latency and througput for sythetic, bushfire and google cluster monitoring datasets
```
result_latency_cost_greedy.dat
result_latency_LRU_greedy.dat
result_latency_cost_non_greedy.dat
result_latency_LRU_non_greedy.dat
result_latency_estimation_noise.dat
result_latency_cache_size.dat
result_latency_transmission_latency.dat

result_throughput_cost_greedy.dat
result_throughput_LRU_greedy.dat
result_throughput_cost_non_greedy.dat
result_throughput_LRU_non_greedy.dat
result_throughput_estimation_noise.dat
result_throughput_cache_size.dat
result_throughput_transmission_latency.dat

result_latency_bushfire.dat
result_throughput_bushfire.dat

result_latency_google_cluster.dat
result_throughput_google_cluster.dat
```
