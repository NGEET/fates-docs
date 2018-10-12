.. _h1_section:

Allocation Hypothesis 1: Allometrically Guided, Carbon Only
-----------------------------------------------------------

Hypothesis 1, the carbon-only allometric hypotheses, assumes there is a single carbon species for each of the six plant organ pools:

1. Leaf
2. Fine-root
3. Sapwood
4. Structural wood
5. Storage
6. Reproductive

This method was designed to be called daily, which is true when called for the FATES model.

The carbon-only allometric hypotheses contains no nutrient species, no growth limitations based on nutrients, and asumes that tissues will grow in proportion to each other based on a set of allometric functions.  These allometric functions are tied to diameter :math:`d`.  For PARTEH, these allometric functions are those used by FATES.  The allometric functions will calculate a target mass for each pool, which essentially is the maximum carrying capacity for that pool based on the plant's size (stem diameter). In the case of grasses, diameter is still used.  And even though it does not have a physical meaning, it is still a usefull mediator to tie all the proportional pools together.  For mature trees, the diameter should reflect the diameter at breast height.  Target leaf mass is slightly more complicated.  The target leaf mass is based also on a trimming function which scales down the maximum leaf biomass to remove unproductive leaves from shaded lower boughs.  Allometric schemes are based off of previous research, which will be noted in intercomparisons.

This documentation will use the generic symbol :math:`C` for carbon masses and fluxes for organs indexed by :math:`o`.  Also see :ref:`conventions_used_section`.

.. _h1_variable_table:

.. table:: Table H1-1
   :align: center

   +------------------------+------------------+---------------------------------+---------+
   | Symbol                 | Dimension        | Description                     | Units   |
   +========================+==================+=================================+=========+
   | State Variables                                                                       |
   +------------------------+------------------+---------------------------------+---------+
   | :math:`C_{(o)}`        | organ            | carbon mass                     | [kg]    |
   +------------------------+------------------+---------------------------------+---------+
   | Input/Output Boundary Conditions                                                      |
   +------------------------+------------------+---------------------------------+---------+
   | :math:`d`              | scalar           | Reference Stem Diameter         | [cm]    |
   +------------------------+------------------+---------------------------------+---------+
   | Input Boundary Conditions                                                             |
   +------------------------+------------------+---------------------------------+---------+
   | :math:`C_{gain}`       | scalar           | Daily carbon gain               | [kg]    |
   +------------------------+------------------+---------------------------------+---------+
   | :math:`f_{trim}`       | scalar           | Canopy Trim Fraction            | [0-1]   |   
   +------------------------+------------------+---------------------------------+---------+
   | Parameters                                                                            |
   +------------------------+------------------+---------------------------------+---------+
   | :math:`p_{tm}`         | pft*             | maintenance replacement priority| [0-1]   |
   +------------------------+------------------+---------------------------------+---------+

*List of key states, boundary conditions and parameters in hypothesis 1, allometric based carbon-only model.  In this notation, o is used to index the organ dimension. :math:`*` Note that the pft index for the maintenance replacement priority is specified for each PFT, but the dimension will be implied in our notation as each plant is uniquely asociated with a PFT.*


.. _h1_assumptions_section:

Assumptions of Other Processes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _h1_daily_c_gain_section:

Daily Carbon Gain
"""""""""""""""""

It is assumed that over the sub-daily time-steps, net gains photosynthesis and losses from total plant respiration (growth and maintenance) are calculated and integrated (or accumulated). The net sum of these terms results in a daily net carbon gain :math:`C_{gain}`.

.. _h1_turnover_phenology_section:

Turnover and Phenology
""""""""""""""""""""""

It is assumed that for the current day, all turnover from live plants has already been removed, and/or any flushing associated with leaf-out (or bud-burst) has already been transferred (most likely from storage).

Different methodologies for calculating turnover exist.  Event based turnover is covered in :ref:`event_turnover_section`, and maintenance turnover is covered in :ref:`maintenance_turnover_section`.

The PARTEH software system provides helper functions to remove, add and transfer carbon and nutrients during these different processes. It is up to various external modules such as fire and phenology to determine the timing and relative magnitudes of the pools being reduced or flushed.

.. _h1_order_of_operations:


Order of Operations
^^^^^^^^^^^^^^^^^^^

Allocation for this hypothesis can be partitioned roughly into two parts, replenishing the plants' existing pools with respect to a target mass that is defined by the stature (size) of the plant (this may be thought of of bringing the plant back to allometric targets).  And then, if resources are still available, the plant will grow in stature where allocation seeks to grow the pools out concurrently with each other.


1. :ref:`h1_replenish_targets_section`
2. :ref:`h1_grow_stature_section`


.. _h1_replenish_targets_section:

Replenish Pools with Respect to Target Levels
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The plant will go through a sequence of allocations into the different plant tissues, ultimately attempting to get each organ's total carbon pool up to the target mass, :math:`\grave{C}_{(o)}`.  These targets are goverened by the stature of the plant, and can be thought of as the desireable pool sizes that the plant would likely have if it were prepared to grow in stature.   Losses from turnover, may have drawn down the masses away from the target values associated with their current stature.  Sometimes this is considered being "off allometry".  In this hypothesis, the base assumption is that the targets are dictated by allometry, but other methods of determining these targets are possible as well.  Allometric targets are dicated here, by a function of plant diameter :math:`\text{d}` , plant functional type :math:`\text{pft}`, and the canopy trim fraction :math:`f_{trim}`.  This last variable, can be described as the fraction of the crown that this particular plant desires to fill out, compared to a prototypical plant in idealized conditions.  This fraction changes slowly over yearly time-scales, responding to the relative productivity of the plant's crown layers with respect to their respiration costs.

.. math::
   :label: h1_c_target_eq

   \grave{C}_{(o)} &= \text{func}(d,\text{pft},f_{trim})


.. _h1_replace_maint_section:

Replace Maintenance Turnover
""""""""""""""""""""""""""""

The first step in replenishing carbon pools is the replacement of maintenance turnover losses in evergreen plants.  Evergreen plants continually loose leaf and fine-root tissues over the course of the year.  We first define an organ set: :math:`o = \mathbb{O}_{lf}`, which is comprised of leaves and fine-roots.  The demand for replacement of each organ in this set :math:`\check{C}_{(\mathbb{O}_{lf})}`, is governed by the amount of carbon each organ lost to turnover on this day :math:`\vec{C}_{turn(\mathbb{O}_{lf})}`.  A parameter governs this prioritization.  When :math:`p_{tm} = 1`, an attempt is made to replace all carbon lost from maintenance turnover. When :math:`p_{tm} = 0`,no attempt is made to replace this turnover. 

.. math::
   :label: h1_c1_demand_eq

   \check{C}_{(\mathbb{O}_{lf})} &=  \quad p_{tm}  \cdot \vec{C}_{turn(\mathbb{O}_{lf})}

The total carbon demanded in this step :math:`\check{C}_1` is summed for both leaf and fine-root tissues:

.. math::
   :label: h1_c1_sum_eq

   \check{C}_1 &= \sum_{o=\mathbb{O}_{lf}} \check{C}_{(o)}


The flux into these two pools :math:`\vec{C}_{(\mathbb{O}_{lf})}` is governed by the minimum between the how much carbon is available (the sum of carbon gain and available storage carbon :math:`C_{(st)}`) for replacement and how much is demanded.

.. math::
   :label: h1_c1_flux_eq

   \vec{C}_{(\mathbb{O}_{lf})} &= \quad \text{min}(\check{C}_{(\mathbb{O}_{lf})}, \text{max}(0,(C_{(st)}+C_{gain})*(\check{C}_{(\mathbb{O}_{lf})}  / \check{C}_1  ) ))

The carbon pools of the leaf and fine-root organs are then incremented, and the daily carbon gain is decremented.  

.. math::
   :label: h1_c1_flux_increment_eq

   C_{(\mathbb{O}_{lf})}  &= \quad C_{(\mathbb{O}_{lf})} + \vec{C}_{(\mathbb{O}_{lf})}

   C_{gain} &= \quad C_{gain} - \sum_{o=\mathbb{O}_{lf}} \vec{C}_{(o)}



Note that this step may push the daily carbon gain to a negative value.  Carbon will be transferred in the next step to "pay" for this negative carbon balance.

.. _h1_transfer_storage_section:

Bi-directional Storage Transfer
"""""""""""""""""""""""""""""""

In the next step, either of two things will happen.  If the daily carbon gain :math:`C_{gain}` is now negative (which may simply be due to more respiration than primary production or because of its payments in the previous step), carbon will be drawn from storage :math:`C_{(st)}` to bring the carbon gain to zero. At which point the daily allocations are complete and the module returns.  Flux into storage is denoted :math:`\vec{C}_{(st)}`.

.. math::
   :label: h1_storeflux_neggain_eq

   \text{if} \quad C_{gain} < 0

   \vec{C}_{(st)} &= \quad C_{gain}

If the daily carbon gain is positive, carbon will flow into storage based on a non-linear rate that increases the flux when stores are low and decreases the flux when stores are high. This is mediated by identifying the fraction of storage with its allometric maximum :math:`f_{st}`, as well as limiting transfer to not exceed the storage demand :math:`\check{C}_{(st)}`.

.. math::
   :label: h1_storeflux_posgain_eq

   \text{if} \quad C_{gain} >= 0

   \check{C}_{(st)}  &= \quad  \grave{C}_{(st)} - C_{(st)}

   f_{st}            &= \quad  C_{(st)} / \grave{C}_{(st)}

   \vec{C}_{(st)}    &= \quad  \text{min}(\check{C}_{(st)},C_{gain} \cdot \text{max}( e^{-f_{st}^4} - e^{-1}, 0))


And the pools are likewise incremented like they were in the first step.

.. math::
   :label: h1_storeflux_increment_eq

   C_{gain}          &= \quad C_{gain} - \vec{C}_{(st)}

   C_{(st)}          &= \quad C_{(st)} + \vec{C}_{(st)}


   
.. _h1_replenish_leafroot_part2_section:

Replenish Remaining Allometric Deficit of Leaves and Fine-roots
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

In this next step, leaves and fine-roots, again get preferrential access to any available carbon from the daily gains to replenish their pools towards the target values.  For leaf and fineroot organs, in index set :math:`\mathbb{O}_{lf}`, we estimate the deficit from target :math:`\check{C}_{(\mathbb{O}_{lf})}`, and their sum deficit :math:`\check{C}_2`.


.. math::
   :label: h1_c2_demand_eq

   \check{C}_{(\mathbb{O}_{lf})} &= \quad \grave{C}_{(\mathbb{O}_{lf})} - C_{(\mathbb{O}_{lf})}

.. math::
   :label: h1_c2_sumdemand_eq

   \check{C}_2 &= \quad  \sum_{o=\mathbb{O}_{lf}} \check{C}_{(o)}


The flux into leaves and fine-roots is handled proportional to their demands, and based on the minimum between the carbon available and the total demanded from both pools.

.. math::
   :label: h1_c2_flux_eq
	   
   \vec{C}_{(\mathbb{O}_{lf})} &=  \quad \text{min}(\check{C}_{(\mathbb{O}_{lf})}, C_{gain} \cdot \check{C}_{(\mathbb{O}_{lf})}/ \check{C}_2 )
	   
   C_{(\mathbb{O}_{lf})} &=  \quad C_{(\mathbb{O}_{lf})} + \vec{C}_{(\mathbb{O}_{lf})}

   C_{gain} &= C_{gain} -  \sum_{o=\mathbb{O}_{lf}} \vec{C}_{(o)}

.. _h1_replenish_livepools_section:

Replenish Remaining Live Pools Toward Allometric Targets
""""""""""""""""""""""""""""""""""""""""""""""""""""""""

After the prioritized replenishment of leaves and fine-roots, remaining daily carbon gain is allocated to sapwood and storage tissues. The math in this process is the same as in the previous section, :ref:`h1_replenish_leafroot_part2_section`. The only difference is the set of relevant organs changes.

.. _h1_replenish_structure_section:

Replenish Structural Pool Toward Allometric Target
""""""""""""""""""""""""""""""""""""""""""""""""""

Due to branchfall, the plant's current structural carbon pool may be lower than the target size dictated by allometry and stature.  Following the algorithm of the previous two sections, :ref:`h1_replenish_leafroot_part2_section` and :ref:`h1_replenish_livepools_section`, any remaining daily carbon gain is transferred into the structural pool.

At this point, if any daily carbon gain :math:`C_{gain}` remains, the plant must be "on-allometry". With this assumption, concurrent stature growth can proceed.


.. _h1_grow_stature_section:


Grow Stature Concurrently
^^^^^^^^^^^^^^^^^^^^^^^^^

Note, it possible that some pools may be larger than their allometric targets.  This is due to numerical imprecisions, and also may be an artifact of the fusion process in demographic scaling routines (FATES/ED). While it has not been explicitly stated yet, many of the previous calculations in this chapter prevent negative fluxes out of the carbon pools in such cases.  In this step, we define the set of organs :math:`o` that match allometry with some precision, but not above. This is the stature growth set, :math:`\mathbb{O}_{sg}`.

The allometric functions governing the target sizes of the carbon pools also provide the rate of change in the pool with respect to plant diameter. These functions can be related to the plants diameter, and uses parameterizations that are tuned to the plant's functional type.  Since the plant is on allometry, the actual carbon pools should match the target pools, to some precision.  Therefore, for any diameter :math:`d`, we also know for each organ :math:`o` in :math:`\mathbb{O}_{sg}` we can obtain the differential from the allometry module.  A description of the allometry module is provided in the :ref:`allometry_chapter`.

.. math::
   :label: h1_sgrowth_allom_genic

   \frac{dC_{(\mathbb{O}_{sg})}}{dd} &= \quad \text{function}(d, \text{pft})



This concurrent growth step is facilitated by numerical integration, where we integrate over the independant variable, remaining daily carbon gain :math:`C_{gain}`.  For any given point in the integration process, we first identify the sum change in carbon with respect to diameter :math:`\frac{dC_{sg}}{dd}`

.. math::
   :label: h1_sgrowth_sumdiff_eq

   \frac{dC_{sg}}{dd} &= \quad  \sum_{o=\mathbb{O}_{sg}} \frac{dC_{(o)}}{dd}


With the sum change in flux, we can then identify the proportional amount of carbon demanded from any individual pool, relative to the total carbon allocated over all pools in the set.  


.. math::
   :label: h1_sgrowth_proportion_eq

   \frac{dC_{(\mathbb{O}_{sg})}}{dC_{sg}} &= \quad \frac{dC_{(\mathbb{O}_{sg} )}}{dd} / \frac{dC_{sg}}{dd}


It is also at this time, that reproductive carbon flux is allocated.  This is a special case, that does not use allometric scaling, and instead retrieves a reproductive allocation fraction :math:`f_{repro}` that it retrieves from parameterization or a more sophisticated seed/flower allocation algorithm.  The fraction :math:`\frac{d \grave{C}_{(\mathbb{O}_{sg})}}{d C_{sg}}` changes dynamically as the plant grows along its allometric curve, and could potentially require numerical solvers that enforce improved stability or precision.  For most cases, since plant growth is slow, and the interval of integration is small compared to the size of the derivatives, a simple Euler integration suffices.  The integration for organs in set :math:`\mathbb{O}_{sg}`, and reproduction are defined as follows.

.. math::
   :label: h1_sgrowth_integration_eq

   \vec{C}_{(\mathbb{O}_{sg})} =& \quad  \int_{dC=0}^{C_{gain}}  \frac{d \grave{C}_{(\mathbb{O}_{sg})}}{d C_{sg}} (1-f_{repro}) \quad dC

.. math::
   :label: h1_repro_integration_eq

   \vec{C}_{(repro)} =& \quad \int_{dC=0}^{C_{gain}} f_{repro} \quad dC


Following this calculation, the integrated fluxes (including reproduction) are summed up, and then normalized such that their sum matches :math:`C_{gain}` to ensure that the exact amount of remaining carbon is used up.  The change in diameter can also be determined by simply relating one of the integrations back to diameter.

.. math::
   :label: h1_sgrowth_diamintegration_eq

   \Delta d =& \quad  \int_{dC=0}^{C_{gain}}  \frac{d \grave{C}_{(\mathbb{O}_{sg})}}{d C_{sg}} \frac{dd}{dC_{(\mathbb{O}_{sg} )}}  (1-f_{repro}) \quad dC



