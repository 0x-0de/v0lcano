# `polygonizers`

Contains polygonization algorithms that used signed-distance fields to generate a mesh around an isosurface. Examples of this include marching cubes and surface nets.

## Types

`pub const PolygonizeCallback: type = *const fn(SDFCell, *ash.mesh.Vertex, [3]f32) ash.rendering.mesh.VertexError!void` - When the polygonization algorithms want to add vertices to a Mesh, this function type is what needs to be called.

## SDFCell (`struct`)

Represents a cell in a signed-distance field. A cell is a cubic space where the points at the 8 corners of the cube have known distances from the isosurface. Ashbloom's polygonizers mandate that *negative* distance values mean that the point is **outside** the isosurface, while *positive* values indicate that the point is **inside** the isosurface.

### Fields

`pos: [3]f32` - Position of the cell in **mesh**-space. This determines where the geometry is placed in the mesh.

`scl: [3]f32` - Scale of the cell in **mesh**-space. This determines the size of the geometry in the mesh.

`values: [8]f32` - This array is ordered as \[X,Y,Z\] in ascending binary => \[-,-,-\], \[+,-,-\], \[-,+,-\], \[+,+,-\], \[-,-,+\], \[+,-,+\], \[-,+,+\], \[+,+,+\], where '-' represents the smaller x/y/z value and '+' represents the larger x/y/z value.

## MarchingCubes (`struct`)

This structure is essentially just a namespace that houses the marching cubes polygonization algorithm.

### Public Functions

**`add_cell_to_mesh(mesh: *ash.Mesh, cell: SDFCell, callback: ?PolygonizeCallback) !void`**

Adds triangles to the mesh, created and calcuated using the marching cubes algorithm. This algorithm is intended to be used one cell at a time, so this function only adds triangles for a single cell. You will likely need to pass a function to handle adding actual vertex data to the mesh, which is the purpose of `callback`. If **null** is passed, the `polygonizers.default_vertex_addition` is used in place of `callback`, and this function provides an example of how `callback` should be used.

## MarchingTetrahedra (`struct`)

This structure is essentially just a namespace that houses the marching tetrahedra polygonization algorithm.

### Public Functions

**`add_cell_to_mesh(mesh: *ash.Mesh, cell: SDFCell, callback: ?PolygonizeCallback) !void`**

Adds triangles to the mesh, created and calcuated using the marching tetrahedra algorithm. This algorithm is intended to be used one cell at a time, so this function only adds triangles for a single cell. You will likely need to pass a function to handle adding actual vertex data to the mesh, which is the purpose of `callback`. If **null** is passed, the `polygonizers.default_vertex_addition` is used in place of `callback`, and this function provides an example of how `callback` should be used.

### Private Functions

**`add_tetra(mesh: *ash.Mesh, cell: SDFCell, callback: ?PolygonizeCallback, positions: [4][3]f32, values: [4]f32) !void`**

After splitting a cell into 6 tetrahedra, this function is called once per tetrahedron to add triangles to that space.

## SurfaceNets (`struct`)

This structure is essentially just a namespace that houses the surface nets polygonization algorithm.

### Structures

#### NetCell (`packed struct(u6)`)

Utility structure used in the surface nets algorithm.

##### Fields

`x: bool`

`y: bool`

`z: bool`

`dir_x: bool`

`dir_y: bool`

`dir_z: bool`

### Public Functions

**`build(mesh: *ash.Mesh, cells: [][][]SDFCell, callback: ?PolygonizeCallback) !void`**

Using a list of `SDFCell`s, adds all vertices to the mesh.

### Private Functions

**`cell_collides_with_isosurface(cell: SDFCell) bool`**

Returns true if an SDF cell contains a mixture of inside and outside points.

**`get_cell_surface_point(cell: SDFCell) [3]f32`**

For a cell which collides with an isosurface, returns the best-fitting point for where the cell collides with the isosurface.

## Public Functions

**`default_vertex_addition(cell: SDFCell, vertex: *ash.mesh.Vertex, vertex_position: [3]f32) ash.mesh.VertexError!void`**

The default polygonizer callback, called if **null** is passed to any of the polygonizer algorithms contained in Ashbloom. Adds the position vertex to the mesh, and assumes the first and only attribute of the mesh is the vertex position.

## Private Functions

**`midpoint_linear(a: f32, b: f32, t: f32) f32`**

Returns where the value `t` is between `a` and `b`.