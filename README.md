# Potholes Severity Analysis
 
> This is a draft for implementing the solution, as the project is still a WIP.


This project is an extension on `sm_city` solution, that focuses on providing additional analysis to potholes, such as depth and dimensions.
 

## Technical Description
 
To achieve end-goal following steps need to be implemented:
1. Detect potholes using general-purpose AI model that segments street abnormalities and problems.
2. Extract potholes box boundary from event.
3. Pass-on the pothole segmentation event and its details to AI that is dedicated to analyse the event.
4. Finalise a decision on severity of the pothole that is calculated from values obtained from AI model along side pre-defined conditions.
 
### General-purpose AI Model
 
By evaluating several model architectures, and in terms of model efficiency, performance, and portability, we found out `efficientnet lite` architecture meets the requirements, especially the deployment target is edge devices (Power-constraint IoT devices, work with CPU with Intel XNNPACK if GPU not present, and work with Edge-TPU). With the model architecture decided, next step is to train a model on dataset that covers the street problems.
 
### Potholes Analysis AI Model
 
In 2019, Niantic, Inc., presented [paper to use AI to analyse depths](https://arxiv.org/abs/1806.01260) from input visual assets, and provided [`monodepth2`](https://github.com/nianticlabs/monodepth2) as `PyTorch` implementation of the paper. The model was trained on mono and stereo assets with the goal to create depth data that can be represented as heat-maps if trained on mono (single-angle, single-lens cameras) assets, and for stereo (two-angles, 3D cameras), the model would be able to predict the depth in meters.
 
Testing the pre-trained models against potholes assets proved not to provide meaningful results, as the model training is out-of-scope of this purpose. However, the stereo model could be fine-tuned with additional assets to finally achieve the required results with it. Such task would require following steps:
1. Get 3D camera, such as [OAK-D of OpenCV AI Kit](https://docs.luxonis.com/projects/hardware/en/latest/pages/BW1098OAK.html).
2. Build assets set for potholes for use in depth analysis compatible with datasets used to train models of `monodepth2` project.
3. Fine-tune pre-trained `mono+stereo` model of `monodepth2` with potholes assets.
 
Technically, the dataset is the only challenge as the rest is conventional ML process.
 
Once complete successfully, the model should be able to provide users with expected depths analysis.
 
### Decide on Severity
 
Severity decision is a simple business-rule that can be finalised with following steps:
1. Receive depths values of potholes from potholes analysis model.
2. As the values are showing only values of potholes, since the original event is cropped-down to the pothole segmentation box boundary only, it is safe to assume that lowest depth-value is the distance between spectator (camera) and the pothole sides.
3. Following on #2 assumption, highest depth-value is the lowest point in the pothole from the spectator PoV.
4. By subtracting lowest depth-value from highest we get the difference in depth between pothole sides and its lowest point, which is the pothole depth.
5. By mathematically assuming a triangle between spectator and the pothole sides, using Pythagorean theorem would yield the distance in meters between the two farthest sides of the pothole.
6. Having pothole depth and width (size), a business-rule that classify the pothole severity could by executed on the values to finally decide the pothole severity.
 

## Execution
 
The challenge of executing this solution is in creating the required dataset for potholes analysis model. Theoretically, this is straightforward solution based on proven work (`monodepth2`) and require simple additional tasks (general-purpose street problems model, potholes stereo dataset, business-rules) around it.
 
This solution doesn't build-in the process of the business-rule (deciding the severity using AI) leaving the room to tuning the business-rule to the requirements of every city or municipality, rather than baking this in it and making it come with rigid decisions that doesn't fit all.
 
While the solution is practical, it needs more time and effort to be executed and limited-timeframe doesn't allow such execution.
