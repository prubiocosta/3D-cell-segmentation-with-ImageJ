# 3D-cell-segmentation-with-ImageJ
_**Requirements**_:

This cell segmentation was made using ImageJ (Fiji), version 1.53f51. 
You need to have the **MorphoLibJ** and the ImageJ **3D Suite** Plugins installed. 

To install them:

-**3D Suite**: Add **Java8** and **ImageScience** to the ImageJ updater.

-**MorphoLibJ**: Add the **IJPB-plugins** to the ImageJ updater.

## Introduction
Here, I will show you an ImageJ pipeline for 3D cell segmentation. In this example, we are working on a set of z-stack images composed by three different channel: the first channel, which represents the fluorophore intensity of the protein of interest; the second channel, where you can find the cytoplasm, and the third channel, which represents the nuclei. 

![image](https://user-images.githubusercontent.com/91415505/145820849-c57179aa-5be3-43b0-8aee-024fdca582fb.png)

The goal of this pipeline is to correctly segment the nuclei of the cells, then extend them using a morphological operation to estimate the cytoplasm region and finally measuring and quantifying the intensity of the first channel in both the nuclei and the whole cell. For this purpose, here we will only use the first and third channel. 

### Step 1

Run **Split Channels**. You can rename the first channel as “**Intensity**”, and the third channel as “**Nuclei**”. 

![image](https://user-images.githubusercontent.com/91415505/145821230-af10218c-48ea-4fd6-ab66-3ff42e796af7.png)

### Step 2

**Select the Nuclei window.** 

### Step 3

**Apply a 3D filter**** to reduce noise and smooth the image. On the ImageJ main menu, we will go to **Plugins > 3D > 3D Fast Filters**.  I recommend you to use a **Median Filter**. In our case, we will use the following options. In your case, change the parameters until you are happy with the results.

![image](https://user-images.githubusercontent.com/91415505/145821669-094da6c0-6d25-437f-a887-62f5332d5fdb.png)

### Step 4

Get a **value** for **thresholding**. You need to manually select a pixel value for creating a binary image which delimitates the all the nuclei. For this purpose, we will use the measurement tools within ImageJ. Do as it follows:

**Zoom** to one of the nuclei. Trace a **straight line** that clearly passes the entire nuclei and also the background:

![image](https://user-images.githubusercontent.com/91415505/145822064-6dfdad9c-1779-43da-a271-719f198cc056.png)

In the ImageJ main menu, do **Analyze > Plot profile**. You will get a representation like this:

![image](https://user-images.githubusercontent.com/91415505/145822088-6187ba1b-2af0-4b61-90fe-d157489017aa.png)

With the profile’s plot, you can now **decide which value you will use** for the manual thresholding. In our case, we will use a **value** of **20**, as it seems clearly that everything up to that value is nucleus.

![image](https://user-images.githubusercontent.com/91415505/145822349-cb873034-3ea2-4ae2-8a8f-7a8e0e19d920.png)

### Step 5

**Manual thresholding**. We need to apply the threshold value that we just selected. We will go, in the ImageJ main menu, to **Plugins > 3D > 3D Simple segmentation**. At “Low threshold (Included)” we will write the chosen threshold value. We will leave the other options as they are. When we click in “Ok”, two images will appear: one called “Bin”, which is the binary image where we will work from now on, and another called “Seg”, that we will not use. You can close the “Seg” one and continue with the next step.

![image](https://user-images.githubusercontent.com/91415505/145822614-bea1319e-7942-4d38-bed9-4041e04d8a58.png)

### Step 6

**Opening**. Remember to select the Bin window. We need to apply a **3D morphological operation**, an **Opening** (Erosion + Dilation), for filling small holes and suppressing noise. We go to **Plugins > MorphoLibJ > Morphological Filters (3D)**. We select Opening in “Operation” and Ball in “Element shape”. For the X, Y and Z radius, you can select different values of the X and Y radius according to your images, and choose the one that gives you a better result, but always leave the Z radius in 0. In our case, we use a value of 10 pixels for the X and Y radius. 

![image](https://user-images.githubusercontent.com/91415505/145822798-8a9b509a-762d-49c2-b803-b8382ff1061d.png)

### Step 7

**Duplicate the stack**. Select the Bin-Opening window that you just created. Duplicate the image (duplicate stack). Rename one of the images as “Binary” and the other as “Seeds”. Save the Binary image in your computer as you might need it after.

### Step 8

**Assure that the binary options are correct**. Before continue, you need to check if the binary options are correct, as the next steps could go wrong if not. Go to **Process > Binary > Options…** Check that everything is settled as the following and pass to the next step. The most important feature is that **Black background** is selected.

![image](https://user-images.githubusercontent.com/91415505/145828365-57c3fde3-7280-46ed-8ac9-9f5d3c3ac7d3.png)

### Step 9

**Select the Seeds window.**


### Step 10 

Get the **Ultimate Points**. We need them for the seed creation. Go to **Process > Binary > Ultimate Points**. Choose “**Yes**” when ImageJ asks you if you want to process all the stack. 

### Step 11

Make it **Binary**. We need to assure that the stack is binary. Go to **Process > Binary > Make Binary**. Check that all options are as they follow. 

![image](https://user-images.githubusercontent.com/91415505/145829138-ef2a9818-bd91-43c4-8356-b8a5e1ca1a2b.png)

### Step 12

**Dilation**. We need to apply a **3D morphological operation**, a Dilation, to get a proper seed size. Go to **Plugins > MorphoLibJ > Morphological Filters (3D)**. We select **Dilation** in “Operation” and **Ball** in “Element shape”. In our case, we select a value of 10 pixels in the X and Y radius. This value is appropriate in this stack, but this does not always happen. You should try with different seed sizes (e.g. radius of 5, 20, 30…) to see which one is the optimal for your image stacks. In the Z radius, select 0 pixels. Rename the result as “Final Seeds”. 

![image](https://user-images.githubusercontent.com/91415505/145829433-386e6a1a-8362-4183-a51c-305a8ceb1dcd.png)

### Step 13

**3D Watershed Split**. Now that we have a proper seed and binary stacks, it is time to start the segmentation. We go to **Plugins > 3D > 3D Watershed Split**. We select **Binary** at the “Binary mask” drop-down, and **Final Seeds** at the “Seeds” drop-down. At the “radius” setting, we leave the default value of 2. Two stacks will appear, one called “EDT”, that we will not use (you can close it) and one called “Split”, which is the one where the segmentation has been done. 

![image](https://user-images.githubusercontent.com/91415505/145829686-66f237f0-6d9b-4d1e-b2cd-3359d1a8a291.png)

### Step 14.

**Manual correction and re-segmentation**. Ideally, at this point the segmentation in Split could be perfect and we would continue with the cytoplasm estimation and measurement/quantification, but this is not the most likely scenario. Normally, the Split stack will contain some errors that we should correct manually before going far away. Therefore, this step is optional, and you can repeat it as many times as you want until you are happy with your nuclei segmentation. 

A.	Apply the “Glasbey on dark” Lookup Table. This is not mandatory, but it is useful to evaluate the segmentation, as you can see the objects painted in different colors. 
B.	Manually divide the nuclei that are grouped as the same object. By this time, you can find two kinds of error:
1.	Individual nuclei that are divided in different objects
2.	Two or more nuclei that are classified as the same object

![image](https://user-images.githubusercontent.com/91415505/145830748-541d354a-2744-4247-aacb-32796ba918b6.png)

At the moment, we will just focus on the B errors. The goal of this step is, with the result of the first segmentation, create new seeds and repeat the Step 12. The A type errors, and also the imperfections generated during the watershed, can be corrected once all the nuclei are correctly separated. For dividing the nuclei, you can do as follow:

•	Select the straight line selection tool and make a line that clearly divide the two nuclei through all the stack. Go ahead and back in the Z-slices and ensure that in every frame the line divides both of them. 

![image](https://user-images.githubusercontent.com/91415505/145831026-bf18fca6-6db7-488c-800e-a1293ce271b8.png)

•	Add your selection to the ROI Manager. Go to Analyze > Tools > ROI Manager… and click on add once is opened or simply use the shortcut T to add the selection directly. 

•	 Repeat this steps until you have all the selection for the nuclei that should be divided in the ROI manager. Then go to the next point.

In the ROI manager, click on the Deselect button. Then go to More» > OR (combine). You should now see a selection that encompasses all ROIs. Click the Add button. Then, at the end of the ROI list, you should see a ROI which is the combination of all. Click on that ROI. Then, in the ImageJ main menu, go to Process> Math > Set… Select a value of 0. Click OK and select yes when ImageJ asks you if you want to process all the stack. At this moment, you should have a black line in the places where you drew lines, and all the nuclei individually divided:

![image](https://user-images.githubusercontent.com/91415505/145831178-55023a52-e5ad-4613-8db7-5a63165825c8.png)

Seed creation. Now, you need to use the edited stack that you just created as seeds for segmentation. First, you need to apply an erosion. Select the stack and go to Plugins > MorphoLibJ > Morphological filters (3D). Select Erotion in “Operation” and Ball in “Element shape”. Select 0 pixels in the z radius. In our case, we select a value of 5 pixels in the X and Y radius. Rename the results as “New seeds”. You should have a result that look like this:

![image](https://user-images.githubusercontent.com/91415505/145831261-afb0c09d-ff07-437d-a894-4b33b330d9e0.png)

4.	Re-segmentation. Now, repeat the Step 12 using the Binary image that you saved at the Step 6 as “Binary Mask” and New Seeds as “Seeds”. 

5.	Evaluation. Now you can evaluate your result. If you can’t see any nuclei joined together in one object, you can pass to the next point. If not, repeat this steps until you can’t. 

6.	Final processing. We solved the problems about nuclei that are grouped together, but we still have individual nuclei that are segmented in different objects and small imperfections. To fix that, we need to go to Plugins > MorphoLibJ > Label images > Label Edition. You will open a menu that looks like this: 

![image](https://user-images.githubusercontent.com/91415505/145831471-22604b38-fd97-479c-8fd0-fc1cb1dddbd8.png)

•	For binding labels, select them by click on it and hit the “Merge” button. If you have any wrong point selection and you want to erase them, use the shortcut Shift + A. 

•	Once you have the correct labels (one label per nuclei) click on the “Close” button to delete any watershed errors and save the result by closing the editor and saving the processed image. 

Now, you should have a correct nuclei segmentation stack. It is time to move on to the next step.

![image](https://user-images.githubusercontent.com/91415505/145831684-1990a28f-5cf7-49f6-bf9b-6170df34418e.png)

### Step 15

Nuclei to cell. Once you have the nuclei segmentation stack, is time to estimate the cell objects. For that purpose, we select the final nuclei segmentation stack and go to Plugins > MorphoLibJ > Label Images > Dilate Labels. Here, you need to select a radius that is high enough to fill all the space between the labels. In our case, we select a value of 100 pixels, as it is big enough to fulfill all the empty space between nuclei. In your case, try different values and see which one is big enough to fill this space (but no too much big, as it could make gigantic cells in the slices where there are few cells, just a medium point). Save the result as your cell segmentation stack.

![image](https://user-images.githubusercontent.com/91415505/145831914-c7269872-5de6-4714-a1c5-e38b80e478a6.png)

### Step 16

Measure and quantification. For this purpose, we will use the 3D ROI Manager. Go to Plugins > 3D > 3D Manager. Click on the settings button and select the measurements that you need for your experiment. Once you did it, it is time to start measuring. Open in ImageJ the Intensity stack and the cell and nuclei segmentation ones. Select the nuclei window and click on “Add image” in the 3D Manager. In the list at the left, you will see how the different objects of your stack appear. Once they are added, you just need to select the Intensity stack and click on the “Measure 3D” and “Quantif 3D”. Do the same with the cell stack. Save your results. 

![image](https://user-images.githubusercontent.com/91415505/145832090-2ce5bd2a-c478-42f8-b901-c261a79f2b68.png)

If you have any doubt, error or suggestion, feel free to contact me (Pau) and I will do my best: Email: prcosta@fc.ul.pt




