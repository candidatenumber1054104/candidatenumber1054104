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

#Menorca Load Buses (50 MW)
ciudad_mw=12.26
calanbosch_mw = 12.26
mahon_mw= 25.47

#Mallorca Load Buses (490 MW)

esbesons_mw=38.74
sonreus_mw=96.84
llubi_mw=27.11
valldurgent_mw=94.90
sonorlandis_mw=103.62
santaponsa_mw=10.65
castresorer_mw=57.13
alcudia_mw=38.74
calamesquida_mw=22.27

#ibiza and formentera combined load at 21:40
ibizaformentera_mw=110.9

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
      <td>22.27</td>
      <td>10.785851</td>
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
      <td>10.65</td>
      <td>5.158029</td>
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
      <td>57.13</td>
      <td>27.669316</td>
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
      <td>110.90</td>
      <td>53.711310</td>
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
      <td>0.00</td>
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
#Menorca Generation (50.70 MW; 35 MW Diesel and 15.70 MW CCGT)

P_Menorca = 50.70

#Mallorca Generation (399.5 MW)

P_cc = 362
P_waste = 34
P_solar = 0.4
P_cogen = 3.1

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
      <td>50.7</td>
      <td>24.555125</td>
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
      <td>399.1</td>
      <td>193.292910</td>
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
      <td>0.4</td>
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
      <td>60.429761</td>
      <td>-79.211780</td>
      <td>-55.450000</td>
      <td>-26.855655</td>
      <td>4.979761</td>
      <td>-106.067435</td>
      <td>0.476452</td>
      <td>0.310568</td>
      <td>0.476452</td>
      <td>0.914617</td>
      <td>-6.023668</td>
      <td>0.867697</td>
      <td>-10.756585</td>
      <td>29.778251</td>
    </tr>
    <tr>
      <th>1</th>
      <td>60.429761</td>
      <td>-79.211780</td>
      <td>-55.450000</td>
      <td>-26.855655</td>
      <td>4.979761</td>
      <td>-106.067435</td>
      <td>0.476452</td>
      <td>0.310568</td>
      <td>0.476452</td>
      <td>0.914617</td>
      <td>-6.023668</td>
      <td>0.867697</td>
      <td>-10.756585</td>
      <td>29.778251</td>
    </tr>
    <tr>
      <th>2</th>
      <td>110.293463</td>
      <td>48.565368</td>
      <td>-110.039217</td>
      <td>-48.540816</td>
      <td>0.254247</td>
      <td>0.024553</td>
      <td>0.316263</td>
      <td>0.317119</td>
      <td>0.317119</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.995294</td>
      <td>-0.313176</td>
      <td>39.639853</td>
    </tr>
    <tr>
      <th>3</th>
      <td>110.293463</td>
      <td>48.565368</td>
      <td>-110.039217</td>
      <td>-48.540816</td>
      <td>0.254247</td>
      <td>0.024553</td>
      <td>0.316263</td>
      <td>0.317119</td>
      <td>0.317119</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.995294</td>
      <td>-0.313176</td>
      <td>39.639853</td>
    </tr>
    <tr>
      <th>4</th>
      <td>62.589217</td>
      <td>25.559737</td>
      <td>-62.400094</td>
      <td>-26.795187</td>
      <td>0.189123</td>
      <td>-1.235450</td>
      <td>0.178261</td>
      <td>0.180156</td>
      <td>0.180156</td>
      <td>0.995294</td>
      <td>-0.313176</td>
      <td>0.989241</td>
      <td>-0.733588</td>
      <td>22.519442</td>
    </tr>
    <tr>
      <th>5</th>
      <td>62.589217</td>
      <td>25.559737</td>
      <td>-62.400094</td>
      <td>-26.795187</td>
      <td>0.189123</td>
      <td>-1.235450</td>
      <td>0.178261</td>
      <td>0.180156</td>
      <td>0.180156</td>
      <td>0.995294</td>
      <td>-0.313176</td>
      <td>0.989241</td>
      <td>-0.733588</td>
      <td>22.519442</td>
    </tr>
    <tr>
      <th>6</th>
      <td>267.176223</td>
      <td>127.117265</td>
      <td>-264.937969</td>
      <td>-120.915185</td>
      <td>2.238254</td>
      <td>6.202080</td>
      <td>0.784915</td>
      <td>0.786176</td>
      <td>0.786176</td>
      <td>0.989241</td>
      <td>-0.733588</td>
      <td>0.972136</td>
      <td>-1.843555</td>
      <td>98.271974</td>
    </tr>
    <tr>
      <th>7</th>
      <td>-28.565000</td>
      <td>-13.834658</td>
      <td>28.576681</td>
      <td>13.390147</td>
      <td>0.011681</td>
      <td>-0.444511</td>
      <td>0.085753</td>
      <td>0.085193</td>
      <td>0.085753</td>
      <td>0.971308</td>
      <td>-1.896874</td>
      <td>0.972136</td>
      <td>-1.843555</td>
      <td>10.719173</td>
    </tr>
    <tr>
      <th>8</th>
      <td>-28.565000</td>
      <td>-13.834658</td>
      <td>28.576681</td>
      <td>13.390147</td>
      <td>0.011681</td>
      <td>-0.444511</td>
      <td>0.085753</td>
      <td>0.085193</td>
      <td>0.085753</td>
      <td>0.971308</td>
      <td>-1.896874</td>
      <td>0.972136</td>
      <td>-1.843555</td>
      <td>10.719173</td>
    </tr>
    <tr>
      <th>9</th>
      <td>159.883965</td>
      <td>72.864276</td>
      <td>-156.922519</td>
      <td>-67.248915</td>
      <td>2.961446</td>
      <td>5.615362</td>
      <td>0.466120</td>
      <td>0.470567</td>
      <td>0.470567</td>
      <td>0.989241</td>
      <td>-0.733588</td>
      <td>0.952123</td>
      <td>-3.265518</td>
      <td>58.820818</td>
    </tr>
    <tr>
      <th>10</th>
      <td>104.164607</td>
      <td>43.949446</td>
      <td>-103.098489</td>
      <td>-43.811568</td>
      <td>1.066118</td>
      <td>0.137878</td>
      <td>0.305201</td>
      <td>0.308762</td>
      <td>0.308762</td>
      <td>0.972136</td>
      <td>-1.843555</td>
      <td>0.952123</td>
      <td>-3.265518</td>
      <td>38.595236</td>
    </tr>
    <tr>
      <th>11</th>
      <td>18.804283</td>
      <td>7.305674</td>
      <td>-18.783030</td>
      <td>-9.153290</td>
      <td>0.021252</td>
      <td>-1.847616</td>
      <td>0.055604</td>
      <td>0.057724</td>
      <td>0.057724</td>
      <td>0.952123</td>
      <td>-3.265518</td>
      <td>0.949929</td>
      <td>-3.419429</td>
      <td>7.215558</td>
    </tr>
    <tr>
      <th>12</th>
      <td>19.979550</td>
      <td>7.878841</td>
      <td>-19.956970</td>
      <td>-9.609344</td>
      <td>0.022581</td>
      <td>-1.730504</td>
      <td>0.059196</td>
      <td>0.061193</td>
      <td>0.061193</td>
      <td>0.952123</td>
      <td>-3.265518</td>
      <td>0.949929</td>
      <td>-3.419429</td>
      <td>7.649063</td>
    </tr>
    <tr>
      <th>13</th>
      <td>97.263588</td>
      <td>41.372999</td>
      <td>-96.753867</td>
      <td>-41.417754</td>
      <td>0.509721</td>
      <td>-0.044754</td>
      <td>0.291332</td>
      <td>0.293193</td>
      <td>0.293193</td>
      <td>0.952123</td>
      <td>-3.265518</td>
      <td>0.942041</td>
      <td>-3.989386</td>
      <td>36.649078</td>
    </tr>
    <tr>
      <th>14</th>
      <td>97.263588</td>
      <td>41.372999</td>
      <td>-96.753867</td>
      <td>-41.417754</td>
      <td>0.509721</td>
      <td>-0.044754</td>
      <td>0.291332</td>
      <td>0.293193</td>
      <td>0.293193</td>
      <td>0.952123</td>
      <td>-3.265518</td>
      <td>0.942041</td>
      <td>-3.989386</td>
      <td>36.649078</td>
    </tr>
    <tr>
      <th>15</th>
      <td>22.404946</td>
      <td>-35.081712</td>
      <td>-21.940224</td>
      <td>35.227657</td>
      <td>0.464723</td>
      <td>0.145945</td>
      <td>0.192844</td>
      <td>0.188008</td>
      <td>0.192844</td>
      <td>0.944107</td>
      <td>-4.992959</td>
      <td>0.965496</td>
      <td>-6.559523</td>
      <td>29.218835</td>
    </tr>
    <tr>
      <th>16</th>
      <td>-0.329776</td>
      <td>-46.013508</td>
      <td>0.551228</td>
      <td>3.748780</td>
      <td>0.221451</td>
      <td>-42.264728</td>
      <td>0.208455</td>
      <td>0.017054</td>
      <td>0.208455</td>
      <td>0.965496</td>
      <td>-6.559523</td>
      <td>0.971769</td>
      <td>-7.060145</td>
      <td>13.028412</td>
    </tr>
    <tr>
      <th>17</th>
      <td>-12.811228</td>
      <td>-9.686567</td>
      <td>12.829420</td>
      <td>9.384035</td>
      <td>0.018192</td>
      <td>-0.302532</td>
      <td>0.072290</td>
      <td>0.071332</td>
      <td>0.072290</td>
      <td>0.971769</td>
      <td>-7.060145</td>
      <td>0.974642</td>
      <td>-6.942045</td>
      <td>10.952978</td>
    </tr>
    <tr>
      <th>18</th>
      <td>-12.877166</td>
      <td>-7.822967</td>
      <td>12.940735</td>
      <td>6.514814</td>
      <td>0.063569</td>
      <td>-1.308153</td>
      <td>0.067616</td>
      <td>0.064369</td>
      <td>0.067616</td>
      <td>0.974642</td>
      <td>-6.942045</td>
      <td>0.984473</td>
      <td>-6.415246</td>
      <td>10.244920</td>
    </tr>
    <tr>
      <th>19</th>
      <td>-12.212254</td>
      <td>-7.498856</td>
      <td>12.247902</td>
      <td>6.696429</td>
      <td>0.035648</td>
      <td>-0.802427</td>
      <td>0.064312</td>
      <td>0.062270</td>
      <td>0.064312</td>
      <td>0.974642</td>
      <td>-6.942045</td>
      <td>0.980485</td>
      <td>-6.639842</td>
      <td>9.744217</td>
    </tr>
    <tr>
      <th>20</th>
      <td>-12.247902</td>
      <td>-6.696429</td>
      <td>12.272579</td>
      <td>6.098890</td>
      <td>0.024677</td>
      <td>-0.597539</td>
      <td>0.062270</td>
      <td>0.060887</td>
      <td>0.062270</td>
      <td>0.980485</td>
      <td>-6.639842</td>
      <td>0.984473</td>
      <td>-6.415246</td>
      <td>9.434838</td>
    </tr>
    <tr>
      <th>21</th>
      <td>-12.606657</td>
      <td>-6.306852</td>
      <td>12.615000</td>
      <td>6.109722</td>
      <td>0.008343</td>
      <td>-0.197130</td>
      <td>0.062628</td>
      <td>0.062192</td>
      <td>0.062628</td>
      <td>0.984473</td>
      <td>-6.415246</td>
      <td>0.985772</td>
      <td>-6.338737</td>
      <td>9.489021</td>
    </tr>
    <tr>
      <th>22</th>
      <td>-12.606657</td>
      <td>-6.306852</td>
      <td>12.615000</td>
      <td>6.109722</td>
      <td>0.008343</td>
      <td>-0.197130</td>
      <td>0.062628</td>
      <td>0.062192</td>
      <td>0.062628</td>
      <td>0.984473</td>
      <td>-6.415246</td>
      <td>0.985772</td>
      <td>-6.338737</td>
      <td>9.489021</td>
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
      <td>0.971769</td>
      <td>-7.060145</td>
      <td>12.260000</td>
      <td>5.937788</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.974642</td>
      <td>-6.942045</td>
      <td>12.260000</td>
      <td>5.937788</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.985772</td>
      <td>-6.338737</td>
      <td>-25.230000</td>
      <td>-12.219444</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.984473</td>
      <td>-6.415246</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.980485</td>
      <td>-6.639842</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0.995294</td>
      <td>-0.313176</td>
      <td>94.900000</td>
      <td>45.962158</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0.952123</td>
      <td>-3.265518</td>
      <td>26.710000</td>
      <td>13.129969</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0.989241</td>
      <td>-0.733588</td>
      <td>-302.260000</td>
      <td>-146.391168</td>
    </tr>
    <tr>
      <th>8</th>
      <td>0.972136</td>
      <td>-1.843555</td>
      <td>103.620000</td>
      <td>50.185446</td>
    </tr>
    <tr>
      <th>9</th>
      <td>0.949929</td>
      <td>-3.419429</td>
      <td>38.740000</td>
      <td>18.762634</td>
    </tr>
    <tr>
      <th>10</th>
      <td>0.965496</td>
      <td>-6.559523</td>
      <td>22.270000</td>
      <td>10.785851</td>
    </tr>
    <tr>
      <th>11</th>
      <td>0.944107</td>
      <td>-4.992959</td>
      <td>38.740000</td>
      <td>18.762634</td>
    </tr>
    <tr>
      <th>12</th>
      <td>0.942041</td>
      <td>-3.989386</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>13</th>
      <td>0.971308</td>
      <td>-1.896874</td>
      <td>57.130000</td>
      <td>27.669316</td>
    </tr>
    <tr>
      <th>14</th>
      <td>0.867697</td>
      <td>-10.756585</td>
      <td>110.900000</td>
      <td>53.711310</td>
    </tr>
    <tr>
      <th>15</th>
      <td>0.914617</td>
      <td>-6.023668</td>
      <td>10.650000</td>
      <td>242.158029</td>
    </tr>
    <tr>
      <th>16</th>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>-220.586927</td>
      <td>-97.130737</td>
    </tr>
  </tbody>
</table>
</div>


