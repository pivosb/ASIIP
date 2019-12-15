# A Stellar Intensity Interferometry Target Planner (ASIIP)
* What this target planner does

-A general purpose Intensity Interferometry target planner that can be used for a given telescope array. It was initally designed for VERITAS but can work with any Intensity Interferometry system. 

-Queries 5 different catalogs with custom constraints defined by the user and constructs a master catalog from them. 
It then cross checks found targets with the Bright Star Catalog and SIMBAD to obtain B magnitudes, radial velocities, 
spectral types and the star id. ASIIP will save any completed master catalog and let the user load it for later analysis

-It then provides an interface which allows users analyze individual stars from the constructed catalog using a 
Monte Carlo analysis, or the user can choose to do a full ranking

-If a full ranking is chosen, using the defined constraints ASIIP analyzes each star in the catalog using a Monto Carlo 
analysis. It does this by repeatedly fitting analytical models to simulated data and then ranking each target by how well
the analytical fits converge to the true angular diameter fit value. 

-Once the ranking is finished, ASIIP allows the user to continue to analyze the ranked targets. It will save any
completed catalog once the user quits.


* Running the Script

Make sure you update your parameter file to match the physical characteristics of your array.

If you need help determining the sigmaTel parameter, the siiInstrumentError.py script can provide a rough estimate.
Simply change the variable in that script to match your SII instrument and it will give an estimate for the error 
associated with a given integration time and target.

This Python script was written using Python 3.7, so a Python 2 compiler might not work as expected. To run the target 
planner, open a terminal (or command prompt) and change your directory to where interferometer.py is located and then 
type this

    python asiip.py

There are some package dependencies that you will need in order to get the script running.

    - astropy (THE MOST UP TO DATE VERSION OF ASTROPY! Older versions use a server that's no longer available)
    - astroquery
    - numpy
    - matplotlib 

All of these should be easily installed using 'pip install (your package)'. If there happens to be any more packages you
need use pip install should easily be able to help you with those as well.


* The Parameter input file

    
    
This is a json file which is used to modify the many differing parameters the script uses to calculate which targets to
analyze. Description of the parameters are given below. If you want to see an example input values, look at the file
IIparameters.json included with this package.
 
    - time(YYYY-MM-DD HH:MM:SS): Specifies the desired date of observation. The target planner expects the midnight time of your array in UTC time
    - raRange(degrees): Specifies the desired RA range query constraints from 0 to 360.
    - decRange(degrees): Specifies the DEC range query constraints from -90 to 90.
    - magRange: Specifies the Magnitude range query constraints
    - wavelength(meters): Specifies the wavelength of the filter being used
    - telLocs(meters): Relative telescope locations of the telescopes you will be using to preform your observation, which will also be used to dynamically calculate the baseline coverage.
    - integrationTime(hour): The length, in hours, of a single observation interval
    - telLat(degree): The central latitude of the telescope array
    - telLon(degree): The central longitude of the telescope array
    - telElv(meters): The central elevation of the telescope array
    - altitudeCutoff(degrees): The lowest possible latitude which can be observed by the given observatory. Targets which never rise above this will be excluded.
    - maxSunAltitude(degrees): The altitude the sun must be below before ASIIP will consider a target observable.
    - observationStart(hours): The desired start time of the observation. An input of 0 starts the observation at midnight, -1.5 starts the observation 1.5 hours before midnight, 1.5 starts the observation 1.5 hours after midnight. An input of null starts the observation as early as possible.
    - observationEnd(hours): The desired end time of the observation.  An input of 0 ends the observation at midnight, -1.5 ends the observation 1.5 hours before midnight, 1.5 ends the observation 1.5 hours after midnight. An input of null ends the observation as late as possible.
    - sigmaTel: The error associated with SII measurements for a given telescope array. This is determined empirically before hand. There is a script included in the ASIIP package which can assist with this calculation.
    - sigmaMag: The magnitude of the star used when empirically determining sigmaTel.
    - sigmaTime(seconds): The integration time used when empirically determining sigmaTel.
    - bootStrapRuns: The amount of Monte Carlo simulations you desire to determine the degeneracy of curve fit. The more bootstrap runs are done, the more accurately the software can rank targets at the cost of computational speed.
    - savePlots: If this is true, ASIIP will save produced plots in a directory. If this is false, ASIIP will simply display them on screen. 
    
   
In the event you want multiple parameter files or you want to change the name of your parameter file you can by running 
the script like this

     python interferometer.py (name of your parameter file).json


* How to interpret the results of the analysis

Once the analysis has run it's course, it will print out a table with data on the input targets. A definition of the
columns is given below

    - Index: This column specifies a unique Index number for each row in the catalog. Use the Index number to choose which star you want to visualize using the graphing software.
    - NAME: The ID given by SIMBAD to the star located at the associated RA and DEC. If SIMBAD failed, it will use the ID given by the bright star catalog.
    - RA(archour): The target J2000 Right Ascension used in ASIIP simulations. 
    - DEC(degree): The target J2000 Declination used in ASIIP simulations.
    - ANGD(mas): The prioritized angular diameter from the corresponding catalog CAT used in ASIIP simulations. The catalog prioritizes the catalogs in the following order: GAIA, JMMC, TESS, CADARS, CHARM2. This prioritization was done to ensure more recent catalog values would be used in ASIIP simulations.
    - DiamMedian(mas): The median diameter calculated from flagged duplicate entry across all of the catalogs.  Caution should be taken when DaimMedian is significantly different from ANGD.
    - DiamStd(mas): The standard deviation of flagged duplicate angular diameter measurement across all of the catalogs. Caution should be taken when DiamStd is large compared to DiamMedian.
    - FILT: The optical bandpass filter that was used when querying the catalog.
    - MAG: The visual magnitude associated with the FILT entry used in ASIIP simulations.
    - CAT: The source catalog for RA, DEC, ANGD, FILT, and MAG.
    - BS\_BMAG: The cross matched Bright Star catalog B magnitude.
    - BS\_VMAG: The cross matched Bright Star catalog V magnitude.
    - BSSkyD: The difference between the input RA and DEC vs the the cross matched Bright Star catalog RA and DEC in mas. If BSSkyD is not 0 it means that the RA and DEC don't match precisely. If this is large, it could indicate a mismatched entry.
    - BSSpT: The spectral type of the cross matched target as given by the Bright Star Catalog.
    - BSRV: The radial velocity of the cross matched target as given by the Bright Star Catalog in km/s.
    - SimBMAG: The SIMBAD catalog B magnitude of the cross matched target.
    - SIMSpT: The spectral type of the cross matched target as given by SIMBAD.
    - SIMSkyD(mas): The difference between the input RA and DEC vs the cross matched SIMBAD RA and DEC. If SIMSkyD is not 0 it means that the RA and DEC don't match precisely. If this is large, it could indicate a mismatched entry.
    - SIMRV(km/s): The radial velocity of the cross matched target as given by SIMBAD.
    - ErrAmp: The calculated error for the given integration time and magnitude.
    - TotObsTime(s): The total available observation time.
    - ObsTime(s): The integrated observation time used in the analysis in seconds used in ASIIP simulations.
    - MeanDiamFit(mas): The mean analytical fit to the simulated Monte Carlo empirical models.
    - PerDiamErr: The percentage error of MeanDiamFit to the true value.
    - PerFitErr: The percentage standard deviation of the custom Monte Carlo simulations, defined by $(1 - \frac{\sigma_{rfit}}{r_{zero}})*100$. This is the main result ASIIP uses to rank targets. Since ASIIP uses an error of 20\% for each guess fit given, If PerFitErr is higher then 20\% then caution should be taken when fitting this data as curve\_fit may simply be converging to a value close to the initial guess value.
    - PerFailFit: The percentage of how many analytical fits failed. If this is not zero, then one or more boot strap runs failed to fit the simulated data and caution should be used before analyzing such a target.
