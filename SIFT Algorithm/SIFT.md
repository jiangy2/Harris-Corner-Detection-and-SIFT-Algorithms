This project is an implementation of SIFT(Scale Invariant Feature Transform, including feature point detection and description), 
which extracts distinctive invariant features from images that can be used to perform reliable matching between different views 
of an object or scene.Following are the major stages of generating the set of image features(a division of stages in my code is 
slightly different from Lowe's paper):
    
1.The first stage is to generate a Difference-of-Gaussian Pyramid. The difference-of-Gaussian function provides a close approxi-
mation to the scale-normalized Laplacian of Gaussian whose maxima and minima produce the most stable image features. In this 
stage, we generate the gaussian Pyramid first, then we can get the DoG by repeating the subtraction of two adjacent image scales
in the same octave.

2.After getting the DoG Pyramid, the next step is the local scale-space extrema detection and key point localization. Maxima and
minima of the DoG images are detected by comparing a pixel to its 26 neighbors in 3×3 regions at the current and adjacent scales.
Once a key point candidate has been found by comparing a pixel to its neighbors, what we need to do is fitting these discrete 
points to accurately localize the position and scale of the key point, which also means the determination of the interpolated 
location is needed(using Taylor expansion (up to the quadratic terms) of the scale-space function). The interpolation also pro-
vides a substantial improvement to matching and stability. We also need to check the contrast value and curvature value to remove
some undesired points. The low contrast point needed to be removed since they have very low intensity which means they are very 
sensitive to noise as well. In addition, the edge-like points should be removed too as the DoG images has a strong response to 
image edge points which means they are easy to be unstable influenced by noise. The last step in this stage is recording the 
information that we need of all key points in the next stage.

3.The third stage is orientation assignment. So far, we have achieved the scale invariance by the above stages. The rotation in-
variance can be achieve by assigning a consistent orientation to each key point based on local image properties. For every key 
point, we need to compute the gradient and weight it to assign a direction to it, which is based on a weighted direction histo-
gram in its neighborhood. To be specific, the first step of this stage is to determine some parameters. Considering the key point
as center, We calculate the gradient of all the pixels within the gaussian-weighted circular window whose radius is 3*1.5*σ where
σ is the accurate scale of the key point. All the gradient magnitude within the circular window have different contribution to 
the center, and that's why we need to weight all of them using gaussian filter whose variance is 1.5*σ. After the preparing step,
we start computing the histogram based on the direction. Given a gradient direction, Divide 360° to 36 bins and put the corre-
sponding weighted magnitude into the bins. Hence the histogram is built. In order to prevent the case that a certain gradient 
direction suddenly change due to the noise, a smoothing process is required. Then we are able to select the highest peak value of 
the histogram. If the local peak is at least 80% of the highest peak, we treat its corresponding direction bin as the main direc-
tion of the key point and use a feature to record the accurate loaction and scale. The bin number may not be the exactly max value,so a parabola interpolation is needed to fit the discrete 
direction histogram to obtain the accurate bin number. Finally, the direction where this accurate bin belongs to is 
gotten, which is the orientation we want to assign to the key point.

4.Since above invariance properties is obtained, the last stage of SIFT is generating a descriptor for the local image area that
is highly distinctive yet is as invariant as possible to remaining variations, such as change in illumination or 3D viewpoint. A
SIFT descriptor is a 4*4*8=128 bins histogram sampling in the neighborhood of a feature point. Generating a SIFT has the follow-
ing major steps: The first step of is to define a sampling region. In a gaussian image, the neighborhood to be sampled is divided
into 4x4 regions. There are also 8 bins in gradient direction hence we get the 128 bins in total. The second step is rotating the
coordinates of the descriptor and the gradient orientations relative to the key point orientation to achieve orientation invari-
ance. After previous rotation, we can compute the histogram using a Gaussian weighting function to avoid sudden changes in the 
descriptor with small changes in the position of the window, and to give less emphasis to gradients that are far from the center
of the descriptor. The last step is using tri-linear interpolation to distribute the value of each gradient sample into adjacent
histogram bins for the purpose of avoiding all boundary affects in which the descriptor abruptly changes as a sample shifts 
smoothly from being within one histogram to another or from one orientation to another. To have illumination variations, Normali-
zing the histogram is needed to get rid of the influence introduced by illumination change. Also, a threshold is needed to miti-
gate the influence introduced by this non-linear illumination change since Non-linear change of illumination still exists. To 
make these key points distinctive, we need to normalize the feature vector again.

So far, a SIFT algorithm is completed. Following is the code with detailed comments.
   
Reference paper: Distinctive Image Features from Scale-Invariant Keypoints, DAVID G. LOWE.

@Project author:Yushan Jiang April 2018
