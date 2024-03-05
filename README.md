# An-Attention-PiDi-UNet-and-Focal-Active-Contour-loss-for-Biomedical-Image-Segmentation
See our [published paper](https://ieeexplore.ieee.org/document/10013852) here.
## Introduction
MRI cardiac segmentation on the [ACDC](https://www.creatis.insa-lyon.fr/Challenge/acdc/databases.html) (3-D images), Skin Lesion segmentation on [ISIC 2018](https://challenge.isic-archive.com/landing/2018/45/) (2-D images), Skin Cancer segmentation on [PH2](https://www.fc.up.pt/addi/ph2%20database.html) (2-D images).
## Our contributions
![Proposed Model](https://github.com/tswizzle141/An-Attention-PiDi-UNet-and-Focal-Active-Contour-loss-for-Biomedical-Image-Segmentation/blob/main/1.jpg)
* Influenced by the ConvMixer layer, we have created a residual operation after the depthwise layer before feature maps are fed into the pointwise layer. Nevertheless, instead of utilizing GeLU activation, we have still used the ReLU activation.
* Starting from the end of each decoder stage, a CDCM is proposed for boundary refinement. A Map Reduce, which is $1 \times 1$ convolutional layer, scales down the feature map into a single channel map for further interpolation and concatenation to the original size of the input image. We have observed that the CDCM output channel could be adjusted among values of the set containing the power of 2 $({1,2,4,8,16})$; the segmentation result of each dataset varies when this value changes.
* Classes imbalance is a common issue in semantic segmentation applications; especially when there exists an imbalanced number of samples between classes, the model performance in generalizing the limited classes degrades. Trinh \textit{et al.} have proposed a loss function based on the active contour model for training their neural network and alleviate the intensity inhomogeneity problems by their unsupervised loss term. However, they have not considered the class imbalance issue in the supervised loss term; hence, we leverage this loss function and add the weight $\alpha_i$ in each class $i$ to the loss function to handle this challenge. Let $\Omega$ be the spatial domain of a prediction mask, $N$ be the number of segmentation classes, and denote $\theta$ be the trainable parameters of the CNN, we have the formula:
![Proposed Loss Function](https://github.com/tswizzle141/An-Attention-PiDi-UNet-and-Focal-Active-Contour-loss-for-Biomedical-Image-Segmentation/blob/main/2.jpg)

where $\mathbf{P_{vi}(\theta)}$ expresses the softmax output of the deep neural network for the $\mathbf{v}^{th}$ pixel value of the class $i^{th}$. $\mathbf{T}$ indicates a one-hot vector of the ground truth, i.e., $\textbf{T}$ consists of $N$ channels, each of which comprises a binary segmentation mask that defines the spatial domain of class $i\in\{1,\ldots,\,N\}$, therefore, $\mathbf{T_{vi}} \in \{0, 1\}$ denotes the label located in the $\mathbf{v}^{th}$ pixel value of the $i^{th}$ class.
* With the additional weight $\alpha_i$, our proposed loss function could balance the sample distribution between classes. Nevertheless, it has not changed the loss function's gradient descent. While the model is trained on the severely imbalanced sample, the dominant class largely influences the gradient descent value. Thus we need a more radical adjustment that escalates the influence of the minority on gradient descent. Inspired by the focal loss and semi-active-contour loss, we propose a new focal active contour loss:
![Proposed Loss Function](https://github.com/tswizzle141/An-Attention-PiDi-UNet-and-Focal-Active-Contour-loss-for-Biomedical-Image-Segmentation/blob/main/2.jpg)

where $\gamma$ is a hyperparameter. Not only adding the weight $\alpha_i$ to $L_{uac}$, we also add the factor $\left ( 1-\mathbf{P_{vi}(\theta)} \right )^\gamma$ and the factor $\left ( \mathbf{P_{vi}(\theta)} \right )^\gamma$ to adjust effectively the effect of labels on the loss function and gradient descent simultaneously. In more detail, in the $i^{th}$ channel of ground truth $\mathbf{T}$, the $v^{th}$ positions in class $i$ have $\mathbf{T_{vi}}=1$, and other classes have $\mathbf{T_{vi}}=0$. In class $i$, we consider the $v^{th}$ position so that $\mathbf{T_{vi}}=1$, this loss becomes $FL(\mathbf{P_{vi}})=-\alpha_i \left ( 1-\mathbf{P_{vi}} \right )^\gamma  log \left (\mathbf{P_{vi}} \right )$.
* This term has properties: For easy-classified regions: intuitively, the training model on the unbalanced sample quickly predicts the majority of samples correctly. The probabilities $\mathbf{P_{vi}}$ of these cases tend to be higher ($\mathbf{P_{vi}}\rightarrow 1$). Hence, the factor $\left ( 1-\mathbf{P_{vi}} \right )^\gamma$  is quite small and does not have a significant influence on the loss function. For hard-classified regions:  $\mathbf{P_{vi}}$ is small. Therefore, its impact on the loss function will be $\left ( 1-\mathbf{P_{vi}} \right )^\gamma$ which closes to 1. This magnitude of impact is many times larger than the easy-classified case. For example, if the case is easy to predict $\mathbf{P_{vi}} = 0.9$ and hard to predict $\mathbf{P_{vi}} = 0.1$, then the difference ratio of the contribution to the loss function when $\gamma =2$ will be: $\frac{\left ( 1-0.1 \right )^2}{\left ( 1-0.9 \right )^2}=81$. Hence, this loss function enhances the importance of correcting hard-classified cases.
![Derivatives](https://github.com/tswizzle141/An-Attention-PiDi-UNet-and-Focal-Active-Contour-loss-for-Biomedical-Image-Segmentation/blob/main/3.jpg)
* The magnitude of the gradient of the focal loss will largely depend on $\left ( 1-\mathbf{P_{vi}} \right )^\gamma$. In easy-classified cases, $\left ( 1-\mathbf{P_{vi}} \right )^\gamma$ tends to be very small thus the effect of gradient descent is negligible. In hard-classified cases, $\left ( 1-\mathbf{P_{vi}} \right )^\gamma$ is closed to 1, the magnitude of gradient descent is many times larger than in the easy-classified cases. On the other hand, when taking the $v^{th}$ positions in other classes so that $\mathbf{T_{vi}}=0$ into consideration, according to the same analysis as above, $L_{wf}=\mathbf{P_{vi}^\gamma(\theta)} log\left ( 1-\mathbf{P_{vi}(\theta)} \right )$ still has the same aforementioned properties. Therefore, our proposed loss function $L_{wf}$ could enable our training model to address the class imbalance problems between each class $i$ and other classes by incrementing the influence of the minority on both the loss function and gradient descent. 

To summary, we propose an active focal loss function for image segmentation by combining two previous loss functions: $L_{ac_focal} = L_{wac}+L_{wf}$
## Results
![table1](https://github.com/tswizzle141/An-Attention-PiDi-UNet-and-Focal-Active-Contour-loss-for-Biomedical-Image-Segmentation/blob/main/4.jpg)
![table2](https://github.com/tswizzle141/An-Attention-PiDi-UNet-and-Focal-Active-Contour-loss-for-Biomedical-Image-Segmentation/blob/main/5.jpg)
![table3](https://github.com/tswizzle141/An-Attention-PiDi-UNet-and-Focal-Active-Contour-loss-for-Biomedical-Image-Segmentation/blob/main/6.jpg)
![table4](https://github.com/tswizzle141/An-Attention-PiDi-UNet-and-Focal-Active-Contour-loss-for-Biomedical-Image-Segmentation/blob/main/7.jpg)
## Citation
If you find our work useful for your research, please cite at:

`@inproceedings{attention_pidiunet_focal_active_contour,
    author={Trinh, Minh-Nhat and Nham, Do-Hai-Ninh and Pham, Van-Truong and Tran, Thi-Thao},
    booktitle={2022 RIVF International Conference on Computing and Communication Technologies (RIVF)}, 
    title= {An Attention-PiDi-UNet and Focal Active Contour loss for Biomedical Image Segmentation},
    year={2022},
    volume={}, number={},
    pages={635-640},
    organization={IEEE},
    doi={10.1109/RIVF55975.2022.10013852}}
`
