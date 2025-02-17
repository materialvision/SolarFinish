# SolarFinish
SolarFinish is an early work in progress. Originally built for my personal use, I've incorporated contributions from others and tried to make it more generally useful so others can benefit from easier post-processing with better results. Feel free to use, but I make no promises. It may fail on images from cameras and telescopes different from my own. Expect it to continue to evolve, and don't expect much tech support.

SolarFinish is a command line python app that can batch process a whole directory, or a single image specified by a local filename or remote URL. If a single file, you have the option of interactively adjusting the parameters and seeing the results immediately.

The process consists of:
- Rotation and flip by either automatic aligning to GONG image given a date, or giving explicit flip and rotation angle, or nothing
- Automatic centering of the solar disk and cropping to a specified solar radius multiple
- Adjust contrast of features on the disk and prominences using Convolutional Normalizing Radial Graded Filter (CNRGF) algorithm
- Clip dark pixels to black to reduce noise
- Brighten resulting grayscale image with a blended gamma-like curve: `weight*np.power(im, gamma) + (1-weight)*(1-np.power(1-im,1/gamma))`
- Apply separate RGB gamma curves to colorize

Assumes a full solar disk input image that has been through typical stacking and sharpening using tools like Autostakkert and Imppg, with no or minimal adjustment of intensity curves. It's also best for the input image to be uncropped to avoid artifacts near edges and to allow arbitrary cropping at the end. That said, it takes what you give it and tries its best as long as it can find a big circle in the image.

If you are comfortable with running python scripts locally, you should be able to use pip to install necessary packages and just run SolarFinish.py directly. If you aren't able to do that but have a Windows PC, I've checked in dist/SolarFinish.exe, built with pyinstaller. You can download it by navigating to the file in github and clicking the "raw" button. Be aware that the executable is rather large--about 100M. This is due to the fact that this script depends on multiple modules, each of which contributes DLLs, sometimes redundant copies. This really adds up, and it's not trivial to trim out unneeded binaries. You'll probably also find that your browser and operating system are (wisely) wary of letting you run random executables downloaded from the internet. I wouldn't knowingly upload any viruses, but you'd be smart to make sure you scan any executables before running them, regardless the source, and even then not run untrusted executables at all if your PC has valuable, unbackedup files.

Some things to try out first... to run in interactive mode to process a local image file or a remote URL image:
```
   SolarFinish.exe --interact mysolarimage.tif
```
or
```
   SolarFinish.exe --interact https://www.cloudynights.com/uploads/gallery/album_24370/gallery_79290_24370_2225872.png
```

As an example, if you run this and cancel the file open dialog to use the sample image:
```
   SolarFinish.exe --brighten 0.6 --brightenweight 0.0 --enhance 1.2,2.5 --crop 1.15 --rotate 0.6328125 --darkclip 0.015 --silent
```
Here's the result:
![result output image](examples/2023-09-09-Lunt50DS-enhancedcolor.png)

This will run in interactive mode starting with all modifications except colorization disabled:
```
   SolarFinish --interact --brighten 1.0 --enhance no --crop no --darkclip 0
```

## Interactive Mode
When running in interactive mode with the --interact or -i command line arguments, you're presented with sliders that change the different parameters, showing the result in realtime. When you are finished, and hit a key in the window, it exits, writing out the result grayscale and colorized images, and displaying the command line parameters corresponding to your parameter choices. The GUI is crude, but functional. Here's an explanation for the sliders and checkboxes:
- **Contrast Enhance (CNRGF)** toggles whether or not to apply contrast enhancement
- **Min/Max Adjust** these control how much contrast enhancement is applied. Min controls how much contrast is exaggerated in areas that already have high contrast, and Max controls contrast enhancement in areas that begin with low contrast. Min should be less than max, and the smaller the value, the less enhancement.
- **Gamma** controls brightening after the contrast enhancement. Lower is brighter, higher is darker.
- **Gamma Weight** shifts the gamma curve. 1 means a simple gamma curve where the bulk of the brightening is on darker pixels -- this can be a problem because it emphasizes background glow and noise beyond the solar limb. 0 means shift emphasis to brighter pixels, generally in solar disk. In the middle blends between the two. It helps to use Quadrant to zoom in on prominences when adjusting this to see the subtle effect.
- **Dark Clip** controls the level at which pixels are clipped to black. This should stay very low, as raising too high will eat away at the faint fringes of prominences. Too low, though, and you might see too much background glow and noise.
- **Center and Crop** toggles whether or not image is centered and expanded/cropped
- **Crop Radius** controls the size of the final image crop
- **Rotation** controls the rotation of the image, to let you manually align
- **Horizontal/Vertical Flip** flips the image horizontally or vertically or both
- **Colorize Red/Green/Blue** adjust gamma curves for each color. Smaller is brighter
- **Align Date / Align** set the date of your capture, then hit Align. It takes a minute, so be patient. At the end, the rotation and flip settings should be the best match for the GONG capture at that date. (https://halpha.nso.edu/)

And view controls that have no effect on the generated image:
- **Show Colorized**: toggles between grayscale and colorized view
- **Zoom** lets you zoom out to see full disk even for large image sizes

More details:
```
usage: SolarFinish.exe [-h] [-v] [-t TYPE] [-p PATTERN] [-o [OUTPUT]] [-s]
                       [-a] [-f {h,v,hv}] [-g GONGALIGN] [-b BRIGHTEN]
                       [-w BRIGHTENWEIGHT] [-e ENHANCE] [-c CROP] [-r ROTATE]
                       [-d DARKCLIP] [-i] [-k COLORIZE]
                       [filename]

Process solar images

positional arguments:
  filename              Image file to process

options:
  -h, --help            show this help message and exit
  -v, --version         show program's version number and exit
  -t TYPE, --type TYPE  filetype to go along with -d, defaults to tif
  -p PATTERN, --pattern PATTERN
                        String pattern to match for -d
  -o [OUTPUT], --output [OUTPUT]
                        Output directory
  -s, --silent          run silently
  -a, --append          append the settings used for gamma, min contrast, and
                        max contrast as part of the output filename
  -f {h,v,hv}, --flip {h,v,hv}
                        rotate final images horizontally, vertically, or both
  -g GONGALIGN, --gongalign GONGALIGN
                        Date of GONG image to compare for auto-align, YYYY-MM-
                        DD
  -b BRIGHTEN, --brighten BRIGHTEN
                        gamma value to brighten by, 1 = none, 0.1 = extreme
                        bright, 2.0 darken
  -w BRIGHTENWEIGHT, --brightenweight BRIGHTENWEIGHT
                        weight to shift gamma brightening, 1 = use gamma
                        curve, 0 = less brightening of darker pixels
  -e ENHANCE, --enhance ENHANCE
                        contrast enhance min,max or no. 1 = no enhance, 5 =
                        probably too much.
  -l DECONVOLUTION --deconvolution DECONVOLUTION
                        add Lucy-Richardson deconvolution to sharpen the disc as 
                        a pre-step before other adjustments
  -c CROP, --crop CROP  final crop radius in solar radii. Or no
  -r ROTATE, --rotate ROTATE
                        rotation in degrees
  -d DARKCLIP, --darkclip DARKCLIP
                        clip minimum after contrast enhancement and before
                        normalization
  -i, --interact        interactively adjust parameters
  -k COLORIZE, --colorize COLORIZE
                        R,G,B weights for colorization
```
