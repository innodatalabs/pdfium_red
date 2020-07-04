# Developer Notes

## Developing (easy way) - with docker

One time task - download image of developer container:
```bash
docker pull mkroutikov/redstork
```

To compile and build wheel:
```bash
docker run -v`pwd`:/pdfium/redstork -it mkroutikov/redstork
make so  # or make sodbg
make wheel
```

## Releasing new version to PyPI

Make sure that `redstork/__init__.py` has correct version (bump if needed).

Tag name should match the `__version__` in `redstork/__init__.py` (and hence the version of the PyPI package).

```
git checkout master
git pull
git tag {tag}
git push --tags
```

Pushing a tag to the `master` branch triggers Travis build and deploy process.

## Building developer container
Build a container with PDFium sources and dependencies compile, so that `redstork` development
can be lightweight and fast.

```bash
make docker
```

This takes a long time (downloads all deps and compiles 1.5K sources for Debug and Release).


## Developing (hard way) - no docker

### Tooling

Build tool-chain from Google includes:
* gclient
* ninja
* gn

```bash
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
```

You **must** add this directory to your path, best do this in your `~/.bashrc`
```
export PATH=$PATH:/home/mike/git/depot_tools
```

### PDFium

From this directory, do:
```bash
gclient config --name pdfium --unmanaged https://pdfium.googlesource.com/pdfium.git
gclient sync

# checkout the right branch
cd pdfium
version="chromium/4192"
git fetch origin $version
git checkout $version

# update again since we changed the version
gclient sync
```

### Checkout redstork
From `pdfium` directory, do:

```bash
git clone https://github.com/innodatalabs/redstork.git
```

### Apply patches

Patch root `BUILD.gn` file
```bash
patch -p0 -i redstork/patches/BUILD.gn.diff
```

Note to myself: how to generate patch files
```bash
git diff --no-prefix >> filename.diff
```

### Generate ninja files

Use `gn` tool (from Google toolchain) to generate `ninja` files:
```
cd redstork
mkdir out out/Debug out/Release
cp src/args.Debug.gn out/Debug/args.gn
cp src/args.Release.gn out/Release/args.gn
gn gen out/Debug
gn gen out/Release
```

Note: You can also set arguments interactively using `gn args out/Debug` command.
If so, use the following settings: (note that you may want to change `is_debug` fo `false`)
```gn
use_goma = false
is_debug = true

pdf_use_skia = false
pdf_use_skia_paths = false
pdf_enable_xfa = false
pdf_enable_v8 = false
is_component_build = true

clang_use_chrome_plugins = false
```

### Build
```bash
ninja -C out/Debug
```
