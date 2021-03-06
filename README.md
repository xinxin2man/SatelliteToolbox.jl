# SatelliteToolbox

[![Build Status](https://travis-ci.org/JuliaSpace/SatelliteToolbox.jl.svg?branch=master)](https://travis-ci.org/JuliaSpace/SatelliteToolbox.jl)
[![Build status](https://ci.appveyor.com/api/projects/status/x7ogyjfx1x5yj78j/branch/master?svg=true)](https://ci.appveyor.com/project/ronisbr/satellitetoolbox-jl/branch/master)
[![codecov](https://codecov.io/gh/JuliaSpace/SatelliteToolbox.jl/branch/master/graph/badge.svg)](https://codecov.io/gh/JuliaSpace/SatelliteToolbox.jl)
[![Coverage Status](https://coveralls.io/repos/github/JuliaSpace/SatelliteToolbox.jl/badge.svg?branch=master)](https://coveralls.io/github/JuliaSpace/SatelliteToolbox.jl?branch=master)

This package contains several functions to build simulations related with
satellites. It is used on a daily basis on projects at the [Brazilian National
Institute for Space Research (INPE)](http://www.inpe.br), and it is the engine
of the [FOrPlan Satellite Simulator](http://old.esaconferencebureau.com/docs/default-source/16c11-secesa-docs/39_chagas_presentation.pdf?sfvrsn=2).

## Requirements

* Julia >= 0.7
* HTTP >= 0.6.13
* Interpolations >= 0.8.0
* Parameters >= 0.10.1
* OptionalData >= 0.2.0
* ReferenceFrameRotations >= 0.2.1
* RemoteFiles >= 0.2.1
* StaticArrays >= 0.8.3

## Documentation

A complete documentation of this package is not available yet. However, the
functions are extensively documented using the Julia documentation system, which
can be accessed by typing `?` followed by the function name in REPL.

## Status

Here is a list of the features currently available in the package.

### Analysis

There are many functions to perform small analysis such as:

* Compute the beta angle of an orbit;
* Compute the eclipse time of the satellite;
* Compute the Sun angle and radiation on a satellite surface;
* Verify if a satellite is inside the visibility circle of a ground station;
* Verify if a satellite is above a country (only Brazil is supported at the
  moment).

For more information, see the files inside `./src/analysis/`.

#### Example

```julia
julia> satellite_beta_angle(JD, 7130982.0, 0.001111, 98.405*pi/180, compute_RAAN_lt(JD, 22.5), 5)
5×2 Array{Float64,2}:
 0.0  16.768
 1.0  16.7666
 2.0  16.7682
 3.0  16.7729
 4.0  16.7806
```

### Earth Atmospheric Models

#### NRLMSISE-00

There is a native Julia implementation of the
[NRLMSISE-00](https://ccmc.gsfc.nasa.gov/modelweb/models/nrlmsise00.php)
atmosphere model. The model can be accessed by two methods:

* Using the function `nrlmsise00`, which simplifies the usage but it is not as
  configurable as the original model;
* Using the low level function `gtd7` and `gtd7d`, which requires as input an
  instance of the structure `NRLMSISE00_Structure` that contains the
  configuration parameters.

Both methods return an instance of the structure `NRLMSISE00_Output` that
contains the following values:

* Nitrogen number density;
* N<sub>2</sub> number density;
* Oxygen number density;
* Anomalous oxygen number density;
* O<sub>2</sub> number density;
* Hydrogen number density;
* Helium number density;
* Argon number density;
* Total mass density;
* Exospheric temperature;
* Temperature at the selected altitude;
* Flags used to compute the NRLMSISE-00 model.

For mode information, see the files inside
`./src/earth/atmospheric_models/nrlmsise00`.

##### Example

```julia
julia> nrlmsise00(DatetoJD(1986, 6, 19, 21, 35, 0), # Julian Day
                  +100e3,                           # Altitude [m]
                  -15.779444*pi/180,                # Latitude [rad]
                  -47.929444*pi/180,                # Longitude [rad]
                  91.0,                             # 81 day average F10.7
                  107.0,                            # Daily F10.7
                  12)                               # Magnetic index (daily)
SatelliteToolbox.NRLMSISE00_Output{Float64}
  den_N: Float64 3.51053422831127e11
  den_N2: Float64 9.272926252367151e18
  den_O: Float64 4.1194861170248454e17
  den_aO: Float64 4.849094744214564e-37
  den_O2: Float64 2.1517629595010985e18
  den_H: Float64 2.1129784854913793e13
  den_He: Float64 1.0505700390044144e14
  den_Ar: Float64 9.605966121639621e16
  den_Total: Float64 5.626277180627406e-7
  T_exo: Float64 1027.3184649
  T_alt: Float64 169.96892906010802
  flags: Dict{Symbol,Bool}

julia> nrlmsise00(DatetoJD(1986, 6, 19, 21, 35, 0), # Julian Day
                  +100e3,                           # Altitude [m]
                  -15.779444*pi/180,                # Latitude [rad]
                  -47.929444*pi/180,                # Longitude [rad]
                  91.0,                             # 81 day average F10.7
                  107.0,                            # Daily F10.7
                  12;                               # Magnetic index (daily)
                  output_si = false)                # Output in g, 1/cm^3, g/cm^3
SatelliteToolbox.NRLMSISE00_Output{Float64}
  den_N: Float64 351053.422831127
  den_N2: Float64 9.27292625236715e12
  den_O: Float64 4.1194861170248456e11
  den_aO: Float64 4.8490947442145636e-43
  den_O2: Float64 2.1517629595010984e12
  den_H: Float64 2.1129784854913794e7
  den_He: Float64 1.0505700390044144e8
  den_Ar: Float64 9.605966121639621e10
  den_Total: Float64 5.626277180627406e-10
  T_exo: Float64 1027.3184649
  T_alt: Float64 169.96892906010802
  flags: Dict{Symbol,Bool}
```

### Earth Geomagnetic Field

#### IGRF v12

There is a native Julia implementation of the [International Geomagnetic
Reference Field (IGRF) v12](https://www.ngdc.noaa.gov/IAGA/vmod/igrf.html). This
can be accessed by two functions:

* `igrf12syn`: This is the native Julia implementation of the original FORTRAN
  source-code with **the same** input parameters.
* `igrf12`: An independent (more readable) implementation of the IGRF model.
  However, it is not as fast as `igrf12syn` yet.

For more information, see the files inside
`./src/earth/geomagnetic_field_models`.

##### Example

```julia
julia> igrf12syn(0, 2017.12313, 1, 0.640, 40, 25)
(19685.59353648455, 2164.874709891016, 45708.25456382237, 49814.19286372505)

julia> igrf12(2017.12313, 640, 50*pi/180, 25*pi/180, Val{:geodetic})
3-element StaticArrays.SArray{Tuple{3},Float64,1,3}:
 19685.6
  2164.87
 45708.3
```

### Earth Gravity Models

There is a native Julia implementation of gravity models. In theory, all static
models available in [ICGEM](http://icgem.gfz-potsdam.de/home) can be used by
reading the `gfc` files with the coefficients (see function `parse_gfc`).

To simplify the operations, the coefficients related to the EGM96, JGM2, and
JGM3 were embedded into the toolbox.

For more information, see the files inside `./src/earth/gravity_models`.

##### Example

```julia
julia> egm96 = load_gravity_model(EGM96());

julia> compute_g(egm96, GeodetictoECEF(-45*pi/180, -22*pi/180, 0))
3-element StaticArrays.SArray{Tuple{3},Float64,1,3}:
 -6.45155
  2.60639
  6.93419

julia> compute_g(egm96, GeodetictoECEF(-45*pi/180, -22*pi/180, 0), 10)
3-element StaticArrays.SArray{Tuple{3},Float64,1,3}:
 -6.45155
  2.60639
  6.93419

julia> jgm3 = load_gravity_model(JGM3());

julia> compute_g(jgm3, GeodetictoECEF(-45*pi/180, -22*pi/180, 0))
3-element StaticArrays.SArray{Tuple{3},Float64,1,3}:
 -6.45155
  2.60641
  6.93412

julia> compute_g(jgm3, GeodetictoECEF(-45*pi/180, -22*pi/180, 0), 10)
3-element StaticArrays.SArray{Tuple{3},Float64,1,3}:
 -6.45155
  2.60641
  6.93412
```

### Orbit Analysis

The are many functions related to orbit design, such as computing the possible
Sun Synchronous orbits.

For more information, see the files inside `./src/orbit`.

#### Example

```julia
julia> list_ss_orbits_by_rep_period(3,4,-1,-1,0.001)
16×7 Array{Float64,2}:
 7.5072e6        1.12906e6  1.7467   6480.0   13.0  1.0  3.0
 7.15312e6       7.74986e5  1.71911  6027.91  14.0  1.0  3.0
 6.83798e6       4.59838e5  1.69735  5634.78  15.0  1.0  3.0
 6.55533e6       1.77189e5  1.67989  5289.8   16.0  1.0  3.0
 7.38438e6       1.00624e6  1.73674  6321.95  13.0  2.0  3.0
 7.0441e6        6.65962e5  1.7113   5890.91  14.0  2.0  3.0
 6.74042e6       3.62285e5  1.69111  5514.89  15.0  2.0  3.0
 6.46743e6   89288.8        1.67484  5184.0   16.0  2.0  3.0
 7.53871e6       1.16057e6  1.74932  6520.75  13.0  1.0  4.0
 7.18104e6       8.02905e5  1.72116  6063.16  14.0  1.0  4.0
 6.86292e6  484781.0        1.69898  5665.57  15.0  1.0  4.0
 6.57777e6       1.99633e5  1.68121  5316.92  16.0  1.0  4.0
 7.35445e6       9.76317e5  1.73437  6283.64  13.0  3.0  4.0
 7.01749e6       6.39349e5  1.70944  5857.63  14.0  3.0  4.0
 6.71657e6       3.38435e5  1.68962  5485.71  15.0  3.0  4.0
 6.44591e6   67769.8        1.67363  5158.21  16.0  3.0  4.0
```

### Orbit Propagators

Currently, there are three orbit propagators available: **Two Body**, **J2**,
and **SGP4**. All coded in Julia (no external libraries required).

For more information, see the files inside `./src/orbit/propagators`.

#### Two Body

The following code propagates the orbit of the Brazilian Satellite Amazonia-1
for 1 day using the Two Body orbit propagator.

```julia
julia> orbp = init_orbit_propagator(Val{:twobody}, Orbit(0.0,7130982.0,0.001111,98.405*pi/180,pi/2,0.0,0.0))
julia> (o,r,v) = propagate!(orbp, collect(0:3:24)*60*60)
julia> r
9-element Array{StaticArrays.SArray{Tuple{3},Float64,1,3},1}:
 [5.30372e-7, 7.12306e6, 3.58655e-6]
 [-987245.0, 2.2796e6, -6.68158e6]
 [-6.3457e5, -5.6651e6, -4.29471e6]
 [5.77611e5, -5.94385e6, 3.90922e6]
 [1.00749e6, 1.82033e6, 6.8186e6]
 [70133.2, 7.1069e6, 4.74655e5]
 [-9.62529e5, 2.72855e6, -6.5143e6]
 [-6.88667e5, -5.3608e6, -4.66083e6]
 [5.18048e5, -6.1958e6, 3.50609e6]
```

#### J2

The following code propagates the orbit of the Brazilian Satellite Amazonia-1
for 1 day using the J2 orbit propagator.

```julia
julia> orbp = init_orbit_propagator(Val{:J2}, Orbit(0.0,7130982.0,0.001111,98.405*pi/180,pi/2,0.0,0.0))
julia> (o,r,v) = propagate!(orbp, collect(0:3:24)*60*60)
julia> r
9-element Array{StaticArrays.SArray{Tuple{3},Float64,1,3},1}:
 [5.30372e-7, 7.12306e6, 3.58655e-6]
 [-9.98335e5, 2.14179e6, -6.72549e6]
 [-5.75909e5, -5.83674e6, -4.06734e6]
 [6.65317e5, -5.69201e6, 4.2545e6]
 [9.62557e5, 2.37418e6, 6.65228e6]
 [-1.10605e5, 7.11845e6, -231186.0]
 [-1.02813e6, 1.90664e6, -6.79145e6]
 [-4.82921e5, -5.97389e6, -3.87579e6]
 [750898.0, -5.53993e6, 4.43709e6]
```

#### SGP4

The following code propagates the orbit of the Brazilian Satellite Amazonia-1
for 1 day using the SGP4 orbit propagator.

```julia
julia> orbp = init_orbit_propagator(Val{:sgp4}, Orbit(0.0,7130982.0,0.001111,98.405*pi/180,pi/2,0.0,0.0))
julia> (o,r,v) = propagate!(orbp, collect(0:3:24)*60*60)
julia> r
9-element Array{StaticArrays.SArray{Tuple{3},Float64,1,3},1}:
 [-2159.7, 7.13166e6, -14607.2]
 [-1.00096e6, 2.1411e6, -6.73899e6]
 [-5.78906e5, -5.83897e6, -4.08451e6]
 [6.64614e5, -5.70129e6, 4.24735e6]
 [9.6287e5, 2.37768e6, 6.64987e6]
 [-1.12629e5, 7.12679e6, -2.45705e5]
 [-1.03066e6, 1.90639e6, -6.80469e6]
 [-4.86132e5, -5.97626e6, -3.89338e6]
 [7.5014e5, -5.54932e6, 4.42998e6]
```

**NOTE**: This algorithm has both methods **SGP4** and **SDP4**, the latter
considers the deep space perturbations. It automatically selects which one to
use given the initial orbit parameters.

### Sun

A simple Sun position algorithm is available.

For more information, see the files inside `./src/sun`.

#### Example

```julia
julia> sun_position_i(DatetoJD(2013,7,19,11,00,00))
3-element Array{Float64,1}:
 -6.68124e10
  1.20616e11
  5.22889e10
```

### Coordinate Transformations

#### ECEF <=> ECI

There are many functions to convert between Earth-Centered, Earth-Fixed (ECEF)
reference frames and Earth-Centered Inertial (ECI) reference frames. For more
information, see the documentation of the functions `rECEFtoECI` and
`rECItoECEF`. Two rotation description formats are supported: Direction Cosine
Matrices (DCM) and Quaternions.

Currently, only the **IAU-76/FK5 model** is implemented. Hence, the IERS EOP
Data is necessary but this package is capable to download them from the official
website. You can also read and parse a pre-downloaded file.

The following reference frames are supported:

* **ITRF**: International Terrestrial Reference Frame (ITRF).
* **PEF**: Pseudo-Earth Fixed (PEF) reference frame.
* **MOD**: Mean of Date (MOD) reference frame.
* **TOD**: True of Date (TOD) reference frame.
* **J2000**: J2000 reference frame.
* **GCRF**: Geocentric Celestial Reference Frame (GCRF).

##### Example

```julia
julia> eop_IAU1980 = get_iers_eop(:IAU1980)
julia> rECEFtoECI(DCM, FK5(), ITRF(), GCRF(), DatetoJD(1986, 06, 19, 21, 35, 0), eop_IAU1980)
3×3 StaticArrays.SArray{Tuple{3,3},Float64,2,9}:
 -0.619267      0.78518     -0.00132979
 -0.78518      -0.619267     3.33483e-5
 -0.000797314   0.00106478   0.999999

julia> rECEFtoECI(ITRF(), GCRF(), DatetoJD(1986, 06, 19, 21, 35, 0), eop_IAU1980)
3×3 StaticArrays.SArray{Tuple{3,3},Float64,2,9}:
 -0.619267      0.78518     -0.00132979
 -0.78518      -0.619267     3.33483e-5
 -0.000797314   0.00106478   0.999999

julia> rECEFtoECI(PEF(), J2000(), DatetoJD(1986, 06, 19, 21, 35, 0))
3×3 StaticArrays.SArray{Tuple{3,3},Float64,2,9}:
 -0.619271      0.785176    -0.00133066
 -0.785177     -0.619272     3.45854e-5
 -0.000796885   0.00106622   0.999999

julia> rECEFtoECI(Quaternion, ITRF(), GCRF(), DatetoJD(1986, 06, 19, 21, 35, 0), eop_IAU1980)
Quaternion{Float64}:
  + 0.43630989232629747 - 0.0005909971869613186.i + 0.00030510471843995434.j + 0.8997962188693898.k

julia> rECItoECEF(DCM, FK5(), GCRF(), ITRF(), DatetoJD(1986, 06, 19, 21, 35, 0), eop_IAU1980)
3×3 StaticArrays.SArray{Tuple{3,3},Float64,2,9}:
 -0.619267    -0.78518     -0.000797314
  0.78518     -0.619267     0.00106478
 -0.00132979   3.33483e-5   0.999999

julia> rECItoECEF(J2000(), PEF(), DatetoJD(1986, 06, 19, 21, 35, 0))
3×3 StaticArrays.SArray{Tuple{3,3},Float64,2,9}:
 -0.619271    -0.785177    -0.000796885
  0.785176    -0.619272     0.00106622
 -0.00133066   3.45854e-5   0.999999

julia> rECItoECEF(Quaternion, ITRF(), GCRF(), DatetoJD(1986, 06, 19, 21, 35, 0), eop_IAU1980)
Quaternion{Float64}:
  + 0.43630989232629747 + 0.0005909971869613186.i - 0.00030510471843995434.j - 0.8997962188693898.k
```

#### Geodetic <=> Geocentric

There are two functions `ECEFtoGeodetic` and `GeodetictoECEF` that can be used
to convert between ECEF coordinates and Geodetic coordinates (WGS-84).

## Roadmap

This package will be continuously be enhanced. It is part of an official project
of the Brazilian National Institute for Space Research (INPE). In the short
term, the following is expected:

* Add IAU-2000A model for coordinate transformations.
* Add a numerical propagator using EGM.
