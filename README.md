# Introduction

We recently added **an integrated pipeline (credit to [Jiaxin Wang](https://github.com/Net-Maker)): [integrated_qmat_coverage_axis.py](https://github.com/Frank-ZY-Dou/Coverage_Axis/blob/main/integrated_qmat_coverage_axis.py)** You will be using Q-MAT (see more instructions [here](https://github.com/Frank-ZY-Dou/Coverage_Axis/blob/main/skel_connection/README_EN.md)).



Some geometry tools for MAT and related topics: [Geometry_Tools](https://github.com/Frank-ZY-Dou/Geometry_Tools).

🐱 **[Coverage Axis: Inner Point Selection for 3D Shape Skeletonization
](https://arxiv.org/abs/2110.00965), Eurographics 2022**.


Authors: [Zhiyang Dou](https://frank-zy-dou.github.io/), 
[Cheng Lin](https://clinplayer.github.io/), 
[Rui Xu](https://xrvitd.github.io/index.html), 
[Lei Yang](https://www.linkedin.cn/incareer/in/lei-yang-842052119),
[Shiqing Xin](http://irc.cs.sdu.edu.cn/~shiqing/index.html),
[Taku Komura](https://i.cs.hku.hk/~taku/), 
[Wenping Wang](https://engineering.tamu.edu/cse/profiles/Wang-Wenping.html).

[[Project Page](https://frank-zy-dou.github.io/projects/CoverageAxis/index.html)][[Paper](https://arxiv.org/abs/2110.00965)][[Code](https://github.com/Frank-ZY-Dou/Coverage_Axis)]



🐱 **[Coverage Axis++: Efficient Inner Point Selection for 3D Shape Skeletonization
](https://arxiv.org/abs/2401.12946), SGP 2024**.

Authors: Zimeng Wang*, 
[Zhiyang Dou*](https://frank-zy-dou.github.io/),
[Rui Xu](https://xrvitd.github.io/index.html),
[Cheng Lin](https://clinplayer.github.io/), 
[Yuan Liu](https://liuyuan-pal.github.io/), 
[Xiaoxiao Long](https://www.xxlong.site/), 
[Shiqing Xin](http://irc.cs.sdu.edu.cn/~shiqing/index.html), 
[Lingjie Liu](https://lingjie0206.github.io/), 
[Taku Komura](https://www.cs.hku.hk/index.php/people/academic-staff/taku), 
[Xiaoming Yuan](https://hkumath.hku.hk/~xmyuan/),
[Wenping Wang](https://engineering.tamu.edu/cse/profiles/Wang-Wenping.html).

[[Project Page](https://frank-zy-dou.github.io/projects/CoverageAxis++/index.html)][[Paper](https://arxiv.org/pdf/2401.12946.pdf)][[Code](https://github.com/Frank-ZY-Dou/Coverage_Axis)]





# Requirements
## System requirements
- Linux Ubuntu 20.04
- Python 3.8
- Nvidia GeForce RTX 3090 (GPU is used for acceleration)
## Installation

```angular2html
git clone https://github.com/Frank-ZY-Dou/Coverage_Axis --recursive
conda env create -f ca.yml
conda activate CA
pip install -r requirements.txt
```

# Usage


## Mesh Input
The input mesh `01Ants-12.off` is placed in the folder `input`. The mesh is normalized.

Specify the settings for Coverage Axis in ```Coverage_Axis_mesh.py```
```angular2html
real_name = '01Ants-12'
surface_sample_num = 2000
dilation = 0.02
# inner_points = "voronoi"
inner_points = "random"
max_time_SCP = 100 # in second
```
For Coverage Axis, Run
```angular2html
python Coverage_Axis_mesh.py
```
For Coverage Axis++, Run
```angular2html
python Coverage_Axis_plusplus_mesh.py
```

The outputs are placed in the folder `output`.
- `mesh_inner_points.obj` contains the candidate inner points.
- `mesh.obj` contains the input mesh.
- `mesh_samples_2000.obj` contains the sampled surface points that are covered.
- `mesh_selected_inner_points.obj` contains the selected inner points.


You may use randomly generated points inside the volume as inner candidate points by setting `inner_points = "random"
`. Notably, we already generate a sample. If you choose to produce candidates by randomly sampling inside the shape, it can be a little time consuming.

For Coverage Axis, run
```angular2html
python Coverage_Axis_mesh.py
```
For Coverage Axis++, run
```angular2html
python Coverage_Axis_plusplus_pc.py
```


## How to use skeleton connection

This script integrates QMAT and CoverageAxis algorithms, providing a complete medial axis simplification pipeline.

### Features

1. **Step 1**: Use QMAT for regular simplification (optional)
2. **Step 2**: Extract VD information from MA file, run CoverageAxis algorithm to select optimal interior points
3. **Step 3**: Use selected poles to run QMAT for constrained simplification


### File Structure

```
.
├── integrated_qmat_coverage_axis.py  # Main script
├── run_example.py                    # Usage example
├── utils.py                          # Utility functions (copy from original project)
├── input/                            # Input files directory
├── output/                           # CoverageAxis intermediate results directory
├── qmat_temp/                        # QMAT step 1 temporary output directory
└── final_output/                     # Final output directory
```

### Usage

```bash
python integrated_qmat_coverage_axis.py \
    --mesh ./input/bird/bird.off \
    --ma ./input/bird/bird.ma \
    --qmat ./build/QMAT \
    --vertices 500 \
    --samples 3000 \
    --dilation 0.05 \
    --temp-dir ./qmat_temp/ \
    --output-dir ./final_output/
```

If you already have a suitable MA file, you can skip the first step:

```bash
python integrated_qmat_coverage_axis.py \
    --mesh ./input/bird/bird.off \
    --ma ./input/bird/bird.ma \
    --qmat ./build/QMAT \
    --vertices 500 \
    --skip-step1 \
    --output-dir ./final_output/
```

### Parameter Description

- `--mesh`: Input mesh file path (must be .off format)
- `--ma`: Input MA file path (.ma format)
- `--qmat`: QMAT executable file path
- `--vertices`: Target number of spheres (default: 500)
- `--samples`: Number of surface sampling points (default: 3000)
- `--dilation`: Dilation parameter (default: 0.05)
- `--temp-dir`: QMAT temporary output directory (default: ./qmat_temp/)
- `--output-dir`: Final output directory (default: ./final_output/)
- `--skip-step1`: Skip QMAT step 1, use original MA file directly

### Output Files

#### CoverageAxis Intermediate Files (./output/)
- `mesh.obj`: Original mesh
- `mesh_samples_N.obj`: Surface sampling points
- `mesh_inner_points.obj`: All interior candidate points
- `mesh_selected_inner_points.obj`: Selected optimal interior points
- `mesh_selected_inner_points.txt`: Coordinates and radius information of selected points
- `selected_points_for_qmat.txt`: Selected points file formatted for QMAT

#### QMAT Temporary Files (./qmat_temp/)
- `export_half___v_X___e_Y___f_Z.ma`: Simplified MA file generated in step 1
- `sim_MA___v_X___e_Y___f_Z.obj`: Simplified MA generated in step 1 (OBJ format)

#### Final Output Files (./final_output/)
- `sim_MA___v_X___e_Y___f_Z.obj`: Final simplified medial axis (OBJ format)
- `export_half___v_X___e_Y___f_Z.ma`: Final simplified medial axis (MA format)
- `test_all_poles.obj`: Visualization file of all poles


### QMAT Output File Format

According to QMAT output specifications:
- `sim_MA___v_X___e_Y___f_Z.obj`: Simplified medial axis (OBJ format)
- `export_half___v_X___e_Y___f_Z.ma`: Simplified medial axis (MA format)
- `test_all_poles.obj`: (Mode 2 only) All poles visualization file

Where X, Y, Z represent the number of vertices, edges, and faces respectively.




# Citation

```angular2html
@inproceedings{dou2022coverage,
  title={Coverage Axis: Inner Point Selection for 3D Shape Skeletonization},
  author={Dou, Zhiyang and Lin, Cheng and Xu, Rui and Yang, Lei and Xin, Shiqing and Komura, Taku and Wang, Wenping},
  booktitle={Computer Graphics Forum},
  volume={41},
  number={2},
  pages={419--432},
  year={2022},
  organization={Wiley Online Library}
}
```

```angular2html
@inproceedings{wang2024coverage,
  title={Coverage Axis++: Efficient Inner Point Selection for 3D Shape Skeletonization},
  author={Wang, Zimeng and Dou, Zhiyang and Xu, Rui and Lin, Cheng and Liu, Yuan and Long, Xiaoxiao and Xin, Shiqing and Komura, Taku and Yuan, Xiaoming and Wang, Wenping},
  booktitle={Computer Graphics Forum},
  volume={43},
  number={5},
  pages={e15143},
  year={2024},
  organization={Wiley Online Library}
}
```

```commandline
@article{xu2023globally,
  title={Globally consistent normal orientation for point clouds by regularizing the winding-number field},
  author={Xu, Rui and Dou, Zhiyang and Wang, Ningna and Xin, Shiqing and Chen, Shuangmin and Jiang, Mingyan and Guo, Xiaohu and Wang, Wenping and Tu, Changhe},
  journal={ACM Transactions on Graphics (TOG)},
  volume={42},
  number={4},
  pages={1--15},
  year={2023},
  publisher={ACM New York, NY, USA}
}
```

# References
- https://libigl.github.io/tutorial/  Many thanks to the contributors of libigl :)
- https://www.cgal.org/
- https://gist.github.com/dendenxu/ee5008acb5607195582e7983a384e644




