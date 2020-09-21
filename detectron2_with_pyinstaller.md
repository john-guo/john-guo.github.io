# win10下基于detectron2的python脚本用pyinstaller打包exe的注意事项


*打包环境的版本信息:*

模块|版本
-|-
python|3.8
fvcore|0.1.2
torch|1.6
torchvision|0.7
detectron2|0.2.1
pyinstaller|4.0


基于以上版本在win10下detectron2制作的程序直接用pyinstallr打包发布不依赖于安装环境的独立exe会碰到打包出来的exe无法正常执行的问题。


假设要打包的文件为test.py，文件开头引入库为:

```
import torch
import detectron2.engine

...
```

执行以下打包命令获得test.exe:

`pyinstaller test.py`

那么在运行时基本上就会发生类似"OSError: could not get source code"的问题。

这个问题是因为pyinstaller无法正确解析detectron2所有依赖的模块造成打包时遗漏了相关模块导致的，可以采用指定hidden-import方式显示通知pyinstaller需要打包哪些依赖模块。

pyinstaller提供了3种方式手动加载依赖模块：命令行、spec文件、hook。
如果指定的依赖模块比较多那命令行输入就会比较繁琐容易出错，而使用spec文件时如果在命令行指定成脚本那就会被pyinstaller覆盖也容易出错，这里就采用hook方式解决这个问题。

pyinstaller的hook其实就是个python脚本，只不过名字必须是"hook-模块完全名.py"这种形式，由于torch自己提供了一个hook，
为了避免冲突这里就采用hook-detectron2.engine.py来对detectron2.engine进行hook。

先给出hook-detectron2.engine.py的脚本内容:

```
import os
from PyInstaller.utils.hooks import get_package_paths

hiddenimports =[
    'fvcore',
    'torchvision',
    'detectron2',
]

datas = [
    (get_package_paths('fvcore')[1], 'fvcore'),
    (get_package_paths('torchvision')[1], 'torchvision'),
    (get_package_paths('detectron2')[1], 'detectron2'),
]

torchlib = os.path.join(get_package_paths('torch')[1], 'lib\\*.dll')

binaries = [
    (torchlib, '.'),
]
```

这里最关键的就是为依赖包没有官方提供pyinstaller-hook包的模块指定加载路径，包括了fvcore, torchvision, detectron2这3个。

关于hook里torch那段，因为我们的目标是打包一个不依赖于安装环境能独立运行的exe，所以我们打包的结果必须包含torch用到的所有dll，
而torch提供的默认hook虽然是复制了torch所有文件但是没有把lib下的dll放到exe同一个目录，这在windows下执行exe时如果搜索路径里没包括exe目录下的torch\lib就会出错，
因此打包时就直接把这些dll复制到exe同级目录方便使用，发布时可以选择删除torch\lib以减小发布尺寸
**或者可以去掉hook torchlib这段在启动程序前把torch\lib设置为dll搜索路径。**

有了这个hook文件后，采用以下打包命令进行打包问题就可以解决(**hook-detectron2.engine.py和test.py必须在同一目录**)：

`pyinstaller --clean --additional-hooks-dir=. --noconfirm test.py`

