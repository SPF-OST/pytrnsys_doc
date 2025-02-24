.. pytrnsys documentation master file, created by
   sphinx-quickstart on Tue Oct 22 16:02:17 2019.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

pytrnsys
========

A short presentation (15 min) of pytrnsys and its features can be found in the following
`YouTube video <https://www.youtube.com/watch?v=B1BSjYRKuVM>`_.

The pytrnsys package provides a complete python-based framework to build, post-process, plot, and report TRNSYS
simulations. It is designed to give users a fast, fully automatized, and reproducible way to execute and share
TRNSYS simulations by the use of a single short configuration file. In addition, a large variety of commands is
accessible to post-process simulation results in one shot. This functionality extends beyond processing TRNSYS generated
data and can also be used for analyzing generic data. These could be, e.g., measurement data that are structured in a
similar way as the TRNSYS results.

The package is developed at the `SPF - Institute for Solar Technology <https://www.spf.ch/>`_ at the `OST - Eastern
Switzerland University of Applied Sciences <https://www.ost.ch/>`_

.. image:: ./guide/resources/logos.svg
      :width: 600
      :alt: logos

|

in collaboration with DCarbo Energy Consulting S.A., Spain

.. image:: ./guide/resources/dcarbo_logo.png
      :width: 300
      :alt: DCarbo logo

|
|

.. toctree::
   :maxdepth: 2

   guide/getting_started
   guide/building_dck
   guide/gui
   guide/run_simulation
   guide/process_data
   guide/ddck_structure

   references

Acknowledgments

A first version of this package was created in 2013 and since then it has evolved considerably. We would like to
thank the Swiss Federal Office Of Energy (SFOE) who supported many projects related to simulations of renewable
energy systems in the context of which this code has been developed. We would also like to thank the European Union’s
Horizon 2020 research and innovation programme for the funding received in TRI-HP under the Grant Agreement No. 81488 and in
PLURAL under the Grant Agreement No. 958218. These projects allowed to dedicate efforts in sharing the code with the
consortium and to make it usable for others.