#####################
Tips & Best Practices
#####################

Although there are countless ways to create objects with build123d, experience
has proven that certain techniques can assist designers in achieving their goals
with the greatest efficiency. The following is a description of these techniques.

*************************
Can't Get There from Here
*************************

Unfortunately, it's a reality that not all parts described using build123d can be
successfully constructed by the underlying CAD core. Designers may have to
explore different design approaches to get the OpenCascade CAD core to successfully
build the target object. For instance, if a multi-section :func:`~operations_generic.sweep`
operation fails, a :func:`~operations_part.loft` operation may be a viable alternative
in certain situations. It's crucial to remember that CAD is a complex field and
patience may be required to achieve the desired results.

************
2D before 3D
************

When creating complex 3D objects, it is generally best to start with 2D work before
moving on to 3D. This is because 3D structures are much more intricate, and 3D operations
can be slower and more prone to failure. For designers who come from a Constructive Solid
Geometry (CSG) background, such as OpenSCAD, this approach may seem counterintuitive. On
the other hand, designers from a GUI BREP CAD background, like Fusion 360 or SolidWorks,
may find this approach more natural.

In practice, this means that 3D objects are often created by applying operations like
:func:`~operations_part.extrude` or :func:`~operations_part.revolve` to 2D sketches, as shown below:

.. code:: python

    with BuildPart() as my_part:
        with BuildSketch() as part_profile:
            ...
        extrude(amount=some_distance)
        ...

With this structure ``part_profile`` may have many objects that are combined and
modified by operations like :func:`~operations_generic.fillet` before being extruded
to a 3D shape.

**************************
Delay Chamfers and Fillets
**************************

Chamfers and fillets can add complexity to a design by transforming simple vertices
or edges into arcs or non-planar faces. This can significantly increase the complexity
of the design. To avoid unnecessary processing costs and potential errors caused by a
needlessly complicated design, it's recommended to perform these operations towards
the end of the object's design. This is especially true for 3D shapes, as it is
sometimes necessary to fillet or chamfer in the 2D design phase. Luckily, these
2D fillets and chamfers are less likely to fail than their 3D counterparts.

************
Parameterize
************

One of the most powerful features of build123d is the ability to design fully
parameterized parts. While it may be faster to use a GUI CAD package for the
initial iteration of a part, subsequent iterations can prove frustratingly
difficult. By using variables for critical dimensions and deriving other dimensions
from these key variables, not only can a single part be created, but a whole set
of parts can be readily available. When inevitable change requests arise, a simple
parameter adjustment may be all that's required to make necessary modifications.

******************
Use Shallow Copies
******************

As discussed in the Assembly section, a :ref:`shallow copy <shallow_copy>` of parts that
are repeated in your design can make a huge difference in performance and usability of
your end design.  Objects like fasteners, bearings, chain links, etc. could be duplicated
tens or even hundreds of times otherwise. Use shallow copies where possible but keep in
mind that if one instance of the object changes all will change.

****************
Object Selection
****************

When selecting features in a design it's sometimes easier to select an object from
higher up in the topology first then select the object from there.  For example let's
consider a plate with four chamfered hole like this:

.. image:: assets/plate.svg
    :align: center

When selecting edges to be chamfered one might first select the face that these edges
belongs to then select the edges as shown here:

.. code-block:: python

    from build123d import *

    svg_opts = {"pixel_scale": 5, "show_axes": False, "show_hidden": True}

    length, width, thickness = 80.0, 60.0, 10.0
    hole_dia = 6.0

    with BuildPart() as plate:
        Box(length, width, thickness)
        with GridLocations(length - 20, width - 20, 2, 2):
            Hole(radius=hole_dia / 2)
        top_face: Face = plate.faces().sort_by(Axis.Z)[-1]
        hole_edges = top_face.edges().filter_by(GeomType.CIRCLE)
        chamfer(hole_edges, length=1)

********************************
Build123d - CadQuery Integration
********************************

As both `CadQuery <https://cadquery.readthedocs.io/en/latest/index.html>`_ and **build123d** use
a common OpenCascade Python wrapper (`OCP <https://github.com/CadQuery/OCP>`_) it's possible to
interchange objects between the two systems by transferring the ``wrapped`` objects as follows:

.. code-block:: python

    import build123d as b3d
    b3d_solid = b3d.Solid.make_box(1,1,1)

    ... some cadquery stuff ...

    b3d_solid.wrapped = cq_solid.wrapped


*****************
Self Intersection
*****************

Avoid creating objects that intersect themselves - even if at a single vertex - as these topoplogies 
will almost certainly be invalid (even if :meth:`~topology.Shape.is_valid` reports a ``True`` value).
An example of where this my arise is with the thread of a screw (or any helical shape) where after
one complete revolution the part may contact itself. One is likely be more successful if the part
is split into multiple sections - say 180° of a helix - which are then stored in an assembly.
