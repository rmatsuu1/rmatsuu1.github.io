This is my report on my project for the Machine Learning for Physical Sciences course. <img align="right" width="220" height="220" src="/assets/IMG/template_logo.png">

# Introduction

The distribution of the energetic charged particles that occupy the near-Earth space constantly changes in response to their interactions with the solar wind. Understanding these dynamics is important since the radiation can be extremely harmful to space-based technological systems our society relies on. While the inner magnetosphere has traditionally been studied using physics-based models, these models are often computationally complex. Furthermore, due to the limited number of spacecraft, it is impossible to take in situ measurements at numerous points in space simultaneously over a long period of time. This makes empirical modeling challenging. Using a machine learning model overcomes many of these challenges and can be helpful in predicting time-dependent distribution of magnetospheric quantities.

In this project, I built a feed forward neural network model to reconstruct the electron density in the near-Earth space, which varies in time. This is a supervised regression problem because my model predicts the density using a large set of density observations available to be used for training. The electron dynamics is very complicated and a linear model would not be able to accurately predict the density profile. Therefore, considering that a large dataset is available, a neural network is an appropriate model for this problem. My goal is to produce a model similar to the one by Bortnik et al. (2016), who have addressed this problem before, and take a step further by optimizing the model parameters. 

# Data

To predict the electron number density, my model uses the satellite location and the time history of the SYM-H index as the input features. 

The dataset containing the observed electron density and the location of measurement was obtained through my research group. The data was collected by the Van Allen Probes (RBSP-A and RBSP-B) and covers the time period of 16 September 2012 to 12 October 2019, providing density measurements at 1 min cadence. The measurement location is defined by the L-shell and the magnetic local time (MLT), both available for every data point. The plot below shows the trajectories of the two spacecrafts between 16 September 2012 and 20 September 2012, colored by the density measured at that location. This shows that in general the density is much higher at lower L.

<p align="center" width="100%">
    <img align="center" width="500" height="220" src="/assets/IMG/satellite_trajectory.png">
</p>

The time history of the SYM-H index was obtained through the [OMNI database](https://omniweb.gsfc.nasa.gov/form/omni_min_def.html). SYM-H index reflects the change in the Earth's magnetic field strength from its normal quiet time value and is commonly used to describe geomagnetic storms in space physics research. SYM-H data is available at 5 min cadence without any data gaps, which is convenient for this project because I don't lose any data points from unavailability of SYM-H data. Many other geomagnetic indices and solar wind parameters are also available on the OMNI database and can be downloaded if I decide to include additional features to my model as an extension to this project.

Prior to any data cleaning, I reduced the density data to 5 min averages so that the time resolution matches that of the SYM-H data.

## Data Cleaning

Datasets often contain data points that do not help with making accurate predictions. For example, there can be measurement errors resulting in data points that are not consistent with the physical understanding of the system. To maximize the performance of my model, I identified and filtered such points. Data points that meet at least one of the following criteria were removed from the dataset.
1. L > 10
2. Observed density < 0.01 cm^-3

For this project, I set the region of interest to be L < 10. Therefore, density measurements taken farther than L = 10 are removed. The filtering criterion based on the observed density ensures that points with negative, zero, or extremely low densities are removed (for instance, the original dataset contained many data points with extremely low densities at lower L-shells which is not consistent with our understanding of the system). This exact filtering criterion was also used by Bortnik et al. (2016). 

# Modeling

As a model to predict the electron density, I chose to use a feed forward neural network. The input layer takes in the following features as the input:
- 5 hour history of the SYM-H index (at 5 min cadence, resulting in 60 features)
- L
- cos(MLT angle)
- sin(MLT angle)

Number of hidden layers and the number of neurons in each of the hidden layers are important hyperparameters of the model and have to be optimized for the particular dataset. The result of hyperparameter tuning is presented below. I used the RELU activation function for all of the neurons except for the output layer for which I used a linear activation function.

To evaluate the performance of the model, I used 15% of the dataset as a test set. I further split the remaining dataset into a validation set and a training set (15% and 70% of the overall dataset, respectively). 

In the training process, the model iterated through the entire training set for multiple epochs while adjusting its weight matrices to minimize the RMSE. The model was tested on the validation set at the end of each epoch, and I continued the training process until the RMSE on the validation set stopped improving for 10 consecutive epochs. At the end of the training, the weight matrices that was used in the epoch with the lowest RMSE were recovered to avoid overfitting.

## Hyperparameter Tuning

To optimize my model, I explored how the number of hidden layers and the number of neurons in each layer affect the model performance. I used either 1 or 2 hidden layers and varied N1 and N2, number of neurons in the first layer and second layer, respectively (N2 = 0 if only one hidden layer is used). I tested different combinations of N1 and N2 through grid search and recorded the RMSE obtained with each combination. Below is a 3D surface showing the model performance.

<p align="center" width="100%">
    <img align="center" width="350" height="300" src="/assets/IMG/hyperparameter_tuning.png">
</p>

With one hidden layer, adding neurons to the layer lowered the RMSE steadily until at N = 80 it started to increase. With two hidden layers, adding more neurons to both layers generally improved the RMSE just like with one layer. However, the RMSE dropped at a slower rate as I added more neurons and the performance eventually stopped improving around when N1 and N2 were 70. As a result my final model contains two hidden layers, both with 70 neurons.

# Results and Discussion

<p align="center" width="100%">
    <img align="center" width="700" height="350" src="/assets/IMG/model_performance.png"> 
</p>

The log_10 of the observed density is plotted against the log_10 of the predicted density above for the test set (left) and the training and validation sets combined (right). The solid line is the line of best fit, and the dashed line is the perfect fit where predicted = observed. The equation of the line of best fit, Pearson correlation coefficient, and the RMSE are listed for both sets below.

<p align="center" width="100%">
    <img align="center" width="250" height="40" src="/assets/IMG/test_performance.png">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    <img align="center" width="250" height="40" src="/assets/IMG/training_performance.png">
</p>

Testing the model on the test set resulted in an RMSE of 0.396 and a correlation of 0.897. While the model performs slightly better on the training data (RMSE of 0.394 and correlation of 0.899), the margin is extremely small and this shows that the model is not overfit. Even though my model performs somewhat well on the test set, I believe that it can certainly be improved (in fact Bortnik et al. achieved a lower RMSE and a higher correlation) and potential changes to my model for improvement are discussed in the conclusion.

In addition to evaluating the model performance on the test set, I selected one storm event (where SYM-H dropped to ~-100nT) and used my model to predict the distribution of electrons over the course of the event. This is important because the purpose of this model is to predict the spatiotemporal density variation of the density during geomagnetic storms. The selected event occurred on 8 March 2008, which is not within the 2012-2019 range of the dataset the model was trained on. In other words, this event period is completely out sample. Figure below is the change in the SYM-H index throughout the event.

<p align="center" width="100%">
    <img align="center" width="360" height="300" src="/assets/IMG/symh_plot.png">
</p>

2D density maps at various times are presented below. Note that for every plot, the left side of the plot is the day side and the right side is the night side (MLT is labeled at 0, 6, 12, and 18). A contour at the density of 50 electrons per cubic centimeter is plotted as a black dashed line for each map to show how the size of the "high density" region changes over time. The result shows that the density is high in a large volume of space at the start of the event, which shrinks as the storm starts. Additionally, the region of high density is extended on the dayside (particularly at 5 am and 10 am on March 9th). Both of these features were also predicted by Bortnik et al. (2016) and is also consistent with the theoretical understanding of the plasmasphere.

<p align="center" width="100%">
    <img align="center" width="950" height="165" src="/assets/IMG/density_distribution.png">
</p>
<p align="center" width="100%">
    <img align="center" width="400" height="50" src="/assets/IMG/colorbar.png">
</p>

# Conclusion

In this project, I built a feed forward neural network model to reconstruct the electron density in the near-Earth space. My model predicts the electron density at any point in space and time using the satellite location and the 5-hour time history of the SYM-H index as the input features. The model includes an input layer, an output layer, and 2 hidden layers in between them with 70 neurons each. The model parameters were optimized by testing various layer sizes in the hyperparameter tuning process. Testing the model resulted in an RMSE of 0.396 and a correlation of 0.897. Finally, I constructed an electron density map during a storm event in March 2008 using my model, and the result shows several features consistent with the physical understanding of plasmasphere. 

My model can most likely be improved in several ways if more time was available for the project. For example, while I optimized the model hyperparameters (number of neurons in the hidden layers), I did not attempt to optimize the feature set. The length of the SYM-H time history was fixed at 5 hours, but including a longer history may potentially capture additional processes that indicate later changes in electron density, in which case the model performance may improve. Therefore the length of the time history can be optimized in the same way the hidden layer sizes were optimized. Furthermore, the model performance could improve by adding time histories of other relevant geomagnetic indices or solar wind parameters to the feature set. 

Even though machine learning proved to be a useful approach for the prediction of electron density, it is important to note that this model can't be used to extrapolate for situations that were not well represented in the training data. Considering this limitation, it is best to use this model together with physics-based models that are better suited for extrapolation.

In the future, this density model can be used to study the inner magnetosphere in different ways. One way is to conduct research on the feature importance after adding more parameters potentially helpful for prediction to the feature set. For example, I can study the contribution of each feature to the density output by computing SHAP value as done by Ma et al. (2023). Then I can potentially identify processes that are most influential in enhancing or reducing the electron density. This is one way these machine learning models can be used do more than simply to predict a particular quantity by aiding in advancing our understanding of the physical system.

Lastly, the techniques used for this project are applicable to research in any scientific field dealing with prediction of quantities that vary with space and time. In space physics reserach in particular, similar methods can be used in building machine learning models that predict other magnetospheric quantities. 

# References

\[1\] Bortnik, Jacob, et al. "A unified approach to inner magnetospheric state prediction." Journal of Geophysical Research: Space Physics 121.3 (2016): 2423-2430.

\[2\]Ma, Donglai, et al. "Opening the black box of the radiation belt machine learning model." Space Weather 21.4 (2023): e2022SW003339.
