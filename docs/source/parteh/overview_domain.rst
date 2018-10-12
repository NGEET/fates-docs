.. _overview_chapter:

Overview
********

The **Plant Allocation** and **Reactive Transport Extensible Hypotheses (PARTEH)** is a suite of modules that handle the processes of allocation, transport and reactions (i.e. thos processes related to movement and change, yet perhaps not the genesis) of various arbitrary species (carbon, nutrients, toxins, etc) within the various organs of live vegetation.  

The modules themselves are written in modern Fortran, with an emphasis on code extensibility (i.e. future proofing).  This software can therefore be compiled and embedded in Land Surface Models (LSMs) and Dynamic Vegetation Models (DVMs).  A functional unit testing infrastructure, written in python, is also used to evaluate how the different modules that define the allocation, reaction and transport hypotheses, operate under contexts of different synthetic boundary conditions.

.. _domain_section:

PARTEH's Domain of Influence
^^^^^^^^^^^^^^^^^^^^^^^^^^^^


Summary of Domain of Influence:

1. Single plant allocation, transport and the associated reactions
2. Not photosynthesis
3. Optional payment of respiration costs, and/or optional calculations of rates associated with transport/allocation (growth respiration)
4. Turnover losses and turnover re-translocation, coordinated with external modules
5. Accepts soil nutrient boundary fluxes, not a soil BGC model

An expanded description of PARTEH's scope follows.

PARTEH's scope is not limited to, but will likely be applied on individual plants.  It may operate as a sub-component of ecosystem scaling models (such as FATES, ED, etc) where its treatment of single plant physics, will be translated into cohorts.  Hypotheses may also be crafted that operate on other scales, such as pools that are disasociated from individual plants.

PARTEH is intended to work in coordination with, but not replicate, photosynthesis modules.

Since some respiratory costs are tightly connected with allocation, reactions and transport, PARTEH is designed to both support alternative hypothesis on and perform calculations of associated respiration costs, or accept respiration costs associated with allocation as an input.  

PARTEH does handle the turnover of live plants.  These calculations may rely on coordination with outside modules that specify the scale and scope of turnover associated with events like deciduous leaf drop, deciduous leaf flushing, fire, herbivory or storms.  Turnover is inextricably linked to re-translocation, which is classified as reaction and transport.  This turnover may be mediated either through continuous (background) type behavior, or event based behavior (e.g. leaf drop).

PARTEH does not simulate the fate of live vegetation turnover, for example litter decomposition, and it is expected that these carbon and nutrient loss fluxes are passed to the host-model (e.g FATES) to be accounted for.

PARTEH does not handle plant mortality.  A host model may ask PARTEH for diagnostics on nutrient concentrations in various plant organs, however decisions on how this is expected to influence plant survival are handled outside of the model.

PARTEH does not handle below-ground soil-biogeochemistry.  The uptake of nutrients from the soil is an expected boundary condition, however it is also expected that the soil biogeochemical model will ask PARTEH to provide information on the state of plant organs and perhaps the concentrations of species present in those organs so that is can properly determine the plant's affinity for nutrient fluxes.  Further, this fluxes will likely be mediated through a vegetation scaling model (e.g. FATES scales PARTEH affinities and fluxes in its communication with a below-ground model (ECA, CNP, PFLOTRAN, etc.).


.. _why_extensible_section:

Why "Extensible Hypotheses"?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The breadth of knowledge of how plants handle reactive transport is increasing, yet many uncertainties remain.  In light of so many uncertainties, it has seemed inappropriate to coalesce around a single functional hypothesis for terrestrial ecosystem modeling.

Moreover, if so many uncertainties remain, a framework that could conveniently intercompare functional hypthesese with the hopes of comparing with available data, seems useful.

This has led to a design strategy that supports extensibility in design and intercomparison of multiple functional hypotheses of Plant Reactive Transport.  Thus, it is considered a suite of modules, and not a model, because it is designed to accomodate many different functional hypotheses about how plant handle nutrient dynamics.  


.. _software_design_section:

Software Design
^^^^^^^^^^^^^^^

PARTEH is written in **modern Fortran** using some object oriented coding principles.  It may be called from terrestrial ecosystem model, or it can be called using its own driver (which is written in python).  

The PARTEH software is packaged with the FATES model.  PARTEH software is found in two locations.  In the fates/parteh folder, the main fortran code that houses the PARTEH class objects and processes is found.  The fates/functional_unit_testing/parteh/ folder contains scripts for functional unit testing of the PARTEH submodule.  The scripts running the tests are written in python. XML files control the tests and set parameters.  There are also fortran codes that serve as wrappers and support to help make the connection between the python unit testing framework and the core PARTEH fortran code found in fates/parteh.

The module system does not yet leverage external numerical libraries such as Petsc, trilos or sundials.  In the first iterations of building this module system, the numerical integration needs have been fairly simple.  However, the solving of numerical integration has been written in such a way that all hypotheses currently call the same generic routines.  Curently an **Euler scheme**, and a **Runge-Kutte-Fehlerg 4/5th order** are offered.  By writing the modules such that they call a self-contained generic integration layer, the code is in a more ready state to upgrade to a more sophisticated external library.

The module system uses the principal of **inheritance** to help facilitate extensible softare design.  Specifically, a **base-class** has been designed, which forces all the different hypotheses to be written as **extensions** of the base.  Thus, each hypothesis will have a template of how to populate the various state-variables and how and when to call its procedures.  Moreover, the base class contains helper procedures that connect to the external needs of PARTEH.  For intance, if a model such as FATES needs to ask PARTEH how much leaf biomass exists, it can call a procedure defined by the base class which will return the leaf biomass wihtout explicitly stating which hypothesis to calculate it for.  By attaching these helper functions to the base class, these operate on all extensions (different hypotheses of the base), therefore buffering the environment outside of PAREH to the choices that are made inside of PARTEH.

.. _conventions_used_section:

Conventions Used
^^^^^^^^^^^^^^^^

Symbology
"""""""""

In the description of all hypotheses, the following symbology will be used.  For a generic variable :math:`X`:

.. _x_convention_table:

.. table:: Table 1
   :align: center
	   
   +------------------+-----------------------------------------+
   | Symbol           | Description                             |
   +==================+=========================================+
   |:math:`\dot{X}`   | Flux rate                               |
   +------------------+-----------------------------------------+
   |:math:`\vec{X}`   | Time integrated (total) flux            |
   +------------------+-----------------------------------------+
   |:math:`\check{X}` | Deficit or demand relative to reference | 
   +------------------+-----------------------------------------+
   |:math:`\grave{X}` | Target quantity                         |
   +------------------+-----------------------------------------+
   |:math:`\tilde{X}` | Time integrated turnover loss           |
   +------------------+-----------------------------------------+

The PARTEH code will mostly be concerned with the mass pools of carbon and nutrients as state variables (those entities which maintain continuity through time and space, and are integrated).  Pools will generally refer to the mass, in absolute units [kg].  Any pool can also be defined by a combination of species (carbon, nitrogen, phosphorous, etc.) and organ (leaf, fine-root, sapwood, etc).  While the code is flexible enough to accomodate different isotopes of carbon, in general carbon species in this document will be denoted generically :math:`C_{(o)}` for any organ indexed :math:`o`.  Nutrients of arbitrary species :math:`s` and organ :math:`o` will be denoted :math:`N_{(o,s)}`.  See :ref:`cn_convention_table`:

.. _cn_convention_table:

.. table:: Table 2
   :align: center


   +------------------------+------------------+---------------------------------+---------+
   | Symbol                 | Dimension        | Description                     | Units   |
   +========================+==================+=================================+=========+
   | State Variables                                                                       |
   +------------------------+------------------+---------------------------------+---------+
   | :math:`C_{(o)}`        | organ            | carbon mass                     | [kg]    |
   +------------------------+------------------+---------------------------------+---------+
   | :math:`N_{(o,s)}`      | organ x species  | nutrient mass                   | [kg]    |
   +------------------------+------------------+---------------------------------+---------+


All Parameters Are PFT Specific
"""""""""""""""""""""""""""""""

*A note about parameters.*  All parameters (thus far) used in PARTEH, can be assumed as PFT specific. In each of the describing tables, this is indicated.  But the notation in describing text does not maintain a "pft" dimension, as it should just be implied.
