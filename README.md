# About
This SDK is meant to help users with basic queries to WattTime’s API (version 3), and to get data returned in specific formats (e.g., JSON, pandas, csv).

Users must first [register for access to the WattTime API here](https://watttime.org/docs-dev/data-plans/).

We encourage users to fork and contribute pull requests to this repository if they find themselves requiring additional capabilities or data formats. Users may also submit technical issues here relating to the code contained within this SDK, however for questions about WattTime's API, data availability, or access please direct questions to [support@watttime.org](support@watttime.org). You may also [view the status of WattTime's API here](https://status.watttime.org/).

Full documentation of WattTime's API, along with response samples and information about [available endpoints is also available](https://docs.watttime.org/).

# Configuration
The SDK can be installed as a python package, we recommend using an environment manager such as [miniconda](https://docs.conda.io/projects/miniconda/en/latest/) or [venv](https://docs.python.org/3/library/venv.html).
```
git clone git@github.com:WattTime/watttime-python-client.git
pip install watttime-python-client/
```

Once registered for the WattTime API, you may set your credentials as environment variables to avoid passing these during class initialization:
```
# linux or mac
export WATTTIME_USER=<your WattTime API username>
export WATTTIME_PASSWORD=<your WattTime API password>
```

# Using the SDK
Users may first want to query the `/v3/my-access` endpoint using the `WattTimeMyAccess` class to get a dataframe of regions and signal types available to them:

```
from watttime_sdk import WattTimeMyAccess

wt_myaccess = WattTimeMyAccess(username, password)

# return a nested json describing signals and regions you have access to
wt_myaccess.get_access_json()

# return a pandas dataframe describing signals and regions you have access to
wt_myaccess.get_access_pandas()
```

Once you confirm your access, you may wish to request data for a particular balancing authority:
```
from watttime_sdk import WattTimeHistorical

wt_hist = WattTimeHistorical(username, password)

# get data as a pandas dataframe
moers = wt_hist.get_historical_pandas(
    start = '2022-01-01 00:00Z', # ISO 8601 format, UTC
    end = '2023-01-01 00:00Z', # ISO 8601 format, UTC
    region = 'CAISO_NORTH',
    signal_type = 'co2_moer' # ['co2_moer', 'co2_aoer', 'health_damage', etc.]
)

# save data as a csv -> csvs/<region>_<signal_type>_<start>_<end>.csv
wt_hist.get_historical_csv(
    start = '2022-01-01 00:00Z', # ISO 8601 format, UTC
    end = '2023-01-01 00:00Z', # ISO 8601 format, UTC
    region = 'CAISO_NORTH',
    signal_type = 'co2_moer' # ['co2_moer', 'co2_aoer', 'health_damage', etc.]
)
```

You could also combine these classes to iterate through all regions where you have access to data:
```
from watttime_sdk import WattTimeMyAccess, WattTimeHistorical
import pandas as pd

wt_myaccess = WattTimeMyAccess(username, password)
wt_hist = WattTimeHistorical(username, password)

access_df = wt_myaccess.get_access_pandas()

moers = pd.DataFrame()
moer_bas = access_df.loc[access_df['signal_type] == 'co2_moer', 'region'].unique()
for ba in moer_bas:
    ba_df = wt_hist.get_historical_pandas(
        start = '2022-01-01 00:00Z',
        end = '2023-01-01 00:00Z',
        region = ba,
        signal_type = 'co2_moer'
    )
    moers = pd.concat([moers, ba_df], axis='rows')
```

You can also use the SDK to request a current forecast for some signal types, such as co2_moer and health_damage:
```
from watttime_sdk import WattTimeForecast

wt_forecast = WattTimeForecast(username, password)
forecast = wt_forecast.get_forecast_json(
    region = 'CAISO_NORTH',
    signal_type = 'health_damage'
)

```

Methods also exist to request historical forecasts, however these responses may be slower as the volume of data can be significant:
```
hist_forecasts = wt_forecast.get_historical_forecast_json(
    start = '2022-12-01 00:00+00:00',
    end = '2022-12-31 23:59+00:00',
    region = 'CAISO_NORTH',
    signal_type = 'healdh_damage'
)
```