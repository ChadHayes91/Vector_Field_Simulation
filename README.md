# Project Description

The goal of this project is converting an input point cloud with corresponding vectors at each point (also known as a vector field) into an initial triangle mesh via delaunay triangulation. The initial triangle mesh is then altered to output a new triangle mesh where each triangle has approximately one of its three edges following the overall vector flow path (and minimizes the number of added vertices and edges such that the triangle mesh is still aesthetically pleasing). The flow path is computed as a collection of individual points with their corresponding vectors derived by their containing triangle’s normalized berycentric coordinates (NBCs).

<p align="center">
  <img width="280" height="280" src="/Images/InputVectorFieldMesh.png">
</p>
<p align = "center">
   Figure 1: example of an input triangle mesh
</p>

## Triangulation, Point Containment, and Normalized Berycentric Coordinates (NBCs)

From the vector field input, the initial step is to construct a preliminary triangle mesh from the point cloud. A simple implementation for this includes using a quadratic approach to the well-known delaunay triangulation algorithm. Triangulation is necessary since a new query point P needs to be contained by three corresponding traingle vertices to express P in the form of NBCs. After triangulation, a few mathematical operations are performed to find the triangle containing query point P, and then P can be expressed as a form of the triple of NBCs from the triangle’s three vertices. Finally, the coefficients of the NBCs of point P can be used to determine the vector (V(P)) corresponding to every single possible point P.

## Traces & Retriangulation

The concept of a trace is to start at an arbitrary query point P, calculate the vector at that query point, and travel a small distance along that vector to a new query point. This process is repeated until one of the following conditions are met:

<ol>
  <li> The trace exits the triangle mesh </li>
  <li> The trace pierces an already traveled to triangle </li>
</ol>

When one trace ends, new traces are started and computed until all triangles have been reached. This creates an approximation of the overall flow of the vector field.

While we trace through the mesh, the collection of traces are computed and we detect when a particular trace crosses into a new triangle. When this happens, we store the midpoint of the local trace for an individual triangle. These stored trace midpoints are combined with the original point cloud vertices and the delaunay triangulation algorithm is ran  using this new collection of vertices. Although this approach introduces a number of new vertices (and consequently new triangles) to the triangle mesh, the edges from the retriangulation better aligns with the vectors from the vector field.

<p align="center">
  <img width="305" height="305" src="/Images/AllTraces.PNG">
</p>
<p align = "center">
   Figure 2: example traces starting from border edges
</p>

## Details

Since the mesh we are dealing with is always a triangle mesh, the right turn test is sufficient for point containment testing. For a triangle ABC and query point P, the right turn test is as follows:

return (AB:AP ≥ 0 == BC:BP ≥ 0) && (BC:BP ≥ 0 == CA:CP ≥ 0)   (1)

Where “ : “ is the det operator between two vectors. The det operator is equal to the dot product after first rotating the second vector in the operation by 90 degrees. Note that if you had a two dimensional vector (x, y), rotation by 90 degrees can be done by mapping (x, y) instead into (y, -x).

Note that the det product equal to zero is included so that a point on the triangle’s edge counts as being contained.

Furthermore, query point P inside triangle ABC (determined by point containment testing) can be expressed as NBCs:

P = aA + bB + cC     (2)

With some algebra, and the fact that a + b + c = 1 (since the berycentric coordinates are normalized), this can be reduced to:

AP = bAB + cAC

where  (AP:AC) / (AB:AC) = b, (AP:AB) / (AC:AB) = c, and a = 1 - b - c

Similarly, the vector at point P (refered to as P’) can be computed with knowing the coefficients a, b, and c along with the vectors at points A, B, and C (denoted as A’, B’ and C’) using the formula:

P' = aA' + bB' + cC'     (3)

After computing P’, the next point in the trace is computed by traveling from point P in the direction of an error-adjusted form of P’ where the methodology for error adjusting is specified in reference [1].  The pseudocode for the generation of all traces for a particular triangle mesh is as follows:
	
![](/Images/TraceGenerationAlg.PNG)
   
Where ComputeTrace(P) is:

 ![](/Images/ComputeTraceAlg.PNG)

As previously mentioned, the midpoint of a trace for each triangle (when MaxTrace is being computed) is stored, and the combination of the original vertices in the point cloud and these additional stored vertices are retrianguled together, using the same delaunay triangulation algorithm. Note that once a trace starting point is selected, that trace is run in both directions of the vector field, since this genreally provides a better looking triangle mesh output.

<p align="center">
  <img width="305" height="305" src="/Images/TracesWithVertices.PNG">
</p>
<p align = "center">
   Figure 3: best (longest) traces with midpoint vertices
</p>

## Results

### Output Mesh Properties

For an input point cloud with a number of points n, the original triangulation creates a mesh with 2n – 2 – b triangles where b is the number of vertices in the convex hull of the point cloud. For normal meshes, the number of triangles exceeds the number of vertices, and since our trace method creates a new vertex for every triangle, the number of vertices more than doubles after retriangulation (b actually remains constant under our method between the two triangulations). However, the new mesh contains a number of edges which follow the trace, approximately equal to the total number of triangles divided by three. 

Since we recompute the triangle mesh using the delaunay triangulation algorithm, there is no guarentee that the number of edges that follow the traces is exactly one third of the number of triangles, but over numerous iterations of testing, we consistently observed this was the case. Note that our approach only generates convex meshes, since the output of delaunay triangulation algorithm will always be convex (this might be desirable or not depending on the application).

<p align="center">
  <img width="305" height="305" src="/Images/TracesWithVertices.PNG">
  <img width="305" height="305" src="/Images/Retriangulation.PNG">
  <img width="305" height="305" src="/Images/RetriangulationWithTraces.PNG">	
</p>

<p align = "center">
   Figure 3: (repeated for reference) &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;
   Figure 4: mesh after retriangulation &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;
   Figure 5: retriangulated mesh with traces 
</p>

