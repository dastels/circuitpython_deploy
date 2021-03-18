# circuitpython_deploy 

Deployment script for CircuitPython.

# Configuration 

The file `deploy_config.py` that lives in the same place as the `deploy` script and contains overall configuration:

  * the directory of your CircuitPython distribution
  * the version of CircuitPython to use
  * the mountpoint of CIRCUITPY


For example, here's my current config:

```
config = {
    "dir": "/home/dastels/Documents/CircuitPython",  #
    "version": "6.0",
    "circuitpy": "/media/dastels/CIRCUITPY",
}
```

So wherever `dir` is, it will have subdirectories (at least one) corresponding to possible values of `version`. I've chosen "5.0", "6.0", etc. for these. In those subdirectories is where I keep the various CircuitPython UF2 files for various boards and various versions (extracted) of the bundles.

Each of these can be overridden with command line arguments to the deploy script.


# Usage

Usage of the script is:

```
usage: deploy [-h] [-d DIR] [-v VERSION] [-c CIRCUITPY] [-f FILE] [-s] [-l]

Deploy/update a CircuitPython project.

optional arguments:
  -h, --help            show this help message and exit
  -d DIR, --dir DIR     location of CircuitPython files
  -v VERSION, --version VERSION
                        version of CircuitPython to use
  -c CIRCUITPY, --circuitpy CIRCUITPY
                        location of the CIRCUITPY drive
  -f FILE, --file FILE  the single file to copy to CIRCUITPY/code.py
  -s, --subdirs         copy top level subdirectories (e.g. sounds or fonts)
  -l, --updatelibs      update library modules (requires a project manifest.json)
```

The FILE argument is useful in simple cases when you have a CircuitPython file that isn't called `code.py` or `main.py`. For example, the classic LED blink test script: `blink.py`. Use the command

```deploy -f blink.py```

to copy `blink.py` to `CIRCUITPY/code.py`. You likely won't use this much, and certainly not for your own projects where you'd likely start by writing `code.py` or `main.py`.

Typically you'll just use the command:

```
deploy
```

which will recursively copy the files in the current directory to CIRCUITPY. If you want to copy subdirectories (e.g. `sounds`, `fonts`, `images`, etc.) use the `-s` option. If you want to update the contents of the lib folder, use the `-l` option. See below.

## manifest.json

If you place a file named `manifest.json` in your project directory you can have `deploy` manage the contents of `CIRCUITPY/lib` for you, making sure all required modules are present, and updating to the latest verson of the bundle as appropriate. It also let's you skip copying specific project files.

The manifest file is a JSON file containing two arrays: `exclude` and `libs`, e.g:
```
{
    "exclude":[
        "3839d364531413.5ad64cb6a6e75.jpg",
        "test.*",
        ".pytest_cache",
        ".*_example.py",
        "recording_state.py"
    ],
    "libs":[
        "adafruit_adt7410.mpy",
        "adafruit_bitmap_font",
        "adafruit_bno055.mpy",
        "adafruit_bus_device",
        "adafruit_debouncer.mpy",
        "adafruit_display_shapes",
        "adafruit_display_text",
        "adafruit_esp32spi",
        "adafruit_imageload",
        "adafruit_io",
        "adafruit_logging.mpy",
        "adafruit_mcp230xx",
        "adafruit_pyportal.mpy",
        "adafruit_register",
        "adafruit_requests.mpy",
        "adafruit_sdcard.mpy",
        "adafruit_touchscreen.mpy",
        "neopixel.mpy"
    ]
}
```

The `exclude` array contains regular expressions (as per the `re` module) which define files in the project directory that *should not* be copied to CIRCUITPY. I.e. any files that match any of the `exclude` regexes won't be copied. In the example, there's a reference image, test files (for pytest) and supporting classes, example files, etc. Think of this like a `gitignore` file. The `manifest.json` file is automatically excluded.

The `libs` array contains the names of files and directories that should be copied into `CIRCUITPY/lib`. Where do those come from? An extracted bundle that lives in `<dir>/<version>` (`dir` and `version` come from the config file or command line arguments. The script will grab libraries from the most recent extracted bundle directory (in case you didn't clean up old versions for some reason).

So you make note of what files/directories not to copy, and what libraries you need, and then just

```deploy -l```

*If you don't provide a manifest, no libaries will ever be copied, and all project files will be.*

Typically you will set up your initial manifest file and run `deploy -l` to put the library modules in place. After that you will ususally just use the `-l` option when you add to the list of libraries in the manifest file or download & extract a new version of the bundle. You'll also probably seldom use the `-s` option to avoid taking the time to copy data files in subdirectories, using it only when they get updated. This is because those data files are usually relatively static (and large) compared to code. So most often while working on a project you'll simply use

```deploy```

to update the desired top level project files, typically the code you're working on.
