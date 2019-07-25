# Configuration of Thunder 3D Refinement

## 1. Preparation

用relion跑完Initial model。（因为thunder是基于relion的，metadata文件也只能与relion互转。因此其他软件的metadata要先转到relion格式，再转到Thunder格式，容易出现各种各样的bug）

在*InitialModel/jobxxx*目录下面找到*run_it_300_data.star*和*run_it300_class001.mrc*两个文件（本例initial model参数设置为300次循环），复制到新的路径下（假设为*/data/thunder_output*）。

## 2. Metadata Configuration

启动Thunder环境。用thunder提供的STAR_2_THU.py将刚刚复制的.star文件转为.thu格式。用法为：

```shell
python STAR_2_THU.py -i xxx.star -o xxx.thu
```

之后用文本编辑器打开thunder提供的demo_3D.json文件

其中Basic部分设置参考如下：

```json
"Basic" :
{
  "Number of Threads Per Process" : , // 每一Process中的线程数
  "2D or 3D Mode" : "3D",
  "Global Search" : true,
  "Local Search" : true,
  "CTF Search" : false, //whether or not to refine defocus of each single particle
  "Number of Classes" : 1,
  "Size of Image" : 256, // pixel
  "Pixel Size (Angstrom)" : 1.77,
  "Radius of Mask on Images (Angstrom)" : 120, //Mask radius
  "Estimated Translation (Pixel)" : 10,
  "Initial Resolution (Angstrom)" : 60,
  "Perform Global Search Under (Angstrom)" : 15,
  "Symmetry" : "D2",
  "Initial Model" : "run_it300_class001.mrc", //设置为从InitialModel复制来的.mrc文件
  ".thu File Storing Paths and CTFs of Images" : "run_it300_data.thu", //python输出的thu文件
  "Path of Particles" : "../", //此项需查看star文件内_rlnImageName项的路径进行设置
  "Path of Output" : "./", //数据输出路径
  "Prefix of Output" : "",
  "Calculate FSC Using Core Region" : true,
  "Calculate FSC Using Masked Region" : false,
  "Particle Grading" : false, //whether or not to use particle grading to weight the contribution of of each single particle image during reconstruction
  "Auto-Recentre Reference" : false
}
```

其他参数的设置尚未弄清楚XD

## 3. Process Initialization

根据上例设置后，在thunder环境内输入：

```shell
mpirun -n 25 thunder_cpu 3D.json
```

即可，Log文件在*Path of Output*设置的路径下，文件名为*thunder.log*

mpirun的进程数最好为奇数，这样在减掉一个master进程后，分配给halfA和halfB的进程数保持一致，-n的参数最少为3

在配置好CUDA和NCCL环境后，可以使用gpu计算模式，即

```shell
mpirun -n 5 thunder_gpu 3D.json
```

## 4. Mask Create

Thunder的3D refinement会输出Reference_000_A_Final.mrc和Reference_000_B_Final.mrc文件

```shell
thunder_genmask -i Reference_000_A_Final.mrc -o mask.mrc --pixelsize 1.77 --threshold 0.005 --ext 0 --edgewidth 6 -j 8
```

其中pixelsize参数与json保持一致，j为进程数，输出mask.mrc文件

## 5. Postprocess

```shell
thunder_postprocess --inputA Reference_000_A_Final.mrc --inputB Reference_000_B_Final.mrc --mask mask.mrc --pixelsize 1.77 -j 8
```



