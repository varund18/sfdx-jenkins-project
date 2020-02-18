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

    println 'KEY IS'
    println JWT_KEY_CRED_ID
    println HUB_ORG
    println SFDC_HOST
    println CONNECTED_APP_CONSUMER_KEY
    def toolbelt = tool 'toolbelt'

    stage('checkout source') {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }

    withEnv(["HOME=${env.WORKSPACE}"]) {
        withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
            stage('Salesforce Logout Org'){
                if (isUnix()) {
                    rc = sh returnStatus: true, script: "${toolbelt} force:auth:logout --targetusername ${HUB_ORG} -p"
                }else{
                    rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:logout --targetusername ${HUB_ORG} -p"
                }
                if (rc != 0) { error 'Salesforce ORG logout failed' }
            }
            stage('Salesforce Authorize Org'){
                if (isUnix()) {
                    rc = sh returnStatus: true, script: "${toolbelt} force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
                }else{
                    rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${jwt_key_file}\" --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
                }
                if (rc != 0) { error 'Salesforce ORG authorization failed' }
            }
            stage('Convert to MDAPI format'){
                if (isUnix()) {
                    rmsg = sh returnStdout: true, script: "${toolbelt} force:source:convert -d mdapi_convert"
                }else{
                    rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:source:convert -d mdapi_convert"
                }
            }
            stage('Salesforce Deploy Code') {
                if (isUnix()) {
                    rmsg = sh returnStdout: true, script: "${toolbelt} force:mdapi:deploy -d mdapi_convert/. -u ${HUB_ORG}"
                }else{
                    rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:mdapi:deploy -d mdapi_convert/. -u ${HUB_ORG}"
                }

                printf rmsg
                println(rmsg)
            }
        }
    }
}