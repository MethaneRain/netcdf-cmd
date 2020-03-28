# netcdf-cmd
Info on working with Unidata's NetCDF Command Line Tool

--- 

### Merging two netCDF files with same variable along time axis

---

Here is a quick example to merge 2 GFS 20km CONUS ```nc``` files with only MSLP variable along the time axis.

This will combine the ```time``` variables into one variable in the final file. It will also combine the ```MSLP``` data into one variable. The two time variables are 1200 and 1800Z respectively.

Using ``cdo`` command ```mergetime```:

```shell
$ cdo mergetime GFS_CONUS_20km_20200325_1200.grib2.nc GFS_CONUS_20km_20200325_1800.grib2.nc merged.nc
```

Using Python (or any other netCDF file program):

```python
from netCDF4 import Dataset 
ds = Dataset("/Users/chowdahead/Downloads/new2.nc")

ds.variables["time"]

>>>
<class 'netCDF4._netCDF4.Variable'>
float64 time(time)
    standard_name: time
    long_name: GRIB forecast or observation time
    units: Hour since 2020-03-25T12:00:00Z
    calendar: proleptic_gregorian
    axis: T
unlimited dimensions: time
current shape = (2,)
filling on, default _FillValue of 9.969209968386869e+36 used

ds.variables["time"][:]

>>>
masked_array(data=[0., 6.],
             mask=False,
       fill_value=1e+20)
```
