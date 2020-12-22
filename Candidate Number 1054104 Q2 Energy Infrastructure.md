```python
import pandas as pd
import pandapower as pp
from pandapower import plotting
import geopandas as gpd
import numpy as np
import matplotlib.pyplot as plt
from shapely.geometry import Point, LineString
import math
import numba
```


```python
#The busiest time for the lines in the Balearic Islands on the 27th of June was found to be 13:20 (671.5 MW of Generation and 672 MW of demand)
# Empty net
net = pp.create_empty_network()

#Menorca Buses with location

mahon_loc=(39.897058, 4.258246)
dragonera_loc=(39.890857, 4.236704)
mercadal_loc=(39.977237, 4.095480)
ciudadela_loc=(40.003247, 3.855644)
calanbosch_loc=(39.932103, 3.833833)

#Mallorca Buses with location
esbesons_loc=(39.581950, 3.158578)
llubi_loc=(39.673730, 3.039215)
sonreus_loc=(39.651175, 2.678874)
valldurgent_loc=(39.584276, 2.549038)
sonorlandis_loc=(39.599929, 2.743859)
alcudia_loc=(39.809501, 3.093032)
calamesquida_loc=(39.739387, 3.428000)
santaponsa_loc=(39.536279, 2.509556)
castresorer_loc=(39.568605, 2.723820)

#Ibiza and Formentera Buses with location

ibizaformentera_loc=(38.919696, 1.430453)

#Cable lengths Menorca (km)

calamesquida_calanbosch=41.3
calanbosch_ciud=8.4
ciud_drag=34.8
ciud_merc=21.2
drag_mahon=5.1
merc_drag=15.5

#Cable lengths Ibiza/Formentera (km)

ibiza_mallorca=126 

#Cable lengths Mallorca (km)

santaponsa_valldurgent=6.5
valldurgent_sonreus1=15.1
valldurgent_sonreus2=15.1
sonreus_sonor=9.3
castres_sonor = 4.1
sonreus_llubi=34.6
sonor_llubi=29
llubi_MTR1=17
llubi_MTR2=16
llubi_esbe1=15.3
llubi_esbe2=15.3
esbe_calamesquida=30.5




```


```python

#power factor = 0.9, Q = P*tan(arccos(0.9))
#tan(arccos(0.9))=0.484322
PF= 0.484322


#Menorca Buses
b_calanbosch = pp.create_bus(net, vn_kv=132, name = "Cal an Bosch", geodata=calanbosch_loc)
b_ciudadela = pp.create_bus(net, vn_kv=132, name = "Ciudadella", geodata=ciudadela_loc)
b_mahon = pp.create_bus(net, vn_kv=132, name = "Mahon", geodata=mahon_loc)
b_dragon = pp.create_bus(net, vn_kv=132, name = "Dragonera", geodata=dragonera_loc)
b_mercadal = pp.create_bus(net, vn_kv=132, name = "Mercadal", geodata=mercadal_loc)

#Mallorca Buses

b_valldurgent = pp.create_bus(net, vn_kv=220, name = "Valldurgent", geodata=valldurgent_loc)
b_llubi = pp.create_bus(net, vn_kv=220, name = "LLubi", geodata=llubi_loc)
b_sonreus = pp.create_bus(net, vn_kv=220, name = "Son Rues", geodata=sonreus_loc)
b_sonorlandis = pp.create_bus(net, vn_kv=220, name = "Son Orlandis", geodata=sonorlandis_loc)
b_alcudia = pp.create_bus(net, vn_kv=220, name = "Alcudia", geodata=alcudia_loc)
b_calamesq = pp.create_bus(net, vn_kv=132, name = "Cala Mesquida", geodata=calamesquida_loc)
b_esbe_lv = pp.create_bus(net, vn_kv=132, name = "Es Bessons LV", geodata=esbesons_loc)
b_esbe_hv = pp.create_bus(net, vn_kv=220, name = "Es Bessons HV", geodata=esbesons_loc)
b_castres = pp.create_bus(net, vn_kv=220, name = "Cas Tresorer", geodata=castresorer_loc)


#Ibiza/Formentera Buses
b_ibizaformentera = pp.create_bus(net, vn_kv=132, name = "Ibiza/Formentera", geodata=ibizaformentera_loc)

#create buses for mainland interconnector and trafo

b_sp_lv = pp.create_bus(net, vn_kv=132, name = "SP HVDC LV", geodata=santaponsa_loc) 
b_sp_hv = pp.create_bus(net, vn_kv=220, name = "SP HVDC HV", geodata=santaponsa_loc)
```


```python
#Load Buses (MW) This was taken on demographic population data across the region

#Menorca Load Buses (53 MW)
ciudad_mw=13
calanbosch_mw = 13
mahon_mw= 27

#Mallorca Load Buses (506 MW)

esbesons_mw=40
sonreus_mw=100
llubi_mw=28
valldurgent_mw=98
sonorlandis_mw=107
santaponsa_mw=11
castresorer_mw=59
alcudia_mw=40
calamesquida_mw=23

#ibiza and formentera combined load minus generation at 1320. 
ibizaformentera_mw=68.1

#Create Menorca Loads

L_calanbosch = pp.create_load(net, bus=b_calanbosch, p_mw=calanbosch_mw, q_mvar=calanbosch_mw*PF, name = "Load Cal an Bosch")
L_ciudadela = pp.create_load(net, bus=b_ciudadela, p_mw=ciudad_mw, q_mvar=ciudad_mw*PF, name = "Load Ciudadella")
L_mahon = pp.create_load(net, bus=b_mahon, p_mw=mahon_mw, q_mvar=mahon_mw*PF, name = "Load Mahon")

#Create Mallorca Loads
L_esbe = pp.create_load(net, bus=b_esbe_lv, p_mw=esbesons_mw, q_mvar=esbesons_mw*PF, name = "Load Es Bessons")
L_llubi = pp.create_load(net, bus=b_llubi, p_mw=llubi_mw, q_mvar=llubi_mw*PF, name = "Load LLubi")
L_sonreus = pp.create_load(net, bus=b_sonreus, p_mw=sonreus_mw, q_mvar=sonreus_mw*PF, name = "Load Son Reus")
L_valldurgent = pp.create_load(net, bus=b_valldurgent, p_mw=valldurgent_mw, q_mvar=valldurgent_mw*PF, name = "Load Valldurgent")
L_sonorlandis = pp.create_load(net, bus=b_sonorlandis, p_mw=sonorlandis_mw, q_mvar=sonorlandis_mw*PF, name = "Load Son Orlandis")
L_alcudia = pp.create_load(net, bus=b_alcudia, p_mw=alcudia_mw, q_mvar=alcudia_mw*PF, name = "Load Alcudia") 
L_calamesquida = pp.create_load(net, bus=b_calamesq, p_mw=calamesquida_mw, q_mvar=calamesquida_mw*PF, name = "Load Cala Mesquida")
L_santaponsa = pp.create_load(net, bus=b_sp_lv, p_mw=santaponsa_mw, q_mvar=santaponsa_mw*PF, name = "Load Santa Ponsa")
l_castresorer =pp.create_load(net, bus=b_castres, p_mw=castresorer_mw, q_mvar=castresorer_mw*PF, name = "Load Cas Tresorer")

#Create Ibiza Loads

L_ibizaformentera = pp.create_load(net, bus=b_ibizaformentera, p_mw=ibizaformentera_mw, q_mvar=ibizaformentera_mw*PF, name = "Load Ibiza/Formentera")

L_sp = pp.create_load(net, bus=b_sp_lv, p_mw=0, q_mvar=237, name = "Reactor")


net.load.tail()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>bus</th>
      <th>p_mw</th>
      <th>q_mvar</th>
      <th>const_z_percent</th>
      <th>const_i_percent</th>
      <th>sn_mva</th>
      <th>scaling</th>
      <th>in_service</th>
      <th>type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>9</th>
      <td>Load Cala Mesquida</td>
      <td>10</td>
      <td>23.0</td>
      <td>11.139406</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>True</td>
      <td>wye</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Load Santa Ponsa</td>
      <td>15</td>
      <td>11.0</td>
      <td>5.327542</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>True</td>
      <td>wye</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Load Cas Tresorer</td>
      <td>13</td>
      <td>59.0</td>
      <td>28.574998</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>True</td>
      <td>wye</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Load Ibiza/Formentera</td>
      <td>14</td>
      <td>68.1</td>
      <td>32.982328</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>True</td>
      <td>wye</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Reactor</td>
      <td>15</td>
      <td>0.0</td>
      <td>237.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>True</td>
      <td>wye</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Menorca Generation (55.9 MW)

P_Menorca = 55.9

#Mallorca Generation (391.2 MW)

P_cc = 316
P_waste = 33
P_solar = 39
P_cogen = 3.2

#Generation Buses

gen_mahon = pp.create_sgen(net, bus = b_mahon, p_mw = P_Menorca, q_mvar = P_Menorca*PF, name = 'Mahon Generation')
gen_sonreus = pp.create_sgen(net, bus = b_sonreus, p_mw = P_cc+P_waste+P_cogen, q_mvar=(P_cc+P_waste+P_cogen)*PF , name = "Son Reus Generation")
gen_llubi = pp.create_sgen(net, bus = b_llubi, p_mw = P_solar, q_mvar=0, name = "LLubi Generation")

#Swing Bus

swing_hvdc = pp.create_ext_grid(net, bus=b_sp_hv, vm_pu=1.00, name="Cometa Swing Bus")

net.sgen.tail()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>bus</th>
      <th>p_mw</th>
      <th>q_mvar</th>
      <th>sn_mva</th>
      <th>scaling</th>
      <th>in_service</th>
      <th>type</th>
      <th>current_source</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Mahon Generation</td>
      <td>2</td>
      <td>55.9</td>
      <td>27.073600</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>True</td>
      <td>wye</td>
      <td>True</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Son Reus Generation</td>
      <td>7</td>
      <td>352.2</td>
      <td>170.578208</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>True</td>
      <td>wye</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2</th>
      <td>LLubi Generation</td>
      <td>6</td>
      <td>39.0</td>
      <td>0.000000</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>True</td>
      <td>wye</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
</div>




```python
#create 220 to 132kV transformers at es bessons and santa ponsa, typical capacity zotero
trans_sp = pp.create_transformer_from_parameters(net, hv_bus=b_esbe_hv, lv_bus=b_sp_lv, sn_mva=400, vn_hv_kv=220,\
                                                  vn_lv_kv=132, vkr_percent=1, vk_percent=10, pfe_kw=400.*0.01, i0_percent=1)

trans_eb = pp.create_transformer_from_parameters(net, hv_bus=b_esbe_hv, lv_bus=b_esbe_lv, sn_mva=400, vn_hv_kv=220,\
                                                  vn_lv_kv=132, vkr_percent=1, vk_percent=10, pfe_kw=400*0.01, i0_percent=1)
```


```python
#Create Lines
#create_line(net, “line1”, from_bus = 0, to_bus = 1, length_km=0.1, std_type=”NAYY 4x50 SE”)

#220 overhead typical properties....gull aw
Rkm_220OH = 0.13
Xkm_220OH = 0.425
Cnfkm_220OH = 8.2
MaxIkA220 = 0.8

#132 overhead properties....hawk aw
Rkm_132OH = 0.14
Xkm_132OH = 0.42
Cnfkm_132OH = 8.2
MaxIkA132 = 0.66

#132 Underwater/ground properties IBIZA MALLORCA
Rkm_132UW = 0.14
Xkm_132UW = 0.10
Cnfkm_132UW = 200
MaxIkA132UW = 1.6

#132 Underwater/ground properties Mallorca Menorca
Rkm_132UWmen = 0.14
Xkm_132UWmen = 0.10
Cnfkm_132UWmen = 200
MaxIkA132men = 1.6


#pandapower.create_line_from_parameters(net, from_bus, to_bus, length_km, r_ohm_per_km, x_ohm_per_km, c_nf_per_km, max_i_ka, name=None, index=None, type=None, geodata=None, in_service=True, df=1.0, parallel=1, g_us_per_km=0.0, max_loading_percent=nan, alpha=None, temperature_degree_celsius=None, r0_ohm_per_km=nan, x0_ohm_per_km=nan, c0_nf_per_km=nan, g0_us_per_km=0, endtemp_degree=None, **kwargs)
#this needs to be updated to reflect underwater 132kV properties #CHANGE STD TYPE p = 2*100MW
#

line_ibiza_mallorca1 = pp.create_line_from_parameters(net, from_bus = b_sp_lv, to_bus = b_ibizaformentera , length_km= ibiza_mallorca , r_ohm_per_km=Rkm_132UW, x_ohm_per_km=Xkm_132UW, \
                                                c_nf_per_km = Cnfkm_132UW, max_i_ka =MaxIkA132UW, name = 'Ibiza_Mallorca1', geodata = [ibizaformentera_loc, santaponsa_loc])
line_ibiza_mallorca2 = pp.create_line_from_parameters(net, from_bus = b_sp_lv, to_bus = b_ibizaformentera, length_km= ibiza_mallorca ,  r_ohm_per_km=Rkm_132UW, x_ohm_per_km=Xkm_132UW, \
                                                c_nf_per_km = Cnfkm_132UW, max_i_ka =MaxIkA132UW , name = 'Ibiza_Mallorca2', geodata = [ibizaformentera_loc, santaponsa_loc])


#Mallorca (220 kV)
line_santaponsa_val1durgent1 = pp.create_line_from_parameters(net,  from_bus = b_sp_hv, to_bus = b_valldurgent , length_km= santaponsa_valldurgent , r_ohm_per_km=Rkm_220OH, x_ohm_per_km=Xkm_220OH, \
                                              c_nf_per_km = Cnfkm_220OH, max_i_ka =MaxIkA220, name = 'santaponsa_valldurgent1', geodata = [santaponsa_loc, valldurgent_loc])
line_santaponsa_valldurgent2 = pp.create_line_from_parameters(net,  from_bus = b_sp_hv, to_bus = b_valldurgent , length_km= santaponsa_valldurgent , r_ohm_per_km=Rkm_220OH, x_ohm_per_km=Xkm_220OH, \
                                              c_nf_per_km = Cnfkm_220OH, max_i_ka =MaxIkA220, name = 'santaponsa_valldurgent2', geodata = [santaponsa_loc, valldurgent_loc])

#two lines from santa ponsa to Valldurgent (overpass.eu)
line_valldurgent_sonreus1 = pp.create_line_from_parameters(net,  from_bus = b_valldurgent, to_bus = b_sonreus , length_km= valldurgent_sonreus1 , r_ohm_per_km=Rkm_220OH, x_ohm_per_km=Xkm_220OH,\
                                              c_nf_per_km = Cnfkm_220OH, max_i_ka =MaxIkA220, name ='valldurgent_sonreus1', geodata = [valldurgent_loc, sonreus_loc])
line_valldurgent_sonreus2 = pp.create_line_from_parameters(net,  from_bus = b_valldurgent, to_bus = b_sonreus , length_km= valldurgent_sonreus2 , r_ohm_per_km=Rkm_220OH, x_ohm_per_km=Xkm_220OH,\
                                              c_nf_per_km = Cnfkm_220OH, max_i_ka =MaxIkA220, name = 'valldurgent_sonreus2', geodata = [valldurgent_loc, sonreus_loc])

line_sonreus_sonorlandis1 = pp.create_line_from_parameters(net,  from_bus = b_sonreus, to_bus = b_sonorlandis , length_km= sonreus_sonor , r_ohm_per_km=Rkm_220OH, x_ohm_per_km=Xkm_220OH, \
                                            c_nf_per_km = Cnfkm_220OH, max_i_ka =MaxIkA220, name = 'sonrreus_sonorlandis1', geodata = [sonreus_loc, sonorlandis_loc])
line_castres_sonorlandis2 = pp.create_line_from_parameters(net, from_bus = b_castres, to_bus = b_sonorlandis, length_km= castres_sonor, r_ohm_per_km=Rkm_220OH, x_ohm_per_km=Xkm_220OH, \
                                              c_nf_per_km = Cnfkm_220OH, max_i_ka =MaxIkA220, name = 'castres_sonorlandis2', geodata = [castresorer_loc, sonorlandis_loc])
line_castres_sonorlandis3 = pp.create_line_from_parameters(net, from_bus = b_castres, to_bus = b_sonorlandis, length_km= castres_sonor, r_ohm_per_km=Rkm_220OH, x_ohm_per_km=Xkm_220OH, \
                                              c_nf_per_km = Cnfkm_220OH, max_i_ka =MaxIkA220, name ='castres_sonorlandis3', geodata = [castresorer_loc, sonorlandis_loc])

line_sonrreus_llubi = pp.create_line_from_parameters(net,  from_bus = b_sonreus, to_bus = b_llubi , length_km= sonreus_llubi , r_ohm_per_km=Rkm_220OH, x_ohm_per_km=Xkm_220OH, \
                                            c_nf_per_km = Cnfkm_220OH, max_i_ka =MaxIkA220, name = 'sonreus_llubi', geodata = [sonreus_loc, llubi_loc])
line_sonorlandi_llubi = pp.create_line_from_parameters(net,  from_bus = b_sonorlandis, to_bus = b_llubi , length_km= sonor_llubi , r_ohm_per_km=Rkm_220OH, x_ohm_per_km=Xkm_220OH, \
                                            c_nf_per_km = Cnfkm_220OH, max_i_ka =MaxIkA220, name = 'sonorlandis_llubi', geodata = [b_sonorlandis, llubi_loc])


line_llubi_alcudia1 = pp.create_line_from_parameters(net, from_bus = b_llubi, to_bus = b_alcudia, length_km= llubi_MTR1, r_ohm_per_km=Rkm_220OH, x_ohm_per_km=Xkm_220OH, \
                                             c_nf_per_km = Cnfkm_220OH, max_i_ka =MaxIkA220, name ='llubi_alcudia1', geodata = [llubi_loc, alcudia_loc])
line_llubi_alcudia2 = pp.create_line_from_parameters(net, from_bus = b_llubi, to_bus = b_alcudia, length_km= llubi_MTR2, r_ohm_per_km=Rkm_220OH, x_ohm_per_km=Xkm_220OH, \
                                             c_nf_per_km = Cnfkm_220OH, max_i_ka =MaxIkA220, name = 'llubi_alcudia2', geodata = [llubi_loc, alcudia_loc])

line_llubi_esbessons1 = pp.create_line_from_parameters(net, from_bus = b_llubi, to_bus = b_esbe_hv, length_km= llubi_esbe1, r_ohm_per_km=Rkm_220OH, x_ohm_per_km=Xkm_220OH, \
                                             c_nf_per_km = Cnfkm_220OH, max_i_ka =MaxIkA220, name ='llubi_esbes1', geodata = [llubi_loc, esbesons_loc])
line_llubi_esbessons2 = pp.create_line_from_parameters(net, from_bus = b_llubi, to_bus = b_esbe_hv, length_km= llubi_esbe2, r_ohm_per_km=Rkm_220OH, x_ohm_per_km=Xkm_220OH, \
                                             c_nf_per_km = Cnfkm_220OH, max_i_ka =MaxIkA220, name ='llubi_esbes2', geodata = [llubi_loc, esbesons_loc])


#132 lines on mallorca
line_esbessons_calamesquides = pp.create_line_from_parameters(net, from_bus = b_esbe_lv, to_bus = b_calamesq, length_km= esbe_calamesquida,  r_ohm_per_km=Rkm_132OH, x_ohm_per_km=Xkm_132OH, \
                                              c_nf_per_km = Cnfkm_132OH, max_i_ka =MaxIkA132, name = 'esbessons_gesa', geodata = [esbesons_loc, calamesquida_loc]) #change std type


#this needs to be updated to reflect underwater 132kV properties
line_calamesquides_calanbos = pp.create_line_from_parameters(net, from_bus = b_calamesq, to_bus = b_calanbosch, length_km= calamesquida_calanbosch,  r_ohm_per_km=Rkm_132UWmen, x_ohm_per_km=Xkm_132UWmen, \
                                              c_nf_per_km = Cnfkm_132UWmen, max_i_ka =MaxIkA132men, name ='gesa_cb', geodata = [calamesquida_loc, calanbosch_loc]) #change std type

#standard 132 overhead lines
line_calanbos_ciudadella = pp.create_line_from_parameters(net, from_bus = b_calanbosch, to_bus = b_ciudadela, length_km= calanbosch_ciud,  r_ohm_per_km=Rkm_132OH, x_ohm_per_km=Xkm_132OH, \
                                              c_nf_per_km = Cnfkm_132OH, max_i_ka =MaxIkA132, name ='calanbos_cuidadela', geodata = [calanbosch_loc, ciudadela_loc])
line_ciudadella_dragonera = pp.create_line_from_parameters(net, from_bus = b_ciudadela, to_bus = b_dragon, length_km= ciud_drag,  r_ohm_per_km=Rkm_132OH, x_ohm_per_km=Xkm_132OH,\
                                                c_nf_per_km = Cnfkm_132OH, max_i_ka =MaxIkA132, name ='ciut_drag', geodata = [ciudadela_loc, dragonera_loc])
line_ciudadella_mercadal = pp.create_line_from_parameters(net, from_bus = b_ciudadela, to_bus = b_mercadal, length_km= ciud_merc,  r_ohm_per_km=Rkm_132OH, x_ohm_per_km=Xkm_132OH, \
                                                c_nf_per_km = Cnfkm_132OH, max_i_ka =MaxIkA132, name ='ciut_mercadal',  geodata = [ciudadela_loc, mercadal_loc])
line_mercadal_dragonera =  pp.create_line_from_parameters(net, from_bus = b_mercadal, to_bus = b_dragon, length_km= merc_drag,   r_ohm_per_km=Rkm_132OH, x_ohm_per_km=Xkm_132OH, \
                                                 c_nf_per_km = Cnfkm_132OH, max_i_ka =MaxIkA132, name ='mercadal_drag', geodata = [mercadal_loc, dragonera_loc] )
line_dragonera_mahon1 = pp.create_line_from_parameters(net, from_bus = b_dragon, to_bus = b_mahon, length_km= drag_mahon,  r_ohm_per_km=Rkm_132OH, x_ohm_per_km=Xkm_132OH,\
                                                  c_nf_per_km = Cnfkm_132OH, max_i_ka =MaxIkA132, name ='drag_mahon1', geodata = [dragonera_loc, mahon_loc])
line_dragonera_mahon2 = pp.create_line_from_parameters(net, from_bus = b_dragon, to_bus = b_mahon, length_km= drag_mahon,  r_ohm_per_km=Rkm_132OH, x_ohm_per_km=Xkm_132OH, \
                                                  c_nf_per_km = Cnfkm_132OH, max_i_ka =MaxIkA132, name ='drag_mahon2' , geodata = [dragonera_loc, mahon_loc])

net.line.head()

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>std_type</th>
      <th>from_bus</th>
      <th>to_bus</th>
      <th>length_km</th>
      <th>r_ohm_per_km</th>
      <th>x_ohm_per_km</th>
      <th>c_nf_per_km</th>
      <th>g_us_per_km</th>
      <th>max_i_ka</th>
      <th>df</th>
      <th>parallel</th>
      <th>type</th>
      <th>in_service</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Ibiza_Mallorca1</td>
      <td>None</td>
      <td>15</td>
      <td>14</td>
      <td>126.0</td>
      <td>0.14</td>
      <td>0.100</td>
      <td>200.0</td>
      <td>0.0</td>
      <td>1.6</td>
      <td>1.0</td>
      <td>1</td>
      <td>None</td>
      <td>True</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Ibiza_Mallorca2</td>
      <td>None</td>
      <td>15</td>
      <td>14</td>
      <td>126.0</td>
      <td>0.14</td>
      <td>0.100</td>
      <td>200.0</td>
      <td>0.0</td>
      <td>1.6</td>
      <td>1.0</td>
      <td>1</td>
      <td>None</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2</th>
      <td>santaponsa_valldurgent1</td>
      <td>None</td>
      <td>16</td>
      <td>5</td>
      <td>6.5</td>
      <td>0.13</td>
      <td>0.425</td>
      <td>8.2</td>
      <td>0.0</td>
      <td>0.8</td>
      <td>1.0</td>
      <td>1</td>
      <td>None</td>
      <td>True</td>
    </tr>
    <tr>
      <th>3</th>
      <td>santaponsa_valldurgent2</td>
      <td>None</td>
      <td>16</td>
      <td>5</td>
      <td>6.5</td>
      <td>0.13</td>
      <td>0.425</td>
      <td>8.2</td>
      <td>0.0</td>
      <td>0.8</td>
      <td>1.0</td>
      <td>1</td>
      <td>None</td>
      <td>True</td>
    </tr>
    <tr>
      <th>4</th>
      <td>valldurgent_sonreus1</td>
      <td>None</td>
      <td>5</td>
      <td>7</td>
      <td>15.1</td>
      <td>0.13</td>
      <td>0.425</td>
      <td>8.2</td>
      <td>0.0</td>
      <td>0.8</td>
      <td>1.0</td>
      <td>1</td>
      <td>None</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
</div>




```python
pp.runpp(net)



net.res_line




```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>p_from_mw</th>
      <th>q_from_mvar</th>
      <th>p_to_mw</th>
      <th>q_to_mvar</th>
      <th>pl_mw</th>
      <th>ql_mvar</th>
      <th>i_from_ka</th>
      <th>i_to_ka</th>
      <th>i_ka</th>
      <th>vm_from_pu</th>
      <th>va_from_degree</th>
      <th>vm_to_pu</th>
      <th>va_to_degree</th>
      <th>loading_percent</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>37.725192</td>
      <td>-107.076541</td>
      <td>-34.050000</td>
      <td>-16.491164</td>
      <td>3.675192</td>
      <td>-123.567706</td>
      <td>0.518092</td>
      <td>0.173368</td>
      <td>0.518092</td>
      <td>0.958431</td>
      <td>-4.390175</td>
      <td>0.954492</td>
      <td>-8.875563</td>
      <td>32.380778</td>
    </tr>
    <tr>
      <th>1</th>
      <td>37.725192</td>
      <td>-107.076541</td>
      <td>-34.050000</td>
      <td>-16.491164</td>
      <td>3.675192</td>
      <td>-123.567706</td>
      <td>0.518092</td>
      <td>0.173368</td>
      <td>0.518092</td>
      <td>0.958431</td>
      <td>-4.390175</td>
      <td>0.954492</td>
      <td>-8.875563</td>
      <td>32.380778</td>
    </tr>
    <tr>
      <th>2</th>
      <td>96.506695</td>
      <td>25.069896</td>
      <td>-96.332762</td>
      <td>-25.309184</td>
      <td>0.173932</td>
      <td>-0.239288</td>
      <td>0.261670</td>
      <td>0.262207</td>
      <td>0.262207</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.996874</td>
      <td>-0.291028</td>
      <td>32.775888</td>
    </tr>
    <tr>
      <th>3</th>
      <td>96.506695</td>
      <td>25.069896</td>
      <td>-96.332762</td>
      <td>-25.309184</td>
      <td>0.173932</td>
      <td>-0.239288</td>
      <td>0.261670</td>
      <td>0.262207</td>
      <td>0.262207</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.996874</td>
      <td>-0.291028</td>
      <td>32.775888</td>
    </tr>
    <tr>
      <th>4</th>
      <td>47.332762</td>
      <td>1.577406</td>
      <td>-47.241069</td>
      <td>-3.144406</td>
      <td>0.091694</td>
      <td>-1.566999</td>
      <td>0.124675</td>
      <td>0.124920</td>
      <td>0.124920</td>
      <td>0.996874</td>
      <td>-0.291028</td>
      <td>0.994633</td>
      <td>-0.647803</td>
      <td>15.615049</td>
    </tr>
    <tr>
      <th>5</th>
      <td>47.332762</td>
      <td>1.577406</td>
      <td>-47.241069</td>
      <td>-3.144406</td>
      <td>0.091694</td>
      <td>-1.566999</td>
      <td>0.124675</td>
      <td>0.124920</td>
      <td>0.124920</td>
      <td>0.996874</td>
      <td>-0.291028</td>
      <td>0.994633</td>
      <td>-0.647803</td>
      <td>15.615049</td>
    </tr>
    <tr>
      <th>6</th>
      <td>230.955996</td>
      <td>93.925565</td>
      <td>-229.383683</td>
      <td>-89.917082</td>
      <td>1.572313</td>
      <td>4.008483</td>
      <td>0.657837</td>
      <td>0.658952</td>
      <td>0.658952</td>
      <td>0.994633</td>
      <td>-0.647803</td>
      <td>0.981215</td>
      <td>-1.616532</td>
      <td>82.369055</td>
    </tr>
    <tr>
      <th>7</th>
      <td>-29.500000</td>
      <td>-14.287499</td>
      <td>29.512230</td>
      <td>13.835731</td>
      <td>0.012230</td>
      <td>-0.451768</td>
      <td>0.087742</td>
      <td>0.087176</td>
      <td>0.087742</td>
      <td>0.980367</td>
      <td>-1.670581</td>
      <td>0.981215</td>
      <td>-1.616532</td>
      <td>10.967749</td>
    </tr>
    <tr>
      <th>8</th>
      <td>-29.500000</td>
      <td>-14.287499</td>
      <td>29.512230</td>
      <td>13.835731</td>
      <td>0.012230</td>
      <td>-0.451768</td>
      <td>0.087742</td>
      <td>0.087176</td>
      <td>0.087742</td>
      <td>0.980367</td>
      <td>-1.670581</td>
      <td>0.981215</td>
      <td>-1.616532</td>
      <td>10.967749</td>
    </tr>
    <tr>
      <th>9</th>
      <td>115.726141</td>
      <td>34.509255</td>
      <td>-114.341918</td>
      <td>-34.160598</td>
      <td>1.384223</td>
      <td>0.348657</td>
      <td>0.318628</td>
      <td>0.321816</td>
      <td>0.321816</td>
      <td>0.994633</td>
      <td>-0.647803</td>
      <td>0.973151</td>
      <td>-2.527840</td>
      <td>40.226939</td>
    </tr>
    <tr>
      <th>10</th>
      <td>63.359222</td>
      <td>10.423165</td>
      <td>-63.022473</td>
      <td>-12.775012</td>
      <td>0.336749</td>
      <td>-2.351847</td>
      <td>0.171736</td>
      <td>0.173411</td>
      <td>0.173411</td>
      <td>0.981215</td>
      <td>-1.616532</td>
      <td>0.973151</td>
      <td>-2.527840</td>
      <td>21.676338</td>
    </tr>
    <tr>
      <th>11</th>
      <td>19.415618</td>
      <td>7.519789</td>
      <td>-19.393939</td>
      <td>-9.451682</td>
      <td>0.021678</td>
      <td>-1.931894</td>
      <td>0.056148</td>
      <td>0.058313</td>
      <td>0.058313</td>
      <td>0.973151</td>
      <td>-2.527840</td>
      <td>0.970936</td>
      <td>-2.679991</td>
      <td>7.289150</td>
    </tr>
    <tr>
      <th>12</th>
      <td>20.629094</td>
      <td>8.111544</td>
      <td>-20.606061</td>
      <td>-9.921198</td>
      <td>0.023033</td>
      <td>-1.809654</td>
      <td>0.059777</td>
      <td>0.061815</td>
      <td>0.061815</td>
      <td>0.973151</td>
      <td>-2.527840</td>
      <td>0.970936</td>
      <td>-2.679991</td>
      <td>7.726868</td>
    </tr>
    <tr>
      <th>13</th>
      <td>74.159839</td>
      <td>8.871631</td>
      <td>-73.917040</td>
      <td>-9.876253</td>
      <td>0.242799</td>
      <td>-1.004622</td>
      <td>0.201415</td>
      <td>0.202025</td>
      <td>0.202025</td>
      <td>0.973151</td>
      <td>-2.527840</td>
      <td>0.968720</td>
      <td>-3.108983</td>
      <td>25.253147</td>
    </tr>
    <tr>
      <th>14</th>
      <td>74.159839</td>
      <td>8.871631</td>
      <td>-73.917040</td>
      <td>-9.876253</td>
      <td>0.242799</td>
      <td>-1.004622</td>
      <td>0.201415</td>
      <td>0.202025</td>
      <td>0.202025</td>
      <td>0.973151</td>
      <td>-2.527840</td>
      <td>0.968720</td>
      <td>-3.108983</td>
      <td>25.253147</td>
    </tr>
    <tr>
      <th>15</th>
      <td>21.041270</td>
      <td>-38.601321</td>
      <td>-20.552209</td>
      <td>38.744555</td>
      <td>0.489061</td>
      <td>0.143234</td>
      <td>0.197941</td>
      <td>0.192755</td>
      <td>0.197941</td>
      <td>0.971456</td>
      <td>-4.060140</td>
      <td>0.995199</td>
      <td>-5.528317</td>
      <td>29.991036</td>
    </tr>
    <tr>
      <th>16</th>
      <td>-2.447791</td>
      <td>-49.883961</td>
      <td>2.703056</td>
      <td>4.950609</td>
      <td>0.255265</td>
      <td>-44.933351</td>
      <td>0.219502</td>
      <td>0.024607</td>
      <td>0.219502</td>
      <td>0.995199</td>
      <td>-5.528317</td>
      <td>1.002600</td>
      <td>-6.018898</td>
      <td>13.718878</td>
    </tr>
    <tr>
      <th>17</th>
      <td>-15.703056</td>
      <td>-11.246795</td>
      <td>15.727822</td>
      <td>10.940840</td>
      <td>0.024766</td>
      <td>-0.305956</td>
      <td>0.084263</td>
      <td>0.083308</td>
      <td>0.084263</td>
      <td>1.002600</td>
      <td>-6.018898</td>
      <td>1.005893</td>
      <td>-5.880659</td>
      <td>12.767109</td>
    </tr>
    <tr>
      <th>18</th>
      <td>-14.744575</td>
      <td>-8.803239</td>
      <td>14.822398</td>
      <td>7.439096</td>
      <td>0.077822</td>
      <td>-1.364143</td>
      <td>0.074671</td>
      <td>0.071345</td>
      <td>0.074671</td>
      <td>1.005893</td>
      <td>-5.880659</td>
      <td>1.016724</td>
      <td>-5.313256</td>
      <td>11.313750</td>
    </tr>
    <tr>
      <th>19</th>
      <td>-13.983247</td>
      <td>-8.433786</td>
      <td>14.026811</td>
      <td>7.595476</td>
      <td>0.043564</td>
      <td>-0.838311</td>
      <td>0.071006</td>
      <td>0.068920</td>
      <td>0.071006</td>
      <td>1.005893</td>
      <td>-5.880659</td>
      <td>1.012318</td>
      <td>-5.554809</td>
      <td>10.758419</td>
    </tr>
    <tr>
      <th>20</th>
      <td>-14.026811</td>
      <td>-7.595476</td>
      <td>14.057090</td>
      <td>6.970224</td>
      <td>0.030279</td>
      <td>-0.625251</td>
      <td>0.068920</td>
      <td>0.067498</td>
      <td>0.068920</td>
      <td>1.012318</td>
      <td>-5.554809</td>
      <td>1.016724</td>
      <td>-5.313256</td>
      <td>10.442384</td>
    </tr>
    <tr>
      <th>21</th>
      <td>-14.439744</td>
      <td>-7.204660</td>
      <td>14.450000</td>
      <td>6.998453</td>
      <td>0.010256</td>
      <td>-0.206207</td>
      <td>0.069421</td>
      <td>0.068972</td>
      <td>0.069421</td>
      <td>1.016724</td>
      <td>-5.313256</td>
      <td>1.018164</td>
      <td>-5.231078</td>
      <td>10.518407</td>
    </tr>
    <tr>
      <th>22</th>
      <td>-14.439744</td>
      <td>-7.204660</td>
      <td>14.450000</td>
      <td>6.998453</td>
      <td>0.010256</td>
      <td>-0.206207</td>
      <td>0.069421</td>
      <td>0.068972</td>
      <td>0.069421</td>
      <td>1.016724</td>
      <td>-5.313256</td>
      <td>1.018164</td>
      <td>-5.231078</td>
      <td>10.518407</td>
    </tr>
  </tbody>
</table>
</div>




```python
net.res_bus
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>vm_pu</th>
      <th>va_degree</th>
      <th>p_mw</th>
      <th>q_mvar</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1.002600</td>
      <td>-6.018898</td>
      <td>13.00000</td>
      <td>6.296186</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1.005893</td>
      <td>-5.880659</td>
      <td>13.00000</td>
      <td>6.296186</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1.018164</td>
      <td>-5.231078</td>
      <td>-28.90000</td>
      <td>-13.996906</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1.016724</td>
      <td>-5.313256</td>
      <td>0.00000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1.012318</td>
      <td>-5.554809</td>
      <td>0.00000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0.996874</td>
      <td>-0.291028</td>
      <td>98.00000</td>
      <td>47.463556</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0.973151</td>
      <td>-2.527840</td>
      <td>-11.00000</td>
      <td>13.561016</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0.994633</td>
      <td>-0.647803</td>
      <td>-252.20000</td>
      <td>-122.146008</td>
    </tr>
    <tr>
      <th>8</th>
      <td>0.981215</td>
      <td>-1.616532</td>
      <td>107.00000</td>
      <td>51.822454</td>
    </tr>
    <tr>
      <th>9</th>
      <td>0.970936</td>
      <td>-2.679991</td>
      <td>40.00000</td>
      <td>19.372880</td>
    </tr>
    <tr>
      <th>10</th>
      <td>0.995199</td>
      <td>-5.528317</td>
      <td>23.00000</td>
      <td>11.139406</td>
    </tr>
    <tr>
      <th>11</th>
      <td>0.971456</td>
      <td>-4.060140</td>
      <td>40.00000</td>
      <td>19.372880</td>
    </tr>
    <tr>
      <th>12</th>
      <td>0.968720</td>
      <td>-3.108983</td>
      <td>0.00000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>13</th>
      <td>0.980367</td>
      <td>-1.670581</td>
      <td>59.00000</td>
      <td>28.574998</td>
    </tr>
    <tr>
      <th>14</th>
      <td>0.954492</td>
      <td>-8.875563</td>
      <td>68.10000</td>
      <td>32.982328</td>
    </tr>
    <tr>
      <th>15</th>
      <td>0.958431</td>
      <td>-4.390175</td>
      <td>11.00000</td>
      <td>242.327542</td>
    </tr>
    <tr>
      <th>16</th>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>-193.01339</td>
      <td>-50.139793</td>
    </tr>
  </tbody>
</table>
</div>




```python
pp.
```
