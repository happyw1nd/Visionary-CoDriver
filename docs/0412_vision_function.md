# A. Road Scene Perception and Processing

**Fig. 3. Typical potential roadside risk caused by pedestrian**

As shown in Figure 3, the input videos come from dash cam installed in vehicles. In the example scene, crossroads and various urban streets represent typical environments of potential risks, with pedestrians possibly ignoring traffic regulations to cross the roads.

To obtain scene information on the road, we designed the following hybrid frame analysis framework:

1. To enable LLM with consistent and continuous reasoning on the image sequence, a Multiple Object Tracking (MOT) backbone model is indispensable. After evaluating various models, we integrated the YoloX with ByteTrack [33], [34] to associate low-confidence detection results for better tracking in crowded pedestrian environments.

   By integrating YoloX with ByteTrack, our system extracts bounding boxes (bboxes) and assigns globally unique IDs to each pedestrian and vehicle in the video stream. The extracted data, including bbox coordinates and unique IDs, are then formatted into JSON, ensuring seamless integration with the HUD interface.

   By analyzing the changes in the same pedestrian's bbox coordinates between different frames, we can anticipate their relative speed to the ego car, categorizing them into ['slow', 'fast'].

2. In complex urban traffic scenarios, determining whether pedestrians are stationed on the roadside is pivotal for intention estimation and risk assessment.

   To acquire more accurate data on the relationship between pedestrians and the road position, we utilized the pixel-level segmentation of Segment Anything Model (SAM) and Grounding DINO [35] to distinguish sidewalks from roadways [36], as Figure 4 and Figure 5 shows.

   **Fig. 4. segmentation result of SAM**

   **Fig. 5. Semantic segmentation of sidewalk and road with SAM and Grounding DINO**

3. To obtain more road scene information from monocular video, we deployed the Dense Prediction Transformer (DPT) [37] model for predicting the depth of each pixel in the current image, as Figure 6 demonstrates.

   Pedestrian distance information, paired with the "safe distance" labels set based on traffic engineering principles, offers more contextual information and warnings to LLM:

   - High Alert Area (0-5 meters): very near
   - Alert Area (5-10 meters): near  
   - Caution Area (10-15 meters): medium
   - Others: far and very far

   Given the distance and roadside masks generated by SAM, we apply a pixel-level algorithm to all of the pedestrians, obtaining their location and the type of surface they are currently on. As depicted in Figure 7, we categorize sidewalks into "left and right" types, and roads into "left, center, right" types. The pedestrian's location is determined by dividing the picture into six sections, which are recorded as lower left, front center, lower right, upper left, far front, and upper right.

   This process converts the pedestrian's bbox into location information that is easy for LLM to understand, thereby reducing the complexity of the input information.

4. Data from various modules is aggregated and translated into an 'Integral scene description' in natural language, which is then fed into the LLM.

**Fig. 6. Dense prediction transformer(DPT) result**

**Fig. 7. Pedestrian location classification**

Through this hybrid frame analysis, the LLM can receive the following types of scenario:

a) The globally unique id of vehicles and pedestrians throughout the video.
b) The bounding box trajectory.
c) The detection confidence.
d) The type of surface upon which the pedestrian is located.
e) The location of pedestrian.
f) The spatial orientation of vehicles and pedestrians, including distance and angle, relative to the ego vehicle.
