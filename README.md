IFermi
------

ifermi is a package which provides tools for plotting Fermi surfaces
from DFT output. ifermi is also useful for visualisation of slices of
the three-dimensional Fermi surface along a specified plane. The idea 
is to provide tools which allow for more tailored Fermi surface plots
than what is currently offered by other packages.

The main features include:

1. **Plotting of three-dimensional Fermi surfaces, with interactive plotting
   supported by [mayavi](https://docs.enthought.com/mayavi/mayavi/), [plotly](https://plot.ly/) and [matplotlib](https://matplotlib.org) (see recommended 
   libraries below).**

2. **Taking a slice of a three-dimensional Fermi surface along a specified 
   plane and plotting the resulting contour.**

3. **Identification and visualisation of vanishingly small Fermi surfaces**


Dependencies on external libraries: 

   - VASP calculations are imported using [Pymatgen](https://docs.enthought.com/mayavi/mayavi/).
   - Band interpolation is carried out using [BoltzTraP2]().
   - Plotting is supported in [Mayavi](https://docs.enthought.com/mayavi/mayavi/), [Plotly](https://plot.ly/) and [Matplotlib](https://matplotlib.org).
   - I reccomned using Mayavi or Plotly for three-dimensional
     Fermi surface visualisation. Plotly is selected by deafult. 

The code currently primarily supports VASP calculations, but will 
soon be extended to other platforms supported by Pymatgen 
(Quantum Espresso, Questaal, etc.)

### Usage

IFermi can be used from the command-line or from a python API. The built-in
help (``-h``) option for each command provides a summary of the
available options.

To generate the three-dimensional Fermi surface with default parameters one can 
just run the command:

```bash
ifermi
```

Alternatively, to plot a two-dimensional slice of a Fermi surface along the plane
specified by the miller indices (A B C) and at a distance d, run the command

```bash
ifermi --slice A B C d
```

#### Python interface

ifermi is made up of a number of classes for building and plotting
Fermi surfaces. This includes:

- `FermiSurface`: stores isosurfaces at the Fermi-level for use in plotting,
   as well as other useful structural information. 
- `FermiSlice`: A two-dimesnional slice of the FermiSurface along a specified plane
(The plane is specified with Miller indices)
- `WignerSeitzCell`: Represents the lattice's Wigner-Seitz brillouin zone
- `ReciprocalCell`: Represents the lattice's standard reciprocal cell 
- `Interpolator`: Takes energies specified on a uniform k-mesh and interpolates 
   this to a finer k-mesh.
- `FermiSurfacePlotter`: Given a FermiSurface object, produces an interactive plot   
- `FermiSlicePlotter`: Given a FermiSlice object, produces a two-dimensional plot of that object

A minimal working example for plotting the 3d Fermi surface of MgB2 from a POSCAR
file and Vasprun.xml file is:

```python
import os
import sys

from pymatgen.io.vasp.outputs import Vasprun

def find_vasprun_file():
    """Search for vasprun files from the current directory.

    Will look for vasprun.xml or vasprun.xml.gz files.
    """
    for file in ["vasprun.xml", "vasprun.xml.gz"]:
        if os.path.exists(file):
            return file

    print("ERROR: No vasprun.xml found in current directory")
    sys.exit()


if __name__ == '__main__':

    from ifermi.fermi_surface import FermiSurface
    from ifermi.interpolator import Interpolater
    from ifermi.plotter import FermiSurfacePlotter
    
    # create a Pymatgen BandStructure object from a vasprun file

    filename = find_vasprun_file()

    vr = Vasprun(filename)
    bs = vr.get_band_structure()

    # interpolate the energies to a finer k-point mesh, specified by the interpolate_factor

    interpolater = Interpolater(bs)
    
    interpolate_factor = 8

    interp_bs, kpoint_dim = interpolater.interpolate_bands(interpolate_factor)
    
    # create a FermiSurface object from the resulting energy mesh
    # the Fermi-level can be displaced by changing 'mu' to a non-zero value
    
    fs = FermiSurface.from_band_structure(
        interp_bs, kpoint_dim, mu=0.0, wigner_seitz=True, 
    )

    # Create a FSPlotter object
    plotter = FermiSurfacePlotter(fs)
    
    # specify the directory and prefix of the plot name
    
    # create and save the plot
    
    plotter.plot(plot_type='mayavi', interactive=True)
```

### Example output

An example of the output generated by ifermi for BaFe<sub>2</sub>As<sub>2</sub>
 is shown below:


![BaFe2As2 fermi slice](docs/source/_static/fermi_surface_3.png)

And for a slice taken along the plane specified by the Miller index (0 0 0.5):
![BaFe2As2 fermi slice](docs/source/_static/fermi_slice_3.png)

## Detailed requirements

ifermi is currently compatible with Python 3.5+ and relies on a number of
open-source python packages, specifically:

- [pymatgen](http://pymatgen.org)
- [numpy](http://www.numpy.org)
- [scipy](https://www.scipy.org)
- [matplotlib](https://matplotlib.org)
- [mayavi](https://docs.enthought.com/mayavi/mayavi/)
- [plotly](https://plot.ly/)



## Contributing

If you think that the code could use some improvement
or added functionality, send a push request to the GitHub page. 
I would greatly appreciate any contributions.

## License

IFermi is made available under the MIT License.

## Acknowledgements

Alex Ganose for help developing/improving code.
Sinead Griffin for suggesting the project.

