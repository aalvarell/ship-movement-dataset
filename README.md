# Ship movements dataset

This dataset contains the movements of 45 ships recorded in the [Outer Port of Punta Langosteira](https://goo.gl/maps/UG8zR274TrZEUSj66 "Outer Port of Punta Langosteira, Spain") (A Coruña, Spain) from 2015 until 2020.

The dataset is intended to be used to train supervised Machine Learning models, so each file has input variables (predictors) and output variables (value(s) to predict). There are three types of __input variables__: meteorological conditions (weather conditions and sea state), ship characteristics, and berthing location. The __output variables__ are the ship's movements.

The [data sources section](#data-sources) explains how the data stored in this data source was obtained. It is not necessary to read that section to use the data, but it explains its nature.
The [Dataset characteristics section](#dataset-characteristics) explains the variables available in the dataset, how to interpret them, and how the data is distributed.

## Data sources

### Input variables

The weather conditions and sea state data were collected using three measurement systems provided by the port technological infrastructure: a directional wave buoy, a weather station, and a tide gauge. All this data is also available in the [Spanish Port System data webpage][5], searching for _ "Langosteira"_ in the search box:

* __The directional wave buoy__ is located 1.8 km off the main breakwater of the port (43º 21' 00" N 8º 33' 36" W). This buoy belongs to the Coastal Buoy Network of Puertos del Estado ([REDCOS][1]) and has the code 1239 in said network. Due to the sensibility of the buoy data to noise, the system provides its data aggregated in 1-hour intervals. This quantization allows calculating some statistical parameters that reflect the sea state while mitigating the noise's effects.
* __The weather station__ is located on the main port's breakwater. It provides the data aggregated in 10 minutes intervals.
* __The tide gauge__ is also placed at the end of the main port's breakwater. This tide gauge belongs to the Coastal Tide Gauges Network of Puertos del Estado ([REDMAR][2]) and has the code 3214 on said network. It provides the data in 1-minute intervals.

The port operator provided all the values for the ship characteristics and berthing location variables.

### Output variables

The ship's movements were recorded using three complementary and fully synchronized measurement systems:
* An __Inertial Measuring Unit__ (IMU), a device that can measure and record the angular rate and specific gravity of the object to which it is attached to.
* __Two laser distance meters__ pointed to the bow and stern areas of the ship.
* __Two cameras__ used together with Computer Vision techniques to track the vessel movement.

The methodology used to measure the ships' movement (in its six degrees of freedom) is described in more detail in two research articles:
* [Field measurements of angular motions of a vessel at berth: Inertial device application
][3].
* [Dynamical Study of a Moored Vessel Using Computer Vision][4].

The combination of these devices allows the recording of the movements at a minimum frequency of 1 Hz, which is enough to detect the movements of a moored vessel.

## Dataset characteristics

The [Spanish Port System data webpage][5] provides a forecast for some ocean-meteorological variables. This dataset is intended to create ships movement models that, when used with weather and sea state forecasts as inputs, can predict future movements of a moored ship. This usage limits the ocean-meteorological variables that can be used to train the models to those that are also provided by the port's forecast system. For this reason, this dataset only stores the variables that are also available in the port's forecast system.

### Variables

These are the meteorological variables that are intended to be used as __input variables__:

* __H<sub>s</sub>__ (m): significant wave height, i.e., the mean of the highest third of the waves in a time series of waves representing a certain sea state measured by a buoy.
* __T<sub>p</sub>__ (s): peak wave period, i.e., the period of the waves with the highest energy, extracted from the spectral analysis of the wave energy.
* __θ<sub>m</sub>__ (deg): mean wave direction, i.e., the mean of all the individual wave directions in a time series representing a certain sea state.
* __W<sub>s</sub>__ (km/h): mean wind speed.
* __W<sub>d</sub>__ (deg): mean wind direction.
* __H<sub>0</sub>__ (m): sea level with respect to the zero of the port.
* __H<sub>sm</sub>__ (m): significant wave height measured by a tide gauge, i.e. the mean of the highest third of the waves in a time series of waves representing a certain sea state.

> **Note**: a buoy provides H<sub>s</sub>, T<sub>p</sub>, and θ<sub>m</sub>, a weather station provides W<sub>s</sub> and W<sub>d</sub>, and a tide gauge provides H<sub>0</sub> and H<sub>sm</sub>. Read the [data sources section](#data-sources) for more details.

The other type of __input variables__ are the ship characteristics and the berthing location:
* __Length__ (m): ship length.
* __Breadth__ (m): ship breadth.
* __DWT__ (tonnes): deadweight tonnage, a measure of how much weight a ship can carry. 
* __Berthing zone__: . The port is divided into 12 berthing zones (the port operator provides it).

> **Note**: the ship's geometric characteristics have high relevance in its dynamic behavior. The dataset uses characteristics that are easy to obtain (even in production). The dataset also uses the berthing zone as input. The berthing zone will allow a model to capture the characteristics of the different locations of the port and create a tool that loops over all available berthing zones and calculates a prediction in each one to choose the best berthing location.

The output variables are ship movements. A moored ship has six degrees of freedom: three displacements (surge, sway, and heave) and three rotations (roll, pitch, and yaw):

* __Surge__ (m): linear longitudinal motion (bow-stern).
* __Sway__ (m): linear lateral motion (port-starboard).
* __Heave__ (m): linear vertical motion.
* __Roll__ (deg): tilting rotation of the vessel about its longitudinal axis (bow-stern).
* __Pitch__ (deg): up/down rotation of the vessel about its lateral axis (port-starboard).
* __Yaw__ (deg): turning rotation of the vessel about its vertical axis. 

![six degrees of freedom of a vessel's movement](https://raw.githubusercontent.com/aalvarell/ship-movement-dataset/36a776101a9b6f4bef3f3f19a10134639bd34d9c/vessel_movements.png)

> **Note**: an Inertial Measuring Unit provides the pitch and roll. Two laser distance meters provide the sway and yaw, and two cameras provide the surge and heave.

The dataset also stores information related to the ship's stay, not intended to be used as predictors: the ship's name (anonymized), the date of the stay, and the ship's type (*General Cargo* or *Bulk carrier*).

### Data frequency

All the data sources used to create this dataset (buoy, weather station, tide gauge, ship's characteristics, and ship's movement monitoring) have different frequencies. The dataset aggregates the data to have the same frequency as the frequency of the source with the largest period (the sea state data). This is the same frequency as the data intended to be used as the models' inputs in production (forecast data): the variables that the forecast systems provide have a period of 1 hour, so the dataset aggregates the rest of the variables and provides them with a **1-hour period**. When aggregating the data for the ship's movements, we calculate the mean, maximum, and significant (mean of the highest third) value for the amplitude of each movement for every hour. i.e., for each movement, we have three variables (mean, max, and significant).

> **Note**: a moored vessel may experience a maximum point motion much greater than its significant or average motion under the action of certain ocean-meteorological conditions. This value abandons the primary trend of the movement. It could occasionally be caused by external agents such as the waves generated by the passage of other ships or the punctual modification of the mooring lines' tension to adapt them to the variations of the tidal range, making them difficult to predict. It usually does not disrupt the operations due to its brief nature. In summary, the best choice for the value to predict for a given movement is the significant value.

### Dataset files, variables names, and dataset size

The dataset stores the data related to **each movement in a separate CSV file**. When using a camera to measure a ship's movement in a port environment, there are many technical limitations, occlusion being the most limiting. When a vessel is loading/unloading, there are many types of machinery such as cranes and trucks that occlude the camera field of view, making it impossible to measure the movements that rely on computer vision. Given this limitation, not all movements are available for all the vessels at all times. To surpass this limitation, to use all data available and to create precise prediction models, the dataset stores the data related to each movement in a separate file, so one could create one prediction model for each vessel movement.

The data is already divided into a **training dataset**  and a **testing dataset**. Usually, the test dataset is created by randomly choosing data from the whole dataset but, had we had done this, the test results would have been inaccurate (overly optimistic). The movement data of a ship for a specific stay is highly correlated, as it is temporal data: the ship's movement for a given hour has a high correlation with the movement of the previous hour (they are close in the data distribution).

Had we chosen random data from the whole dataset to create the testing dataset, the models would have obtained excellent results in testing since they would have been trained with data very similar to that used for testing (belonging to the same ship, similar environmental conditions, and close temporal events). To avoid this, we created the test set by separating the data corresponding to two monitored vessels. The test results should provide a more realistic estimation of how the models will perform in the production environment. 

The two selected vessels have characteristics in the same data distribution used for training: a small general cargo ship (13,000 dwt and 127m length) and a large bulk carrier (71,500 dwt and 225m length). We chose these vessels considering that they should provide enough information to conclude the performance of the models in a production environment, but without reducing too much the training dataset size.

**In summary**: the dataset has one file for each movement. For each movement, one file for training and one for testing. The file naming convention is: 

```
[surge|sway|heave|roll|pitch|yaw]_[train|test].csv
```

#### Variables naming

Each file has the same columns in the same order. To facilitate using the data programmatically, the files use the following names for the variables:

| Variable  | Name in dataset   | Units/format  |
| ---       | ---:      | ---:      |
| Ship name | ship | *ship#*  | 
| Ship type |  type | *"General Cargo"* or *"Bulk carrier"*  |
| Date of stay |  time | MM/DD/YYYY HH:mm |
| Length | length | m | 
| Breath | breath | m | 
| DWT | dwt | tonnes | 
| Berthing zone | zone | [1, 12] | 
| H<sub>s</sub> | h_s | m |
| T<sub>p</sub> | t_p | s | 
| θ<sub>m</sub> | dir | deg | 
| W<sub>s</sub> | wind_speed | km/h | 
| W<sub>d</sub> | wind_dir | deg | 
| H<sub>0</sub> | h_0 | m | 
| H<sub>sm</sub> | h_sm | m |
| Average movement | mov_avg | Surge, sway, and heave: m<br>Roll, pitch, and yaw: deg |
| __Maximum movement__ | mov_max | Surge, sway, and heave: m<br>Roll, pitch, and yaw: deg |
| __Significant movement__ | mov_sig | Surge, sway, and heave: m<br>Roll, pitch, and yaw: deg|

#### Dataset size

| Movement  | Training  | Testing   |
| ---       | ---:      | ---:      |
| Surge     | 364 | 16 |
| Sway      | 1451 | 158 |
| Heave     | 364 | 18 |
| Roll      | 1348 | 51 |
| Pitch     | 1348 | 51 |
| Yaw       | 1248 | 158 |

> **Remember**: each row in each file represents 1 hour.

[1]: https://bancodatos.puertos.es/BD/informes/INT_1.pdf
[2]: https://bancodatos.puertos.es/BD/informes/INT_3.pdf 
[3]: http://urn.kb.se/resolve?urn=urn:nbn:se:umu:diva-153135
[4]: https://doi.org/10.6119/JMST.2018.04_(2).0011
[5]: https://portus.puertos.es/#/

----
[![DOI](https://zenodo.org/badge/387162150.svg)](https://zenodo.org/badge/latestdoi/387162150)

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)
