.. _h2_section:

Allocation Hypothesis 2: Allometrically Guided, Carbon and Nutrients with Prioritization and Flexible Target Stoichiometry
--------------------------------------------------------------------------------------------------------------------------

This hypothesis assumes there is a single carbon species, and an arbitrary number of nutrient species for each of the six plant organ pools:

1. Leaf
2. Fine-root
3. Sapwood
4. Structural wood
5. Storage
6. Reproductive

The PARTEH code for hypothesis two currently only enables Nitrogen and Phosphorous.  The code can be easily extended to handle other nurtient species, however this would increase the number of parameters used and complicate things for a broader user base.  Without an imediate need for other species, they have been left out.  This documentation will use the generic symbol :math:`N` for all nutrients of any species indexed by :math:`s` in organs indexed by :math:`o`.

The state variables, boundary conditions and parameters for hypothesis 2 are described in :ref:`h2_variable_table`.


.. _h2_variable_table:

.. table:: Table H2-1
   :align: center

   +------------------------+------------------------+---------------------------------+---------+
   | Symbol                 | Dimension              | Description                     | Units   |
   +========================+========================+=================================+=========+
   | State Variables                                                                             |
   +------------------------+------------------------+---------------------------------+---------+
   | :math:`C_{(o)}`        | organ                  | carbon mass                     | [kg]    |
   +------------------------+------------------------+---------------------------------+---------+
   | :math:`N_{(o,s)}`      | organ x species        | nutrient mass                   | [kg]    |
   +------------------------+------------------------+---------------------------------+---------+
   | Input/Output Boundary Conditions                                                            |
   +------------------------+------------------------+---------------------------------+---------+
   | :math:`R_{md}`         | scalar                 | Maint. Resp. Deficit            | [kg]    |
   +------------------------+------------------------+---------------------------------+---------+
   | :math:`d`              | scalar                 | Reference Stem Diameter         | [cm]    |
   +------------------------+------------------------+---------------------------------+---------+
   | Input Boundary Conditions                                                                   |
   +------------------------+------------------------+---------------------------------+---------+
   | :math:`f_{trim}`       | scalar                 | Canopy Trim Fraction            | [0-1]   |   
   +------------------------+------------------------+---------------------------------+---------+
   | :math:`C_{gain}`       | scalar                 | Daily carbon gain               | [kg]    |
   +------------------------+------------------------+---------------------------------+---------+
   | :math:`N_{gain(s)}`    | species                | Daily nutrient gain             | [kg]    |
   +------------------------+------------------------+---------------------------------+---------+
   | Output Boundary Conditions                                                                  |
   +------------------------+------------------------+---------------------------------+---------+
   | :math:`C_{exu}`        | scalar                 | Daily carbon exudation          | [kg]    |
   +------------------------+------------------------+---------------------------------+---------+
   | :math:`N_{exu(s)}`     | species                | Daily nutrient exudation        | [kg]    |
   +------------------------+------------------------+---------------------------------+---------+
   | :math:`R_g`            | scalar                 | Growth Respiration              | [kg]    |
   +------------------------+------------------------+---------------------------------+---------+
   | Parameters                                                                                  |
   +------------------------+------------------------+---------------------------------+---------+
   | :math:`\alpha_{(o,s)}` | pft* x organ x species | ideal stoichiometric ratios     | [kg/kg] |
   +------------------------+------------------------+---------------------------------+---------+
   | :math:`\beta_{(o,s)}`  | pft* x organ x species | minimum stoichiometric ratios   | [kg/kg] |
   +------------------------+------------------------+---------------------------------+---------+
   | :math:`p_{tm}`         | pft*                   | tissue vs. resp. prioritization | [0-1]   |
   +------------------------+------------------------+---------------------------------+---------+
   | :math:`\omega_{(o)}`   | pft* x organ           | prioritization level            | [1-6]   |
   +------------------------+------------------------+---------------------------------+---------+
   | :math:`r_{g(o)}`       | pft* x organ           | unit growth respiration rate    | [kg/kg] |
   +------------------------+------------------------+---------------------------------+---------+

*List of key states, boundary conditions and parameters in hypothesis 2, allometric multi-nutrient species with fixed target stoichiometry.  In this notation, o and s are used to index the organ and species (nutrient) dimensions. :math:`*` Note that the parameters are specified explicitly for each pft, but the dimension will be implied in our notation as each plant is already uniquely asociated with a PFT.*




Order of Operations
^^^^^^^^^^^^^^^^^^^

It is assumed that over the sub-daily time-steps, photosynthesis, respiration and nutrient uptake has been accumulated. These provide net carbon and nutrient gains at the end of the day to drive allocations.  The first daily procedure is the removal of biomass from the plant due to turnover coming from leaf-fall, branchfall and turnover of fine-roots.  The second daiy procedure seeks to replenish the plants existing pools with respect to a target mass that is defined by the stature (size) of the plant (this may be thought of of bringing the plant back to allometric targets).  Next, if resources are still available the plant will grow in stature, where allocation seeks to grow the pools out concurrently with each other.  If any nutrient resources remain, they will be allocated towards ideal stoichiometric proportions, which may or may not be greater than the proportionalities needed during stature growth.  And finally, all excess materials are sent to storage pools (if not full) and then exuded through roots.

1. (sub-daily) :ref:`h2_accumulate_cn_section`
2. (daily) Perform Allocations to Pools

   a. :ref:`h2_turnover_section`
   b. :ref:`h2_replenish_targets_section`
   c. :ref:`h2_grow_stature_section`
   d. :ref:`h2_transfer_ideal_nutrients_section`
   e. :ref:`h2_exude_section`



VISUALIZATION


.. _h2_accumulate_cn_section:

Accumulate Carbon and Nutrients
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Photosynthesis and maintenance respiration are sensitive to light levels and tissue temperatures, which vary over sub-daily timescales. In CLM/ELM, this "fast" time-step is 30 minutes.  It is assumed that the host-model (e.g. FATES) will handle the calculation of GPP and maintenance respiration, and integrate these quantities over the course of the day.  There is some flexibility in how PARTEH handles allocations with these two constraints.  Along with nutrient inputs, the host model must provide the boundary conditions of daily carbon gain :math:`C_{gain}`, and optionally, the maintenance respiration deficit :math:`R_{md}`.  

There are two scenarios that this hypothesis accomodates:

1. The host model calculates the difference between daily integrated GPP and maintenance respiration and passes it as :math:`C_{gain}`, which may be positive or negative. No maintenance respiration is tracked, because it is paid instantly, and thus :math:`R_{md} = 0`.

2. The host model passes GPP as :math:`C_{gain}` (always positive), and maintains a running account of maintenance respiration deficit, thereby adding the daily integrated maintenance respiration to :math:`R_{md}`.  The PARTEH model will then attempt to pay for :math:`R_{md}`, and passing back the updated deficit to the host.

The third key boundary condition provided by the host, is the daily integrated flux of nutrients from soil to fine-roots, :math:`N_{gain(s)}`, for each nutrient species :math:`s`.  Depending on the soil biogeochemistry model in use, PARTEH can provide information about the state of the plant, to help the soil biogeochemistry module determine the plant's affinity in a competitive nutrient environment.


.. _h2_turnover_section:

Remove Biomass From All Pools as Turnover
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Different methodologies for calculating turnover exist, and are executed prior to allocations.  Event based turnover is covered in :ref:`event_turnover_section`, and maintenance turnover is covered in :ref:`maintenance_turnover_section`.

.. _h2_replenish_targets_section:

Replenish Pools with Respect to Target Levels
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The organs of each plant have target masses for carbon :math:`\grave{C}_{(o)}` and nutrients :math:`\grave{N}_{(o,s)}`.  These targets are goverened by the stature of the plant, and can be thought of as the desireable pool sizes that the plant would like to have to be ready for further growth in stature.  Turnover, as described in the previous section, draws down the masses away from the target values associated with their current stature.  Sometimes this is considered being "off allometry".  In this hypothesis, the base assumption is that the targets are dictated by allometry, but other methods of determining these targets are possible as well.  Allometric targets are typically a function of plant diameter :math:`\text{dbh}` , plant functional type :math:`\text{pft}`, and an indicator of how much trimming of unproductive lower boughs a plant has executed :math:`\text{trimming}`.

.. math::
   :label: h2_c_target_eq

   \grave{C}_{(o)} &= \text{func}(\text{dbh},\text{pft},\text{trimming})

For nutrient species, the targets are based on a parameter that describes the minimum stoichiometric ratios with carbon :math:`\beta_{(o,s)}` that are required for the plant to grow in stature.

.. math::
   :label: h2_n_target_eq

   \grave{N}_{(o,s)} &= \quad \grave{C}_{(o)} \cdot \beta_{(o,s)}


In this step, the plant must allocate resources to bring its pools up to the targets before growing out the plant's stature again.  This process relies on calculating the targets, and then the carbon :math:`\check{C}_{(o)}` and nutrient :math:`\check{N}_{(o,s)}` demands to reach those targets.  

Given these targets, the demands for each carbon :math:`\check{C}_{(o)}` and nutrient :math:`\check{N}_{(o,s)}` pool are calculated.  In the case of carbon, a growth tax is applied to allocation, which contributes to the demand.  Here, that tax is governed by a unit growth parameter :math:`r_{g(o)}`, however more complicated growth tax functions could be used as well. Likewise, a pool may already be at or above its current target.  Only positive demands are used, so a floor of 0 is imposed on the demand.

.. math::
   :label: h2_c_demand_eq

   \check{C}_{(o)} &=  \quad \text{max}(0,(\grave{C}_{(o)} - C_{(o)}) \cdot (1+r_{g(o)}))


.. math::
   :label: h2_n_demand_eq

   \check{N}_{(o,s)} &=  \quad \text{max}(0,\grave{N}_{(o,s)} - N_{(o,s)} )


Each plant organ is then associated with any priority level, 0 through 6.  Organs associated with priority 1 will get first access to carbon and nutrients and organs associated with priority order will get the remainder.  The priority order levels are ascended sequentially, we indicate the valid set of organ indices in the current priority order level :math:`pr` as set :math:`\mathbb{O}_{pr}`.  Note that priority level 0 is a special bypass level.  This is used for reproductive allocation, which currently is only generated during the stature growth step.  Note, it is not required that ANY organs are classified as priority 1. 

The first priority level (:math:`pr=1,\mathbb{O}_{1}`) has two approaches based on the boundary conditions provided.

1. It is assumed that maintenance respiration costs have not been paid yet by the host model, and thus the maintenance respiration deficit :math:`R_{md}` exists and is non-zero, and that daily carbon gains are greater than or equal to zero.  This is detailed in :ref:`p1_explicit_rmd_section`.

2. It is assumed that boundary condition for :math:`C_{gain}` has already decucted maintenance respiration costs. Here, :math:`R_{md}` is always zero, and if the plant is not metabolically dormant, :math:`C_{gain}` may be positive or negative.  This is detailed in :ref:"p1_implicit_rmd_section'.

.. _p1_explicit_rmd_section:

Priority 1 Carbon Fluxes with explicit Maintenance Respiration Deficit
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

 First, we assess how much total demand is coming from the priority 1 carbon pools.

.. math::
   :label: h2_c_priority1_sum_eq

   \check{C}_1 &= \sum_{o=\mathbb{O}_{1}} \check{C}_{(o)}

The total carbon that can be translocated from storage is :math:`\vec{C}_{st-tran}`.  Any number of models could be used to determine how resistant the storage is to pay off high-priority tissues and maintenance respiration costs.  Below is an example of a simple function where the transferable carbon decreases as the square of the pool's proportion with its target. Where storage is denoted organ index :math:`o=st`:

.. math::
   :label: h2_c_st_trans_eq

   C_{st-tran} &= C_{(st)} \cdot \text{min}(1,C_{(st)} / \grave{C}_{(st)}  )

The total carbon that is transferred :math:`\vec{C}_{tot}` is the minimum between the demanded and what can be transferred from both storage and carbon gains :math:`C_{gain}`.  The fraction of how much is transferred versus demanded, :math:`f_{tot}`, is also useful.

.. math::
   :label: h2_c_vec_tot_eq

   \vec{C}_{tot} &= \quad \text{min}( \check{C}_1 + R_{md} , C_{st-tran} + C_{gain} )

   f_{tot} &= \quad \vec{C}_{tot} / (\check{C}_1 + R_{md})


Preference can be specified to allocate available carbon to either maintenance respiration, or the priority 1 pools. To do so, we define a redistribution flux :math:`\vec{C}_{RD}` that scales the transfer between the two options.  The parameter :math:`p_{tm}`, which varies betwen 0 and 1, sets the relative priority of each.   When the parameter is greater than 0.5, :math:`\vec{C}_{RD}` re-directs flux from relieving maintenance respiration deficit (:math:`\vec{C}_{md}`) towards priority 1 tissues (:math:`\vec{C}_1`).  Alternatively, when the parameter is less than 0.5, :math:`\vec{C}_{RD}` is redictect from replacing priority 1 tissues into maintenance respiration deficit.

.. math::
   :label: h2_redirection_eq

   \text{for }p_{tm}>0.5
   
   \vec{C}_{RD}   &= \quad \text{min}( (p_{tm}-0.5)/0.5 \cdot f_{tot} \cdot R_m, (1-f_{tot}) \cdot \check{C}_1 )
   
   \vec{C}_1      &= \quad f_{tot} \cdot \check{C}_1 + \vec{C}_{RD}
   
   \vec{R}_{md} &= \quad f_{tot} \cdot R_{md} - \vec{C}_{RD}
   
   \text{for }p_{tm}<0.5
   
   \vec{C}_{RD}   &= \quad \text{min}( (0.5-p_{tm})/0.5 \cdot f_{tot} \cdot \check{C}_1, (1-f_{tot}) \cdot R_{md} )

   \vec{C}_1      &= \quad  f_{tot} \cdot \check{C}_1 - C_{RD}
   
   \vec{R}_{md} &= \quad  f_{tot} \cdot R_{md} + \vec{C}_{RD}


The total flux of carbon into each priority 1 pool is then governed, linearly, by the fraction of which their demand constitutes the whole demand.  For any carbon pool in organ found in priority set :math:`\mathbb{O}_1`.

.. math::
   :label: h2_p1_c_vec_eq

   \vec{C}_{(\mathbb{O}_1)} &= \vec{C}_1 \cdot \check{C}_{(\mathbb{O}_1)} / \check{C}_1 
  

With the fluxes known, increment the priority 1 carbon pools, increment their growth respiration.    For each organ in set :math:`\mathbb{O}_1`:

.. math::
   :label: h2_p1_increment_pools_eq

   C_{(\mathbb{O}_1)} &= \quad C_{(\mathbb{O}_1)} + \vec{C}_{(\mathbb{O}_1)} / (1+r_{g(\mathbb{O}_1)})
   
   R_{g(\mathbb{O}_1)} &= \quad R_{g(\mathbb{O}_1)} + \vec{C}_{(\mathbb{O}_1)} \cdot  r_{g(\mathbb{O}_1)} / (1+r_{g(\mathbb{O}_1)})


Decrement maintenance respiration deficit, daily carbon gain, and potentially, storage carbon (where :math:`o=st`).


.. math::
   :label: h2_p1_decrement_pools_eq

   R_{md} &= \quad R_{md} - \vec{R}_{md}

   \vec{C}_{gain} &= \quad \text{min}(C_{gain}, \vec{C}_{tot})

   C_{gain} &= \quad C_{gain} - \vec{C}_{gain}

   C_{(st)} &= \quad C_{(st)} - \text{max}(0,  \vec{C}_{tot} - \vec{C}_{gain})




.. _p1_implicit_rmd_section:

Priority 1 Carbon Fluxes with Implicit Maintenance Respiration
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Recall that as an alternative to :ref:`p1_explicit_rmd_section`, carbon gains may subsume maintenance respiration.  With this assumption, the equations in the previous section are valid in all cases, except for when :math:`C_{gain} < 0`.  For this condition, it is assumed that storage carbon will pay off the negative carbon gain and bring it back to zero.  Caution must be made, in so much that calculations of maintenance respiration are conducted so that the plant does create impossible conditions where storage carbon becomes zero.  PARTEH will fail gracefully in this condition.  The remainder of this section specifically details the condition where :math:`C_{gain}<0`. 

The flux from storage brings negative daily carbon gain up to zero.

.. math::
   :label: h2_implicit_c_gain_eq

   \vec{C}_{gain} &= \quad - C_{gain}

   C_{(st)} &= \quad C_{(st)} - \vec{C}_{gain}
   
   C_{gain} &= 0
   

Any extra flux that can transferred out of storage and into priority 1 tissues, would then be calculated by using the same function that determines the maximum transferrable carbon, as in :eq:`h2_c_st_trans_eq`.  However, in this case :math:`C_{st-trans}` is calculated after the carbon to replace the negative :math:`C_{gain}` is removed in :eq:`h2_implicit_c_gain_eq`.  The demand for carbon to priority 1 tissues follows the same methods as :ref:`p1_explicit_rmd_section`.

.. math::
   :label: h2_implicit_extra_eq

   \text{if} \quad C_{gain} < 0

   \vec{C}_1 &= \quad \text{min}(C_{st-trans},\check{C}_1)


Transfer from storage into priority 1 tissues also follows the same logic as :ref:`p1_explicit_rmd_section`, specifically :eq:`h2_p1_c_vec_eq`.  And finally, decrement storage again, as per total flux into priority 1 organs :math:`\vec{C}_1`.

.. math::
   :label: h2_p1_implicit_store2c1_eq

   C_{(st)} &= \quad C_{(st)} + \vec{C}_{1}



.. _p1_nutrient_section:

Priority 1 Nutrient Fluxes
""""""""""""""""""""""""""

With the priority 1 carbon pools updated, the fluxes of nutrients into those pools can proceed.  The targets :math:`\grave{N}_{(o,s)}`, and subsequently the deficit from the target :math:`\check{N}_{(o,s)}`, is set by the organ of interest's current (and newly updated) carbon mass. For all nutrient species :math:`s`, and all organs :math:`o` in set :math:`\mathbb{O}_1`, the targets and demands are updated via :eq:`h2_n_target_eq` and :eq:`h2_n_demand_eq`.

The total demand for each nutrient species $s$ across priority 1 tissues is thus:

.. math::
   :label: h2_p1_groupnsum_eq

   \check{N}_{1(s)} = & \quad  \sum_{o=\mathbb{O}_{1}} \check{N}_{(o,s)}
  

And therefore the fluxes for each species :math:`s` and each organ in priority set :math:`\mathbb{O}_1` are transferred into their respective pools.

.. math::
   :label: h2_p1_nvec_eq
	   
   \vec{N}_{(\mathbb{O}_1,s)} = & \quad \text{min}(N_{gain(s)},\check{N}_{1(s)}) \cdot ( \check{N}_{(\mathbb{O}_1,s)} / \check{N}_{1(s)} )

   N_{(\mathbb{O}_1,s)} = & \quad N_{(\mathbb{O}_1,s)} + \vec{N}_{(\mathbb{O}_1,s)}


The daily nutrient gains for each species are correspondingly decremented.

.. math::
   :label: h2_p1_n_gain

   N_{gain(s)} = & \quad N_{gain(s)} - \text{min}(N_{gain(s)},\check{N}_{1(s)})


.. _high_p_carbon_nutrient_section:

Carbon and Nutrient Fluxes after Priority Level 1
"""""""""""""""""""""""""""""""""""""""""""""""""

At this point, all priority 1 fluxes have been allocated.  The next priority level fluxes are enacted sequentially, and the procedure is much the same as priority 1, without the complications of shunting carbon to maintenance respiration or paying back negative carbon gains, or **transfering from storage to pay for priority 1 demands**.

For each priority level :math:`pr`, a new set of organs is sub-set into group :math:`\mathbb{O}_{pr}`, thereby calculating fluxes of carbon and nutrients and decrementing :math:`C_{gain}` and :math:`N_{gain(s)}` correspondingly.  The algorithm follows generally:

1. Sum the carbon demands of the set, via :eq:`h2_c_priority1_sum_eq`
2. Calculate carbon fluxes based on relative demand, similar to :eq:`h2_p1_c_vec_eq`
3. Increment carbon pools, growth respiration and decrement carbon gain, via :eq:`h2_p1_increment_pools_eq` and :eq:`h2_p1_decrement_pools_eq` (ignoring parts where storage is translocated)
4. Re-assess nutrient demands, via :eq:`h2_n_demand_eq`
5. Sum the nutrient demands of the set, similar too :eq:`h2_p1_groupnsum_eq`
6. Calculate nutrient fluxes and perform transfers, similar to :eq:`h2_p1_nvec_eq` and :eq:`h2_p1_n_gain`


.. _h2_grow_stature_section:

Grow Stature Concurrently
^^^^^^^^^^^^^^^^^^^^^^^^^

If there is at some of each daily carbon gain, and daily nutrient gain for all species remaining, the plant will grow out its stature.  This method assumes that the organs will grow out concurrently.  

As a default, the carbon in these organs will be allocated as dictated by the derivatives of the allometric functions.  Other hypotheses, such as those that seek to optimize root tissues to increase nutrient acquisition will break from this.  

Of important note, is that for either reasons governed outside of the PARTEH framework, or because of numerical integration errors, some organs may have slightly more carbon than their allometric target.  In doing so, we remove these organs from the set to be grown out.  Structural carbon is an exception, and is always "on-allometry", since it is directly tied to stature and dbh.  This is actually forced by adjusting the plant's diameter to match the structural carbon in cases where structural carbon was higher than its allometric target.  

Broadly, the first objective in this section, is to determine which species, be it carbon or nutrient, will limit growth.  To do this, we calculate an approximation of how much equivalent growth in carbon each of them could provide, by extrapolating the derivatives at the current plant's stature.  The derivatives for target carbon per change in diameter, :math:`\frac{dC_{(o)}}{dd}`, are provided by allometric functions. In the following set of organs, we exlude reproduction (which does not have a derivative wrt size), creating subset of organs :math:`\mathbb{O}_{sg}`.

.. math::
   :label: h2_sg_sum_dcgdd_eq

   \frac{dC_{sg}}{dd}     =& \quad \sum_{o=\mathbb{O}_{sg}} \frac{d\grave{C}_{(o)}}{dd}

With this sum, we can determine the relative fraction of carbon that is sent to each organ in set :math:`\mathbb{O}_{sg}`, that is directed to stature growth, denoted:  :math:`f_{sg(\mathbb{O}_{sg})}`.  The fraction of flux that is directed towards reproductive organs, :math:`f_{sg(o=repro)}` is special, and is **calculated from an external module**.  

For the other organs, in set :math:`\mathbb{O}_{sg}`:

.. math::
   :label: h2_sg_c_reflrac_eq

   f_{sg(\mathbb{O}_{sg})} &= \frac{d\grave{C}_{(\mathbb{O}_{sg})}}{dd} / \frac{dC_{sg}}{dd} \cdot (1 - f_{sg(repro)})


The approximated amount of carbon would be transferred into plant tissues :math:`\vec{C}^*_{sg}`, is calculated via assembling these relative fractions, and divesting the total available carbon gain of the growth respiration rates for each pool.  Note the asterisk in the symbology is meant to reflect an approximate value.

.. math::
   :label: h2_sg_c_limiting_sum_eq

   \vec{C}^*_{sg}  \quad    &= \quad C_{gain} \cdot \left( \frac{ f_{sg(repro)} }{ 1 + r_{g(repro)}} + \sum_{o=\mathbb{O}_{sg}} \frac{ f_{sg(o)} }{ 1 + r_g(o) }  \right)
	   
   
The approximated amount of nutrient of each species :math:`s` that would be transferred into plant tissues :math:`\vec{N}^*_{sg(s)}`, is calculated much in the same way, however there is no growth respiration tax.  Also, it is possible that a nutrient pool may have a mass that is already greater than the mass equivalent to the target associated with the minimum stoichiometry. Such cases must be accounted for, because they will reduce the likelihood of nutrient needs in that organ limiting growth.

.. math::
   :label: h2_sg_n_limiting_sum_eq

   \vec{N}^*_{sg(s)} \quad  &= \quad N_{gain(s)} \cdot \left( f_{sg(repro)} / \beta_{(repro,s)}  + \sum_{o=\mathbb{O}_{sg}} f_{sg(o)} / \beta_{(o,s)} \right) \\
                     &+ \quad \text{max}(0,N_{(repro,s)} - \grave{N}_{(repro,s)}) / \beta_{(repro,s)} \\
		     &+ \quad \sum_{o=\mathbb{O}_{sg}} \text{max}(0,N_{(o,s)} - \grave{N}_{(o,s)}) / \beta_{(o,s)}


The actual carbon that is then set aside for stature growth :math:`\vec{C}_{sg}` , based on the minimum of approximations :math:`\vec{C}^*_{sg}` and :math:`\vec{N}^*_{sg(s)}`.


.. math::
   :label: h2_sg_c_gstature_eq

   \vec{C}_{sg} &= \quad C_{gain} \cdot \text{min}( \vec{C}^*_{sg}, \vec{N}^*_{sg(s)} ) /  \vec{C}^*_{sg}


Carbon fluxes into each of the plant's organs are conducted via numerical integration, which is a coupled set of ordinary differential equations, integrated over :math:`\vec{C}_{sg}`.  For each organ in set :math:`\mathbb{O}_{sg}`.

Where the rate of change of carbon for a given organ is its proportionality relative to the whole:

.. math::
   :label: h2_sg_ode_eq

   \frac{d C_{(\mathbb{O}_{sg})}}{d C_{sg}} &=  \quad \overbrace{\frac{dC_{(\mathbb{O}_{sg})}}{dd} \cdot \frac{dd}{d C_{sg}}}^{ \text{Continuous allometry equations} } 

.. math::
   :label: h2_sg_ode_int_eq

   \vec{C}_{sg(\mathbb{O}_{sg})} &=  \quad  \int_{dC_{sg}=0}^{\vec{C}_{sg}}  \frac{d C_{(\mathbb{O}_{sg})}}{d C_{sg}} dC_{sg}

The fluxes are then transferred to increment the carbon pools, increment the growth respiration and decrement the carbon gain.


.. _h2_transfer_ideal_nutrients_section:

Allocate Nutrients Towards Ideal Stoichiometric Ratios
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


.. _h2_exude_section:

Send Excess Quantities to Storage or Exude to Soil
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
