circuitpython_deploy

Deployment script for CircuitPython.

The file `deploy_config.py` that lives in the same place as the `deploy` script and contains overall configuration:

  * the directory of your CircuitPython distribution
  * the version of CircuitPython to use
  * the mountpoint of CIRCUITPY

for example, here's my current config:

```
config = {
    "dir": "/home/dastels/Documents/CircuitPython",  #
    "version": "6.0",
    "circuitpy": "/media/dastels/CIRCUITPY",
}
```

So wherever `dir` is, it will have subdirectories (at least one) corresponding to possible values of `version`. I've chosen "5.0", "6.0", etc. for these. In those subdirectories is where I keep the various CircuitPython UF2 files for various boards and various versions (extracted) of the bundles.

Each of these can be overridden with command line arguments to the deploy script.

Usage of the script is:

```
usage: deploy [-h] [-d DIR] [-v VERSION] [-c CIRCUITPY] [-f FILE]

Deploy/update a CircuitPython project.

optional arguments:
  -h, --help            show this help message and exit
  -d DIR, --dir DIR
  -v VERSION, --version VERSION
  -c CIRCUITPY, --circuitpy CIRCUITPY
  -f FILE, --file FILE
```

The FILE argument is useful in simple cases when you have a CircuitPython file that isn't called `code.py` or `main.py`. For example, the classic LED blink test script: `blink.py`. Use the command
```deploy -f blink.py```
to copy `blink.py` to `CIRCUITPY/code.py`. You likely won't use this much, and certainly not for your own projects where you'd likely start by writing `code.py` or `main.py`.

Typically you'll just use the command:
```
deploy
```
which will update the required libraries and copy the required files to CIRCUITPY.

You can place a file named `manifest.json` in your project directory. This directory also contains whatever should be copied to CIRCUITPY, other than the lib directory.

The manifest file is a JSON file containing two lists: `exclude` and `libs`, e.g:
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

The `exclude` list contains regular expressions (as per the `re` module) which define files in the project directory that *should not* be copied to CIRCUITPY. I.e. any files that match any of the `exclude` regexes won't be copied. In the example, there's a reference image, test files (for pytest) and supporting classes, example files, etc. Think of this like a `gitignore` file. The `manifest.json` file is automatically excluded.

The `libs` list contains the names of files and directories that should be copied into `CIRCUITPY/lib`. Where do those come from? An extracted bundle that lives in `<dir>/<version>` (`dir` and `version` come from the config file or command line arguments. The script will grab libraries from the most recent bundle directory (in case you didn't clean up old versions for some reason).

So you make note of what files/directories not to copy, and what libraries you need, and then just
```deploy```

If you don't provide a manifest, no libaries will be copied, and all project files will be copied to CIRCUITPY.
