This is my report on my project for the Machine Learning for Physical Sciences course.

# Introduction

Talk about stuff

# Data

To predict the electron number density, my model uses the satellite location and the time history of the SYM-H index as the input features. 

The dataset containing the observed electron density and the location of measurement was obtained through my research group. The data was collected by the Van Allen Probes (RBSP-A and RBSP-B) and covers the time period of 16 September 2012 to 12 October 2019, providing density measurements at 1 min cadence. The measurement location is defined by the L-shell and the magnetic local time (MLT), both available for every data point. 

<p align="center" width="100%">
    <img align="center" width="500" height="220" src="/assets/IMG/satellite_trajectory.png">
</p>

The time history of the SYM-H index was obtained through the [OMNI database](https://omniweb.gsfc.nasa.gov/form/omni_min_def.html). SYM-H data is available at 5 min cadence without any data gaps, which is convenient for this project because I don't lose any data points from unavailability of SYM-H data. 

Prior to any data cleaning, I reduced the density data to 5 min averages so that the time resolution matches that of the SYM-H data.

## Data Cleaning

Data points that meet at least one of the following criteria were removed from the dataset.
1. L > 10
2. Observed density < 0.01 cm^-3

For this project, I set the region of interest to be L < 10. Therefore, density measurements taken farther than L = 10 are removed. The filtering criterion based on the observed density ensures that points with negative, zero, or extremely low densities are removed (for instance, the original dataset contained many data points with extremely low densities at lower L-shells which is not consistent with our understanding of the system) This exact filtering criterion was also used by Bortnik et al. (2016). 

# Modeling

As a model to predict the electron density, I chose to use a feed forward neural network. The input layer takes in the following features as the input:
- 5 hour history of the SYM-H index (at 5 min cadence, resulting in 60 features)
- L
- cos(MLT angle)
- sin(MLT angle)
Number of hidden layers and number of neurons in each of the hidden layers are important hyperparameters of the model. The result of hyperparameter tuning is presented below. I used the RELU activation function for all of the neurons except for the output layer for which I used a linear activation function.

To evaluate the performance of the model, I used 15% of the dataset as a test set. I further split the remaining dataset into validation set and training set (15% and 70% of the overall dataset, respectively). 

In the training process, the model iterated through the entire training set for multiple epochs while adjusting its weight matrices to minimize the RMSE. At the end of epoch, the model was tested on the validation set, and I continued the training process until the RMSE on the validation set stopped improving for 10 consecutive epochs. At the end of the training, the weight matrices that led to the best performance were recovered to avoid overfitting.

## Hyperparameter Tuning

To optimize my model, I explored how the number of hidden layers and the number of neurons in each layer affect the model performance. I used either 1 or 2 hidden layers and varied the N1 and N2, number of neurons in the first layer and second layer, respectively (N2 = 0 if only one hidden layer is used). I tested different combinations of N1 and N2 through grid search and recorded the RMSE obtained with each combination. Below is a 3D surface showing the model performance, and the vertical axis is the RMSE.

<p align="center" width="100%">
    <img align="center" width="350" height="300" src="/assets/IMG/hyperparameter_tuning.png">
</p>

WRITE A SENTENCE OR TWO ABOUT THE TUNING

# Results

<p align="center" width="100%">
    <img align="center" width="700" height="350" src="/assets/IMG/model_performance.png"> 
</p>

The log_10 of the observed density is plotted against the log_10 of the predicted density above for the test set (left) and the training and validation sets combined (right). The solid line is the line of best fit, and the dashed line is where predicted = observed. The equation of the line of best fit, Pearson correlation coefficient, and the RMSE are listed for both sets below.

<p align="center" width="100%">
    <img align="center" width="250" height="40" src="/assets/IMG/test_performance.png">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    <img align="center" width="250" height="40" src="/assets/IMG/training_performance.png">
</p>

In addition to evaluating the model performance on the test set, I selected one storm event (where SYM-H dropped to ~-100nT) and used my model to predict the distribution of electrons over the course of the event. This is important because the purpose of this model is to predict the spatiotemporal density variation of the density during geomagnetic storms. The selected event occurred on 8 March 2008, which is not within the 2012-2019 range of the dataset the model was trained on. Figure below is the change in the SYM-H index throughout the event.

<p align="center" width="100%">
    <img align="center" width="350" height="300" src="/assets/IMG/symh_plot.png">
</p>

2D density maps at various times are presented below. Note that for every plot, the left side of the plot is the day side and the right side is the night side.

<p align="center" width="100%">
    <img align="center" width="950" height="150" src="/assets/IMG/density_distribution.png">
</p>
<p align="center" width="100%">
    <img align="center" width="400" height="50" src="/assets/IMG/colorbar.png">
</p>

# Discussion

Talk about how to interpret RMSE and r squared errors.
Describe how the density distribution is changing, and what the SYM-H is doing at corresponding times

# Conclusion

In this project, I built a feed forward neural network model to reconstruct the electron density in the near-Earth space. My model predicts the electron density at any point in space and time using the satellite location and the 5-hour time history of the SYM-H index as the input features. The model includes an input layer, an output layer, and 2 hidden layers in between them with N1 and N2 neurons each. The model parameters were optimized by testing various layer sizes in the hyperparameter tuning process. Testing the model resulted in an RMSE of RMSE and an r^2 error of R_SQUARED. Finally, I constructed an electron density map during a storm event in March 2008 using my model, and result shows RESULT. 

My model can most likely be improved in several ways if more time was availble for the project. For example, while I optimized the model hyperparameters (number of neurons in the hidden layers), I did not attempt to optimize the feature set. The length of the SYM-H time history was fixed at 5 hours, but including a longer history may potentially capture additional processes that indicate later changes in electron density, in which case the model performance may improve. Therefore the length of the time history can be optimized in the same way the hidden layer sizes were optimized. Furthermore, the model performance could improve by adding time histories of other magnetospheric indices that may be relevant to the feature set. 

Lastly, the techniques used for this project are applicable to many future projects since similar methods can be used in building machine learning models that predict other magnetospheric quantities relevant in space physics research. 

# References

\[1\] Bortnik, Jacob, et al. "A unified approach to inner magnetospheric state prediction." Journal of Geophysical Research: Space Physics 121.3 (2016): 2423-2430.

**Hi class, welcome to the AOS C111/204 final project!** <img align="right" width="220" height="220" src="/assets/IMG/template_logo.png">

For this project, you will be applying your skills to train a machine learning model using real-world data, then publishing a report on your own website.

* To get data for your project, you could:
  * use **your own data** from a separate research activity
  * **scour the internet** to find something original, then preprocess it yourself - see the Module Overview on BruinLearn for some resources
  * browse an archive of data designed for machine learning problems, such as the [UC Irvine Machine Learning Repository](https://archive.ics.uci.edu/datasets)

* Your report should be in the region of 2000-2500 words with three to four figures, and written in a scientific language and style. [This template page](/project.md) gives an example structure that you could use, but feel free to make it your own.

Your website will be a great addition to your CV, and a place to host future projects too since it doubles as a GitHub repository. The first step is to set up a project website like this one by following the instructions below. 

## How does this website work?

First, check out the Github repository for this site: [https://github.com/atmosalex/atmosalex.github.io/](https://github.com/atmosalex/atmosalex.github.io/).

Using GitHub pages, you can write a website using markdown syntax - the same syntax we use to write comments in Google Colab notebooks. GitHub pages then takes the markdown file and renders it as a web page using a Jekyll theme. The markdown source code for this page [is shown here](https://github.com/atmosalex/atmosalex.github.io/blob/main/README.md?plain=1).

## Setting up your Project Website

### How to copy this site as a template
1. Create [a GitHub account](https://github.com/)
2.	Create a new GitHub repository with the name **username.github.io**, where **username** is your GitHub username as shown below. Select *public* and do not tick *Add a README file*. [![screenshot][1]][1]
3.	From your new repository, you should see a *Quick setup* guide. Scroll down to the bottom of the page and click *Import code*, as shown: [![screenshot][2]][2]
4.	In the box that says *Your old repository’s clone URL*, copy and paste this URL: `https://github.com/atmosalex/atmosalex.github.io/`, then proceed.
5.	Go to the *Settings* tab, then click *Pages* (under *Code and automation*), and check that the *Build and deployment* section looks like this: [![screenshot][3]][3]
6.	If you can see the *Actions* tab, click it and check that the build and deployment action has finished. Once it has, navigate to **[username].github.io** to see your site, which should be a copy of this one! If you cannot see an *Actions* tab, just wait a few minutes then go to your URL to check it is live.

Now you are ready to customize your site! To add your name to the site, go to your Github page, edit `_config.yml`, and replace the temporary title with your name.

[1]: /assets/IMG/instr_create.png
[2]: /assets/IMG/instr_import.png
[3]: /assets/IMG/instr_bd.png

### How to change the theme (optional)
1.	You can choose any theme [listed on this page](https://pages.github.com/themes/), though some do not work as well on mobile devices.
2.	From GitHub, edit `_config.yml` and replace the `theme:` line with `theme: jekyll-theme-name` where `name` is the name of the theme from the above list. **For the `minima` theme, use a shortened preface like so `theme: minima`**, the others seem to need the whole preface `theme: jekyll-theme-`. You can check the *Actions* tab (as in step 6. above) to make sure the site is building successfully.

### How to change your site logo (optional)
1. Some themes, such as `jekyll-theme-minimal`, show a logo. In your repository, upload a logo or profile picture to the `assets/IMG/` directory
2. Open `_config.yml` and modify the line `logo: /assets/IMG/template_logo.png` to point to your new image

***

## Guide to Adding Content
* Your repository's `README.md` file (the file you are reading now) acts like a home page. Replace its contents with whatever you want the world to see by editing the file on GitHub.
* If you want to turn this page into a CV or blog, etc., it may be useful to refer to a [guide for writing Markdown](https://www.markdownguide.org/basic-syntax/).
* You can create other markdown files (.md) in your repository and navigate to them from this page using links, i.e.: [here is a link to another file, `project.md`](project.md)
* When editing a markdown file on GitHub, it is useful to wrap text by selecting the *Soft wrap* option as shown: ![screenshot](/assets/IMG/instr_wrap.png)
* If you want to get even more technical, you can also write HTML in your .md files, and GitHub Pages will render it. For example, the image below is displayed by writing the following (edit this file to see!): `<img align="right" width="200" height="200" src="/assets/IMG/template_frog.png">`
<img align="right" width="337" height="200" src="/assets/IMG/template_frog.png"> 

***

## Delivering your Project

Your final project has three components: a dataset, a report, and your code.

### Dataset

A link to the dataset you used must be submitted on BruinLearn so that your course instructor can use it to run your code. To do this, it might be easiest to upload the dataset to Google Drive, then share a link (click *Share* then *Copy link*). If your dataset is too big to upload, try to find a way to reduce its size, or speak with your instructor if this is not possible.

### Report

Your report should be **delivered via your website**. Submit a link to your website on BruinLearn so that your instructor can browse it to find your report. 

To make this simple, you can write the report using a word processor or Latex, then export it as a .pdf file and upload it to the `assets` directory. You can then link to it [like so](/assets/project_demo.pdf). However, you can also type the report directly onto the website using another markdown page - [here is](/project.md) a template for that.

### Code

A link to your code must be submitted on BruinLearn, and the course instructor must be able to download and run your code using the dataset. The code could be in a Google Colab notebook (make sure to *share* the notebook so access is set to **Anyone with the link**), or you could upload the code into a separate GitHub repository, or you could upload the code into the `assets` directory of your website and link to it. 
