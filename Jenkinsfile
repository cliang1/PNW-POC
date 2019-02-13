#!groovy
import groovy.json.JsonSlurperClassic
node {

    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
    def SFDC_USERNAME

    def HUB_ORG=env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DH

    def toolbelt = tool 'toolbelt'

    stage('checkout source') {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }

    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
    
        stage('Initialize Variables') {
            when {
                branch 'dev' 
            }
            steps {
                echo 'Hello from dev' 
                //HUB_ORG_DH = Jenkins Username or Org Alias
                //CONNECTED_APP_CONSUMER_KEY_DH = Dev Consumer Key
            }
        }
        stage('Initializing Variables') {
            when {
                branch 'test'  
            }
            steps {
                echo 'Hello from test' 
                //HUB_ORG_DH = Jenkins Username or Org Alias
                //CONNECTED_APP_CONSUMER_KEY_DH = test Consumer Key
            }
        }


        stage('Authorize Org') {
            rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${jwt_key_file}\" --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"            
            if (rc != 0) { error 'hub org authorization failed' }            
        }

        //Creates directory called metadataFormat and converts sfdx project source code to metadata format
        stage('Convert Source to Metadata Format') {
            rc = bat returnStatus: true, script: "\"${toolbelt}\" force:source:convert -r ./force-app/ -d ./metadataFormat"            
            if (rc != 0) { error 'convert to metadata format failed' }
        }

        //Deploys from metadataFormat directory to environment
        stage('Deploying') {
            rc = bat returnStatus: true, script: "\"${toolbelt}\" force:mdapi:deploy -d ./metadataFormat -u ${HUB_ORG} -w 5"            
            if (rc != 0) { error 'deployment failed' }
        }     
}