## Building and Uploading your Custom UI to Automate
**Summary:** 
Now that you have a local developed custom UI, you'll need to build it into a package and upload it to your Custom-UI configuration within your process in Automate in order for your intended audience to see it.


**Package the UI**
1. In Terminal, navigate to the root level of your local custom UI.
2. Use the following command to build and package your UI: 
```
npm run pack-build workspace-hxp
```
3. Once this command is complete, a .zip file for this UI will be placed within the following directory: ```dist/```.
   - **NOTE:** The maximum UI file size allowed to be uploaded is 10MB, so if your .zip file exceeds this then you'll receive an error on the next step. The likely issue causing the file size to be too large might be that the images you used for your HTML page are too large. If this is the case, use an application that has a save-for-web feature (like Photoshop) or try and reduce the file size of the image(s). 
4. In Automate, go into **Studio Modelling** and open the process that you created this Custom UI from. On the left hand panel, toggle down the **UI** header and select your custom UI to open it's configuration. Use the blue **Upload** button to upload the .zip archive created in Step 3, confirming replacement when prompted to do so. (Use the following screenshot as a guide):
![alt text](images/upload-ui.jpeg "Upload Custom UI.")
5. Save the process, release, then navigate to **Studio Admin** and Upgrade the project. Test all is working by launching the custom UI name instead of the Workspace UI when Upgrade is complete.

--- 
