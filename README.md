pyob
====

Python PyObject wrapper for C++ matplotlib numpy wx etc


Requirements
------------

matplotlibcpp.h https://github.com/lava/matplotlib-cpp

tested matplotlib-cpp version (hash 00a4766) 2018-10-06

tested with Python 3.6.1 (64-bit)

tested with Visual C++ 2017


Environments
------------

set QT_QPA_PLATFORM_PLUGIN_PATH=%PYTHONHOME%/Library/plugins/platforms


Sample
------

``` c++
#define _USE_MATH_DEFINES // M_PI = 3.14159265358979323846...
#include <cmath>
#include <vector>
#include <iomanip>
#include <iostream>
#include <stdio.h>

#include "pyob.h"
// #include "matplotlibcpp.h"
#pragma comment(lib, "python36.lib")

using namespace std;
using namespace pyob;
namespace plt = matplotlibcpp;

int main(int argc, char **argv)
{
  PyBase::version();

  int n = 500;
  vector<double> x(n), y(n);
  for(int i = 0; i < n; ++i){
    x[i] = i;
    y[i] = sin(i * M_PI / 180.0);
  }
  plt::plot(x, y, "--r");

  PyBase::begin(L"dummy");

try{
  PyMod np("numpy");
  fprintf(stdout, "%20.17lf\n", double(np|"pi")); // 3.14159265358979323846...
  fprintf(stdout, "%20.17lf\n", double(np|"e")); // 2.71828182845904523536...

  PyBase x = (np|"arange")(tie(PYLNG(-99), PYLNG(99), PYDBL(.2)), {});
  PyBase y = (np|"arange")(tie(PYLNG(-30), PYLNG(30), PYDBL(.1)), {});
  PyBase m = (np|"meshgrid")(tie(x, y), {});
  PyBase zz = (np|"sqrt")(MKTPL((m[0] ^ 2) + (m[1] ^ 2)), {});

  PyMod pyplot("matplotlib.pyplot");
  PyBase ax = (pyplot|"contour")(MKTPL(m[0] * 3.5, m[1] / 5.0, zz), {});
}catch(const std::exception &e){
  fprintf(stderr, "exception[%s]\n", e.what());
}

  plt::show();

try{
  PyMod _builtins("builtins");
  PyBase _h = (_builtins|"bytes"|"fromhex")(tie(PYSTR("55AA")), {});
  PyBase::s((wchar_t *)_h);

  PyMod wx("wx");
  PyBase app = (wx|"App")();
  PyBase frm = (wx|"Frame")(tie(PYNONE, PYLNG(-1)), {
      {"title", PYSTR("TEST wx")},
      {"size", PYBV("ii", 640, 480)}, {"pos", PYBV("ii", 320, 240)}});
  const char *ascicon =
    "789cc592310e8330100497e006450afe41a8229e41e187f184b4f9457a1a3f27\x0A"
    "657a8acbad6d029130a241597b59e6245b823ba0c0094d63350d5e27e006c0da\x0A"
    "c86d015c4aa0d55aa3eed4ac6fab0f1b7d1f834b5fbcf79aac48d81089c1c512\x0A"
    "1f07ab7a50f73c4be2f3403df32cbb395cbc60e35439d6d346f2ccab220fdbbc\x0A"
    "f8e6c0aedbcf6e9b8f94ac08e5648f55e7eefaba9455af88736e1167fd1f735e\x0A"
    "a571a999dacf902313212b9d966e649f6bd70d29e389d473f3d3fbecbf91e882\x0A"
    "ee672ff5a6afb3a733f407b4ecff08\x0A";
  PyStr ascdat(ascicon);
  PyBase hex = (ascdat|"replace")(tie(PYSTR("\n"), PYSTR("")), {});
  PyMod binascii("binascii");
  PyBase binicon = (binascii|"a2b_hex")(tie(hex), {});
  PyMod zlib("zlib");
  PyBase bindat = (zlib|"decompress")(tie(binicon), {});
  PyMod pyio("io");
  PyBase strm = (pyio|"BytesIO")(tie(bindat), {});
  PyBase im = (wx|"Image")(tie(strm), {});
  PyBase bm = (wx|"Bitmap")(tie(im), {});
  PyBase icon = (wx|"Icon")();
  (icon|"CopyFromBitmap")(tie(bm), {});
  (frm|"SetIcon")(tie(icon), {});
  (frm|"Show")();
  (app|"SetTopWindow")(tie(frm), {});
  (app|"MainLoop")();

  PyMod sys("sys");
  (sys|"path"|"append")(tie(PYSTR(".")), {});

  PyMod mm("mm"); // mm.py (test module) must be at '.'
  PyBase mc = (mm|"mc")(tie(PYLNG(1234)), {});
  PyBase r = (mc|"pi")(tie(mm, mc), {{"a", mm}, {"b", mc}});
}catch(const std::exception &e){
  fprintf(stderr, "exception[%s]\n", e.what());
}

  PyBase::end();
  return 0;
}
```


mm.py (test module)

``` python
class mc(object):
  '''test data overwritten by self.n after construction'''
  n = 9999

  def __init__(self, n):
    self.n = n
    print('TEST mc')

  def pi(self, *args, **kwargs):
    print(f'TEST mc.pi ({len(args)}), {kwargs.keys()}, self.n={self.n}')
    # print(f'{args[0].__dict__}') # OK
    # print(f'{kwargs["a"].__dict__}') # OK
    for i, v in enumerate(args): print(f'{i}: [{v}]')
    for k in kwargs.keys(): print(f'[{k}]: [{kwargs[k]}]')
    return {'result': 'XYZ', 'args': args, 'kwargs': kwargs}
```


License
-------

MIT license
