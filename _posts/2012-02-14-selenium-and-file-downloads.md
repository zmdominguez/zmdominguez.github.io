---
layout: post
title: Selenium and File Downloads
date: '2012-02-14T15:35:00.000+11:00'
author: Zarah Dominguez
tags:
  - selenium
  - JUnit
  - file download
modified_time: '2012-02-14T15:42:54.703+11:00'
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-8440745014661270945
blogger_orig_url: http://www.zdominguez.com/2012/02/selenium-and-file-downloads.html
---

Lately, one of my tasks has been to automate regression tests on one of our apps. Since this is a web app, we are using [Selenium](http://seleniumhq.org/). Here, I enumerate the steps to configure Firefox for file downloading using Selenium and JUnit by foregoing the downloads dialog box.

##### Setting up the profile
We will create a new profile to be used for testing. A profile is simply a set of configuration that Firefox will use when you start it. You can use your existing profile as well, but I opted to create a new one so that the Firefox instance for testing will be as pristine as possible (i.e. no unnecessary plugins, no bookmarks, etc). A note: I am working on a Windows machine.

1. Bring up the command prompt and start the profile chooser by typing in `firefox.exe -ProfileManager -no-remote`
2. Click on `Create Profile`.
3. Enter the profile name you wish to use. I suggest you include the name of your project instead of just naming it "Test".
4. Click on `Choose Folder` and navigate to the folder where you wish to store the profile files. Make sure you can access that folder easily; write the path down because you will need this in JUnit.
5. Click on `Finish`.
6. You will be brought back to the profile chooser dialog. Click on the `Don't ask at start-up checkbox`

Note: To make Firefox use your original configurations when you are browsing, bring up the profile chooser dialog, choose the profile named _default_, and start Firefox.

##### Configuring Firefox
Now we will configure Firefox to behave like how we want it to.

1. From the profile chooser dialog, highlight your test profile and click the `Start Firefox button`.
2. (Optional if Firefox is not your default browser) Uncheck `Always perform this check when starting Firefox`.
3. Click on `Tools > Options`. We will go through the steps for each tab in the `Options` dialog box.
4. General tab
- Under Startup, choose `Show a blank page` from the `When Firefox starts` dropdown.
- Under Downloads, uncheck the `Show the Downloads window when downloading a file` checkbox.
- Still under Downloads, click on `Browse` for the `Save files to` option. Navigate to the folder where you want Firefox to put downloaded files. Take note of this folder as well because we will use this in JUnit.
5. Tabs tab
- Uncheck `Open new windows in a new tab instead` and uncheck all warnings.
6. Content tab
- Uncheck `Block pop-up windows`.
7. (Optional) Privacy tab
- Under History, choose `Never remember history` from the `Firefox will:` dropdown.
8. Advanced tab
- Under the General sub-tab, uncheck `Use autoscrolling` and uncheck `Always check to see if Firefox is the default browser on startup`.
- If you will be using a proxy server, choose the Network sub-tab and input your proxy settings there.

#### Adding MIME Types

Now we need to tell Firefox how to handle downloading each specific type of file.

1. Navigate to the folder where you saved your custom profile and open the `mimeTypes.rdf` file in a text editor.
2. Add the following lines towards the end of the file, right above the closing `</RDF:RDF>` tag.

    ```xml
    <RDF:Seq RDF:about="urn:mimetypes:root">
        <RDF:li RDF:resource="urn:mimetype:text/plain"/>
    </RDF:Seq>
    
    <RDF:Description RDF:about="urn:mimetype:handler:text/plain" NC:alwaysAsk="false" NC:saveToDisk="true">
        <NC:externalApplication RDF:resource="urn:mimetype:externalApplication:text/plain"/>
    </RDF:Description>
    
    <RDF:Description RDF:about="urn:mimetype:text/plain" 
                     NC:value="text/plain" NC:editable="true" NC:fileExtensions="txt" NC:description="Text Document">
        <NC:handlerProp RDF:resource="urn:mimetype:handler:text/plain"/>
    </RDF:Description>
    ```

    What we did there is we told Firefox to directly download files with MIME type `text/plain`. If you need to test downloading other file types like _.doc_ or _.pdf_, you would need to add their MIME types to this file too. 

3. There are two ways to add MIME types to the mimeTypes.rdf file.
- Via the Firefox Applications GUI
  - From Firefox, choose `Tools > Options > Applications`
  - If you have previously downloaded a file of the required MIME type, look for it in the list. In the dropdown menu on the right, under `Action` choose `Save File`.
  - If the MIME type you want is not in the list, you would need to go to a website that allows you to download a sample file. If the file downloads automatically without showing the download pop-up, you would not have to do anything. If the pop-up shows up, activate the `Save File` radio button and check the `Do this automatically for files like this from now on` checkbox and click `Okay`.
- Manually editing the file
  - <span style="color: red;">**Note**: This is generally not the advised way to edit the file due to its complexity. Care is required! </span>
  - Open the `mimeTypes.rdf` file.
  - Look for the `RDF:Seq` node and add your desired MIME type.  
  
        ```xml
          <RDF:Seq RDF:about="urn:mimetypes:root">
            <RDF:li RDF:resource="urn:mimetype:text/plain"/>
            <RDF:li RDF:resource="urn:mimetype:application/pdf"/>
          </RDF:Seq>
        ```
  
      - Add the `RDF: Description` nodes for that MIME type.  

        ```xml
         <RDF:Description RDF:about="urn:mimetype:application/pdf" NC:fileExtensions="pdf" NC:description="Adobe Acrobat Document" NC:value="application/pdf" NC:editable="true">
           <NC:handlerProp RDF:resource="urn:mimetype:handler:application/pdf"/>
         </RDF:Description>
         
         <RDF:Description RDF:about="urn:mimetype:handler:application/pdf" NC:saveToDisk="true" NC:alwaysAsk="false" />
        ``` 

To use this profile in our Selenium test:
```java
FirefoxProfile profile = new FirefoxProfile(new File("path/to/your/profile"));

// We can also set the download folder via code
File testFile = new File("path/to/download/folder");
String downloadFolder = testFile.getAbsolutePath();
profile.setPreference("browser.download.dir", downloadFolder);

WebDriver driver = new FirefoxDriver(profile);
```

And you're done. :)