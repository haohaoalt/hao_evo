<!--
 * @Author: zhanghao
 * @Date: 2022-09-06 12:59:44
 * @LastEditTime: 2022-09-06 20:17:04
 * @FilePath: /hao_evo/readme.md
 * @Description: 
-->
# hao_evo

## 01 EVO install
```
sudo apt-get install python3
sudo apt install python3-pip
pip install evo --upgrade --no-binary evo
//reboot
evo_ape -h
```
## 02 EVO 

`evo_config show`

```
evo_config set plot_export_format pdf
evo_config set plot_figsize 6 6
evo_config set plot_fontscale 1.8
evo_config set plot_linewidth 1.8
evo_config set plot_reference_color black
evo_config set plot_reference_linestyle -
evo_config set plot_seaborn_style white








```

```

evo_traj bag 2022-08-31-21-42-33.bag /odom --ref /location_pose -p --plot_mode xy --align

evo_ape bag 2022-08-31-21-42-33.bag /odom  /location_pose -p --plot_mode xy --align -r full

```