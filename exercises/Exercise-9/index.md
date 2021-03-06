# Exercise 9 - Putting it all together
    
In this lab, you will build a complete CI/CD pipeline utilizing a shared function library and declaring a reusable delivery pipeline passing in input parameters.  

You can view the shared function library here:
<https://github.com/Vizuri/openshift-cicd-pipeline/blob/master/src/com/vizuri/openshift/PipelineSteps.groovy>

You can view the reusable delivery pipeline here:
<https://github.com/Vizuri/openshift-cicd-pipeline/blob/master/src/com/vizuri/openshift/JavaDeliveryPipeline.groovy>

This reusable pipeline supports a Feature, Develop and Release Branch.

   * If the branch starts with `feature/...`, a featue pipeline is executed. This just builds and performs code analysis of the feature branch code. 

    <img src="../images/image15.png" alt="image15" width="50%">

   * If the branch is develop, a development pipeline is executed.
     This builds the code, performs code analysis, pushes the build artifact to Nexus, 
     builds a container, pushes it to a Quay registry, deploys the container to an OpenShift 
     development project and then runs integration tests on the deployed code.

    <img src="../images/Image-101.png" alt="Image-101" width="50%">

   * If the branch starts with release/..., a release pipeline is executed.
     This builds the code, performs code analysis, pushes the build artifact to Nexus, 
     builds a container, pushes it to a Quay registry, prompts the user to confirm 
     deployment to test, deploys the container to an OpenShift test project, 
     runs integration tests against test, prompts the user to confirm a deployment to production, 
     deploys the container to production and then runs integration tests against production.

    <img src="../images/Image-102.png" alt="Image-102" width="50%">

## Test Shared Library

* Backup your Jenkinsfile to Jenkinsfile.BAC.
* Update the Jenkins File by replacing everything with the following contents:

```

    #!/usr/bin/groovy
    
    @Library('github.com/vizuri/openshift-cicd-pipeline@master')
    
    def javaDeliveryPipeline = new com.vizuri.openshift.JavaDeliveryPipeline();
    
    
    javaDeliveryPipeline {
        ocpAppSuffix = 'apps.ocpws.kee.vizuri.com'
        imageNamespace = 'student_{{ student_number }}';
        registryUsername = 'student-{{ student_number }}'
        imageBase = 'quay.{{ ocp_app_suffix }}'
        registryUsername = 'student-{{ student_number }}'
        registryPassword = '{{ student_pwd }}'
        app_name = 'customer'
        ocp_dev_cluster = 'ocp-ws'
        ocp_dev_project = 'student-{{ student_number }}-customer-dev'
        ocp_test_cluster = 'ocp-ws'
        ocp_test_project = 'student-{{ student_number }}-customer-test'
        ocp_prod_cluster = 'ocp-ws'
        ocp_prod_project = 'student-{{ student_number }}-customer-prod'
    }

```

* Rebuild Project by returning to the customer-service develop job and trigger a build.

### Test Feature Branch
Create a new feature branch in the Gogs Repository and test it in Jenkins:

   * To create a new branch you have to create a new file (call it *dummy.txt*).
   * Add the text *dummy* inside the file
        
    <img src="../images/create_dummy_file.png" alt="create_dummy_file" width="30%">
     
   * Before you click on the *Commit Changes* button, change the default commmit option 
     from *Commit directly ...* to *Create a new branch ...*
           
   * Enter the branch name `feature/Feature-1`. 
       
    <img src="../images/new_branch.png" alt="new_branch" width="30%">
    
   * Click on the *Commit Changes* button
     >IMPORTANT: At this point do not perform a pull request yet
     
   * From the Jenkins console, go to the *custom-service* project and click on the *Scan Multibranch Pipeline Now* option on the left. 
     This will scan the project for new branches and kick off the feature branch build.  

     >Note that in practice a webhook should be configured to automatically trigger the build when a new branch is created. 

    <img src="../images/scan_multibranch_pipeline.png" alt="scan_multibranch_pipeline" width="15%">

   * You should have a new job under *custom-service* called `feature/Feature1`
   
    <img src="../images/feature_build.png" alt="feature_build" width="50%">
   
   
### Test Develop Branch
Create a Pull Request and merge the Feature Branch into the Develop Branch. 
   
   * Go back to Gogs and now complete the pull request:
    
    <img src="../images/pull_request.png" alt="pull_request" width="30%">  

   * Now merge the pull request back into the *develop* branch:
   
    <img src="../images/merge_pull_request.png" alt="merge_pull_request" width="30%">

   * After the merge is complete you should see this:
   
    <img src="../images/merge_complete.png" alt="merge_complete" width="30%">
   
   * Go back to Jenkins and click on *Scan Multibranch Pipeline Now* link on the left.  
     This will trigger the develop branch build again.

### Release Develop Branch
Create a release branch called release/1.0 from the develop branch. 

   * To create a new branch you have to create a new file (call it *dummy_release.txt*).
   * Add the text *dummy release* inside the file
    
    <img src="../images/create_dummy_release.png" alt="create_dummy_release" width="30%">
        
   * Before you click on the *Commit Changes* button, change the default commmit option 
        from *Commit directly ...* to *Create a new branch ...*
              
   * Enter the branch name `release/1.0`. 
   
    <img src="../images/new_release_branch.png" alt="new_release_branch" width="30%"> 
 
   * Click on the *Commit Changes* button
     >IMPORTANT: At this point do not perform a pull request yet
     
   * Go to Jenkins and click on *Scan Multibranch Pipeline Now* button.  
     This will trigger a develop and release branch build.  
   
   * When prompted choose "Proceed" to deploy container to test and production environments.
 
### Complete the Pull Request of the release branch into the develop branch   
   * Go back to Gogs and now complete the pull request:
       
    <img src="../images/release_pull_request.png" alt="pull_request" width="30%">  
   
   * Now merge the pull request back into the *develop* branch:
      
    <img src="../images/merge_release_pull_request.png" alt="merge_release_pull_request" width="30%"> 

   * Cick on *Scan Multibranch Pipeline Now* button in Jenkins.
     This will trigger a develop and release branch build.
          
    <img src="../images/release_build.png" alt="release_build" width="30%"> 
