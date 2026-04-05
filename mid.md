---
layout: default
---

# GraphCast Architecture Overview


---
layout: default
---

<!-- Image placeholder: GraphCast Architecture Flow (Figure 1d, 1e, 1f) -->
![](https://via.placeholder.com/800x250?text=Insert+Image:+GraphCast+Architecture+Flow)

GraphCast consists of three major components:
- **Encoder:** Maps input data from the original latitude-longitude grid into learned features on a multi-mesh.
- **Processor:** Uses a 16-layer deep GNN to perform learned message-passing on the multi-mesh, allowing efficient propagation of information across space.
- **Decoder:** Maps the multi-mesh representation back to the original grid and combines it with input state via a residual connection.







---
layout: default
transition: fade-out
---

# Graph Representation: Nodes

<!-- Image placeholder: Grid and Mesh Nodes representation -->
![](https://via.placeholder.com/800x250?text=Insert+Image:+Grid+and+Mesh+Nodes)

**Grid Nodes**
- Represent a vertical atmospheric slice at a latitude-longitude point.
- 0.25° resolution $\rightarrow$ **1,038,240 grid nodes** (721 $\times$ 1440).
- **474 input features** per node (surface/atmospheric variables, levels, steps, forcings).

**Mesh Nodes**
- Placed uniformly around the globe in an R-refined icosahedral mesh ($M_R$).
- Refined 6 times to $M_6$.
- **40,962 mesh nodes**, each with **3 input features**.


---
layout: default
transition: fade-out
---

# Graph Representation: Edges

<!-- Image placeholder: Types of Edges in GraphCast -->
![](https://via.placeholder.com/800x250?text=Insert+Image:+Mesh+and+Bipartite+Edges)

- **Mesh Edges:** Bidirectional, connect mesh nodes. **327,660 edges** (4 input features/edge).
- **Grid2Mesh Edges:** Unidirectional (Grid $\rightarrow$ Mesh). Connects every grid node to at least one mesh node (distance $\le 0.6 \times$ $M_6$ edge length). **1,618,746 edges**.
- **Mesh2Grid Edges:** Unidirectional (Mesh $\rightarrow$ Grid). Connects grid points to the 3 adjacent mesh nodes of their enclosing triangular face. **3,114,720 edges**.





---
layout: image-right
image: images/encoder.png
backgroundSize: 80%
transition: fade-out
---

# Phase 1: The Encoder

<!-- Image placeholder: Encoder embedding and Grid2Mesh GNN -->
![](https://via.placeholder.com/800x250?text=Insert+Image:+The+Encoder)

- **Goal:** Prepare data into latent representations to run exclusively on the multi-mesh.
- **Feature Embedding:** 5 MLPs project features of grid nodes, mesh nodes, and all edges into a fixed-size latent space.
- **Grid2Mesh GNN:** 
  - Transfers atmospheric state information from grid nodes to mesh nodes.
  - Performs a single message-passing step over the bipartite subgraph connecting grid nodes to mesh nodes.




---
layout: image-right
image: images/encoder.png
backgroundSize: 80%
transition: fade-out
---



# Phase 2: The Processor

<!-- Image placeholder: Deep multi-mesh GNN Processor -->
![](https://via.placeholder.com/800x250?text=Insert+Image:+The+Processor+%28Deep+GNN%29)

- **Operation:** Deep GNN operating exclusively on the Mesh subgraph.
- **Long-Distance Communication:** Utilizes edges from all mesh resolutions ($M_0$ to $M_6$).
- **Multi-Mesh GNN:**
  - Iteratively applies message-passing **16 times** using unshared MLP weights.
  - **Step 1:** Updates mesh edges using adjacent node information.
  - **Step 2:** Updates mesh nodes by aggregating all arriving edge information.
  - **Step 3:** Applies residual connections.






---
layout: default
transition: fade-out
---

# Phase 3: The Decoder

<!-- Image placeholder: Decoder representation mapping and Output -->
![](https://via.placeholder.com/800x250?text=Insert+Image:+The+Decoder+and+Output)

- **Goal:** Extract an output by bringing information back from the multi-mesh to the grid.
- **Mesh2Grid GNN:**
  - Performs a single message passing over the bipartite Mesh2Grid subgraph.
  - Updates Grid2Mesh edges using adjacent node information, then updates grid nodes by aggregating arriving information.
- **Output Function:** 
  - Produces per-node predictions ($\hat{Y}_t$) using an MLP (227 variables).
  - Matches the next state, computed via residual connection for all grid nodes:  
    **$\hat{X}_{t+1} = \hat{X}_t + \hat{Y}_t$**