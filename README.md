# PlatformIO Helpers

Python3 is only supported.

These helpers work by using [PlatformIO pre-script system.](https://docs.platformio.org/en/latest/projectconf/advanced_scripting.html)


## mbedignore.py

This script allows to ignore specific directories within the mbed-os framework run under PlatformIO.

You should be able to use this script freely across different projects without interfering each other,
when ignoring specific paths within the mbed-os framework. 

The script needs `.mbedignore` file as input. See [How to use it](#how-to-use-it).

`.mbedignore` shall contain relative paths which are to be ignored within the mbed-os framework, e.g.:

```
features/cellular
drivers/source/usb
components/wifi
```

> Before you use the script, it is suggested to run:
>
> ```
> git init
> git add .
> git commit -m "Baseline"
> ```
>
> in the mbed-os framework directory, which is most probably `~/.platformio/packages/framework-mbed`.
>
> You'll be protected against unintended changes made by the script. Alternatively, if something goes wrong, you can always delete `framework-mbed` folder, and PIO will download it again.
>

### How to use it

1. Create your `.mbedignore` file and put it in the root directory of your PlatformIO-based project. Alternatively, you can use one from the example folder of this repository.
2. Copy `mbedignore.py` script to the root directory of your PIO project.
3. Add this line to the environment you want in your `paltformio.ini`:

```
extra_scripts = pre:mbedignore.py
```

### How to remove it

1. Clear the content of `.mbedignore` but do not delete it.
2. Do a PIO build (to launch the script at least one time). When you see "MBED_OS:" line (at the beginning), you can stop the build.
3. Now, all mbed ignore files have been removed from PIO mbed framework folder by the script. You can remove the `extra_scripts` line from your `paltformio.ini`, remove `mbedignore.py` script, and remove `.mbedignore` file.

### TODO

* Add native support of official `.mbedignore` files.
* Found a better way to remove properly mbedignore file when not needed.
* (Secure python main function ?) 
* Warning about wrong path instead of Error.

## custom_library_json.py

This script allows to apply a custom `library.json` file to any external library. It is assumed that the external
library is downloaded using `lib_deps` inside `platformio.ini`.

*NOTE:* When using the program in the pre-script the `library.json` will be not copied on the first build. This is
because the `library.json` is applied before the PIO's `build` target which downloads all the dependencies on the first
run. On the second build invocation the `library.json` should be applied correctly.


### How to use it

1. Prepare your custom `library.json` file for a specific library.

2. Put the file into a subdirectory within the project directory tree. The path shall be constructed like that:

```
<root directory of the project>/<path1>/<library name>
```

`path1` will contain all the custom `library.json` files. Within this path each subdirectory will have a name of a
library you want to apply the custom `library.json`. Each of those subdirectories will contain respective
`library.json` file, e.g.:

```
lib_overlay
    |____boost
            |____library.json
    |____unity
            |____library.json
    |____googleTest
            |____library.json
```

In the example `path1` is `lib_overlay`.

3. Create an `extra_script.py` with this content inside:

```
from os import path
import sys
import custom_library_json

Import("env")

root_dir = env['PROJECT_DIR']

sys.path.append(path.join(root_dir, 'scripts'))
```

4. Put that section to the `extra_script.py` and repeat the operation depending the number of library you want to add:

```
custom_library_json.apply(env, 'path1', 'library name')
```

Tune the paths according to your environment.

5. Add this line to the environment you want in your `paltformio.ini`:

```
extra_scripts = pre:extra_script.py
```

If you are already using mbed ignore script, do a multi line:

```
extra_scripts =
	pre:mbedignore.py
	pre:extra_script.py
```

