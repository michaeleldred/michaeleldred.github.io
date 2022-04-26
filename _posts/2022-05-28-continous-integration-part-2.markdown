---
layout: post
title:  "Mobile Continuous Integration Using Godot and Azure Devops, Part 2: iOS Deploy"
blurb: "Build and Deploy a Godot app to iOS"
og_image: /assets/header2.jpg
date:   2022-05-28 19:34:57 -0600
categories: ci cd azure devops godot gamedev ios
---
<img src="{{ "/assets/header2.jpg" | absolute_url }}" alt="bay" class="post-pic"/>

## Why
One of the biggest headaches for my <a href="/infinibreak/">First Project</a> was managing the releases for iOS and Android. Manually building and uploading the files to both of the stores was a error prone process with many manual steps. This is why I wanted to build an automatic release process for my latest project.

Although I own an Apple computer required to deploy to iOS from my own machine, having continued access to one for building all of the time can be a problem as they are expensive (though worth it in my opinion). Azure Devops provides access to macOS machines in the cloud for building, so if you can get access to an Apple computer once to set all this up, this will run automatically for about a year, with no need of access to an apple computer.

Note: this will not necessarily be a tutorial on exporting to iOS, but I will try and cover it the best I can. The official guide for exporting to iOS can be found <a href="https://docs.godotengine.org/en/stable/tutorials/export/exporting_for_ios.html">here</a>.

## Local Setup
# iOS Project Setup
The first step to releasing an app to the Apple App store is to sign up for an <a href="https://developer.apple.com">Apple Developer account.</a> After that is set up, go to the <a href="https://developer.apple.com/account/#!/membership/">membership page</a> and record the Team ID for use later in the project

Once the developer account is setup, go to <a href="http://appstoreconnect.apple.com/">App Store Connect</a> and create a new app. As part of that you will need to create a bundle ID. Save the bundle id for adding to the godot project later.

Next, you will need to create the iconset. I find using an icon generator such as <a href="https://appicon.co">this one</a> simplifies the process of generating the different sizes of icons. Save those icons for later as well.


We will be starting with the project created in my <a href="/2022/03/continuous-integration-part-1">Last Post</a>, so if you haven't followed that tutorial, I would encourage doing that before starting with iOS.


# Godot Export
For the project to run, it needs a main scene. I just created one called Application.tscn in <code>source</code> and set the main scene in settings

Before setting up the export, I will create a couple of folders to store the icons generated previously and the ios build. For the build, I will copy the format of the android build and create a <code>ios/build</code> folder to hold the XCode Project. Create a file called <code>.gdignore</code> in the ios folder, telling Godot to ignore this folder for the purpose of working in the editor. I will also add an icons folder to build_data and move the ios icons generated previously into that folder. The final folder structure should look something like this:

<img src="/assets/ci2-1screenshot.png" />

After that, the next step is creating the export settings for iOS. Open up the export dialog by selecting <code>Project > Export</code>. In that dialog, click <code>Add...</code> and then in the dialog select iOS. After that, fill in the Team ID and icons you generated before. Finally choose an identifier for your app, for example <code>io.itch.michaelreldred.cdtestproject</code>. Once you have the project setup, set the export path to <code>ios/build/&lt;Project Name&gt;</code> and then Export the project with the default settings. After that is run you will have a project that is ready to export. We will use the project name later ##TODO

# YAML
With the project setup, we can now go through the changes to the yaml build file. In my case, I have two separate pipelines, one for unit tests, and another that runs all of the build actions. So create a copy of the run-tests.yml file called master-build.yml and make the modifications there. As before, I will post the whole file and then go through each section to explain it's purpose:

```
trigger:
  branches: 
    include: 
      - '*'
    exclude:
      - master

pool:
  vmImage: 'macOS-10.15'

variables:
  - name: godot_version
    value: '3.3.3'
  - name: export_name
    value: 'CDTestProject'

steps:
  - checkout: self

  - task: InstallAppleCertificate@2
    inputs:
      certSecureFile: 'michaelreldred.p12'
      certPwd: $(P12Password)
  
  - task: InstallAppleProvisioningProfile@1
    inputs:
      provProfileSecureFile: '$(export_name).mobileprovision'

  - task: CmdLine@2
    displayName: 'Download Godot'
    inputs:
      script: |
        wget https://downloads.tuxfamily.org/godotengine/$(godot_version)/Godot_v$(godot_version)-stable_osx.universal.zip -O $(Agent.TempDirectory)/godot.zip
        wget https://downloads.tuxfamily.org/godotengine/$(godot_version)/Godot_v$(godot_version)-stable_export_templates.tpz -O $(Agent.TempDirectory)/templates.tpz
        mkdir -p ~/Library/Application\ Support/Godot/templates/$(godot_version).stable
        unzip -j $(Agent.TempDirectory)/templates.tpz -d  ~/Library/Application\ Support/Godot/templates/$(godot_version).stable  

  - task: ExtractFiles@1
    displayName: "Extract Godot executable files"
    inputs:
      archiveFilePatterns: '$(Agent.TempDirectory)/godot.zip'
      destinationFolder: '$(Agent.ToolsDirectory)'


  - task: CmdLine@2
    displayName: 'Run Tests'
    continueOnError: true
    inputs:
      script: '$(Agent.ToolsDirectory)/Godot.app/Contents/MacOS/Godot addons/WAT/cli.tscn run=all --verbose'


  - task: CmdLine@2
    displayName: 'Export iOS from Godot'
    inputs:
      script: |
        $(Agent.ToolsDirectory)/Godot.app/Contents/MacOS/Godot --export-pack "iOS" ios/$(export_name).pck --no-window
  

  - task: CmdLine@2
    displayName: 'Update iOS Version'
    inputs:
      script: |
        echo 'Setting iOS version to $(Build.BuildId)'
        /usr/libexec/PlistBuddy -c "Set :CFBundleVersion $(Build.BuildId)" "$(Build.SourcesDirectory)/ios/$(export_name)/$(export_name)-Info.plist"
        /usr/libexec/PlistBuddy -c "Set :CFBundleShortVersionString $(Build.BuildId)" "$(Build.SourcesDirectory)/ios/$(export_name)/$(export_name)-Info.plist"
        cat "$(Build.SourcesDirectory)/ios/$(export_name)/$(export_name)-Info.plist"


  - task: Xcode@5
    inputs:
      actions: 'build'
      scheme: '$(export_name)'
      sdk: 'iphoneos'
      configuration: 'Release'
      xcWorkspacePath: '$(Build.SourcesDirectory)/ios/$(export_name).xcodeproj/project.xcworkspace'
      xcodeVersion: 'default'
      signingOption: 'manual'
      signingIdentity: '$(APPLE_CERTIFICATE_SIGNING_IDENTITY)'
      provisioningProfileUuid: '$(APPLE_PROV_PROFILE_UUID)'
      useXcpretty: false
      exportPath: $(Build.ArtifactStagingDirectory)
      packageApp: true


  - task: PublishTestResults@2
    inputs:
      testResultsFormat: 'JUnit'
      testResultsFiles: 'results.xml' 
  

  - task: PublishPipelineArtifact@1
    inputs:
      targetPath: '$(Build.ArtifactStagingDirectory)'
      artifact: '$(export_name)_drop'
      publishLocation: 'pipeline'
```

Now to go over the new code:
```
- task: InstallAppleCertificate@2
    inputs:
      certSecureFile: 'michaelreldred.p12'
      certPwd: $(P12Password)
  
  - task: InstallAppleProvisioningProfile@1
    inputs:
      provProfileSecureFile: '$(export_name).mobileprovision'
```
These install certificate files required to sign the generated file. I'll go over how to get these in the pipelines step.
```
- task: CmdLine@2
    displayName: 'Export iOS from Godot'
    inputs:
      script: |
        $(Agent.ToolsDirectory)/Godot.app/Contents/MacOS/Godot --export-pack "iOS" ios/$(export_name).pck --no-window
```

Export a pck file from godot. The only part of the xcode project that needs updating is the pck data file generated by Godot, so add that file to ios/build

```
- task: CmdLine@2
    displayName: 'Update iOS Version'
    inputs:
      script: |
        echo 'Setting iOS version to $(Build.BuildId)'
        /usr/libexec/PlistBuddy -c "Set :CFBundleVersion $(Build.BuildId)" "$(Build.SourcesDirectory)/ios/$(export_name)/$(export_name)-Info.plist"
        /usr/libexec/PlistBuddy -c "Set :CFBundleShortVersionString $(Build.BuildId)" "$(Build.SourcesDirectory)/ios/$(export_name)/$(export_name)-Info.plist"
        cat "$(Build.SourcesDirectory)/ios/$(export_name)/$(export_name)-Info.plist"
```

Update the build version of the ios file using the buildid of the pipeline. Each time a build is uploaded to the app store, a new, higher, version number is required to provide the latesst version to users.

```
  - task: Xcode@5
    inputs:
      actions: 'build'
      scheme: '$(export_name)'
      sdk: 'iphoneos'
      configuration: 'Release'
      xcWorkspacePath: '$(Build.SourcesDirectory)/ios/$(export_name).xcodeproj/project.xcworkspace'
      xcodeVersion: 'default'
      signingOption: 'manual'
      signingIdentity: '$(APPLE_CERTIFICATE_SIGNING_IDENTITY)'
      provisioningProfileUuid: '$(APPLE_PROV_PROFILE_UUID)'
      useXcpretty: false
      exportPath: $(Build.ArtifactStagingDirectory)
      packageApp: true
```
Build the iOS app using Xcode, it generates a ipa file in the Artifact staging directory that will be published in the next step.

```
  - task: PublishPipelineArtifact@1
    inputs:
      targetPath: '$(Build.ArtifactStagingDirectory)'
      artifact: '$(export_name)_drop'
      publishLocation: 'pipeline'
```

Publish the files so that they can be uploaded to the app store in a separate step

# .gitignore

Add this section to the .gitignore file, these are build files generated by xcode that will cause problems in the pipeline.
``` 
# iOS Files
**.ipa
**/*.xcarchive
**/project.xcworkspace/xcshareddata/
**/project.xcworkspace/xcuserdata/
```

## Azure Pipeline Setup
# iOS Certificate Files
You will need two certificate files to build the iOS app. You will only need to generate one p12 file for all your projects, I use <A href="https://support.magplus.com/hc/en-us/articles/203808748-iOS-Creating-a-Distribution-Certificate-and-p12-File">this guide</a> to generate the p12 certificate file. For each mobile project you do, you will need a mobileprovision file, I used <a href="https://support.attendify.com/hc/en-us/articles/4409331246228-How-to-Export-a-Mobile-Provision-Certificate-">this guide to generate that file, using the bundle id created earlier in the process</a> 

Once you have those files, add them to the azure devops using the secure files functionality. Go to <code>Pipelines > Library</code> and then open the Secure Files tab. Upload both the p12 and mobileprovision file as a secure file.

# Pipeline Modifications
Create a new pipeline in Azure Devops using the master-build.yml file. Update the file to change the p12 and mobileprovision file names to use the secure file names.

After that you can run the pipeline. The first time you do it will have to provide access to the secure files to the pipeline, after that it will run normally.

## Releases Setup
The release part is fairly simple, in the pipeline menu go the the release page. Create a new release pipeline, using the empty job. Add your new master-build pipeline as the Artifact and go to the first state. I choose to rename the stage to testing. This is what mine looks like

<img src="/assets/ci2-2screenshot.png" />

After that, go into the job and add a "Apple App Store Release" task to the 