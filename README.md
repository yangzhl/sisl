# sids #

[![Build Status](https://travis-ci.org/zerothi/sids.svg?branch=master)](https://travis-ci.org/zerothi/sids)

The system incentive for dense simulation toolbox provides a simple
toolbox for manipulating, constructing and creating tight-binding matrices 
in a standard and uniform way.

It provides easy interfaces for creating and calculating band-structures of
simple tight-binding models as well as interfacing to more advanced DFT
programs.

## Usage ##

### Geometry manipulation ###

sids contain a class for manipulating geometries in a consistent and easy
way. Without using any other feature this enables you to easily create and
manipulate structures in a consistent manner. 

For instance to create a huge graphene flake

	sq3h  = 3.**.5 * 0.5
    gr = Geometry(cell=np.array([[1.5, sq3h,  0.],
                                 [1.5,-sq3h,  0.],
                                 [ 0.,   0., 10.]],np.float) * 1.42,
                  xyz=np.array([[ 0., 0., 0.],
                                [ 1., 0., 0.]],np.float) * 1.42,
                  atoms = Atom(Z=6,R = 1.42), nsc = [3,3,1])
    huge = gr.tile(100,axis=0).tile(100,axis=1)

Which results in a 20000 atom big graphene flake.

#### IO-manipulation ####

sids employs a variety of IO interfaces for managing geometries.

The hard-coded file formats are:

1. ___xyz___, standard coordinate format
 Note that the the _xyz_ file format does not _per see_ contain the cell size.
 The `XYZSile` writes the cell information in the `xyz` file comment section (2nd line). Hence if the file was written with sids you retain the cell information.
2. ___got___, reads geometries from GULP output
3. ___nc___, reads/writes NetCDF4 files created by SIESTA
4. ___tb___, intrinsic file format for geometry/tight-binding models
5. ___fdf___, SIESTA native format
6. ___XV___, SIESTA coordinate format with velocities

All file formats in sids are called a _Sile_ (sids file). This small difference
prohibits name clashes with other implementations.

To read a file one can do

    import sids
	fxyz = sids.get_Sile('file.xyz')

which returns an `XYZSile` file object that enables reading the information in
`file.xyz`. To read the geometry and obtain a geometry object

	geom = fxyz.read_geom()

and now you can interact with that geometry at will. 

Even though these are hard coded you can easily extend your own file format

	sids.add_Sile(<file ending>,<SileObject>)

for instance the `XYZSile` is hooked using:

	sids.add_Sile('xyz',XYZSile)
	sids.add_Sile('XYZ',XYZSile)

which means that `sids.get_Sile` understands files `*.xyz` and `*.XYZ` files as
an `XYZSile` object. You can put whatever file-endings here and classes to retain API
compatibility. See the `sids.io` package for more information. Note that a call to
`add_Sile` with an already existing file ending results in overwriting the initial
meaning of that file object.

__NOTE__: if you know the file is in _xyz_ file format but the ending is erroneous, you can force the `XYZSile` by instantiating using that class

	sids.XYZSile(<filename>)

which disregards the ending check. 

### Tight-binding ###

To create a tight-binding model you _extend_ a geometry to a `TightBinding` class which
contains the required sparse pattern.

To create the nearest neighbour tight-binding model for graphene you simply do

    # Create nearest-neighbour tight-binding
    # graphene lattice constant 1.42
    dR = ( 0.1 , 1.5 )
	on = (0.,1.)
    nn = (-0.5,0.)

	# Ensure that graphene has supercell connections
	gr.set_supercell([3,3,1])
    tb = TightBinding(gr)
    for ia in tb.geom:
        idx_a = tb.close_all(ia,dR=dR)
        tb[ia,idx_a[0]] = on
        tb[ia,idx_a[1]] = nn

at this point you have the tight-binding model for graphene and you can easily create
the Hamiltonian using this construct:

    H, S = tb.tocsr(k=[0.,0.5,0])

which returns the Hamiltonian and the overlap matrices in the `scipy.sparse.csr_matrix`
format. To calculate the dispersion you diagonalize and plot the eigenvalues

	import matplotlib.pyplot as plt
    klist = ... # dispersion curve
	eigs = np.empty([len(klist),tb.no])
	for ik,k in enumerate(klist):
		H, S = tb.tocsr(k)
		eigs[ik,:] = sli.eigh(H.todense(),S.todense(),eigvals_only=True)
	for i in range(tb.no):
		plt.plot(eigs[:,i])

Very large tight-binding models are notoriously slow to create, however, sids
implement a much faster method to loop over huge geometries

	for ias, idxs in tb.geom.iter_block(iR = 10):
		for ia in ias:
			idx_a = tb.geom.close(ia, dR = dR, idx = idxs)
			tb[ia,idx_a[0]] = on
			tb[ia,idx_a[1]] = nn

which accomplishes the same thing, but at much faster execution. `iR` should be a
number such that `tb.geom.close(<any index>,dR = tb.geom.dR * iR)` is approximately
1000 atoms.


## Downloading and installation ##

Installing sids requires the following packages:

   - __numpy__
   - __scipy__
   - __netCDF4__, this module is only required if you need interface to construct
    the transport tight-binding model for `TBtrans`

Installing sids is as any simple Python package

	python setup.py install --prefix=<prefix>


## Contributions, issues and bugs ##

I would advice any users to contribute as much feedback and/or PRs to further
maintain and expand this library.

Please do not hesitate to contribute!

If you find any bugs please form a [bug report/issue][issue].

If you have a fix please consider adding a [pull request][pr].


## License ##

The sids license is [LGPL][lgpl], please see the LICENSE file.


<!---
Links to external and internal sites.
-->
[sids@git]: https://github.com/zerothi/sids
[sids-doc]: http://github.com/zerothi/sids
[issue]: https://github.com/zerothi/sids/issues
[pr]: https://github.com/zerothi/sids/pulls
[lgpl]: http://www.gnu.org/licenses/lgpl.html


<!---
Local variables for emacs to turn on flyspell-mode
% Local Variables:
%   mode: flyspell
% End:
-->

