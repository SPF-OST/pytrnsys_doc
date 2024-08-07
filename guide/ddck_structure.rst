.. _ddck_structure:

Structure of a ddck-file
========================

The file sctructure of the ddck files is shown in the example.ddck file in the pytrnsys_ddck repository root.
Each ddck should contain the following parts. Some parts can be empty if not used.

Local and global variables
--------------------------

Pytrnsys knows two kinds of variables. Consider::

    EQUATIONS 1
    heatingOn = LT($Tamb, 15)

Here, `heatingOn` is a "local" variable, i.e., it can only be accessed from within the current ddck
but not from other ddcks. `$Tamb`, on the other hand, is a "global" variable: it can be accessed from
any ddck. You can also define your own global variables; just prefix them with a dollar sign like so:
`$myGlobalVariable`.

If you really need to access a local variable from outside its ddck (e.g. to change a solar collector's area from the
run.config) you need to prefix the variable with the component's name. I.e., if your collector is called `Coll` and its
area is called `A` locally, you can access the variable from outside by the name `CollA`.

Header
------
In the header the name of the ddck as well as a contact person, the creation date and some information
about the last changes are specified. In addition, there is a section for a general description of the ddck::

    *******************************
    **BEGIN example.ddck
    *******************************

    *****************************************
    ** Contact person : author
    ** Creation date  : date
    ** Last changes   : date, name
    *****************************************

    ***************************************************************************
    ** Description:
    ** Overall description of the ddck file
    ** TODO: Any improvements needed
    ***************************************************************************

Inputs from hydraulic solver
-----------------------------
The main inputs to a particular component/ddck are the input mass flow rates and the input temperatures for each input port.
There is a special syntax to access these values. To access the input temperature and mass flow rates at the condenser and
evaporator inlet (called `Cond` and `Evap`, respectively, in this example) within a heat pump ddck file, e.g., do the following::

    ***********************************
    ** inputs from hydraulic solver
    ***********************************

    TCondIn = @temp(CondIn) ! Condenser input temperature
    MCond = @mfr(CondIn)  ! Condenser mass flow rate
    
    TEvapIn = @temp(EvapIn) ! Evaporator input temperature
    MEvap = @mfr(EvapIn)  ! Evaporator mass flow rate

Besides temperature (`@temp`) and mass flow rate (`@mfr`) you can also access the heat capacity (`@cp`) and the density
(`@rho`) of the fluid circulating through a given port in the same way as shown above.

Outputs to the hydraulic solver
-------------------------------
The main outputs of a component/ddck are the temperatures at its output ports. These can be
specified as follows (again using the heat pump example [see above])::

    ***********************************
    ** outputs to hydraulic solver
    ***********************************

    @temp(CondOut) = [20,2]
    @temp(EvapOut) = [20,4]

Here, `20` would be the unit number of your heat pump type and `2` and `4` the type's outputs corresponding
to the condenser and evaporator output temperatures.

Outputs to the energy balance
-----------------------------
Pytrnsys can automatically compute an energy balance of your system. In order to do that
it needs to be told what should go into the energy balance. You can tell pytrnsys about your component/ddck's
contribution to the energy balance like so::

    ******************************************************************************************
    ** outputs to energy balance in kWh and ABSOLUTE value
    ******************************************************************************************

    @energy(heat, in, cond) = [20, 6]   ! heat going into the system and out of our HP component
    @energy(heat, out, evap) = [20, 8]  ! heat going out of the system and into our HP component

The `in` and `out` are from the system's perspective: `out` would be out of the system and into a storage, e.g.
(i.e. when the storage is charged `@energy(heat, out, ...)` would be positive).

Dependencies with other ddck files
----------------------------------
In order to enhance modularization, dependencies with other ddcks should be kept minimal. Dependencies that
cannot be avoided and are neither part of the component-database relation or the general variables should be
declarated and reassigned to an internally used variable in this part::

    ***********************************
    ** Dependencies with other ddck
    ***********************************

    ** Re-assing here the variables necessary from other types
    ** variableInternal = variableExternal
    ** Exception: those from general variables

Outputs to other ddck files
---------------------------
Variables that are designated to be used in other ddck files should be added here::

    ***********************************
    ** outputs to other ddck
    ***********************************

    ** Add here the outputs of the TYPE or TYPES that will be used in other types
    ** Exception: those for printers and so on dont need to be here.

Precalculations related to parameter scaling and pre-processing
---------------------------------------------------------------
Usually, in the declaration of a TRNSYS component, many parameters are calculated out of more general
system variables. All calculations to determine the right parameters inputs for the type go here::

    ***********************************
    ** Begin CONSTANTS
    ***********************************

Type section
------------
TRNSYS has its own syntax that calls the type dll files. This core part of the ddck goes here::

    ***********************************
    ** Begin TYPE
    ***********************************

Component printers
------------------
Each component should have a monthly as well as an hourly printer. This helps to simplify the setup
and the processing of the simulation. In addition, an online plotter is a nice tool for the debugging
of the system::

    ***********************************
    ** Monthly printer
    ***********************************

    ***********************************
    ** Hourly printer
    ***********************************

    ***********************************
    ** Online plotter
    ***********************************

Hydraulics files
----------------

The hydraulics file represents the systems hydraulics layout. Each pytrnsys example system except
the pv battery system has its own hydraulic layout file. In order to create your own hydraulic files
that represent the hydraulics of your choice you need access to the pytrnsys GUI. The hydraulics file
are not part of the ddck repository. The hydraulic files of the example systems are located in the
example system folder of **pytrnsys_examples**.

Examples
--------
The following example shows the ddck file of the solar collector type 1 used in the solar domestic
hot water system::

    *******************************
    **BEGIN Type1.ddck
    *******************************

    *****************************************
    ** Contact person : Dani Carbonell
    ** Creation date  : 10.01.2010
    ** Last changes   : 03.2020 Jeremias Schmidli
    *****************************************

    ***************************************************************************
    ** Description:
    ** Collector model using efficiency curve efficiency
    ***************************************************************************

    ***********************************
    ** inputs from hydraulic solver
    ***********************************

    EQUATIONS 2
    TCollIn = @temp(CollIn)
    MfrColl = @mfr(CollIn)

    ***********************************
    ** outputs to hydraulic solver
    ***********************************

    EQUATIONS 1
    TCollOut = [28,1]

    ***********************************
    ** outputs to other ddck
    ***********************************

    ******************************************************************************************
    ** outputs to energy balance in kWh and ABSOLUTE value
    ** Following this naming standard : qSysIn_name, qSysOut_name, elSysIn_name, elSysOut_name
    ******************************************************************************************

    EQUATIONS 1
    qSysIn_Collector = PColl_kW

    ***********************************
    ** Dependencies with other ddck
    ***********************************

    EQUATIONS 1
    pumpColOn = puColOn

    CONSTANTS 2
    C_tilt = slopeSurfUser_1  ! @dependencyDdck Collector tilt angle / slope [°]
    C_azim = aziSurfUSer_1    ! @dependencyDdck Collector azimuth  (0:s, 90:w, 270: e) [°]

    EQUATIONS 4
    **surface-8
    IT_Coll_kJhm2 = IT_surfUser_1  ! Incident total radiation on collector plane, kJ/hm2
    IB_Coll_kJhm2 = IB_surfUser_1  ! incident beam radiation on collector plane, kJ/hm2
    ID_Coll_kJhm2 = ID_surfUser_1  ! diffuse and ground reflected irradiance on collector tilt
    AI_Coll = AI_surfUser_1  ! incident angle on collector plane, °

    EQUATIONS 5
    IT_Coll_kW = IT_Coll_kJhm2/3600     ! Incident total radiation on collector plane, kW/m2
    IB_Coll_kW = IB_Coll_kJhm2/3600     ! incident beam radiation on collector plane, kW/m2
    ID_Coll_kW = ID_Coll_kJhm2/3600     ! diffuse and ground reflected irradiance on collector tilt (kW/m2)
    IT_Coll_Wm2 = IT_surfUser_1/3.6
    IT_Coll_kWm2 = IT_surfUser_1/3600

    ***********************************
    ** Begin CONSTANTS
    ***********************************

    CONSTANTS 3
    MfrCPriSpec = 15  ! Coll. Prim. loop spec. mass flow [kg/hm2]
    AcollAp=5         ! Collector area
    MfrCPriNom = MfrCPriSpec*AcollAp !

    ***********************************
    ** Begin TYPE
    ***********************************

    UNIT 28 TYPE 1
    PARAMETERS 11
    nSeries       ! number in series
    AcollAp       ! collector area
    cpBri          ! fluid specific heat kj(kgK
    efficiencyMode ! efficiency mode
    testedMfr      ! tested flow rate kg/(hm2)
    Eta0          ! intercept efficiency
    a1            ! efficiency slope kJ/hm^2K
    a2            ! efficiency curvature kJ/hm^2K^2
    2             ! optical mode
    FirstOrderIAM  ! 1st order IAM
    SecondOrderIAM ! 2nd order IAM
    INPUTS 9
    TCollIn
    MfrColl
    Tamb
    IT_Coll_kJhm2
    IT_H
    ID_Coll_kJhm2
    0,0
    AI_Coll !Flo check ! JS: This was defined wrong before (C_azim, even though it is incident angle input). Now it should be correct.
    C_tilt !Flo check  ! JS: This should be correct
    *** INITIAL INPUT VALUES
    20 0 10 0 0 0 GroundReflectance 45 0

    EQUATIONS 4
    **MfrCout = [700,2]
    Pcoll = [28,3] !kJ/h
    PColl_kW = Pcoll/3600
    PColl_kWm2 = PColl_kW/(AcollAp+1e-30)
    PColl_Wm2  = PColl_kWm2*1000


    ***********************************
    ** Monthly printer
    ***********************************

    CONSTANTS 1
    unitPrintSol = 31

    ASSIGN temp\SOLAR_MO.Prt unitPrintSol

    UNIT 32 TYPE 46
    PARAMETERS 6
    unitPrintSol ! 1: Logical unit number, -
    -1           ! 2: Logical unit for monthly summaries, -
    1            ! 3: Relative or absolute start time. 0: print at time intervals relative to the simulation start time. 1: print at absolute time intervals. No effect for monthly integrations
    -1           ! 4: Printing & integrating interval, h. -1 for monthly integration
    1            ! 5: Number of inputs to avoid integration, -
    1            ! 6: Output number to avoid integration
    INPUTS 4
    Time  Pcoll_kW  PColl_kWm2  IT_Coll_kWm2
    **
    Time  Pcoll_kW  PColl_kWm2  IT_Coll_kWm2

    ***********************************
    ** Hourly printer
    ***********************************

    CONSTANTS 1
    unitHourlyCol = 33

    ASSIGN    temp\SOLAR_HR.Prt    unitHourlyCol

    UNIT 34 TYPE 46     ! Printegrator Monthly Values for System
    PARAMETERS 7
    unitHourlyCol ! 1: Logical unit number, -
    -1            ! 2: Logical unit for monthly summaries, -
    1             ! 3: Relative or absolute start time. 0: print at time intervals relative to the simulation start time. 1: print at absolute time intervals. No effect for monthly integrations
    1             ! 4: Printing & integrating interval, h. -1 for monthly integration
    2             ! 5: Number of inputs to avoid integration, -
    4             ! 6: Output number to avoid integration
    5             ! 7: Output number to avoid integration
    INPUTS 6
    Pcoll_kW  PColl_kWm2  IT_Coll_kWm2 TCollOut TCollIn MfrColl
    **
    Pcoll_kW  PColl_kWm2  IT_Coll_kWm2 TCollOut TCollIn MfrColl


A specific parametrization can be added by using a ddck from the database for example the
type1_CONSTANTS_cOBRAak2_8v.ddck::

    ******************************
    **BEGIN Type1_Constants_CobraAK2_8V.ddck
    *******************************

    *****************************************
    ** Solar Thermal Data for covered collector.
    ** Very well performing collector Cobra AK 2.8V
    ** Version : v0.0
    ** Last Changes: Jeremias Schmidli
    ** Date: 10.03.2020
    ******************************************

    CONSTANTS 11

    Eta0= 0.857     ! Eta0 (a0) of collector (zero heat loss efficiency)
    a1 = 4.16*3.6    ! linear heat loss coefficient of collector [kJ/hm^2K] ![W/m2K]*3.6
    a2 = 0.0089*3.6   ! quadratic heat loss coefficient of collector [kJ/hm^2K^2] ![W/m2K2]*3.6

    AbsorberArea = 2.435 !m2
    TotArea = 2.768 !m2

    nSeries = 1
    efficiencyMode = 1
    testedMfr = 200/AbsorberArea !l/hm2

    GroundReflectance = 0.2

    FirstOrderIAM = 0.108
    SecondOrderIAM = 0
    *******************************
    **END Type1_Constants_Test.ddck
    *******************************


Placeholder statements with specific syntax can be added to the ``inputs from hydraulic solver`` and ``Outputs to the 
hydraulic solver`` in ddck::

    *******************************
    **BEGIN Type1.ddck 
    *******************************
    
    *****************************************
    ** Contact person : Dani Carbonell    
    ** Creation date  : 10.01.2010
    ** Last changes   : 18.05.2022
    *****************************************
    
    ***************************************************************************
    ** Description: 
    ** Collector model using efficiency curve efficiency
    ***************************************************************************
    
    ***********************************
    ** inputs from hydraulic solver
    ***********************************
    EQUATIONS 2
    TCollIn = @temp(In, TPiColIn)
    MfrColl = ABS(@mfr(In, MfrPiColIn))
    
    ***********************************
    ** outputs to hydraulic solver
    ***********************************
    EQUATIONS 2
    TCollOut = [28,1]
    @temp(Out) = TCollOut
    
    ***********************************
    ** outputs to other ddck
    ***********************************
    
    ******************************************************************************************
    ** outputs to energy balance in kWh and ABSOLUTE value
    ** Following this naming standard : 
    ** qSysIn_name, qSysOut_name, elSysIn_name, elSysOut_name
    ******************************************************************************************
    EQUATIONS 1
    qSysIn_Collector = PColl_kW  
    
    ***********************************
    ** Dependencies with other ddck
    ***********************************
    EQUATIONS 1
    pumpColOn = puColOn
    
    CONSTANTS 2
    C_tilt = slopeSurfUser_1		! @dependencyDdck Collector tilt angle / slope [°]
    C_azim = aziSurfUSer_1    		! @dependencyDdck Collector azimuth  (0:s, 90:w, 270: e) [°]
    
    EQUATIONS 4
    **surface-8
    IT_Coll_kJhm2 = IT_surfUser_1		! Incident total radiation on collector plane, kJ/hm2 
    IB_Coll_kJhm2 = IB_surfUser_1  		! incident beam radiation on collector plane, kJ/hm2
    ID_Coll_kJhm2 = ID_surfUser_1  		! diffuse and ground reflected irradiance on collector tilt
    AI_Coll = AI_surfUser_1  			! incident angle on collector plane, °
    
    EQUATIONS 5
    IT_Coll_kW = IT_Coll_kJhm2/3600		! Incident total radiation on collector plane, kW/m2
    IB_Coll_kW = IB_Coll_kJhm2/3600     ! incident beam radiation on collector plane, kW/m2
    ID_Coll_kW = ID_Coll_kJhm2/3600     ! diffuse and ground reflected irradiance on collector tilt (kW/m2)
    IT_Coll_Wm2 = IT_surfUser_1/3.6
    IT_Coll_kWm2 = IT_surfUser_1/3600
    
    ***********************************
    ** Begin CONSTANTS
    ***********************************
    CONSTANTS 3  
    MfrCPriSpec = 15		! Coll. Prim. loop spec. mass flow [kg/hm2]
    AcollAp = 5         	! Collector area  
    MfrCPriNom = MfrCPriSpec*AcollAp
    
    ***********************************
    ** Begin TYPE
    ***********************************
    UNIT 28 TYPE 1
    PARAMETERS 11
    nSeries       		! number in series
    AcollAp       		! collector area
    cpBri          		! fluid specific heat kj(kgK
    efficiencyMode		! efficiency mode
    testedMfr      		! tested flow rate kg/(hm2)
    Eta0          		! intercept efficiency 
    a1            		! efficiency slope kJ/hm^2K
    a2            		! efficiency curvature kJ/hm^2K^2
    2             		! optical mode
    FirstOrderIAM  		! 1st order IAM
    SecondOrderIAM		! 2nd order IAM
    INPUTS 9
    TCollIn
    MfrColl
    Tamb
    IT_Coll_kJhm2
    IT_H
    ID_Coll_kJhm2
    0,0
    AI_Coll		!Flo check		! JS: This was defined wrong before (C_azim, even though it is incident angle input). Now it should be correct.
    C_tilt 		!Flo check		! JS: This should be correct
    *** INITIAL INPUT VALUES
    20 0 10 0 0 0 GroundReflectance 45 0 
    
    EQUATIONS 4
    **MfrCout = [700,2]
    Pcoll = [28,3]		!kJ/h
    PColl_kW = Pcoll/3600
    PColl_kWm2 = PColl_kW/(AcollAp+1e-30)   
    PColl_Wm2  = PColl_kWm2*1000   
    
    ***********************************
    ** Monthly printer
    ***********************************
    CONSTANTS 1
    unitPrintSol = 31
    
    ASSIGN temp\SOLAR_MO.Prt unitPrintSol 
    
    UNIT 32 TYPE 46      
    PARAMETERS 6   
    unitPrintSol		! 1: Logical unit number, -
    -1           		! 2: Logical unit for monthly summaries, -
    1            		! 3: Relative or absolute start time. 0: print at time intervals relative to the simulation start time. 1: print at absolute time intervals. No effect for monthly integrations
    -1           		! 4: Printing & integrating interval, h. -1 for monthly integration
    1            		! 5: Number of inputs to avoid integration, -
    1            		! 6: Output number to avoid integration
    INPUTS 4
    Time  Pcoll_kW  PColl_kWm2  IT_Coll_kWm2
    **
    Time  Pcoll_kW  PColl_kWm2  IT_Coll_kWm2
    
    ***********************************
    ** Hourly printer
    ***********************************
    CONSTANTS 1
    unitHourlyCol = 33
    
    ASSIGN    temp\SOLAR_HR.Prt    unitHourlyCol     
    
    UNIT 34 TYPE 46		! Printegrator Monthly Values for System
    PARAMETERS 7    
    unitHourlyCol		! 1: Logical unit number, -
    -1            		! 2: Logical unit for monthly summaries, -
    1             		! 3: Relative or absolute start time. 0: print at time intervals relative to the simulation start time. 1: print at absolute time intervals. No effect for monthly integrations
    1             		! 4: Printing & integrating interval, h. -1 for monthly integration
    2             		! 5: Number of inputs to avoid integration, -
    4             		! 6: Output number to avoid integration
    5             		! 7: Output number to avoid integration
    INPUTS 6
    Pcoll_kW  PColl_kWm2  IT_Coll_kWm2 TCollOut TCollIn MfrColl
    **  
    Pcoll_kW  PColl_kWm2  IT_Coll_kWm2 TCollOut TCollIn MfrColl
    
    ***********************************
    ** Online Plotter
    ***********************************
    UNIT 103 TYPE 65		!Changed automatically
    PARAMETERS 12     
    4     				! 1: Nb. of left-axis variables
    2     				! 2: Nb. of right-axis variables
    0     				! 3: Left axis minimum
    10     				! 4: Left axis maximum
    0     				! 5: Right axis minimum
    100  				! 6: Right axis maximum
    nPlotsPerSim		! 7: Number of plots per simulation
    12     				! 8: X-axis gridpoints
    1     				! 9: Shut off Online w/o removing
    -1     				! 10: Logical unit for output file
    0     				! 11: Output file units
    0     				! 12: Output file delimiter
    INPUTS 6    
    Pcoll_kW  PColl_kWm2  IT_Coll_kWm2  MfrColl
    TCollOut TCollIn
    Pcoll_kW  PColl_kWm2  IT_Coll_kWm2  MfrColl
    TCollOut TCollIn
    LABELS  3     
    Power_and_Mfr 
    Temperatures   
    Collector
    
    *******************************
    **END Type1.ddck
    *******************************