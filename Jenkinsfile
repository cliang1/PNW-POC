#!groovy
import groovy.json.JsonSlurperClassic
node {

    def SFDC_HOST = env.SFDC_HOST
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID
    def HUB_ORG
    def CONNECTED_APP_CONSUMER_KEY

    def toolbelt = tool 'toolbelt'

    stage('checkout source') {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }

    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
		
         if (env.BRANCH_NAME == 'dev') {
            echo 'I only execute on the dev branch'
            HUB_ORG=env.HUB_ORG_DEV
            CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DEV
        } else if (env.BRANCH_NAME == 'test') {
            echo 'I only execute on the test branch'
            HUB_ORG=env.HUB_ORG_TEST            
            CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_TEST
        } else if (env.BRANCH_NAME == 'stage') {
            echo 'I only execute on the stage branch'
        } else {
            echo 'I only execute if not on the dev, test, or stage branch'
            echo env.BRANCH_NAME
            HUB_ORG=env.HUB_ORG_DEV
            CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DEV
        }
        
        if (env.BRANCH_NAME == 'dev' || env.BRANCH_NAME == 'test' || env.BRANCH_NAME == 'stage') {
            stage('Authorize Org') {
                rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${jwt_key_file}\" --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"            
                if (rc != 0) { error 'hub org authorization failed' }            
            }

            stage('Convert Source to Metadata Format') {
                rc = bat returnStatus: true, script: "\"${toolbelt}\" force:source:convert -r ./force-app/ -d ./metadataFormat"            
                if (rc != 0) { error 'convert to metadata format failed' }
            }

            stage('Deploy to Dev') {
                rc = bat returnStatus: true, script: "\"${toolbelt}\" force:mdapi:deploy -d ./metadataFormat -u ${HUB_ORG} -w 5"            
                if (rc != 0) { error 'deploy to dev failed' }
            }
        }
    }
}