$ ./gaseous-giganticus --help
usage: gaseous-giganticus [-b bands] [-i inputfile] [-o outputfile]
       [-w w-offset] [-h] [-n] [-v velocity factor] [-B band-vel-factor]

Options:
   -a, --pole-attenuation: attenuate band velocity near poles.  If
                 attenuation is zero, band velocity at poles will be
                 relatively high due to short distance around band.
                 Default is 0.5. Min is 0.0, max is 1.0.
   -b, --bands : Number of counter rotating bands.  Default is 6.0
   -B, --band-vel-factor: Multiply band velocity by this number when
                   computing velocity field.  Default is 2.9
   -c, --count : Number of iterations to run the simulation.
                 Default is 1000
   -d, --dump-velocity-field : dump velocity field data to specified file
                               (see -r option, below)
   -D, --faderate : Rate at which particles fade, default = 0.01
   --dump-flowmap : dump velocity field data to a series of
                    six RGB png files encoding x, y in R, G channels
   -e, --band-speed-power: Band speed is modulated by
        cos(K*latitude) ^ band-speed-power, where band-speed-power is an
        odd integer. By default, 1, higher values make the fast moving part
        of the bands narrow and the parts between wider.
   -E, --equirectangular image-height. Output an equirectangular image in
         addition to the usual cubemap images.  The image height must be
         an integer power of 2.
   -f, --fbm-falloff: Use specified falloff for FBM noise.  Default is 0.5
   -F, --vfdim: Set size of velocity field.  Default:2048. Min: 16. Max: 2048
   -g, --gain, 2nd and later octaves are multiplied by pow(fbm-falloff, (octave-1)*gain)
   -h, --hot-pink: Gradually fade pixels to hot pink.  This will allow
                   divergences in the velocity field to be clearly seen,
                   as pixels that contain no particles wil not be painted
                   and will become hot pink.
   -i, --input : Input image filename.  Must be RGB or RGBA png file.
   -I, --image-save-period: Interval of simulation iterations after which
         to output images.  Default is every 25 iterations
   -k, --cubemap: input 6 RGB or RGBA png cubemap images to initialize particle color
   -K, --cache-aware fraction:  initialize the first fraction of particles in a cache
         aware way (faster) and remaining particles with random locations.
         E.g. -K 0.5 will initialize 50% of particles in a cache aware way and
         50% randomly.
   -l, --large-pixels: particles are 3x3 pixels instead of 1 pixel.  Allows
                       for fewer particles and thus faster rendering, but this
                       WILL leave visible artifacts at cubemap face boundaries.
   -L, --noise-levels: Number of fractal noise levels to consider.  Default 4, max 7
   -m, --speed-multiplier:  band_speed_factor and velocity_factor are
         by this number.  It is a single option to affect both by a
         multiplier which is a bit easier than setting an absolute value
         for them individually.
   -n, --no-fade:  Do not fade the image at all, divergences will be hidden
   -N, --sequence-number:  Do not overwrite output images, instead embed a
                           sequence number in the filenames.
   -o, --output : Output image filename template.
               Example: 'out-' will produces 6 output files
               out-0.png, out-1.png, ..., out-5.png
   -O, --opacity: Specify minimum opacity of particles between 0 and 1.
                  default is 0.2
   -p, --particles: Use specified number of particles. Default is 8000000.
   -P, --plainmap  Do not use sinusoidal image mapping, instead repeat image
                   on six sides of a cubemap.
   -r, --restore-velocity-field: Instead of computing the velocity field from
                                 scratch, import a previously saved velocity
                                 field (see -d option, above)
   -R, --random: Random values are used for bands, band-vel-factor, velocity-factor
                 noise-scale, and w-offset.  -S and -V options are also set.
   -s, --stripe: Begin with stripes from a vertical strip of input image
   -S, --sinusoidal: Use sinusoidal projection for input image
                 Note: sinusoidal is the default projection.
                 Note: --stripe and --sinusoidal are mutually exclusive
   -t, --threads: Use the specified number of CPU threads up to the
                   number of online CPUs
   -T, --thread-iterations: Number of iterations of thread movement particles
         before threads join. Default is 1. Higher values will reduce the number
         of times threads are created and destroyed.
   --trap-nans: trap divide by zero, overflow and invalid floating point
                exceptions
   -v, --velocity-factor: Multiply velocity field by this number when
                   moving particles.  Default is 1200.0
   -V, --vertical-bands:  Make bands rotate around Y axis instead of X
                          Also affects --stripe option.
   --vortex-band-threshold: controls distribution of vortices (see man page)
   --vortex-size: radius of vortices as a fraction of planet radius. Default 0.04
   --vortex-size-variance: Range of vortex sizes, default is plus or minus 0.02
   -w, --w-offset: w dimension offset in 4D open simplex noise field
                   Use -w to avoid (or obtain) repetitive results.
   -W, --wstep: w coordinate of noise field is incremented by specified
                amount periodically and velocity field is recalculated
   -x, --vortices: how many artificial circular vortices to add into the v-field
   -z, --noise-scale: default is 2,600000

Note: As noise-scale increases, velocity-factor should generally decrease.

Examples:

   ./gaseous-giganticus --noise-scale 2.5 --velocity-factor 1300 --bands 10\
        -i input_image.png -o output_image
   ./gaseous-giganticus -R -i input_image.png -o output_image

Hints for speeding things up:

0. If your system does CPU frequency scaling, make sure whatever governs this is
   in a 'performance' mode rather than a 'power saving' mode. (On linux, see cpufreq-set)
1. First choose bands, velocity-factor, and noise-scale with a small number of
   particles, say, -p 50000. This will allow you to quickly iterate these values
   to get the general shape of the turbulence and overall look of the output the
   way you want it, and to determine roughly how many iterations are needed.
2. Once you establish roughly how many iterations you need (200-250 is usually good)
   use the -c and -I options.  PNG encoding takes significant time, so doing it
   less frequently speeds things up considerably.
3. Once you have the general shape of the turbulence, use the -d option to save
   the velocity field to a file, and in subsequent runs, use -r to load it from
   this file rather than calculating it from scratch each time.
4. Next iterate on your input image colors.  For this, use -K 1.0 with 2500000 to
   3000000 particles or so.
5. Once satisfied with your input image, run without the -K option and with the default
   of 8 million particles for better coverage and smoother output.



---------- OWN COMMANDS ----------

   --vortex-speed: Vortex speed every 25 frames in fraction of half the planet [-1.0, 1.0]
   --vortex-lat: Latitude in which vortex moves [-90, 90]

Own command to compile everything into the exe (without dependencies):
 make LDFLAGS="-static -static-libgcc -static-libstdc++ -lz"


Own tried commands:
./gaseous-giganticus -V --sinusoidal --noise-scale 2.5 --velocity-factor 1300 -i ./colors.png -o ./output/frame --bands 7 --equirectangular 2048

./gaseous-giganticus -V --sinusoidal --noise-scale 2.5 --velocity-factor 1300 -i ./colors.png -o ./output/frame --bands 7 --equirectangular 1024 --count 15000

./gaseous-giganticus -V --sinusoidal --noise-scale 2.5 --velocity-factor 5000 -i ./colors.png -o ./output/frame --bands 7 --equirectangular 1024 --count 1000 --pole-attenuation 1.0 --band-vel-factor 1.0 --faderate 0.0 --opacity 1.0

./gaseous-giganticus -V --sinusoidal --noise-scale 1.2 --velocity-factor 10000 -i ./colors.png -o ./output/frame --bands 3 --equirectangular 1024 --count 1000 --pole-attenuation 1.0 --band-vel-factor 4.0 --faderate 0.0 --opacity 1.0 --vortex-size 1.0 --vortex-size-variance 0.0 --vortices 1 --w-offset 10 --wstep 10

./gaseous-giganticus -V --sinusoidal --noise-scale 3.0 --velocity-factor 800 --bands 20 -i ./colors.png -o ./output/frame --equirectangular 1024 --opacity 1.0


DAMIT DEZIMALZAHLEN GEHEN MUSS MAN EIN FUCKING "," KOMMA VERWENDEN ANSTATT EINEM "." PUNKT!!!!!!!!!!!!!


From Steven Cameron Jupiter Example: ./gaseous-giganticus -V --sinusoidal --noise-scale 2.8 --velocity-factor 800 --bands 20 -i ./colors.png -o ./output/frame --equirectangular 1024

Own Variations for Jupiter:

   -> Test for Vortex: ./gaseous-giganticus -V --sinusoidal --noise-scale 1,0 --velocity-factor 800 --bands 0 -i ./colors.png -o ./output/frame --equirectangular 1024 --vortex-band-threshold 0,01 --vortex-size 0,1 --vortices 1 --wstep 100

   -> ./gaseous-giganticus -V --sinusoidal --noise-scale 2.8 --velocity-factor 800 --bands 20 -i ./colors.png -o ./output/frame --equirectangular 1024 --vortex-band-threshold 0,01 --vortex-size 0,1 --vortices 1 --wstep 100

   -> Good Vortex Size: ./gaseous-giganticus -V --sinusoidal --noise-scale 2.8 --velocity-factor 800 --bands 20 -i ./colors.png -o ./output/frame --equirectangular 1024 --vortex-band-threshold 0,01 --vortex-size 0,2 --vortices 1

Bestes bisher (large pixels gegen fragmente, hab jupiter original mit large pixels und no fade verglichen) -> ./gaseous-giganticus -V --sinusoidal --noise-scale 2,0 --velocity-factor 800 --bands 20 -i ./colors.png -o ./output/frame --equirectangular 1024 --vortex-band-threshold 0,01 --vortex-size 0,22 --vortices 1 --large-pixels --count 1500

Mit Bewegendem Vektorfeld (wstep) -> ./gaseous-giganticus -V --sinusoidal --noise-scale 2,0 --velocity-factor 800 --bands 20 -i ./colors.png -o ./output/frame --equirectangular 1024 --vortex-band-threshold 0,01 --vortex-size 0,22 --vortices 1 --large-pixels --count 1500 --wstep 0,1

Mit input parametern für vortex: ./gaseous-giganticus -V --sinusoidal --noise-scale 2,0 --velocity-factor 800 --bands 20 -i ./colors.png -o ./output/frame --equirectangular 1024 --vortex-band-threshold 0,01 --vortex-size 0,22 --vortices 1 --large-pixels --count 1500 --wstep 0,1 --vortex-speed 0,0001 --vortex-lat -22 --image-save-period 1

Angepasst an die realität mit 600 frames insgesamt (ein frame entspricht 4.95 echt tagen): ./gaseous-giganticus -V --sinusoidal --noise-scale 2,0 --velocity-factor 1000 --bands 20 -i ./colors.png -o ./output/frame --equirectangular 1024 --vortex-band-threshold 0,01 --vortex-size 0,22 --vortices 1 --large-pixels --count 6000 --wstep -0,125 --vortex-speed 0,0016 --vortex-lat -22 --image-save-period 10

Angepasst an die realität mit 60 frames insgesamt (ein frame entspricht 49.5 echt tagen): ./gaseous-giganticus -V --sinusoidal --noise-scale 1,0 --velocity-factor 10000 --bands 20 -i ./colors.png -o ./output/frame --equirectangular 1024 --vortex-band-threshold 0,01 --vortex-size 0,22 --vortices 1 --large-pixels --count 600 --wstep -1,25 --vortex-speed 0,016 --vortex-lat -22 --image-save-period 10

Low Res Test (Wirbel passt so mit speed 1/600, auf 600 Frames genormt, 4.95 tage / Frame): ./gaseous-giganticus -V --sinusoidal --noise-scale 2,0 --velocity-factor 800 --bands 20 -i ./colors.png -o ./output/frame --equirectangular 128 --vortex-band-threshold 0,01 --vortex-size 0,22 --vortices 1 --large-pixels --count 6000 --wstep -0,125 --vortex-speed -0,0016666666666667 --vortex-lat -22 --image-save-period 10 --particles 300000 --vfdim 256

Higher Res Test (Wirbel passt so mit speed 1/600, auf 600 Frames genormt, 4.95 tage / Frame): ./gaseous-giganticus -V --sinusoidal --noise-scale 2,0 --velocity-factor 800 --bands 20 -i ./colors.png -o ./output/frame --equirectangular 256 --vortex-size 0,22 --vortices 1 --large-pixels --count 8000 --wstep -0,01 --vortex-speed -0,0016666666666667 --vortex-lat -22 --image-save-period 10 --vfdim 512

test Mit Rot Speed Vortex: ./gaseous-giganticus -V --sinusoidal --noise-scale 2,0 --velocity-factor 800 --bands 20 -i ./input/jupiter-colors.png -o ./output/frame --equirectangular 1024 --vortex-size 0,22 --vortices 1 --large-pixels --count 8000 --wstep -0,01 --vortex-speed -0,1 --vortex-lat -22 --image-save-period 10 --vortex-rot-speed 1,0

Neuer Test: ./gaseous-giganticus -V --sinusoidal --noise-scale 2,0 --velocity-factor 800 --bands 20 -i ./input/jupiter-colors.png -o ./output/frame --equirectangular 1024 --vortex-size 0,22 --vortices 1 --large-pixels --count 8000 --wstep -0,005 --vortex-speed -0,0016666666666667 --vortex-lat -22 --image-save-period 10 --vortex-rot-speed 2,0

------- FOR TESTING -------

0. ---> For orientation (Steven Cameron Jupiter):

./gaseous-giganticus -V --sinusoidal --noise-scale 2,8 --velocity-factor 800 --bands 20 -i ./input/jupiter-colors.png -o ./output/frame --equirectangular 1024


1. ---> Start Point (Example from above):

./gaseous-giganticus -V --sinusoidal --noise-scale 2,5 --velocity-factor 1300 --bands 10 -i ./input/jupiter-colors.png -o ./output/frame --equirectangular 1024


2. ---> Correct Number of Bands:

./gaseous-giganticus -V --sinusoidal --noise-scale 2,5 --velocity-factor 1300 --bands 12 -i ./input/jupiter-colors.png -o ./output/frame --equirectangular 1024


3. ---> Correct Jet Speed (Normalized to N=500 Frames in final animation -> jet (not equatorial) crossing planet in 160/2970 * N = 26.94 Frames), also output image every 10 iterations :

-> Band Speed Next to Equator (band-vel-factor -24,0 is perfect, negative to have equatorial jet go from west to east):
./gaseous-giganticus -V --sinusoidal --noise-scale 2,5 --velocity-factor 1300 --bands 12 -i ./input/jupiter-colors.png -o ./output/frame --equirectangular 1024 --image-save-period 10 --band-vel-factor -24,0

-> Speed Fallof Towards Poles (have been tested for, is not mathematically like 1/20 of equator speed or smth; set to max: 1,0):
./gaseous-giganticus -V --sinusoidal --noise-scale 2,5 --velocity-factor 1300 --bands 12 -i ./input/jupiter-colors.png -o ./output/frame --equirectangular 1024 --image-save-period 10 --band-vel-factor -24,0 --pole-attenuation 1,0


4. ---> Correct Noise:

-> Noise detail (fbm falloff, 0,1 means few details, 0,9 means high details, details means more small noise vortices; trial and error -> fvm 0,7 is good detail level)
./gaseous-giganticus -V --sinusoidal --noise-scale 2,5 --velocity-factor 1300 --bands 12 -i ./input/jupiter-colors.png -o ./output/frame --equirectangular 1024 --image-save-period 10 --band-vel-factor -24,0 --pole-attenuation 1,0 --fbm-falloff 0,5

-> Noise scale (means how much noise, 2,2 seems like a good value):
./gaseous-giganticus -V --sinusoidal --velocity-factor 1300 --bands 12 -i ./input/jupiter-colors.png -o ./output/frame --equirectangular 1024 --image-save-period 10 --band-vel-factor -24,0 --pole-attenuation 1,0 --fbm-falloff 0,5 --noise-scale 2,2

-> Velocity Factor (how strong the noise vortices influence the actual bands -> speed of particles in vortex):
./gaseous-giganticus -V --sinusoidal --velocity-factor 1300 --bands 12 -i ./input/jupiter-colors.png -o ./output/frame --equirectangular 1024 --image-save-period 10 --band-vel-factor -24,0 --pole-attenuation 1,0 --fbm-falloff 0,5 --noise-scale 2,2 --velocity-factor 800

[ONLY MAYBE] -> Slightly move noise field to move the edge vortices (instead of just staying in the same place):
./gaseous-giganticus -V --sinusoidal --velocity-factor 1300 --bands 12 -i ./input/jupiter-colors.png -o ./output/frame --equirectangular 1024 --image-save-period 10 --band-vel-factor -24,0 --pole-attenuation 1,0 --fbm-falloff 0,5 --noise-scale 2,2 --velocity-factor 800 --wstep 0,002



5. ---> Vortex. Artificially place 1 vortex with size 0,22 (tested what value is good for GRS). Placed it at lat 22°S (correct), with speed -0,002 (1/N speed. So one circumnavigation at N=500 frames). Vortex rot speed 5,5 for good shape (not real wind speed).

./gaseous-giganticus -V --sinusoidal --bands 12 -i ./input/jupiter-colors.png -o ./output/frame --equirectangular 1024 --image-save-period 10 --band-vel-factor -24,0 --pole-attenuation 1,0  --noise-scale 2,2 --velocity-factor 800 --fbm-falloff 0,5 --vortices 1 --vortex-size 0,25 --vortex-lat -22 --vortex-speed 0,002 --vortex-rot-speed 5,5 --wstep 0,0



6. ---> Reduce Fragments

-> Large Pixels to reduce fragments:
./gaseous-giganticus -V --sinusoidal --bands 12 -i ./input/jupiter-colors.png -o ./output/frame --equirectangular 1024 --image-save-period 10 --band-vel-factor -24,0 --pole-attenuation 1,0  --noise-scale 2,2 --velocity-factor 800 --fbm-falloff 0,5 --vortices 1 --vortex-size 0,25 --vortex-lat -22 --vortex-speed 0,002 --vortex-rot-speed 5,5 --wstep 0,0 --large-pixels



7. ---> Final Render (500 Frames + 100 Fade Frames + 50 Frames to let Simulation balance at start)

./gaseous-giganticus -V --sinusoidal --bands 12 -i ./input/jupiter-colors.png -o ./output/frame --equirectangular 1024 --image-save-period 10 --band-vel-factor -24,0 --pole-attenuation 1,0  --noise-scale 2,2 --velocity-factor 800 --fbm-falloff 0,5 --vortices 1 --vortex-size 0,25 --vortex-lat -22 --vortex-speed -0,002 --vortex-rot-speed 5,5 --wstep 0,003 --large-pixels --count 7500


A Unrealistic: --> Physikalisch incorrekter test aber mit wirbel korrekt (laufzeit 500 frames wie ebend):

./gaseous-giganticus -V --sinusoidal --bands 12 -i ./input/jupiter-colors.png -o ./output/frame --equirectangular 1024 --image-save-period 10 --band-vel-factor -2,4 --pole-attenuation 1,0  --noise-scale 2,0 --velocity-factor 800 --fbm-falloff 0,5 --vortices 1 --vortex-size 0,22 --vortex-lat -22 --vortex-speed -0,001 --vortex-rot-speed 1,0 --wstep 0,01 --large-pixels --count 7500


B Unrealistic: --> Physikalisch incorrekter test aber mit wirbel korrekt (laufzeit 500 frames wie ebend):

./gaseous-giganticus -V --sinusoidal --bands 12 -i ./input/jupiter-colors.png -o ./output/frame --equirectangular 1024 --image-save-period 10 --band-vel-factor -2,4 --pole-attenuation 1,0  --noise-scale 2,0 --velocity-factor 400 --fbm-falloff 0,5 --vortices 1 --vortex-size 0,22 --vortex-lat -22 --vortex-speed -0,002 --vortex-rot-speed 1,0 --wstep 0,002 --large-pixels --count 7500


C Unrealistic: --> Physikalisch incorrekter test aber mit wirbel korrekt (laufzeit 500 frames wie ebend)[JETZT MIT KORREKTEM VORTEX SPEED 0,002]:

./gaseous-giganticus -V --sinusoidal --bands 12 -i ./input/jupiter-colors.png -o ./output/frame --equirectangular 1024 --image-save-period 10 --band-vel-factor -2,4 --pole-attenuation 1,0  --noise-scale 2,0 --velocity-factor 400 --fbm-falloff 0,5 --vortices 1 --vortex-size 0,22 --vortex-lat -22 --vortex-speed -0,002 --vortex-rot-speed 1,0 --wstep -0,003 --large-pixels --count 7500
