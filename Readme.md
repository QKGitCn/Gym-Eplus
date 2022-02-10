# PraeklimaControl Baseline-Rule-Based-Control
## 1. run .py script in Jupyter
```
%run pyScripts/framework.py -d RUNDIRs/test/ -w weather/DEU_Berlin.103840_IWEC.epw -r models/test/211123_ModelOffice.idf
```
## 2. Working Repository:

- PraeklimaControl(main repo)
	- models 
		- test ((keeps all the IDF models here, you can choose to run one by commands))
	- pyScripts (keeps all the .py python scripts here, I can send you updated .py files by .RAR and you can just replace them with new ones)
		- Baseline_logs (keeps all the .CSV file and figures that created by the python scripts during simulations)
		- framework.py
		- variabels.py
		- estIllum.py (define some calculation functions)
	- RUNDIRs
		- test (keeps all the .EDD, RDD, .ERR, .ESO runing log files after simulation, will be overwriten every time you run the simulation)
	- weather (keeps the .EPW weather files)
	- run_command.ipynb (Jupyter notebook file, run the command inside)
	- md_images (local images in README.md)
	- RL_prototype (old RL scripts for single actuator)

## 3. E+ APIs
### State:
Prior to calling into most of the API functions, the API client must create a new state instance. This state instance is not to be directly accessed by the client, but instead the client must simply be a courier of this state instance and pass it in and out of the API calls. Whenever the client is done with that instance, the instance can be reset to be prepped for another run of EnergyPlus, or destroyed if the client is done with it.
### Runtime:
The “Runtime” API allows for users to hook into a running simulation. The client creates functions, and using this runtime API, they can register these functions to be called back at specific points in a simulation.
```
import sys
from pyenergyplus.api import EnergyPlusAPI

sys.path.insert(0,"path to Energyplus")
sys.path.append("path to work repository")

api = EnergyPlusAPI()
state = api.state_manager.new_state()
api.runtime.run_energyplus(state, sys.argv[1:])
api.state_manager.reset_state(state)
```
### Data exchange:
Define all variables in "variables.py", imported as "my_vars"
Get sensor handler ID:
```
my_vars.varID_ppl_occ_cnt = api.exchange.get_variable_handle(state, my_vars.varStr_ppl_occ_cnt, u"PEOPLESCHED 1")
```
Get actuator-sensor handler ID:
```
my_vars.varID_win_blind_slat_angle = api.exchange.get_variable_handle(state, my_vars.varStr_win_blind_slat_angle, u"Sub Surface 1")
```
Read a sensor:
```
my_vars.var_ppl_occ_cnt = api.exchange.get_variable_value(state, my_vars.varID_ppl_occ_cnt)
```
Get actuator handler ID:
```
my_vars.actID_win_slat_angle = api.exchange.get_actuator_handle(state, "Window Shading Control", my_vars.actStr_win_slat_angle, "Sub Surface 1")
```
Assign value to actuator:
```
api.exchange.set_actuator_value(state, my_vars.actID_light_act, my_vars.act_light_act)
```
Others:
```
hour = api.exchange.hour(state) # Time in Hour from 0-24
day = api.exchange.day_of_week(state) # weekday from 0-6
month = api.exchange.month(state) # month from 1-12
daytime = api.exchange.sun_is_up(state) # True before sun down
```
The user’s choice for Number of Timesteps per Hour must be evenly divisible into 60; the allowable choices are 1, 2, 3, 4, 5, 6, 10, 12, 15, 20, 30, and 60.
```
current_timestep = api.exchange.system_time_step(state) # current timestep in model.idf
```
### Sensors/Actuators in the algorithm
| Object | Variable_Name | Variable_Key | Unit |Range | Variable_Handle |
| :---: | :---: | :---: | :---: | :---: | :---: |
|Sensor|Zone Air CO2 Concentration|THERMAL ZONE 1|	ppm	||	41|
|Sensor|Zone Operative Temperature|THERMAL ZONE 1 |C||23|
|Sensor	|Zone Air Relative Humidity|THERMAL ZONE 1|	%||40|
|Sensor	|"Zonellumminance(daylight in zone)"|EMS|Lux||16|
|Sensor	|People Occupant Count|THERMAL ZONE 1|per/m2||11|
|Sensor	|Site Outdoor Air Drybulb Temperature|Environment|C||1|
|Sensor	|Site Outdoor Air Dewpoint Temperature|Environment|C||2|
|Act_sensor	|Surface Window Blind Slat Angle|SUB SURFACE 1|	degree| 0-180|21|
|Act_sensor	|Lights Electricity Rate|THERMAL ZONE 1 LIGHTS|	watt||12|
|Act_sensor	|AFN Surface Venting Window or Door Opening Factor|SUB SURFACE 1|fraction |0-1|32|
|Act_sensor	|Zone Ventilation Current Density Volume Flow Rate|THERMAL ZONE 1|m3/s||27|
						
|Object	|Control_Type|	Component_Type|	Component_Name|	Unit|Range|	Actuator_Handle|
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|Actuator	|Slat Angle|	Window Shading Control	|SUB SURFACE 1	|degree|	0-180|	117|
|Actuator	|Lights Electricity Rate|	Lights|	THERMAL ZONE 1 LIGHTS|	watt	||	114|
|Actuator	|Venting Opening Factor|AirFlow Network Window/Door Opening	|SUB SURFACE 1|	fraction|0-1	|165|
|Actuator	|"Air Exchange Flow Rate???"|	Zone Ventilation	|"THERMAL ZONE 1 VENTILATION PER AREA"/ "WINDOWSTACKED AREA1"|	m3/s|	|163/164|

![image](/md_image/Isolated.png)
