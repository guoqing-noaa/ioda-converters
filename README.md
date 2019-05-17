# ioda-converters

The intended way to use this repository is to install in `/usr/local` by default, and if you cannot write into `/usr/local` use the `--prefix` option of `ecbuild` to install in your home directory under `tools`.  Run the scripts from the place they are installed, not the source directory. That way the scripts can reference each other without providing a path.

For example,
```
ecbuild --prefix=$HOME/tools  path_to_top_level_cmakelists_file
make
make install
ctest
```
## bufr2nc

Python script, `bufr2nc.py`, for converting BUFR to netCDF4. bufr2nc is built upon the py-ncepbufr package.

```
Usage: bufr2nc.py [-h] [-m <max_num_msgs>] [-c] [-p] obs_type input_bufr output_netcdf
```
  * The idea is for bufr2nc.py to create separate netCDF files for different observation types

The short term plan is to handle aircraft, radiance, radiosonde and GPSRO observation types.
AOD observation type may also be included in the short term plans

Currently supported obs types

| Obs Type           | raw BUFR | prepBUFR |
|:-------------------|:--------:|:--------:|
| Aircraft           | Y        | Y        |
| Radiosonde         | N        | Y        |
| Radiance (AMSU-A)  | Y        | N/A      |
| GPSRO              | Y        | N/A      |
| AOD                | N        | N/A      |

## gsi-ncdiag
These scripts use classes defined in the gsincdiag Python library to convert output from GSI netCDF diag files into
IODA observation files and GeoVaLs for UFO. To run GSI and produce the necessary files, see the feature/files_for_jedi
branch in the ProdGSI repository.

The following executable scripts are to be used by the user:
* `proc_gsi_ncdiag.py`
    * This script uses Python multiprocessing to run multiple instances in separate processes to convert the files in
      parallel.
    * `usage: python proc_gsi_ncdiag.py -n NPROCS -o /path/to/obsout -g /path/to/geovalsout /path/to/diagfiles`
       where NPROCS is the number of parallel processes that will run at one time (should be equal to the number of
       cores on one node.
* `subset_files.py`
    * `usage: subset_files.py -m/-s -n NPROCS /path/to/directory`
    * Subsets all of the files in the input directory to an output of only 100 (m) or 1 (s) location.
    * NPROCS controls how many files can be subsetted at once to speed up the process.
* `combine_conv.py`
    * `usage: combine_conv.py -i /path/to/file1.nc /path/to/filen.nc -o /path/to/outputfile.nc`
    * Finds observations for conventional data at the same locations and combines them from multiple files into one
      output file for additional processing or analysis.
* `test_gsidiag.py`
    * `usage: test_gsidiag.py -i /path/to/inputfile.nc -o /path/to/outdir/ -t conv|rad|aod|oz`
    * A script to convert just a single input GSI diag file into one Obs file and one GeoVaLs file
    * This is called by ctest but can also be used by a user rather than `proc_gsi_ncdiag.py`

For developers, or for those who need to change the names of input/output variables in the scripts, see the README in
src/gsi-ncdiag for details.



## marine
The marine converters all take the following format, with some converters taking additional optional arguments as noted:
 
```
Usage: <converter.py> -i INPUT_FILE(S) -o OUTPUT_FILE -d YYYYMMDDHH
```

* `emc_ice2ioda.py` - Ice concentration observations from EMC. Optional thinning available with `--thin AMOUNT` argument.
* `gds2_sst2ioda.py` - Generic SST/skin-SST converter for use with any GHRSST GD2.0 L2 or L3 data file. Parallel processing of files available with `--threads THREADS` argument. Thinning of data available with `--thin AMOUNT` argument.
* `gmao_obs2ioda.py` - NASA/GMAO ocean observations
* GODAE insitu temperature and salinity ocean profiles from the Fleet Numerical Meteorology and Oceanography Center(FNMOC). Observations available from [here](https://www.usgodae.org/ftp/outgoing/fnmoc/data/ocn/)
  * `godae_profile2ioda.py`
  * `godae_ship2ioda.py`
  * `godae_trak2ioda.py`
* Hybrid-GODAS - preprocessed, suberobbed, and QCd observations of altimetry, insitu T/S, and SST. Used in the NCEP HGODAS project _(these are likely to be removed at some point)_
  * `hgodas_adt2ioda.py`
  * `hgodas_insitu2ioda.py`
  * `hgodas_sst2ioda.py`  
* `rads_adt2ioda.py` - absolute dynamic topography observations from NOAA/NESDIS. Observations available from `ftp://ftp.star.nesdis.noaa.gov/pub/sod/lsa/rads/adt`
* `smap_sss2ioda.py` - SMAP satellite sea surface salinity observations. Observations available from `ftp://podaac-ftp.jpl.nasa.gov/allData/smap/L2/RSS/V3/SCI`

* `ncep_classes.py` - Convert (prep-)BUFR with embedded BUFR table to IODA format. See [here](src/ncep/README.md) for usage.


## odbapi2nc

Python script, `odbapi2nc.py`, for converting Met Office or ECMWF ODB2 files to netCDF4 files formatted for use by IODA.
```
Usage: odbapi2nc.py [-h] [-c] [-q] [-t] [-v] [-b] input_odb2 definition_yaml output_netcdf
```
Definition YAML files currently created and tested:
* Met Office Radiosonde
* Met Office Aircraft
* Met Office AMSU-A from atovs report
* ECMWF Radiosonde
* ECMWF Aircraft

## odbapi2json

Python script, `odbapi2json.py`, for converting Met Office ODB2 files to JSON files which can be used to load the data
to MongoDB.

This script used to work, but is currently not being maintained and no longer does. The code is being kept as a starting point if we want to update it later.
```
Usage: odbapi2json.py [-h] [-c] [-q] input_odbapi output_temp > output.json
```

