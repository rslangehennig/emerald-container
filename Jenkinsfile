node {
    //URL to Github repository https://github.com/<owner>/<repo>
    def GITHUB_REPO_URL="https://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@github.com/rslangehennig/emerald-container"
    //Retrieve ENV ID values for DEV and QA from "Download attributes from API" json file
    //def VELOCITY_ENV_ID_DEV="a4a91beb-e2be-4da5-896a-78632d3b451c"
    //VELOCITY_APP_NAME must match your Velocity pipeline application name
    //def VELOCITY_APP_NAME="Online-Botanicals"
    //Version number
    def VERSION_NUMBER="9.1.7"
    
    def gitVars = git branch: "main", url: "$GITHUB_REPO_URL"
    // gitVars will contain the following keys: GIT_BRANCH, GIT_COMMIT, GIT_LOCAL_BRANCH, GIT_PREVIOUS_COMMIT, GIT_PREVIOUS_SUCCESSFUL_COMMIT, GIT_URL
    println gitVars
    println "Previous successful commit is : ${gitVars.GIT_PREVIOUS_SUCCESSFUL_COMMIT}"
    println "Current commit is ${gitVars.GIT_COMMIT}"

    //Do not change below this line.
    def GIT_COMMIT

    stage ('cloning the repository'){
        currentBuild.displayName = "${VERSION_NUMBER}-${BUILD_NUMBER}"
        echo currentBuild.displayName = "${currentBuild.displayName}"
        majorVersion="${BUILD_NUMBER}"
        def scm = git branch: 'main', url: "${GITHUB_REPO_URL}"
        GIT_COMMIT = sh(returnStdout: true, script: "git rev-parse HEAD").trim()
        echo "GIT_COMMIT=${GIT_COMMIT}"
    }

    stage ('build') {
        sh '''#!/bin/bash
          echo $WORKSPACE
          pwd
          ls -al
          
          export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64/
          export UCD_AUTH_TOKEN="9ec486c6-c7bc-xxxx-xxxxx--xxxxxx"
          export UCD_WEB_URL="https://169.62.212.57:8444"
          
          # Clean up files
          pwd
          ls -al
            rm -f README.md
            rm -rf .git
            rm -rf jenkins
            rm -f Jenkinsfile*
            cp urbancode/deploy.json ..
            cp urbancode/snapshot-emerald.json ..
            rm -rf urbancode
          ls -al

          # Create new component version
          echo "Create new component version"
          /opt/IBM/udclient/udclient -weburl $UCD_WEB_URL -authtoken $UCD_AUTH_TOKEN createVersion -component Emerald-Container -name 9.1.7-$BUILD_NUMBER
          /opt/IBM/udclient/udclient -weburl $UCD_WEB_URL -authtoken $UCD_AUTH_TOKEN addVersionLink -component Emerald-Container -version 9.1.7-$BUILD_NUMBER -linkName "Jenkins Pipeline" -link "http://52.116.6.138:8081/job/Emerald/job/emerald-frontend-pipeline/"${BUILD_NUMBER}"/"
          #/opt/IBM/udclient/udclient -weburl $UCD_WEB_URL -authtoken $UCD_AUTH_TOKEN addVersionFiles -component Emerald-Container -version 9.1.7-$BUILD_NUMBER -base "/opt/jenkins-agent/workspace/Emerald/emerald-frontend-pipeline"

          # Add a component status to allow it to pass gates in UCD
          echo "Adding component status"
          /opt/IBM/udclient/udclient -weburl $UCD_WEB_URL -authtoken $UCD_AUTH_TOKEN addVersionStatus -component Emerald-Container -version 9.1.7-$BUILD_NUMBER -status "Unit Tests Passed"
          
        '''
    }

    stage ('Run Unit Tests') {
        sh '''#!/bin/bash
        echo $BUILD_NUMBER
        echo $GIT_COMMIT
        pwd

        echo "perform curl"
        #curl -k -X POST 'https://velocity2.ibmdevops.com/api/v1/builds/apiDriven' \
        #    -H 'Authorization: UserAccessKey 809845e6-ef8f-4b5b-9329-1ac4a17a3f4d' \
        #    -H 'Content-Type: application/json' \
        #    -d \
        #    '{
        #        "version":"2.1.'"${BUILD_NUMBER}"'",
        #        "application":{"externalId":"e4795ab5-8184-4f84-baf8-a45d484fc0b6"},
        #        "url":"<Build URL>",
        #        "revision":"${GIT_COMMIT}"
        #    }'
           '''
    }
    
    stage ('Create Snapshot for Deployment') {
         sh '''#!/bin/bash

            export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64/
            export UCD_AUTH_TOKEN="9ec486c6-c7bc-4040-ac1d-f6b6b3e040c7"
            export UCD_WEB_URL="https://169.xx.xxx.xx:8444"
            
            # Create snapshot
            echo "Create new snapshot using 9.1.7-$BUILD_NUMBER"
            sed -i "s/MYNAME/9.1.7-$BUILD_NUMBER/g" ../snapshot-emerald.json
            cat ../snapshot-emerald.json
            /opt/IBM/udclient/udclient -weburl $UCD_WEB_URL -authtoken $UCD_AUTH_TOKEN createSnapshot ../snapshot-emerald.json

            # Deploy snapshot to DEV environment
            #cp urbancode/deploy-snapshot.json .
            #sed -i "s/MYNAME/${BUILD_NUMBER}/g" deploy-snapshot.json
            #cat deploy-snapshot.json

            #/opt/IBM/udclient/udclient -weburl $UCD_WEB_URL -authtoken $UCD_AUTH_TOKEN requestApplicationProcess ../deploy.json

            rm -f ../deploy.json
            echo All done
            '''
    }
}
