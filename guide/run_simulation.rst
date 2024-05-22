.. _run_simulations:

Running simulations
===================

pytrnsys runs TRNSYS simulations based on configuration files with the extension ``.config``. The general idea behind
this is to provide a fast and easily accessible way to define, run and analyse both single simulations as well as
parametric studies.

.. note::
    The configuration file does not require a header. It should contain different keyword commands on single lines.
    Comments start with ``#`` characters. End of line comments are possbile.

The existing commands are listed per topic in the following.

Simulation control
------------------

``outputLevel``
    Output message level according to the logging package. (Options: "DEBUG", "INFO" (default), "WARNING", "ERROR", and
    "CRITICAL")::

        string outputLevel "INFO"

``ignoreOnlinePlotter``
    If set to True, the TRNSYS online plotters are commented out in all the dck-files. No online plotters are shown
    during the simulation run. The TRNSYS progress bar window is still displayed (default: False)::

        bool ignoreOnlinePlotter False
``keepOnlinePlotter``
    If set to True, the TRNSYS online plotters will be left open until they are closed manually. 
    This requires ignoreOnlinePlotter to be set to False at the same time (default: False)::

    bool keepOnlinePlotter True

``removePopUpWindow``
    Online plotters as well as the progress bar window are suppressed during the simulations, which corresponds to the
    TRNSYS hidden mode (default: False)::

        bool removePopUpWindow False

``reduceCpu``
    When running simulations in parallel on different logical cores of your CPU, you can specify how many cores will be
    left unoccupied (default: 0)::

        int reduceCpu 0

``checkDeck``
    If set to True, during merging the ddck-files, the specified and given amount of Equations and Parameters in
    each block are checked for inconsistencies (default: True)::

        bool checkDeck True

``parseFileCreated``
    Saves the parsed dck-file that can be used to locate the line where ``checkDeck`` found errors (default: True)::

        bool parseFileCreated True

``runCases``
    If set to False, the dck-files are created and saved in the normal structure but not executed (default: True)::

        bool runCases True

``doAutoUnitNumbering``
    If set to True, the units of the merged dck-file are renumbered to avoid duplicates. This parameter should usually
    be set to the default True::

        bool doAutoUnitNumbering True

``generateUnitTypesUsed``
    If set to True, a file called ``UnitType.info`` containing the TRNSYS-Type numbers used is saved in the main
    run-folder (default True)::

        bool generateUnitTypesUsed True

``addAutomaticEnergyBalance``
    If set to True, an automatic energy balance printer is created in the dck file. For more information
    see :ref:`ref-defaultPlotting` (default: True)::

        bool addAutomaticEnergyBalance True

``nameRef``
    Base name of the dck-file(s) created. Default base name is "pytrnsysRun"::

        string nameRef "pytrnsysRun"

``rerunCases``
    TBD::

        bool rerunCases False

``runFromCases``
    TBD::

        bool runFromCases False

``copyBuildingData``
    TBD::

        bool copyBuildingData False

``runType``
    TBD (Options: "runFromConfig" (default), "runFromCases", "runFromFolder")::

        string runType "runFromConfig"

``pathToConnectionInfo``
    Specify the path to the DdckPlaceHolderValues.json, which you would like to use to replace the placeholders in the
    ddck-files. It overrules the behaviour of replacing the placeholders with the defaults::

        string pathToConnectionInfo "path-to-your-project-folder\DdckPlaceHolderValues.json"

Tracking
--------

``trackingFile``
    When running multiple simulations the status of each simulation can be tracked with the help of a json-file. When
    a simulation is started, this is entered with a timestamp into this file. Once the simulation is finished this entry
    will be overwritten accordingly. Like this one can keep track of which simulations were aborted (for whatever
    reason) after having been launched. To activate this functionality you need to specify the full path of the
    json-file to be created::

        string trackingFile "...\[name].json"

``masterFile``
    If several simulations are run from different instances, the tracking can be taken one step further by employing
    a "master-file" in the form of a csv-file. It also tracks the status of different simulations based on the tracking
    json-files. One important feature is that, when it is used, simulations (identified by the name of the dck-file)
    that are already entered as a "success" won't be run again. This is useful for redoing parametric studies where
    single simulations failed. If this is the case one can do the needed corrections and then simply launch the same
    parametric study again and the "master-file" will ensure that no unnecessary repetitions of simualtions are
    executed. To activate this functionality you need to specify the full path of the csv-file to be created::

        string masterFile "...\[name].csv"

Paths
-----

``trnsysExePath``
    Specify the path to the exe-file of TRNSYS, which you would like to use to run the simulations::

        string trnsysExePath "C:\TRNSYS18\Exe\TrnEXE.exe"

``pathBaseSimulations``
    If specified, the location of where the simulation is run is changed to the given path. It overrules the normal
    behavior of executing the simulations in the current working directory::

        string pathBaseSimulations "path-to-your-simulation-folder"

``addResultsFolder``
    Specify the path to which you would like to save your simulation results::

        string addResultsFolder "path-to-your-results-folder"

Definition of path alias
    You can define an alias for a path to be used in a different place. If, e.g., you want to load many ddck-files from
    "C:\\GIT\\pytrnsys\\data\\ddcks" you can give this path an alias such as "PYTRNSYS$". The "$" at the end of the alias
    needs to be included always to mark it as such::

        string PYTRNSYS$ "C:\GIT\pytrnsys\data\ddcks"

Scaling
-------

.. _ref-scaling:

``scaling``
    If this is set to "toDemand" the scaling functionality is activated for the parameter variation. (Options: "False"
    (default), "toDemand")::

        string scaling "False"

``scaleHP``
    Specify the size of the heat pump of the system in kW through some numerical value ``x`` and some ``variable`` that
    is defined in the ``scalingReference`` file (see below)::

        string scaleHP "x*variable"

.. _ref-scalingVariable:

``scalingVariable``
    This defines a variable from the ``scalingReference`` file (see below) that is used for scaling the parameter
    variations::

        string scalingVariable "name-of-your-scaling variable"

``scalingReference``
   Full path to a json-file containing the variables used for scaling (see above)::

        string scalingReference "...\[name].json"

Parameter variation
-------------------

A core feature of pytrnsys is the parameter variation. pytrnsys allows either to modify TRNSYS simulation parameters in
the configuration file statically or with variations that result in parametric runs.

``deck``
    A certain ``trnsysVariable`` defined in the dck-file can be set to a certain ``value`` by overwriting the previous
    one through::

        deck trnsysVariable value

    This feature can for example be used to change the starting time of the simulation(s)::

        deck START 4344

``variation``
    A parametric study, i.e. several TRNSYS simulations with different ``values`` for a certain ``trnsysVariable``, can
    be launched with the following command::

        variation variationName trnsysVariable value1 value2 value3 ...

    Here, ``variationName`` defines how the variation will be noted in the names of the dck-files to be generated. In
    general the ``values`` are absolute values of the respective ``trnsysVariable``. If :ref:`scaling <ref-scaling>`
    is set to "toDemand", however, then the ``values`` are the factors by which the ``scalingVariable`` is mulitplied to
    receive the actual numerical value of the ``trnsysVariable``. If, e.g., the ``scalingVariable`` is the yearly heat
    demand of a system in MWh and the ``trnsysVariable`` to be varied is the area of the solar collector field called
    ``AcollAp`` in m2, then this area can be varied as multiples of the yearly heat demand (in m2/MWh) like this::

        variation Ac AcollAp 1.0 1.5 2.0 2.5 3.0

``combineAllCases``
    If several ``variations`` are defined, this parameter controls their combination. If it is set to the default True,
    all combinations are created. So if n values are given for variation 1 and m values are in variation 2 the total
    amount of simulations executed will be (m x n). If it is set to False, the amount of values of all variations has to
    be equal and they are combined according to their order::

        bool combineAllCases True

``changeDdckFile``
    Instead of only varying one or more variable, due to its modular nature, pytrnsys also allows to vary through
    different ddck-files::

        changeDDckFile originalDdck ddckVariation1 ddckVariation2 ddckVariation3 ...

    This can be used, e.g., to change the weather location for a simulation very swiftly. Assuming the weather data is
    specified through a file of the type ``City..._dryK`` and the locations ``BAS``, ``CDF`` and ``LUG`` should be
    simulated, this can be done with the command::

        changeDDckFile CityBAS_dryK CityBAS_dryK CityCDF_dryK CityLUG_dryK
        
random variations
~~~~~~~~~~~~~~~~~
If the influence of many different parameters is of interest, random variations might be needed. Random variations can be done with the keywords ``randvar``, ``randvarddck``, ``nrandvar`` and ``randseed``. If using random variations, do not include any regular variations and be aware, that only one ddck can be changed with ``randvarddck``.

``randvar``
    Random variations of trnsys constants can be executed by adding one or multiple lines like the following::

        randvar variationName trnsysVariable minValue maxValue stepSize

    Here, ``variationName`` defines how the variation will be noted in the names of the dck-files to be generated. The ``minValue`` and ``maxValue`` are the minimum and maximum Value that the ``trnsysVariable`` will take, respectively. ``stepSize`` describes the step size of the values that can be taken between ``minValue`` and ``maxValue``. Make sure that ``maxValue`` = ``minValue`` + ``n`` * ``stepSize``. Where ``n`` is an integer. In the following example, the storage size will be taken between 0.5 m3 and 1 m3 in steps of 0.1, so it will have the options 0.5, 0.6, 0.7, 0.8, 0.9, 1::
    
        randvar Vtes storageSize 0.5 1 0.1

    
``randvarddck``
    Random variations of ddcks can be included with the following command::
    
        randvarddck originalDdck ddckVariation1 ddckVariation2 ddckVariation3 ... ddckVariationn
        
    For every iteration pytrnsys then takes randomly one of the n ``ddckVariation`` instead of the ``originalDdck``, that is specified in the used ddcks section. Currently, only one ddck can be randomly varied.
    
``nrandvar``
    This keyword describes the total number of random variations to be simulated, if e.g. 1000 variations should be simulated, the following line has to be added::
    
        nrandvar 1000

    Default value of nrandvar is 100.

``randseed``
    This keyword describes an integer, that is used as a seed. If a seed is set, then rerunning a simulation will yield the same variations. A different integer will yield different random variations.

    Example::

        randseed 1

    If randseed is not defined or of it is set to None, then the variations will be different every time.


ddck files
----------

The core of the run configuration file is the ddck section. In this part of the configuration file, the different
modular ddck files that should be used in the simulation are specified::

    PATH_ALIAS_1$ head
    PATH_ALIAS_2$ ddck_1
    PATH_ALIAS_2$ ddck_2
    ...
    PATH_ALIAS_m$ ddck_n
    PATH_ALIAS_1$ end

An example can be found in the example section below. The path to the repository root can be either absolute or
relative. If a relative path is detected, pytrnsys will interpret it as relative to the configuration file location.

Example
-------
Here is an example of a run configuration file. It is taken from the example project solar_dhw
(``run_solar_dhw.config``)::

    ######### Generic ########################
    bool ignoreOnlinePlotter  True
    int reduceCpu  4
    bool parseFileCreated True
    bool runCases True
    bool checkDeck True

    ############# AUTOMATIC WORK BOOL##############################

    bool doAutoUnitNumbering True
    bool generateUnitTypesUsed True
    bool addAutomaticEnergyBalance True

    #############PATHS################################

    string trnsysExePath "C:\Trnsys17\Exe\TRNExe.exe"
    string addResultsFolder "solar_dhw"
    string PYTRNSYS$ "..\..\pytrnsys_ddck\"
    string LOCAL$ ".\"

    ################SCALING#########################

    string scaling "False" #"toDemand"
    string nameRef "SFH_DHW"
    string runType "runFromConfig"

    #############PARAMETRIC VARIATIONS##################

    bool combineAllCases True
    variation Ac AcollAp 2 3 4 6 8 10
    variation VTes volPerM2Col 75 100

    #############FIXED CHANGED IN DDCK##################

    deck START 0    # 0 is midnight new year
    deck STOP  8760 #
    deck sizeAux 3

    #############USED DDCKs##################

    PYTRNSYS$ generic\head
    PYTRNSYS$ demands\dhw\dhw_sfh_task44
    PYTRNSYS$ weather\weather_data_base
    PYTRNSYS$ weather\SIA\normal\CitySMA_dryN
    PYTRNSYS$ solar_collector\type1\database\type1_constants_CobraAK2_8V
    PYTRNSYS$ solar_collector\type1\type1
    LOCAL$ solar_dhw_control
    LOCAL$ solar_dhw_storage1
    LOCAL$ solar_dhw_hydraulic
    LOCAL$ solar_dhw_control_plotter
    PYTRNSYS$ generic\end
