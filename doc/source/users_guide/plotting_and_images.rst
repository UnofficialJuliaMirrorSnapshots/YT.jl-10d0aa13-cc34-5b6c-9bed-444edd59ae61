.. _plotting-and-images:

.. |yt_plotting_docs| replace:: ``yt`` plotting documentation
.. _yt_plotting_docs: http://yt-project.org/doc/visualizing/plots.html

Plotting and Images
===================

.. _plots:

Plots
-----

``YT`` provides an interface to two of the most common plotting routines in ``yt``: ``SlicePlot``
and ``ProjectionPlot``:

.. code-block:: julia

    function SlicePlot(ds::Dataset, axis, fields; center="c", width=nothing, 
                       field_parameters=nothing, args...)

    function ProjectionPlot(ds::Dataset, axis, fields; weight_field=nothing,
                            center="c", width=nothing, field_parameters=nothing, 
                            data_source=nothing, args...)

Unlike other methods in ``YT``, these return the native ``yt`` Python-based objects. This is
mainly for convenience; it allows one to use all of the annotation and plot modification methods
that hang off these objects. The API for these objects is the same as it is in ``yt``,
which can be found in the |yt_plotting_docs|_.

We'll illustrate the plotting functionality with a ``SlicePlot`` as an example:

.. code-block:: jlcon

    julia> slc = YT.SlicePlot(ds, "z", ["density","temperature"], width=(500.,"kpc"))

.. image:: ../images/slice_density.png

.. image:: ../images/slice_temperature.png

which produces a ``SlicePlot`` ``PyObject`` of the ``fields`` "density" and "temperature",
along the "z" ``axis``. The optional argument ``width`` has been used to zoom on on the central
500 kpc of the domain. The ``SlicePlot`` object has available to it all of the methods for
annotating the plot that one would have access to in ``yt``. For example, one can annotate grids:

.. code-block:: jlcon

    julia> slc.annotate_grids()

.. image:: ../images/slice_density_grids.png

.. image:: ../images/slice_temperature_grids.png

or velocity vectors:

.. code-block:: jlcon

    julia> slc.annotate_velocity()

.. image:: ../images/slice_density_velocity.png

.. image:: ../images/slice_temperature_velocity.png

Logging can be set for specific fields:

.. code-block:: jlcon

    julia> slc.set_log("temperature", false)

.. image:: ../images/slice_temperature_linear.png

or the colormap can be changed:

.. code-block:: jlcon

    julia> slc.set_cmap("density", "kamae")

.. image:: ../images/slice_density_colormap.png

.. code-block:: jlcon

To save a plot:

.. code-block:: jlcon

    julia> slc.save("my_awesome_plot.png")
    
If one is in the `IJulia notebook <http://github.com/JuliaLang/IJulia.jl>`_, the ``show_plot``
method can be used to display the plot inline:

.. code-block:: jlcon

    julia> YT.show_plot(slc)

The full set of options for these plots can be found in the |yt_plotting_docs|_.

.. _images:

Images
------

To create a raw 2D image from a ``Slice``, ``Proj``, or ``Cutting`` object,
one can create a ``FixedResolutionBuffer`` object using the ``to_frb`` method:

.. code-block:: julia

    function to_frb(cont::Union{Slice,Proj}, width::Length,
                    nx::Union{Integer,Tuple{Integer,Integer}}; 
                    center=nothing, height=nothing, periodic=false)
                    
    function to_frb(cont::Cutting, width::Length,
                    nx::Union{Integer,Tuple{Integer,Integer}}; 
                    height=nothing, periodic=false)

where ``cont`` is the ``Slice``, ``Proj``, or ``Cutting`` object, ``width`` is the width of the plot,
``nx`` is the resolution of the image, ``center`` is the center of the image (defaults to the
``center`` of the ``cont``), and ``height`` is the height of the image (defaults to the
``width``). The resolution ``nx`` can either be a single value or a tuple of two values,
depending on how you want to set the width and height. 

.. note::
    
    The ``center`` keyword argument is not available when calling ``to_frb`` on a ``Cutting``.
    
This is an example of how to create a ``FixedResolutionBuffer`` from a ``Slice``:

.. code-block:: jlcon

    julia> slc = YT.Slice(ds, "z", 0.0)
    YTSlice (sloshing_nomag2_hdf5_plt_cnt_0100): axis=2, coord=0.0

    julia> frb = YT.to_frb(slc, (500.,"kpc"), 800)
    FixedResolutionBuffer (800x800)

which can be plotted with a plotting package such as
`PyPlot <http://github.com/stevengj/PyPlot.jl>`_ or `Winston <http://github.com/nolta/Winston.jl>`_:

.. code-block:: jlcon

    julia> using Winston

    julia> imagesc(frb["kT"].value)

which yields the following image:

.. image:: ../images/winston.png