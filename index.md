<!-- MathJax Configuration -->
<script>
  window.MathJax = {
    tex: {
      inlineMath: [['$', '$'], ['\\(', '\\)']],
      displayMath: [['$$', '$$'], ['\\[', '\\]']]
    },
    svg: {
      fontCache: 'global'
    }
  };
</script>

<!-- MathJax CDN -->
<script src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-svg.js" async=""></script>

<p>Paper link: <a href="https://arxiv.org/pdf/2303.08336">Progressive Frame Patching for FoV-based Point Cloud  Video Streaming
</a></p>

<h1 id="abstract">Abstract</h1>

<p>Many XR applications require the delivery of volumetric video to users with six degrees of freedom (6-DoF) movements. Point Cloud has become a popular volumetric video format. A dense point cloud consumes much higher bandwidth than a 2D/360 degree video frame. User Field of View (FoV) is more dynamic with 6-DoF movement than 3-DoF movement. To save bandwidth, FoV-adaptive streaming predicts a user's FoV and only downloads point cloud data falling in the predicted FoV. However, it is vulnerable to FoV prediction errors, which can be significant when a long buffer is utilized for smoothed streaming. In this work, we propose a multi-round progressive refinement framework for point cloud video streaming. Instead of sequentially downloading point cloud frames, our solution simultaneously downloads/patches multiple frames falling into a sliding time-window, leveraging the inherent scalability of octree-based point-cloud coding. The optimal rate allocation among all tiles of active frames are solved analytically using the heterogeneous tile rate-quality functions calibrated by the predicted user  FoV. Multi-frame downloading/patching simultaneously takes advantage of the streaming smoothness resulting from long buffer and the FoV prediction accuracy at short buffer length. We evaluate our streaming solution using simulations driven by real point cloud videos, real bandwidth traces, and 6-DoF FoV traces of real users. Our solution is robust against the bandwidth/FoV prediction errors, and can deliver high and smooth view quality in the face of bandwidth variations and dynamic user and point cloud movements.</p>


<h1 id="FoV-adaptive Coding and Streaming">FoV-adaptive Coding and Streaming</h1>
<h2 id="Octree-based Scalable PCV Coding">Octree-based Scalable PCV Coding</h2>

<p><img src="images/tile_tree.jpg" alt="Figure 1: octree-based tiles" width="100%" /></p>

<p>A point cloud can be coded using an octree which is recursively constructed: the root node represents the entire space spanned by all the points, which is equally partitioned along the three dimensions into $2\times 2\times 2$ cubes; the root has eight children, each of which is the root of a sub-octree representing each of the eight cubes. The recursion stops at either an empty cube, or the maximum octree level $L$.  Each sub-octree rooted at level $l$ represents a cube with side lengths that are $1/2^l$ of the side lengths of the original space. To code the geometry, the root node records the $(x,y,z)$ coordinates of the center of the space, based on which the coordinates of the center of any cube at any level are uniquely determined. Consequently, each non-root node only needs one bit to indicate whether the corresponding cube is empty or not. For the octree in Fig. (b), the nodes with value $1$ at the three levels represent one, two and three non-empty cubes with side lengths of $1/2$, $1/4$ and $1/8$ in Fig. (a), respectively. All the non-empty nodes at level $l$ collectively represent the geometry information of the point cloud at the spatial granularity of $1/2^l$. This makes octree <i>spatially scalable</i>: with one more level of nodes delivered, the spatial resolution doubles (i.e., the distance between points is halved). The color attributes of a point cloud are coded following the octree structure as well, with each node stores the average color attributes of all the points in the corresponding cube. Scalable color coding can be achieved by only coding  the difference between a node and its parent. The serialized octree can be further losslessly compressed using entropy coding. With MPEG G-PCC, at each tree level $l$, the  status of a node ($0$ or $1$) is coded using context-based arithmetic coding, which uses a context consisting of its neighboring siblings that have been coded and its neighboring parent nodes to estimate the probability $p$ that the node is $1$.</p>

<h2 id="Tile-based Coding and Streaming">Tile-based Coding and Streaming</h2>

<!-- <img src="images/system_cap.png" alt="Figure 2: Experimental Setup" width="90%"/> -->

<!-- <p><img src="images/system.png" alt="Figure 2: Experimental Setup" width="90%" /></p> -->

<p>Considering the limited viewer Field-of-View (FoV) (dependent on the 6-DoF viewpoint), occlusions between objects and parts of the same object, and the reduced human visual sensitivity at long distances, only a subset of points of a PCV frame are <i>visible and discernible</i> to the viewer at any given time. FoV-adaptive PCV  streaming significantly reduces PCV bandwidth consumption by only streaming points visible to the viewer at the spatial granularity that is discernible at the current viewing distance. To support FoV-adaptive streaming, octree nodes are partitioned into slices that are selectively transmitted for <b>FoV adaptability</b> and <b> spatial scalability</b>. Each slice consists of a subset of nodes at one tree level that are independently decodable provided that the slice containing their parents is received. To enable FoV adaptability, a popular approach known as tile-based coding is to partition the entire space into non-overlapping 3D tiles and code each tile independently using a separate tile octree. Only tiles  falling into the predicted FoV will be transmitted. To enable spatial scalability within a tile, we need to put the nodes at each level of the tile octree into a separate slice. As illustrated in Fig. (c), each sub-octree rooted at level $L_T$ represents a 3D-tile with side length of $1/2^{L_T}$. Within each tile sub-octree, nodes down to the $L_B$ level are packaged into a base layer slice, and nodes at the lower layers are packaged into additional enhancement layer slices.</p>

<p>When the streaming server is close to the viewer, one can conduct reactive FoV-adaptive streaming: the client collects and uploads the viewer's FoV to the server; the server renders the view within the FoV, and streams the rendered view as a 2D video to the viewer. To facilitate seamless interaction and avoid motion sickness, the rendered view has to be delivered with short latency (e.g. $<$20 ms) after the viewer movement, the so called Motion-to-Photon (MTP) latency constraint. To address the MTP challenge, we consider predictive streaming that predicts the viewer's FoVs for future frames and prefetches tiles within the predicted FoV.</p>

<h1 id="Progressive PCV Streaming">Progressive PCV Streaming</h1>

<!-- <img src="images/featurea_democap.png" alt="Figure 2: Experimental Setup" width="50%"/> -->

<p><img src="images/seg_download.png" alt="Figure 2: Progressive Streaming Illustration" width="40%" /></p>

<p>Due to the close interaction with PCV objects, viewers are highly sensitive to  QoE impairments, such as black screen, freezes, restarts, excessively long latency, etc. Not only the network and viewer dynamics have direct QoE impacts, bandwidth and FoV prediction errors are also critical for the  effectiveness of predictive streaming. We propose a novel <i>progressive FoV-adaptive PCV  streaming design</i> to minimize the occurrence of the above impairments and deliver a high level of viewer QoE.</p>

<p>Scalable PCV coding opens up a new dimension to reconcile the conflicting desires for long streaming buffer and short FoV prediction interval. It allows us to <b>progressively download and patch</b> tiles: when a frame's playback deadline is still far ahead, we only have a rough FoV estimate for it, and will download  low resolution slices of tiles overlapping with the predicted FoV; as the deadline approaches, we have more accurate FoV prediction, and will <i>patch</i> the frame by downloading additional enhancement slices for tiles falling into the predicted FoV. <i>Progressive streaming is promising to simultaneously achieve streaming smoothness and robustness against bandwidth variations and FoV prediction errors.</i> On one hand, as shown in Fig. 2, each tile in each segment is downloaded over multiple rounds, the interval of which is $\Delta$. For example, tiles of $seg_{i+3}$ are downloaded in both round $i\Delta$ and round $(i+1)\Delta$. Traditional methods  download the segment only once in round $i\Delta$ based on the FoV prediction at this moment. The benefit of downloading $seg_{i+3}$ in two rounds is that FoV prediction in round $(i+1)\Delta$ is more accurate than in round $i\Delta$. Therefore, the bandwidth can be allocated to the tiles that would be more likely viewed by user.</p>

<p>In our paper, we detailed the formulation of the tile rate allocation problem, the KKT Condition based optimization algorithm design, as well as the proposed view-distance based tile utility model.</p>

<h1 id="selected-results">Selected Results</h1>

<p><img src="images/render_results.jpg" alt="Figure 3: rendered views" width="100%" /></p>


<p>In the above figures, we show the example reandered views of different baselines. The artifacts of "Equal Split" and "Non-Progressive" can be easily observed. Non-progressive result looks weird because it downloads each frame only once when the the frame is about to be download into buffer and the FoV prediction is very inaccurate. The Equal Split equally split the predicted available bandwidth over all the tiles within the predicted FoV of every frame, which fails ti consider the heterogeneity of different tiles in terms of marginal utility. For state-of-art "RUMA", when we zoom in we can see the obvious artifacts around hair, right face and chest as well as left thigh, while there's no obvious artifacts for "KKT-exp" which is the best version of our KKT Condition based progressive streaming strategy. We also show the average PSNR and SSIM results in the table below. We conclude that the proposed KKT-Condition based progressive downloading strategy performs better than the baselines in terms of the projected results, which also proves the rationality of the proposed tile utility model.</p>

<p><em>Table 1: Rendered Quality: Average PSNR and SSIM</em></p>

<table>
  <thead>
    <tr>
      <th><strong>Metrics</strong></th>
      <th><strong>KKT-exp</strong></th>
      <th><strong>RUMA</strong></th>
      <th><strong>Equal Split</strong></th>
      <th><strong>Non-Progressive</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>PSNR</strong></td>
      <td>43.05</td>
      <td>42.89</td>
      <td>42.76</td>
      <td>41.88</td>
    </tr>
    <tr>
      <td><strong>SSIM</strong></td>
      <td>0.9725</td>
      <td>0.9695</td>
      <td>0.9691</td>
      <td>0.9610</td>
    </tr>
  </tbody>
</table>

<h1 id="conclusion">Conclusion</h1>

<p>As volumetric video is on the rise, we explore the design space of point cloud video streaming which is challenging due to its high communication and computation requirements. By relying on the inherent scalability of octree-based point cloud coding, we proposed a novel <i>sliding-window based progressive streaming framework</i> to gradually refine the spatial resolution of each tile as its playback time approaches and FoV prediction accuracy improves. In this way, we managed to balance needs of long streaming buffer for absorbing bandwidth variations and short streaming buffer for accurate FoV prediction. We developed an analytically optimal algorithm based on the Karush–Kuhn–Tucker (KKT) conditions to solve the tile rate allocation problem. We also proposed a novel tile rate-utility model that explicitly considers the viewing distance to better reflect the true user QoE.</p>
