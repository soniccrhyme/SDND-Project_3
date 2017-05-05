## **README**
---

### **Behavioral Cloning using a Driving Simulator and Keras**

#### **Victor Roy**

[GitHub Link](https://github.com/soniccrhyme/SDND-Project_3)

---

[//]: # (Image References)

[image1]: ./report_images/model_architecture.png "Model Visualization"
[image2]: ./report_images/center_driving.jpg "Center Lane Driving"
[image3]: ./report_images/right_recovery.jpg "Right-Hand Side Recovery"
[image4]: ./report_images/left_recovery.jpg "Left-Hand Side Recovery"
[image5]: ./report_images/original_image.jpg "Original Image"
[image6]: ./report_images/flipped_image.jpg "Flipped Image"
[image7]: ./report_images/camera_center.jpg "Center Camera View"
[image8]: ./report_images/camera_right.jpg "Right Camera View"
[image9]: ./report_images/camera_left.jpg "Left Camera View"


### Files, Prerequisites, Usage

#### 1. Files

Repository contains the following files:
* model.py containing the script which creates and trains a model
* drive.py for driving the car in autonomous mode
* model.h5 containing a pre-trained convolution neural network (CNN)

#### 2. Prerequisites

Python packages: see Udacity's package requirement list [here](https://github.com/udacity/CarND-Term1-Starter-Kit/blob/master/environment.yml)
The model.h5 file is compatible with Keras 2.0.4
The simulator for your respective OS can be found:
* [Linux](https://d17h27t6h515a5.cloudfront.net/topher/2017/February/58ae46bb_linux-sim/linux-sim.zip)
* [macOS](https://d17h27t6h515a5.cloudfront.net/topher/2017/February/58ae4594_mac-sim.app/mac-sim.app.zip)
* [Windows](https://d17h27t6h515a5.cloudfront.net/topher/2017/February/58ae4419_windows-sim/windows-sim.zip)

#### 3. Execution Instructions
Using the Udacity provided simulator as well as the drive.py and model.h5 provided here, the car can be driven autonomously around the track by executing
```sh
python drive.py model.h5
```
then running the simulator and selecting Autonomous Mode. The current model was designed to work with Track #1 (on the left - the one through the desert).

---

### Model Architecture and Training Strategy

#### 1. Solution Design Approach

The overall strategy for deriving a model architecture was to keep it relatively simple, yet complex enough to deal with the various situations the car might face on the simulator's tracks. I used an architecture derived from NVIDIA's which uses as series of CNN's, fed into a series of fully-connected layers. NVIDIA, afterall, was recently successful of being able to run a self-driving car on camera inputs alone based on a similar architecture ([source](http://images.nvidia.com/content/tegra/automotive/images/2016/solutions/pdf/end-to-end-dl-using-px.pdf)). So this architecture seemed sufficient for this significantly less intensive purpose.

Training data was generated by running the simulator several times, both for whole laps and separately for more the more potentially challenging sections of the track. Data was also generated by simulating recovery situations, where the car, too close to the curb, is steered back toward the road's center.

Training was carried out by splitting the data into training and validation sets and ensuring that the model did not overfit by ensuring that validation loss was monotonically decreasing over all epochs. I initially tried fitting the model without including dropout layers, but the validation loss remained high; dropout layers were subsequently added and successfully combated overfitting (lns 107, 109).

Once the model was trained, it was saved as an hdf file (ln 115). The simulator was run in autonomous mode using the model.h5 file. On the first few occasions, the car did not navigate the track successfully, so I went back and tried to add more data, especially data representing situations vehicle seemed to find challenging. I also added data running the track backwards in hopes that would also further the models ability to generalize (as opposed to simply memorizing the track). This strategy of training data generation, in addition to dataset augmentation (discussed in #3 below) yielded a successful result. At the end of the process, the vehicle was able to drive autonomously around the track without leaving the road (as seen in the included video).

#### 2. Final Model Architecture

The final model architecture (lns 97-113) consists of 5 CNN layers and 4 fully connected NNs. The first 3 CNN layers have increasing filter depths of 24, 32, & 48, using a kernel size of 5x5 and a stride of 2x2. The last two CNNs have filter depths of 64, using a kernel size of 3x3 and a stride of 1x1. All CNN layers include a ReLU activation to allow for nonlinearities.

The four fully-connected NNs have decreasing filter sizes of 100, 50, 10 and, finally, 1 - the last of which yields the steering angle. The first two fully-connected layers feature dropout layers with keep probabilities equal to 0.75. These dropout layers are included in order to encourage the model to redundantly represent important features, thus reducing its propensity toward overfitting.

Furthermore, the model was trained and validated on different sets of simulator runs (in different directions and tracks) - this diversity of training data is also aimed to reduce overfitting. The model was tested by running it through the simulator and ensuring that the vehicle could stay on the track for multiple laps, during different instantiations of the simulator (as to randomize the car's initial state, etc.)

The model used an Adam optimizer, precluding the necessity of manually tuning a learning rate. The number of epochs were chosen based on when both training and validation loss ceased to improve. Mean standard error (MSE) was used as the loss function.

Here is a visualization of the architecture, from NVIDIA's paper on End to End Learning (referenced below):

![Model Architecture][image1]


#### 3. Creation of the Training Set & Training Process

Training data was produced with the goal of keeping the vehicle driving safely within the limits of the road. I used a combination of center lane driving, as well as recovery situations, where the car, located too closely to the curb, was shown to correct itself back toward the center of the road. I also included more data on sections which featured particularly sharp turns, using different angles of approach and different gradients of lateral acceleration. The idea was to feed enough of a variety of situational data so that the model could more easily generalize.

Training set generation strategy needed to account for all likely scenarios in which the car might find itself - these include both best, worst and weird case scenarios.

In the best case scenario, the vehicle would never leave the center of the road. Thus a few laps were completed in the simulator while attempting to keep the car in the center of the road as much as possible.  

![Center Lane Driving][image2]

I then recorded the vehicle recovering from the left side and right sides of the road back to center so that the vehicle would learn to. The following images depict such instances, in which the car is positioned too far to one side:

![Right-Hand Side Recovery][image3]
![Left-Hand Side Recovery][image4]

To augment the data sat, I also flipped images and angles as this would yield twice the amount of data:

![Original Image][image5]
![Flipped Image][image6]

The output from the simulation also gave left- and right-hand side camera angles. I ended up including these images as well, but while adjusting the steering angle by a certain amount of offset. The offset is required because the model assumes the images are from the center camera. As shown below, the right-hand camera would yield an image suggesting the car was too far to the right, so a negative offset was added to the steering angle in this instance; the converse was done for the left-hand camera images.

Left-Hand Side Camera View:

![alt text][image9]

Center Camera View

![alt text][image7]

Right-Hand Side Camera View:

![alt text][image8]

After the collection process, I had ~42,000 left, center and right camera images, yielding ~84,000 total images once their flipped versions were also included.  

Images were trimmed (erasing anything above the horizon, or anything including the hood of the car) and pixel values normalized. The data was then fed into a model architecture inspired by NVIDIA's work, and then trained for 10 epochs. A generator was also implemented so that training the model wouldn't be prohibitively memory intensive.

---

### References

Bojarski et al. April 25, 2016. End to End Learning for Self-Driving Cars. http://images.nvidia.com/content/tegra/automotive/images/2016/solutions/pdf/end-to-end-dl-using-px.pdf
