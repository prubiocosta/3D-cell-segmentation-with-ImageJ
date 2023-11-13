# **3D cell segmentation using ImageJ**
## **Overview**
Here, I will show you an ImageJ pipeline for 3D cell segmentation. In this example, we are working on a set of z-stack images composed by three different channel: the **first channel**, which represents the fluorophore **intensity** of the protein of interest; the **second channel**, where you can find a stain that localizes on the **cytoplasm**, and the **third channel**, which is similar to the second channel but with a stain that localizes on the **nucleus**. 

![image](https://user-images.githubusercontent.com/91415505/145820849-c57179aa-5be3-43b0-8aee-024fdca582fb.png)

The goal of this pipeline is to correctly **segment the nuclei**, then extend it using a morphological operation to **estimate the cytoplasm** region and finally **measuring and quantifying the intensity** of the first channel in both the nuclei and the whole cell. For this purpose, here we will only use the first and third channel. 

_**Requirements**_:

This cell segmentation was made using [**ImageJ (Fiji)**](https://imagej.net/software/fiji/), version 1.53f51. 
You need to have the [**MorphoLibJ**](https://imagej.net/plugins/morpholibj) and the [**ImageJ 3D Suite**](https://imagej.net/plugins/3d-imagej-suite/Plugins) installed.

To install them:

- **3D Suite**: Add **Java8** and **ImageScience** to the ImageJ updater.

- **MorphoLibJ**: Add the **IJPB-plugins** to the ImageJ updater.

### **1. Split Channels**

Open and select the [**Original**](https://github.com/prubiocosta/3D-cell-segmentation-with-ImageJ/blob/main/Stacks/Original.tif) stack with ImageJ. Run **Split Channels**. In the ImageJ main menu, go to **_Image > Color > Split Channels_**. Rename the first channel as “**Intensity**”, and the third channel as “**Nuclei**”.
![image](https://user-images.githubusercontent.com/91415505/145821230-af10218c-48ea-4fd6-ab66-3ff42e796af7.png)

### **2. 3D filtering**

 **Select** the **Nuclei** window. Apply a **3D filter** to reduce noise and smooth the image. On the ImageJ main menu, go to **_Plugins > 3D > 3D Fast Filters_**.  I recommend you to use a **Median Filter**. In our case, we will use the following options. In your stacks, change the parameters until you are happy with the results.

![image](https://user-images.githubusercontent.com/91415505/145821669-094da6c0-6d25-437f-a887-62f5332d5fdb.png)

### 3. **Get a thresholding value**

We need to manually select a pixel value for creating a binary image which delimitates the nuclei. For this purpose, we will use the measurement tools within ImageJ. Do as it follows:

- **Zoom** to one of the nuclei. Trace a **straight line** that clearly passes the entire nuclei and also the background:

![image](https://user-images.githubusercontent.com/91415505/145822064-6dfdad9c-1779-43da-a271-719f198cc056.png)

- In the ImageJ main menu, do **_Analyze > Plot profile_**. You will get a representation like this:

![image](https://user-images.githubusercontent.com/91415505/145822088-6187ba1b-2af0-4b61-90fe-d157489017aa.png)

- With the profile’s plot, you can now **decide which value you will use** for the manual thresholding. We will use a **value** of **20**, as it seems clearly that everything up to that value is nucleus.

![image](https://user-images.githubusercontent.com/91415505/145822349-cb873034-3ea2-4ae2-8a8f-7a8e0e19d920.png)

### **4. Manual thresholding**

We need to apply the threshold value that we just selected. Go to **_Plugins > 3D > 3D Simple segmentation_**. At “_Low threshold (Included)_” we put the chosen threshold value. Leave the other options as they are. Click in “_Ok_”, and then, two stacks will appear: one called “**Bin**”, which is the binary one where we will work from now on, and another called “**Seg**”. Close the **Seg** window and continue with the next step.

![image](https://user-images.githubusercontent.com/91415505/145822614-bea1319e-7942-4d38-bed9-4041e04d8a58.png)

### **5. Opening**

Select the **Bin** window. We need to apply a **3D morphological operation**, an **Opening** (Erosion + Dilation), for filling small holes and suppressing noise. Go to **_Plugins > MorphoLibJ > Morphological Filters (3D)_**. Select **Opening** in “_Operation_” and **Ball** in “_Element shape_”. For the X, Y and Z radius, elect different values of the X and Y radius according to your images, and choose the one that gives you a better result, but leave the Z radius in 0. In our case, we use a value of 10 pixels for the X and Y radius. 

![image](https://user-images.githubusercontent.com/91415505/145822798-8a9b509a-762d-49c2-b803-b8382ff1061d.png)

### **6. Duplicate the stack**

Select the **Bin-Opening** window. **Duplicate** the **stack**. Go to **_Image > Duplicate_**. Rename one of the stacks as **Binary** and the other as **Seeds**. **Save** the **Binary** stack in your computer, as you might need it after.

### **7. Assure that the binary settings are correct**

Before continue, we need to check if the binary settings are correct, as the next steps could go wrong if not. Go to **_Process > Binary > Options…_** Check that everything is settled as the following and pass to the next step. The most important feature is that **Black background** is selected.

![image](https://user-images.githubusercontent.com/91415505/145828365-57c3fde3-7280-46ed-8ac9-9f5d3c3ac7d3.png)

### **8. Ultimate Points**

Select the **Seeds** window. Get the **Ultimate Points**. We need them for the seed creation. Go to **_Process > Binary > Ultimate Points_**. Choose “**Yes**” when ImageJ asks you if you want to process all the stack. 

### **9. Make it Binary**

We need to assure that the stack is binary. Go to **_Process > Binary > Make Binary_**. Check that all options are as they follow. 

![image](https://user-images.githubusercontent.com/91415505/145829138-ef2a9818-bd91-43c4-8356-b8a5e1ca1a2b.png)

### **10. Dilation**

We need to apply a **3D morphological operation**, a Dilation, to get a proper seed size. Go to **_Plugins > MorphoLibJ > Morphological Filters (3D)_**. We select **Dilation** in “_Operation_” and **Ball** in “_Element shape_”. In our case, we select a value of 10 pixels in the X and Y radius. This value is appropriate in this stack, but this does not always happen. You should try with different seed sizes (e.g. radius of 5, 20, 30…) to see which one is the optimal for your image stacks. In the Z radius, select 0 pixels. Rename the result as “**Final Seeds**”. 

![image](https://user-images.githubusercontent.com/91415505/145829433-386e6a1a-8362-4183-a51c-305a8ceb1dcd.png)

### **11. 3D Watershed Split**

Now that we have a proper seed and binary stacks, it is time to start the segmentation. Go to **_Plugins > 3D > 3D Watershed Split_**. Select **Binary** at the “_Binary mask_” drop-down, and **Final Seeds** at the “_Seeds_” drop-down. At the “radius” setting, leave the default value of 2. Two stacks will appear, one called **EDT**, that we will not use (you can close it) and one called **Split**, which is the one where the segmentation has been done. 

![image](https://user-images.githubusercontent.com/91415505/145829686-66f237f0-6d9b-4d1e-b2cd-3359d1a8a291.png)

### **12. Manual correction and re-segmentation**

Ideally, at this point the segmentation in Split could be perfect and we would continue with the cytoplasm estimation and measurement/quantification, but this is not the most likely scenario. Normally, the **Split** stack will contain some errors that we should correct manually before going far away. Therefore, this step is optional. Repeat it as many times as you want until you are happy with your nuclei segmentation. 

**A.**	**Apply the “Glasbey on dark” Lookup Table**. 

Go to **_Image > Lookup Tables > glasbey on dark_**. This is not mandatory, but it is useful to evaluate the segmentation, as you can see the objects painted in different colors. 

**B.**	**Manually divide the nuclei that are grouped as the same object**. 

By this time, you can find two kinds of error:

1.	Individual nuclei that are divided in different objects
2.	Two or more nuclei that are classified as the same object

![image](https://user-images.githubusercontent.com/91415505/145835273-a823b3a3-8ae5-4a67-bbbb-49eeb32a1453.png)

At the moment, we will just focus on the 2 errors. The goal of this step is, with the result of the first segmentation, create new seeds and repeat the Step 12. The A type errors, and also the imperfections generated during the watershed, can be corrected once all the nuclei are correctly separated. For dividing the nuclei, do as it follows:

**B.1.** Select the straight line selection tool and **make a line** that clearly divide the two nuclei through all the stack. Go ahead and back in the Z-slices and ensure that in every frame the line divides both of them. 

![image](https://user-images.githubusercontent.com/91415505/145831026-bf18fca6-6db7-488c-800e-a1293ce271b8.png)

**B.2.** **Add your selection** to the ROI Manager. Go to **_Analyze > Tools > ROI Manager…_** and click on add once is opened or simply use the shortcut T to add the selection directly. 

**B.3.** **Repeat** this steps for all the nuclei that should be divided. Add their divisions to the ROI manager. Then go to the next point.

**B.4.** In the ROI manager, click on the **Deselect** button. Then go to **_More» > OR (combine)_**. You should now see a selection that encompasses all ROIs. Click the "_Add_" button. Then, at the end of the ROI list, you should see a ROI which is the combination of all. Click on that ROI. Then, in the ImageJ main menu, go to **_Process> Math > Set…_** Select a value of 0. Click OK and select yes when ImageJ asks you if you want to process all the stack. At this moment, you should have a black line in the places where you drew lines, and all the nuclei individually divided:

![image](https://user-images.githubusercontent.com/91415505/145831178-55023a52-e5ad-4613-8db7-5a63165825c8.png)

**C.** **Seed creation**. Now, you need to use the edited stack that you just created as seeds for segmentation. First, you need to apply an erosion. Select the stack and go to **_Plugins > MorphoLibJ > Morphological filters (3D)_**. Select **Erotion** in “_Operation_” and **Ball** in “_Element shape_”. Select 0 pixels in the z radius. In our case, we select a value of 5 pixels in the X and Y radius. Rename the results as “**New seeds**”. You should have a result that look like this:

![image](https://user-images.githubusercontent.com/91415505/145831261-afb0c09d-ff07-437d-a894-4b33b330d9e0.png)

**D.**	**Re-segmentation**. Now, repeat the Step 12 using the **Binary** image that you saved at the Step 6 as “_Binary Mask_” and **New Seeds** as “_Seeds_”. 

**E.** **Evaluation**. Now you can evaluate your result. If you can’t see any nuclei joined together in one object, you can pass to the next point. If not, repeat this steps until you can’t. 

**F.**	**Final processing**. We solved the problems about nuclei that are grouped together, but we still have individual nuclei that are segmented in different objects and small imperfections. To fix that, go to **_Plugins > MorphoLibJ > Label images > Label Edition_**. You will open a menu that looks like this: 

![image](https://user-images.githubusercontent.com/91415505/145831471-22604b38-fd97-479c-8fd0-fc1cb1dddbd8.png)

- For binding labels, select them by click on them and hit the “_Merge_” button. If you have any wrong point selection and you want to erase them, use the shortcut _**Shift + A**_. 

- Once you have the correct labels (one label per nuclei) click on the “_Close_” button to delete any watershed errors and save the result by closing the editor and saving the processed image. 

Now, you should have a correct [**nuclei segmentation**](https://github.com/prubiocosta/3D-cell-segmentation-with-ImageJ/blob/main/Stacks/Nuclei_segmentation.tif) stack. It is time to move on to the next step.

![image](https://user-images.githubusercontent.com/91415505/145831684-1990a28f-5cf7-49f6-bf9b-6170df34418e.png)

### **13. Nuclei to cell**

Now it is time to estimate the cell objects. For that purpose, select the final nuclei segmentation stack and go to **_Plugins > MorphoLibJ > Label Images > Dilate Labels_**. You need to select a radius that is high enough to fill all the space between the labels. In our case, we select a value of 100 pixels, as it is big enough to fulfill all the empty space between nuclei. In your case, try different values and see which one is big enough to fill this space (but no too much big, as it could make gigantic cells in the slices where there are few cells, just a medium point). Save the result as your [**cell segmentation**](https://github.com/prubiocosta/3D-cell-segmentation-with-ImageJ/blob/main/Stacks/Cell_segmentation.tif) stack.

![image](https://user-images.githubusercontent.com/91415505/145831914-c7269872-5de6-4714-a1c5-e38b80e478a6.png)

### **14. Measure and quantification**

For this purpose, we will use the **3D ROI Manager**. Go to **_Plugins > 3D > 3D Manager_**. Click on the settings button and select the measurements that you need for your experiment. Once you did it, it is time to start measuring. Open in ImageJ the **intensity** stack and the **cell** and **nuclei** segmentation. Select the nuclei window and click on “_Add image_” in the 3D Manager. In the list at the left, you will see how the different objects of your stack appear. Once they are added, you just need to select the **intensity** stack and click on the “_Measure 3D_” and “_Quantif 3D_”. Do the same with the cell stack. **Save your results**. 

![image](https://user-images.githubusercontent.com/91415505/145832090-2ce5bd2a-c478-42f8-b901-c261a79f2b68.png)

If you have any doubt, error or suggestion, feel free to contact me (Pau) and I will do my best: 

Email: paurubiocosta@gmail.com




