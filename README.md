# netcdf-cmd
Info on working with Unidata's NetCDF Command Line Tool

### Taking a quick looks at netCDF file info 

This will quickly show the variables and their attributes 

The ```ncdump``` command will produce all the info on the file, but if ran with the ```-h``` flag, only the variables metadata will be displayed (not the actual values)

```shell
$ ncdump -h GFS_CONUS_20km_20200325_1200.grib2.nc

>>>
netcdf GFS_CONUS_20km_20200325_1200.grib2 {
dimensions:
	time = 1 ;
	y = 257 ;
	x = 369 ;
variables:
	float MSLP_Eta_model_reduction_msl(time, y, x) ;
		MSLP_Eta_model_reduction_msl:long_name = "MSLP (Eta model reduction) @ Mean sea level" ;
		MSLP_Eta_model_reduction_msl:units = "Pa" ;
		MSLP_Eta_model_reduction_msl:abbreviation = "MSLET" ;
		MSLP_Eta_model_reduction_msl:missing_value = NaNf ;
		MSLP_Eta_model_reduction_msl:grid_mapping = "LambertConformal_Projection" ;
		MSLP_Eta_model_reduction_msl:coordinates = "time y x " ;
		MSLP_Eta_model_reduction_msl:Grib_Variable_Id = "VAR_0-3-192_L101" ;
		MSLP_Eta_model_reduction_msl:Grib2_Parameter = 0, 3, 192 ;
		MSLP_Eta_model_reduction_msl:Grib2_Parameter_Discipline = "Meteorological products" ;
		MSLP_Eta_model_reduction_msl:Grib2_Parameter_Category = "Mass" ;
		MSLP_Eta_model_reduction_msl:Grib2_Parameter_Name = "MSLP (Eta model reduction)" ;
		MSLP_Eta_model_reduction_msl:Grib2_Level_Type = 101 ;
		MSLP_Eta_model_reduction_msl:Grib2_Level_Desc = "Mean sea level" ;
		MSLP_Eta_model_reduction_msl:Grib2_Generating_Process_Type = "Forecast" ;
	double time(time) ;
		time:units = "Hour since 2020-03-25T12:00:00Z" ;
		time:standard_name = "time" ;
		time:long_name = "GRIB forecast or observation time" ;
		time:calendar = "proleptic_gregorian" ;
		time:_CoordinateAxisType = "Time" ;
	float y(y) ;
		y:standard_name = "projection_y_coordinate" ;
		y:units = "km" ;
		y:_CoordinateAxisType = "GeoY" ;
	float x(x) ;
		x:standard_name = "projection_x_coordinate" ;
		x:units = "km" ;
		x:_CoordinateAxisType = "GeoX" ;
	int LambertConformal_Projection ;
		LambertConformal_Projection:grid_mapping_name = "lambert_conformal_conic" ;
		LambertConformal_Projection:latitude_of_projection_origin = 25. ;
		LambertConformal_Projection:longitude_of_central_meridian = 265. ;
		LambertConformal_Projection:standard_parallel = 25. ;
		LambertConformal_Projection:earth_radius = 6371229. ;
		LambertConformal_Projection:_CoordinateTransformType = "Projection" ;
		LambertConformal_Projection:_CoordinateAxisTypes = "GeoX GeoY" ;
	double lat(y, x) ;
		lat:units = "degrees_north" ;
		lat:long_name = "latitude coordinate" ;
		lat:standard_name = "latitude" ;
		lat:_CoordinateAxisType = "Lat" ;
	double lon(y, x) ;
		lon:units = "degrees_east" ;
		lon:long_name = "longitude coordinate" ;
		lon:standard_name = "longitude" ;
		lon:_CoordinateAxisType = "Lon" ;

// global attributes:
		:Originating_or_generating_Center = "US National Weather Service, National Centres for Environmental Prediction (NCEP)" ;
		:Originating_or_generating_Subcenter = "0" ;
		:GRIB_table_version = "2,1" ;
		:Type_of_generating_process = "Forecast" ;
		:Analysis_or_forecast_generating_process_identifier_defined_by_originating_centre = "Analysis from GFS (Global Forecast System)" ;
		:Conventions = "CF-1.6" ;
		:history = "Read using CDM IOSP GribCollection v3" ;
		:featureType = "GRID" ;
		:History = "Translated to CF-1.0 Conventions by Netcdf-Java CDM (CFGridWriter2)\n",
			"Original Dataset = /data/ldm/pub/native/grid/NCEP/GFS/CONUS_20km/GFS_CONUS_20km_20200325_1200.grib2.ncx3#LambertConformal_257X369-40p69N-100p4W; Translation Date = 2020-03-28T15:51:13.464Z" ;
		:geospatial_lat_min = 12.0794601004802 ;
		:geospatial_lat_max = 57.3359502197929 ;
		:geospatial_lon_min = -153.034550162116 ;
		:geospatial_lon_max = -49.2052976797846 ;
}

```

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
ds = Dataset("merged.nc")

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

ds.variables["MSLP_Eta_model_reduction_msl"]

>>>
<class 'netCDF4._netCDF4.Variable'>
float32 MSLP_Eta_model_reduction_msl(time, y, x)
    long_name: MSLP (Eta model reduction) @ Mean sea level
    units: Pa
    grid_mapping: LambertConformal_Projection
    _FillValue: nan
    missing_value: nan
    abbreviation: MSLET
    Grib_Variable_Id: VAR_0-3-192_L101
    Grib2_Parameter: [  0   3 192]
    Grib2_Parameter_Discipline: Meteorological products
    Grib2_Parameter_Category: Mass
    Grib2_Parameter_Name: MSLP (Eta model reduction)
    Grib2_Level_Type: 101
    Grib2_Level_Desc: Mean sea level
    Grib2_Generating_Process_Type: Forecast
unlimited dimensions: time
current shape = (2, 257, 369)
filling on
```
