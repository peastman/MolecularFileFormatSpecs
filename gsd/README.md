# GSD file format

The **GSD** file format is the native file format for
[HOOMD-blue](https://glotzerlab.engin.umich.edu/hoomd-blue/). **GSD**
files store trajectories of the HOOMD-blue system state in a binary file
with efficient random access to frames. **GSD** allows all particle and
topology properties to vary from one frame to the next. Use the **GSD**
Python API to specify the initial condition for a HOOMD-blue simulation
or analyze trajectory output with a script. Read a **GSD** trajectory
with a visualization tool to explore the behavior of the simulation.

- [GSD Documentation](https://gsd.readthedocs.io/): Docs for the **GSD**
  Python package, C API, and data schema.
- [GitHub Repository](https://github.com/glotzerlab/gsd): **GSD**
  source code and issue tracker.
- [HOOMD-blue](https://glotzerlab.engin.umich.edu/hoomd-blue/):
  Simulation engine that reads and writes GSD files.
- [hoomd-users Google
  Group](https://groups.google.com/d/forum/hoomd-users): Ask questions
  to the **HOOMD-blue** community.
- [OVITO](https://www.ovito.org/): The Open Visualization Tool works
  with GSD files.
- [gsd-vmd plugin](https://github.com/mphoward/gsd-vmd): VMD plugin to
  support GSD files.

## HOOMD Schema

HOOMD trajectories use the
[HOOMD Schema](https://gsd.readthedocs.io/en/stable/specification.html).
The documentation for this schema defines its intended use cases and chunk
information, e.g. `particles/position` is in the `property` category, has
data type `float`, size `Nx3` where `N` is the number of particles, defaults to
`(0, 0, 0)`, and has units of `length`. Most properties are defined for each
particle, not each type. Please refer to the link above for the full table
describing the schema.

## Notable Features of GSD

HOOMD-blue is a general purpose simulation engine, and includes support for
anisotropic particles and polyhedral shapes (e.g. of nanoparticles), among
other features. The list below is *not exhaustive* but documents some of the
features of GSD that are uncommon or may contrast the implicit assumptions of
simulation formats targeting biomolecular simulations.

### Types

Some coarse-grained simulations in HOOMD have types named like `'A'` and
`'B'`, which might correspond to the user's definition of octahedral
nanoparticles and hard spheres, instead of corresponding to a chemical
element. In general, there is no correspondence between types in a HOOMD
simulation and a periodic table or atom-typed force field.

### Anisotropy

The **GSD** format is capable of storing particles' orientations, angular
momenta, and moments of inertia. Orientation and angular momentum are stored
as quaternions, and the moment of inertia is stored as the diagonal 3 entries
of the moment of inertia tensor (the particle shape must be in principal
reference frame so that the inertia tensor is diagonal).

### 2D Simulations

The HOOMD schema is defined with both 3D and 2D simulations in mind. The
chunk `configuration/dimensions` must be 2 or 3. For 2D simulations, the
positions are written with `z=0`, e.g. `(x, y, 0)` for each particle.
Similarly, the orientation quaternions are rotations about the z-axis.
By convention, the simulation box has a height of 1 in 2D simulations.

### Shape Visualization

To simplify the visualization of particle shapes in a HOOMD simulation (e.g.
tetrahedral nanoparticles), the chunk `particles/type_shapes` stores a
JSON-encoded string that defines a shape class for each type. The GSD
`type_shapes` specification is
[documented here](https://gsd.readthedocs.io/en/stable/shapes.html).

### State Data (force fields and similar information)

The HOOMD schema separates particle data (e.g. positions, charges) from the
internal state of integrators, updaters, and other classes (which might store
information like potential parameters / force fields). The
[state data](https://gsd.readthedocs.io/en/stable/schema-hoomd.html#state-data)
in HOOMD version 2 is limited to integration parameters for hard particle
Monte Carlo (HPMC) simulations. Current work on HOOMD version 3 expands
this to include state information from many other classes, making it simpler
to initialize systems with complex force fields that may be configured by
other packages.

### Logged data (user-defined quantities)

Users may store logged data in `log/*` data chunks. Logged data encompasses
values computed at simulation time that are too expensive or cumbersome to
re-compute in post processing. This specification does not define specific
chunk names or define logged data. Users may select any valid name for logged
data chunks as appropriate for their workflow. Please refer to the
[Logged Data documentation](https://gsd.readthedocs.io/en/stable/schema-hoomd.html#logged-data)
for more information.