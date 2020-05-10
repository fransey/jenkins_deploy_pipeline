import hudson.model.*;
import java.util.regex.Matcher;
import jenkins.util.*;
import jenkins.model.*;
import jenkins.scm.*;
import groovy.json.*;

pipeline {
    environment {
       GROOVY_HOME = tool name: 'groovy 2.4.6', type: 'hudson.plugins.groovy.GroovyInstallation'
    }
    agent {
        node {
            label 'master'
        }
    }
    stages {
        stage('Validate Artifact Version') {
            steps {
                script {
                    checkArtifactExistsInNexus("maven-public","${env.group_id}","${env.artifact_id}","${version}","${env.NEXUS_API}")
                }
            }
        }
        stage('Validate Version on SCC') {
            steps {
                script {
                    def sccResponse = "";
                    def get = new URL("${env.SCC_API}/${env.artifact_id}/${env.springboot_profile}/${ConfigVersion}").openConnection();
                    def getRC = get.getResponseCode();
                    
                    if(getRC.equals(200)) {
                        sccResponse = get.getInputStream().getText();
                        env.application_port = extractDataRegex(sccResponse,/("server.port":"{0,1})(\d{1,5})/,2);
                        
                        if(env.application_port.toString().isInteger()) {
                            println("Application Port retreived from SCC, Port: " + env.application_port);
                        }
                        else {
                            throw new RuntimeException("ERROR: Invalid Application Port returned by SCC, Please verify the configuration is correct on Gitlab.")
                        }
                    }
                    else {
                        throw new RuntimeException("ERROR: Invalid Response returned by SCC, Response Code: " + getRC + ". Please verify the version provided is correct.")    
                    }
                }
            }
        }
        stage('Pre-Deployment Checks') {
            steps {
ansibleTower async: false, credential: '', extraVars: '''artifactid: ${artifact_id}
nexus_url: ${NEXUS_API}
groupid: ${group_id}
springboot_deploy_folder: ${springboot_deploy_dir}
springboot_group: ${springboot_group}
springboot_user: ${springboot_user}
role: pre-deployment-checks
version: ${version}
java_dir: ${java_dir}
config_version: ${ConfigVersion}
springboot_profile: ${springboot_profile}
deployment_host: ${deployment_host}
application_port: ${application_port}
startup_timeout: ${startup_timeout}''', importTowerLogs: true, importWorkflowChildLogs: false, inventory: '', jobTags: '', jobTemplate: "${env.deployment_job}", jobType: 'run', limit: '', removeColor: false, skipJobTags: '', templateType: 'job', throwExceptionWhenFail: true, towerCredentialsId: 'b199b399-2469-418d-aae1-08ae76d0aa65', towerServer: 'tstawx', verbose: false
            }
        }
        stage('Deploy Artifact') {
            steps {
ansibleTower async: false, credential: '', extraVars: '''artifactid: ${artifact_id}
nexus_url: ${NEXUS_API}
groupid: ${group_id}
springboot_deploy_folder: ${springboot_deploy_dir}
springboot_group: ${springboot_group}
springboot_user: ${springboot_user}
role: maven-deployment
version: ${version}
java_dir: ${java_dir}
config_version: ${ConfigVersion}
springboot_profile: ${springboot_profile}
deployment_host: ${deployment_host}
application_port: ${application_port}
startup_timeout: ${startup_timeout}''', importTowerLogs: true, importWorkflowChildLogs: false, inventory: '', jobTags: '', jobTemplate: "${env.deployment_job}", jobType: 'run', limit: '', removeColor: false, skipJobTags: '', templateType: 'job', throwExceptionWhenFail: true, towerCredentialsId: 'b199b399-2469-418d-aae1-08ae76d0aa65', towerServer: 'tstawx', verbose: false
            }
        }
        stage('Configure and Start App') {
            steps {
                catchError (stageResult: 'FAILURE') {
ansibleTower async: false, credential: '', extraVars: '''artifactid: ${artifact_id}
nexus_url: ${NEXUS_API}
groupid: ${group_id}
springboot_deploy_folder: ${springboot_deploy_dir}
springboot_group: ${springboot_group}
springboot_user: ${springboot_user}
role: systemd-configure-start-app
version: ${version}
java_dir: ${java_dir}
config_version: ${ConfigVersion}
springboot_profile: ${springboot_profile}
deployment_host: ${deployment_host}
application_port: ${application_port}
startup_timeout: ${startup_timeout}''', importTowerLogs: true, importWorkflowChildLogs: false, inventory: '', jobTags: '', jobTemplate: "${env.deployment_job}", jobType: 'run', limit: '', removeColor: false, skipJobTags: '', templateType: 'job', throwExceptionWhenFail: true, towerCredentialsId: 'b199b399-2469-418d-aae1-08ae76d0aa65', towerServer: 'tstawx', verbose: false
                }
            }
        }
        stage('Health Checks') {
            when { 
                expression { currentBuild.result != 'FAILURE' } 
            }
            steps {
                catchError (stageResult: 'FAILURE') {
ansibleTower async: false, credential: '', extraVars: '''artifactid: ${artifact_id}
nexus_url: ${NEXUS_API}
groupid: ${group_id}
springboot_deploy_folder: ${springboot_deploy_dir}
springboot_group: ${springboot_group}
springboot_user: ${springboot_user}
role: healthcheck
version: ${version}
java_dir: ${java_dir}
config_version: ${ConfigVersion}
springboot_profile: ${springboot_profile}
deployment_host: ${deployment_host}
application_port: ${application_port}
startup_timeout: ${startup_timeout}''', importTowerLogs: true, importWorkflowChildLogs: false, inventory: '', jobTags: '', jobTemplate: "${env.deployment_job}", jobType: 'run', limit: '', removeColor: false, skipJobTags: '', templateType: 'job', throwExceptionWhenFail: true, towerCredentialsId: 'b199b399-2469-418d-aae1-08ae76d0aa65', towerServer: 'tstawx', verbose: false
                }
            }
        }
        stage('Verify Test Suite Exists') {
            when { 
                expression { currentBuild.result != 'FAILURE' } 
            }
            steps {
                script {
                    if (jenkins.model.Jenkins.instance.getItem("pipeline-demoapp") != null) {
                        env.executeTests = 'true'
                        println("Test Suite job found, executeTests: " + env.executeTests);
                    }
                    else
                    { 
                        env.executeTests = 'false'
                        println("Test Suite job not found, executeTests: " + env.executeTests);
                    }
                }
            }
        }
        stage('Execute Test Suite') {
            when { 
                expression { currentBuild.result != 'FAILURE' && env.executeTests == 'true' } 
            }
            steps {
                catchError (stageResult: 'FAILURE') {
                    build job: 'pipeline-demoapp', parameters: [[$class: 'StringParameterValue', name: 'source_branch', value: 'develop']]
                }
            }
        }
        stage('Rollback') {
            when { 
                expression { currentBuild.result == 'FAILURE' }
            }
            steps {
ansibleTower async: false, credential: '', extraVars: '''artifactid: ${artifact_id}
nexus_url: ${NEXUS_API}
groupid: ${group_id}
springboot_deploy_folder: ${springboot_deploy_dir}
springboot_group: ${springboot_group}
springboot_user: ${springboot_user}
role: rollback
version: ${version}
java_dir: ${java_dir}
config_version: ${ConfigVersion}
springboot_profile: ${springboot_profile}
deployment_host: ${deployment_host}
application_port: ${application_port}
startup_timeout: ${startup_timeout}''', importTowerLogs: true, importWorkflowChildLogs: false, inventory: '', jobTags: '', jobTemplate: "${env.deployment_job}", jobType: 'run', limit: '', removeColor: false, skipJobTags: '', templateType: 'job', throwExceptionWhenFail: true, towerCredentialsId: 'b199b399-2469-418d-aae1-08ae76d0aa65', towerServer: 'tstawx', verbose: false
            }
        }
        stage('Rollback - Health Checks') {
            when { 
                expression { currentBuild.result == 'FAILURE' } 
            }
            steps {
                catchError (stageResult: 'FAILURE') {
ansibleTower async: false, credential: '', extraVars: '''artifactid: ${artifact_id}
nexus_url: ${NEXUS_API}
groupid: ${group_id}
springboot_deploy_folder: ${springboot_deploy_dir}
springboot_group: ${springboot_group}
springboot_user: ${springboot_user}
role: healthcheck
version: ${version}
java_dir: ${java_dir}
config_version: ${ConfigVersion}
springboot_profile: ${springboot_profile}
deployment_host: ${deployment_host}
application_port: ${application_port}
startup_timeout: ${startup_timeout}''', importTowerLogs: true, importWorkflowChildLogs: false, inventory: '', jobTags: '', jobTemplate: "${env.deployment_job}", jobType: 'run', limit: '', removeColor: false, skipJobTags: '', templateType: 'job', throwExceptionWhenFail: true, towerCredentialsId: 'b199b399-2469-418d-aae1-08ae76d0aa65', towerServer: 'tstawx', verbose: false
                }
            }
        }
        stage('Cleanup') {
            when { 
                expression { currentBuild.result != 'FAILURE' }
            }
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
ansibleTower async: false, credential: '', extraVars: '''artifactid: ${artifact_id}
nexus_url: ${NEXUS_API}
groupid: ${group_id}
springboot_deploy_folder: ${springboot_deploy_dir}
springboot_group: ${springboot_group}
springboot_user: ${springboot_user}
role: cleanup
version: ${version}
java_dir: ${java_dir}
config_version: ${ConfigVersion}
springboot_profile: ${springboot_profile}
deployment_host: ${deployment_host}
application_port: ${application_port}
startup_timeout: ${startup_timeout}''', importTowerLogs: true, importWorkflowChildLogs: false, inventory: '', jobTags: '', jobTemplate: "${env.deployment_job}", jobType: 'run', limit: '', removeColor: false, skipJobTags: '', templateType: 'job', throwExceptionWhenFail: true, towerCredentialsId: 'b199b399-2469-418d-aae1-08ae76d0aa65', towerServer: 'tstawx', verbose: false
                }
            }
        }
    }
}

void checkArtifactExistsInNexus(repo,groupid,artifactid,version,nexusapi) {
	println("[checkArtifactExistsInNexus] - paramRepository Value: " + repo);
	println("[checkArtifactExistsInNexus] - paramGroupIdStr Value: " + groupid);
    println("[checkArtifactExistsInNexus] - paramArtifactIdStr Value: " + artifactid);
	println("[checkArtifactExistsInNexus] - paramVersion Value: " + version);
	println("[checkArtifactExistsInNexus] - Nexus API URL Value: " + nexusapi);
	
	println("[checkArtifactExistsInNexus] - building nexus query and connection end point.");
	println("[checkArtifactExistsInNexus] - URL: $nexus_api/service/rest/v1/search?sort=version&direction=desc&repository=$repo&group=$groupid&name=$artifactid&version=$version");
	
	def connection = new URL("$nexus_api/service/rest/v1/search?sort=version&direction=desc&repository=$repo&group=$groupid&name=$artifactid&maven.baseVersion=$version").openConnection()
	connection.setRequestProperty( 'User-Agent', 'groovy-2.4.4' )
	connection.setRequestProperty( 'Accept', 'application/json' )

	if ( connection.responseCode == 200 ) {
		println("[checkArtifactExistsInNexus] - connected and received a valid response from Nexus. Response code: " + connection.responseCode.toString());
		// get the JSON response
		def response = connection.content.text;
		println("[checkArtifactExistsInNexus] - parsing JSON response.");
		def json = new JsonSlurper().parseText( response );
		println("[checkArtifactExistsInNexus] - response parsed: " + json);

		// Check if version exists in Nexus
		println("[checkArtifactExistsInNexus] - checking the response contains one or more artifact metadata.");
		if(json."items".isEmpty()) {
			throw new RuntimeException("ERROR: The Artifact is not on Nexus, please verify the version provided is correct.")
		}
		else {
			println("[checkArtifactExistsInNexus] - The artifact exists in Nexus, proceeding with SCC config check.");
		}
	} else {
		throw new RuntimeException("ERROR: Invalid response code: " + connection.responseCode + ", response: " + connection.inputStream.text);
	}
}

def extractDataRegex(String data, String regex, int group) {
    println("[extractDataRegex] -  data" + data);
	println("[extractDataRegex] -  regex" + regex);     
    def match = (data =~ regex)     
    if (match.find()) {         
        return match.group(group);     
        }     
        return data;   
}