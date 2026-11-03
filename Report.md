# Step 0 -  Defining the Problem

I am following the guideline of the project from the local app and the lab description.

In the lab description this is the design inputs that we can modify/Optimise.
```
The following additional design inputs need to be optimised:
- Wall thermal insulation (e.g. thickness of the insulation material)
- Roof thermal insulation
- Window types (Sgl, Dbl, Tpl)
- WWR
- Lighting system (among 3 choices)
- Ventilative cooling (ACH-ventilation – summer) control temperatures
- Ventilative cooling (ACH-ventilation summer) airflow values
- Shading control temperature & GHI
- (mechanical ventilation flow rates for IAQ)
```

*Q: Can we modify the external layer of the building ? --> the outside brick layer*\
*A: We can only modify the Insulation Layer and its thickness*
*Need for the range of the thickness*\
*Q: Can we just select all or Swap the whole build-up ?*\
*A: The Swap the whole build-up is going to change the whole structure of the build that is simulated.*




I have define the problem (that will be modify) and got this cell problem:

<details>
<summary>Here</summary>

```
# Define the problem: what to vary (parameters), what to improve (objectives), how EnergyPlus is run.
# Written by the platform from the problem you defined; change it as you like.
# It needs two names from the cell above: idf (the model, bdopt.load_idf) and epw_file (the weather file).
# Every selector is checked against the model when the evaluator is created, with suggestions for a typo.
from bdopt import EnergyPlusEvaluator, FieldSelector, Parameter, Problem, RangeParameter
from bdopt.sampling import lhs

# 8 parameters = 8 columns of `samples`, however many objects each one writes.
parameters = []

# External walls conductivity Brickwork Outer_.1
#   writes 1 Material object, field "Conductivity"
external_walls_conductivity_sel = FieldSelector(
    class_name="Material",
    object_name="Brickwork Outer_.1",
    field_name="Conductivity",
)
parameters.append(Parameter(
    selector=external_walls_conductivity_sel,
    value_descriptor=RangeParameter(0.42, 1.3),
    name="External walls conductivity Brickwork Outer_.1",
))

# External walls conductivity _InsulationEX_.01
#   writes 1 Material object, field "Conductivity"
external_walls_conductivity_2_sel = FieldSelector(
    class_name="Material",
    object_name="_InsulationEX_.01",
    field_name="Conductivity",
)
parameters.append(Parameter(
    selector=external_walls_conductivity_2_sel,
    value_descriptor=RangeParameter(0.017, 0.051),
    name="External walls conductivity _InsulationEX_.01",
))

# External walls conductivity Concrete Block (Medium)_.1
#   writes 1 Material object, field "Conductivity"
external_walls_conductivity_3_sel = FieldSelector(
    class_name="Material",
    object_name="Concrete Block (Medium)_.1",
    field_name="Conductivity",
)
parameters.append(Parameter(
    selector=external_walls_conductivity_3_sel,
    value_descriptor=RangeParameter(0.26, 0.77),
    name="External walls conductivity Concrete Block (Medium)_.1",
))

# External walls conductivity Gypsum Plastering_.013
#   writes 1 Material object, field "Conductivity"
external_walls_conductivity_4_sel = FieldSelector(
    class_name="Material",
    object_name="Gypsum Plastering_.013",
    field_name="Conductivity",
)
parameters.append(Parameter(
    selector=external_walls_conductivity_4_sel,
    value_descriptor=RangeParameter(0.2, 0.6),
    name="External walls conductivity Gypsum Plastering_.013",
))

# Roof conductivity Plasterboard_.013
#   writes 1 Material object, field "Conductivity"
roof_conductivity_plasterboa_sel = FieldSelector(
    class_name="Material",
    object_name="Plasterboard_.013",
    field_name="Conductivity",
)
parameters.append(Parameter(
    selector=roof_conductivity_plasterboa_sel,
    value_descriptor=RangeParameter(0.13, 0.38),
    name="Roof conductivity Plasterboard_.013",
))

# Roof thermal resistance 3_RVAL_3
#   writes 1 Material:NoMass object, field "Thermal Resistance"
roof_thermal_resistance_3_rv_sel = FieldSelector(
    class_name="Material:NoMass",
    object_name="3_RVAL_3",
    field_name="Thermal Resistance",
)
parameters.append(Parameter(
    selector=roof_thermal_resistance_3_rv_sel,
    value_descriptor=RangeParameter(0.09, 0.27),
    name="Roof thermal resistance 3_RVAL_3",
))

# Roof conductivity _InsulationEX_Roof_.01
#   writes 1 Material object, field "Conductivity"
roof_conductivity_insulation_sel = FieldSelector(
    class_name="Material",
    object_name="_InsulationEX_Roof_.01",
    field_name="Conductivity",
)
parameters.append(Parameter(
    selector=roof_conductivity_insulation_sel,
    value_descriptor=RangeParameter(0.02, 0.06),
    name="Roof conductivity _InsulationEX_Roof_.01",
))

# Roof conductivity Asphalt 1_.01
#   writes 1 Material object, field "Conductivity"
roof_conductivity_asphalt_1_sel = FieldSelector(
    class_name="Material",
    object_name="Asphalt 1_.01",
    field_name="Conductivity",
)
parameters.append(Parameter(
    selector=roof_conductivity_asphalt_1_sel,
    value_descriptor=RangeParameter(0.35, 1.1),
    name="Roof conductivity Asphalt 1_.01",
))

# Objectives: meters, summed over the run period, in joules.
# A plain name is minimised; MeterReader(name, direction="max") maximises.
objectives = [
    "Electricity:Facility",
    "DistrictCooling:Facility",
    "DistrictHeating:Facility",
]
problem = Problem(parameters, objectives)

# EnergyPlus writes every Output:Variable the model asks for into its results database, and a detailed model
# asks for hundreds. The objectives are meters, which the evaluator requests by itself, so none of those
# variables is needed: they are removed from the copy that is simulated (idf itself is left as it is).
# An objective that reads a variable, e.g. VariableReader, makes the evaluator request that variable itself.
sim_idf = idf.copy()
sim_idf.remove(sim_idf.by_class("Output:Variable"))
evaluator = EnergyPlusEvaluator(
    problem, sim_idf, epw_file,
    out_dir="/tmp/bdopt_runs",  # run folders on the container's own disk: much faster than a shared folder
    err_dir="outputdir",        # a run that fails is kept here, with EnergyPlus' error file
    processes=2,                # EnergyPlus runs in parallel
)

samples = lhs(problem, 20)  # 20 Latin hypercube samples: a row per design, a column per parameter
# lhs(problem, 20, seed=1) gives the same samples every time, and runs already done are not repeated (the cache lives in /tmp/bdopt_runs, so until the container is recreated).

```

</details>

*Q: What is the use of the problem cell ?*

First version of the surrogate model with Problem definition : [Surrogate.ipynb](/SurrogateNotebook.ipynb)


After Talking with the teacher I modify the problem cell.

<details>
<summary>Here</summary>

```
# Define the problem: what to vary (parameters), what to improve (objectives), how EnergyPlus is run.
# Written by the platform from the problem you defined; change it as you like.
# It needs two names from the cell above: idf (the model, bdopt.load_idf) and epw_file (the weather file).
# Every selector is checked against the model when the evaluator is created, with suggestions for a typo.
from bdopt import (
    CategoryParameter, EnergyPlusEvaluator, FieldSelector, Parameter, Problem, RangeParameter,
)
from bdopt.sampling import lhs

# 8 parameters = 8 columns of `samples`, however many objects each one writes.
parameters = []

# External walls conductivity _InsulationEX_.01
#   writes 1 Material object, field "Conductivity"
external_walls_conductivity_sel = FieldSelector(
    class_name="Material",
    object_name="_InsulationEX_.01",
    field_name="Conductivity",
)
parameters.append(Parameter(
    selector=external_walls_conductivity_sel,
    value_descriptor=RangeParameter(0.017, 0.051),
    name="External walls conductivity _InsulationEX_.01",
))

# Roof conductivity _InsulationEX_Roof_.01
#   writes 1 Material object, field "Conductivity"
roof_conductivity_insulation_sel = FieldSelector(
    class_name="Material",
    object_name="_InsulationEX_Roof_.01",
    field_name="Conductivity",
)
parameters.append(Parameter(
    selector=roof_conductivity_insulation_sel,
    value_descriptor=RangeParameter(0.02, 0.06),
    name="Roof conductivity _InsulationEX_Roof_.01",
))

# External walls thickness _InsulationEX_.01
#   writes 1 Material object, field "Thickness"
external_walls_thickness_ins_sel = FieldSelector(
    class_name="Material",
    object_name="_InsulationEX_.01",
    field_name="Thickness",
)
parameters.append(Parameter(
    selector=external_walls_thickness_ins_sel,
    value_descriptor=RangeParameter(0.02, 0.3),
    name="External walls thickness _InsulationEX_.01",
))

# Roof thickness _InsulationEX_Roof_.01
#   writes 1 Material object, field "Thickness"
roof_thickness_insulationex_sel = FieldSelector(
    class_name="Material",
    object_name="_InsulationEX_Roof_.01",
    field_name="Thickness",
)
parameters.append(Parameter(
    selector=roof_thickness_insulationex_sel,
    value_descriptor=RangeParameter(0.02, 0.3),
    name="Roof thickness _InsulationEX_Roof_.01",
))

# People Activity Level Schedule Name
#   writes every People object, field "Activity Level Schedule Name"
#   The column holds the option itself; problem.json numbered the options:
#     1 = Activity Schedule 285, 2 = Activity Schedule 292, 3 = Activity Schedule 299,
#     4 = Activity Schedule 306, 5 = Activity Schedule 313
people_activity_level_schedu_sel = FieldSelector(
    class_name="People",
    object_name="*",
    field_name="Activity Level Schedule Name",
)
people_activity_level_schedu_options = CategoryParameter([
    "Activity Schedule 285", "Activity Schedule 292", "Activity Schedule 299",
    "Activity Schedule 306", "Activity Schedule 313",
])
parameters.append(Parameter(
    selector=people_activity_level_schedu_sel,
    value_descriptor=people_activity_level_schedu_options,
    name="People Activity Level Schedule Name",
))

# ZoneVentilation:DesignFlowRate Air Changes per Hour
#   writes every ZoneVentilation:DesignFlowRate object, field "Air Changes per Hour"
zoneventilation_designflowra_sel = FieldSelector(
    class_name="ZoneVentilation:DesignFlowRate",
    object_name="*",
    field_name="Air Changes per Hour",
)
parameters.append(Parameter(
    selector=zoneventilation_designflowra_sel,
    value_descriptor=RangeParameter(0.5, 1.5),
    name="ZoneVentilation:DesignFlowRate Air Changes per Hour",
))

# ZoneInfiltration:DesignFlowRate Air Changes per Hour
#   writes every ZoneInfiltration:DesignFlowRate object, field "Air Changes per Hour"
zoneinfiltration_designflowr_sel = FieldSelector(
    class_name="ZoneInfiltration:DesignFlowRate",
    object_name="*",
    field_name="Air Changes per Hour",
)
parameters.append(Parameter(
    selector=zoneinfiltration_designflowr_sel,
    value_descriptor=RangeParameter(0.05, 0.15),
    name="ZoneInfiltration:DesignFlowRate Air Changes per Hour",
))

# ZoneHVAC:IdealLoadsAirSystem Cooling Availability Schedule Name
#   writes every ZoneHVAC:IdealLoadsAirSystem object, field "Cooling Availability Schedule Name"
#   The column holds the option itself; problem.json numbered the options:
#     1 = Block1:OfficeXSWX1f Cooling Availability Sch,
#     2 = Block1:OfficeXSEX1f Cooling Availability Sch,
#     3 = Block1:OfficeXNWX1f Cooling Availability Sch,
#     4 = Block1:OfficeXNEX1f Cooling Availability Sch,
#     5 = Block1:CorridorX1f Cooling Availability Sch
zonehvac_idealloadsairsystem_sel = FieldSelector(
    class_name="ZoneHVAC:IdealLoadsAirSystem",
    object_name="*",
    field_name="Cooling Availability Schedule Name",
)
zonehvac_idealloadsairsystem_options = CategoryParameter([
    "Block1:OfficeXSWX1f Cooling Availability Sch", "Block1:OfficeXSEX1f Cooling Availability Sch",
    "Block1:OfficeXNWX1f Cooling Availability Sch", "Block1:OfficeXNEX1f Cooling Availability Sch",
    "Block1:CorridorX1f Cooling Availability Sch",
])
parameters.append(Parameter(
    selector=zonehvac_idealloadsairsystem_sel,
    value_descriptor=zonehvac_idealloadsairsystem_options,
    name="ZoneHVAC:IdealLoadsAirSystem Cooling Availability Schedule Name",
))

# Objectives: meters, summed over the run period, in joules.
# A plain name is minimised; MeterReader(name, direction="max") maximises.
objectives = [
    "Electricity:Facility",
    "DistrictCooling:Facility",
    "DistrictHeating:Facility",
]
problem = Problem(parameters, objectives)

# EnergyPlus writes every Output:Variable the model asks for into its results database, and a detailed model
# asks for hundreds. The objectives are meters, which the evaluator requests by itself, so none of those
# variables is needed: they are removed from the copy that is simulated (idf itself is left as it is).
# An objective that reads a variable, e.g. VariableReader, makes the evaluator request that variable itself.
sim_idf = idf.copy()
sim_idf.remove(sim_idf.by_class("Output:Variable"))
evaluator = EnergyPlusEvaluator(
    problem, sim_idf, epw_file,
    out_dir="/tmp/bdopt_runs",  # run folders on the container's own disk: much faster than a shared folder
    err_dir="outputdir",        # a run that fails is kept here, with EnergyPlus' error file
    processes=2,                # EnergyPlus runs in parallel
)

samples = lhs(problem, 20)  # 20 Latin hypercube samples: a row per design, a column per parameter
# lhs(problem, 20, seed=1) gives the same samples every time, and runs already done are not repeated (the cache lives in /tmp/bdopt_runs, so until the container is recreated).

```

</details>

