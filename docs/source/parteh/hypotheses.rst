.. _hypotheses_section:

Hypotheses
**********

The bulk of the more complicated processes in PARTEH are subsumed in the allocation and transport of newly acquired resources. However, since the process of re-translocation during turnover is a reactive transport process, this falls within PARTEH's domain.  Therefore, broadly, reactive transport in PARTEH is partitioned into allocation and turnover.  

The PARTEH software is written to enable flexibility in the frequency and order at which allocation and turnover processes are used/called.  In FATES, both are called daily and tunover (all forms) are called first.

The PARTEH software is also written such that allocation hypothesis are each relegated inside their own modules.  Turnover, for which existing hypotheses are somewhat simpler than allocation, are written generically and can operate in concert with any allocation hypothesis for any arbitrary set of organs and nutrients.

.. _turnover_hypotheses_section:

Turnover Hypotheses
====================

.. toctree::
   :maxdepth: 2

   turnover.rst


.. _allocation_hypotheses_section:

Allocation Hypotheses
=====================

.. toctree::
   :maxdepth: 2

   h1_callom_only.rst
   h2_callom_flexstoich.rst
