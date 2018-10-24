
.. _turnover_overview_section:

Overview of Turnover
--------------------

In the turnover phase, biomass is removed from the plant due to turnover associated with both maintenance and event based processes.

1. Maintenance turnover is continuous, and typically applies to the constant overturn of ageing leaves and fine-roots in evergreens, and continuous branchfall in most all species
2. Event based turnover refers to phenology and/or seasonal losses of leaves and fine-roots in deciduous plants, or losses due to the damage from storms or fire.


The parameters that govern the rate of turnover are applicable to the maintenance rates.  The severity of losses due to events are governed by external modules.  Since a plant must be exclusively one of evergreen or deciduous, the parameters that govern retranslocation are applicable in each context.  If a plant is evergreen, the leaf retranslocation parameter for that functional type is relevant to maintenance turnover process in leaves.  If a plant is deciduous, the leaf retranslocation parameter is relevant to the seasonal or stress induced drop processes.  The table :ref:`turnover_params_table` describes the parameters.


.. _turnover_params_table: 

.. table:: Turnover Parameters
   :align: center

   +------------------------+----------------+------------------------------------------+---------+
   | Symbol                 | Dimension      | Description                              | Units   |
   +========================+================+==========================================+=========+
   | :math:`\tau_l`         | pft*           | leaf maintenance turnover timescale      | [years] |
   +------------------------+----------------+------------------------------------------+---------+
   | :math:`\tau_f`         | pft*           | fine-root maintenance turnover timescale | [years] |
   +------------------------+----------------+------------------------------------------+---------+
   | :math:`\tau_{b}`       | pft*           | branch turnover timescale                | [years] |
   +------------------------+----------------+------------------------------------------+---------+
   | :math:`\eta_{c(o)}`    | pft* x organ   | carbon retranslocation fraction          | [kg/kg] |
   +------------------------+----------------+------------------------------------------+---------+
   | :math:`\eta_{n(o)}`    | pft* x organ   | nitrogen retranslocation fraction        | [kg/kg] |
   +------------------------+----------------+------------------------------------------+---------+
   | :math:`\eta_{p(o)}`    | pft* x organ   | phosphorous retranslocation fraction     | [kg/kg] |
   +------------------------+----------------+------------------------------------------+---------+

*List of key parameters used for turnover processes. :math:`*` Note that the parameters are specified explicitly for each pft, but the dimension will be implied in our notation as each plant is already uniquely asociated with a PFT.*


.. _maintenance_turnover_section:

Maintenance Turnover Hypotheses
-------------------------------

.. _constant_fraction_turnover_section:

Constant Fraction Maintenance Turnover and Retranslocation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Constant fraction turnover can be applied to any arbitrary mass pool.  The loss rates are governed by the turnover parameters for leaves, fine-roots and branches, as well as the re-translocation fractions.  See :ref:`turnover_params_table`

Turnover losses to leaves (organ set :math:`o=\mathbb{O}_l`) and fineroots (organ set :math:`o=\mathbb{O}_f`) are dictated by their turnover timescale parameters :math:`\tau_l` and :math:`\tau_f` respectively.  Branchfall affects the pools of sapwood, structure, storage and reproduction (if non-zero), which have the branchfall set of organs :math:`o = \mathbb{O}_b`.  Note that with no re-translocation of nutrients, these rates apply to all nutrient species.  The turnover timescale is in units of [years-1], the elapsed time :math:`\Delta_{yr}` is in units of years (which in practice is 1/365).  Some amount of nutrient of each species :math:`s` may be re-translocated  directly back into the existing pool as proportions dictated by :math:`\eta_{l(s)}` and :math:`\eta_{f(s)}` in leaves and fine-root respectively.  The turnover flux   for carbon :math:`\tilde{C}` and nutrient species :math:`\tilde{N}` are calculated as:

.. math::
   :label: maint_turn_lossfluxes_eq

   \text{leaves}

   \tilde{C}_{(\mathbb{O}_l)} &= C_{(\mathbb{O}_l)} \cdot \tau_l \cdot \Delta_{yr}

   \tilde{N}_{(\mathbb{O}_l,s)} &=  N_{(\mathbb{O}_l,s)} \cdot \tau_l \cdot  \Delta_{yr}  \cdot (1-\eta_{c(ft,\mathbb{O}_l )})

   \text{fine-roots}

   \tilde{C}_{(\mathbb{O}_f)} &=  C_{(\mathbb{O}_f)}  \cdot \tau_f \cdot \Delta_{yr}

   \tilde{N}_{(\mathbb{O}_f,s)} &= N_{(\mathbb{O}_f,s)} \cdot \tau_f \cdot \Delta_{yr} \cdot ((1-\eta_{*(ft,\mathbb{O}_f )})

   \text{branches}

   \tilde{C}_{(\mathbb{O}_b,s)} &=  C_{(\mathbb{O}_b,s)} \cdot  \tau_{b} \cdot \Delta_{yr}

   \tilde{N}_{(\mathbb{O}_b,s)} &=  N_{(\mathbb{O}_b,s)} \cdot  \tau_{b} \cdot \Delta_{yr}

Note that as an end-user of the FATES model, the retranslocation factors are defined in separate arrays by species.  The notation we use in the above equations are simplified to indicate that the nutrient retranslocation factors specific to the species of interest, e.g. :math:`\eta_{*(ft,\mathbb{O}_f )}`.
These loss fluxes are directly removed from the state variables for any organ :math:`o` and species :math:`s`:

.. math::

   C_{(o)} &= C_{(o)} - \tilde{C}_{(o)}

   N_{(o,s)} &= N_{(o,s)} - \tilde{N}_{(o,s)}




.. _event_turnover_section:

Event Based Turnover Hypotheses
-----------------------------------------------------------


.. _const_frac_event_turnover_section:

Event Based Turnover with Simple Retranslocation Hypotheses
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For event-based turnover, the host model must provide PARTEH with the fractions of the biomass that should be removed from each organ in the event.  Depending on the context, PARTEH will or will-not implement re-translocation of nutrients.  

PARTEH will implement re-translocation for these events:

1. Deciduous leaf drop


PARTEH will not implement re-translocation for these events:

1. Fire losses
2. Herbivory
3. Storms

The procedures for both contexts are similar, where in the later, the re-translocation factors can be assumed as zero.

In all situations, when the events are triggered, a fraction mass lost must be passed in as the argument.  As an example, for deciduous leaf drop, the fraction of dropped leave :math:`f_{drop}` is assessed from the phenology module and passed into the PARTEH module. We define a mass :math:`M` which is represented for any carbon or nutrient species present, (defined by species set :math:`s=\mathbb{S}`), and each organ in that set :math:`o=\mathbb{O}_l` (perhaps there are multiple leaf organs). For all species and organs in that set, we define the turnover (or loss) mass :math:`\vec{M}_{loss(\mathbb{S},\mathbb{O}_l)}` and the re-translocated mass :math:`\vec{M}_{retr(\mathbb{S},\mathbb{O}_l)}` which is destined for storage :math:`M_{(\mathbb{S},st)}`.


.. math::
   :label: event_turn_lossfluxes_eq

   \vec{M}_{loss(\mathbb{S},\mathbb{O}_l)} &= \quad  (1-\eta_{*(ft,\mathbb{O}_l )}) \cdot M_{(\mathbb{S},\mathbb{O}_l)} \cdot f_{drop}

   \vec{M}_{retr(\mathbb{S},\mathbb{O}_l)} &= \quad  \eta_{*(ft,\mathbb{O}_l )}  \cdot M_{(\mathbb{S},\mathbb{O}_l)} \cdot f_{drop}

Both fluxes decrement the pool of interest, while the loss flux leaves the live plant's control volume, and the retranslocated mass increments storage carbon.


.. math::
   :label: event_turn_incrementfluxes_eq

   M_{(\mathbb{S},\mathbb{O}_l)} &= \quad M_{(\mathbb{S},\mathbb{O}_l)} - (\vec{M}_{loss(\mathbb{S},\mathbb{O}_l)} + \vec{M}_{retr(\mathbb{S},\mathbb{O}_l)})

   M_{(\mathbb{S},st)} &=\quad M_{(\mathbb{S},st)} + \vec{M}_{retr(\mathbb{S},\mathbb{O}_l)}

   

   
