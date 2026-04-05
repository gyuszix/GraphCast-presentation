# Opening slide
𝑋𝑡+1 = GraphCast(𝑋𝑡,𝑋𝑡−1).

𝑋𝑡+1:𝑡+𝑇
= (GraphCast(𝑋𝑡,𝑋𝑡−1),GraphCast(ˆ
𝑋𝑡+1,𝑋𝑡),...,GraphCast(ˆ
𝑋𝑡+𝑇−1
ˆ
𝑋𝑡+𝑇−2)

# Physical Variables
## insert phy-var-table

# GraphCast componenets
## grid nodes: 
### Each grid node represents a vertical slice of the atmosphere at a given latitude-longitude point
At 0.25° resolution, there is a total of 721 ×1440= 1,038,240 grid nodes. each with (5 surface variables + 6 atmospheric variables
×37 levels) ×2 steps + 5 forcings ×3 steps + 5 constant = 474 input features.

## Mesh nodes: 
### are placed uniformly around the globe in a R-refined icosahedral mesh 𝑀𝑅. The mesh is iteratively refined 𝑀𝑟 →𝑀𝑟+1 by splitting each triangular face into 4 smaller faces, resulting in an extra node in the middle of each edge. GraphCast works with a mesh that has been refined 𝑅 = 6 times, 𝑀6, resulting in 40,962 mesh nodes, each with the 3 input features.
mesh-refinement.png

## Mesh edges: 
### are bidirectional edges added between mesh nodes that are connected in the mesh. This results in a total of 327,660 mesh edges,, each with 4 input features.

## Grid2Mesh edges: 
### are unidirectional edges that connect sender grid nodes to receiver mesh nodes. An edge is added if the distance between the mesh node and the grid node is smaller or equal than 0.6 times5 the length of the edges in mesh 𝑀6 (see Figure 1) which ensures every grid node is connected to at least one mesh node. This results on a total of 1,618,746 Grid2Mesh edges, each with 4 input features

## Mesh2Grid edges:
### are unidirectional edges that connect sender mesh nodes to receiver grid nodes. For each grid point, we find the triangular face in the mesh 𝑀6 that contains it and add three Mesh2Grid edges of the form 𝑒M2G 𝑣M, to connect the grid node to the three mesh nodes adjacent s →𝑣G r to that face (see Figure 1). Features eM2G,features 𝑣M are built on the same way as those for the mesh s →𝑣G r edges. This results on a total of 3,114,720 Mesh2Grid edges (3 mesh nodes connected to each of the 721 ×1440 latitude-longitude grid points), each with four input features.

# GraphCast Architecture
# talk about standerd encode-process-decode arch. Inset arch-verview image.

## Encode
### The purpose of the encoder is to prepare data into latent representations for the processor, which will run exclusively on the multi-mesh. Embedding the input features As part of the encoder, we first embed the features of each of the grid nodes, mesh nodes, mesh edges, grid to mesh edges, and mesh to grid edges into a latent space of fixed size using five multi-layer perceptrons (MLP). Grid2Mesh GNN Next, in order to transfer information of the state of atmosphere from the grid nodes to the mesh nodes, we perform a single message passing step over the Grid2Mesh bipartite subgraph GG2M(VG ,VM ,EG2M)connecting grid nodes to mesh nodes
Insert encode.png

## Processor
### The processor is a deep GNN that operates on the Mesh subgraph GM(VM ,EM)which only contains the Mesh nodes and and the Mesh edges. Note the Mesh edges contain the full multi-mesh, with not only the edges of 𝑀6, but all of the edges of 𝑀5, 𝑀4, 𝑀3, 𝑀2 , 𝑀1 and 𝑀0, which will enable long distance communication.
Multi-mesh GNN A single layer of the Mesh GNN is a standard interaction network [5, 6] which first updates each of the mesh edges using information of the adjacent nodes, Then it updates each of the mesh nodes, aggregating information from all of the edges arriving at that mesh node. And after updating both, the representations are updated with a residual connection and for simplicity of the notation, also reassigned to the input variables. The previous paragraph describes a single layer of message passing, but following a similar approach to [43, 39], we applied this layer iteratively 16 times, using unshared neural network  weights for the MLPs in each layer

## Decoder
The role of the decoder is to bring back information to the grid, and extract an output. Mesh2Grid GNN Analogous to the Grid2Mesh GNN, the Mesh2Grid GNN performs a single message passing over the Mesh2Grid bipartite subgraph GM2G(VG ,VM, ,EM2G). The GNN first updates each of the Grid2Mesh edges using information of the adjacent nodes. Then it updates each of the grid nodes, aggregating information from all of the edges arriving at that grid node

Output function Finally the predictionˆ
y𝑖 for each of the grid nodes is produced using another MLP, which contains all 227 predicted variables for that grid node. Similar to [43, 39], the next weather state,ˆ 𝑋𝑡+1, is computed by adding the per-node prediction,ˆ 𝑌𝑡, to the input state for all grid nodes
