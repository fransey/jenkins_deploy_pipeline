
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
startup_timeout: ${startup_timeout}''', importTowerLogs: true, importWorkflowChildLogs: false, inventory: '', jobTags: '', jobTemplate: 'deploy_demo-app_test', jobType: 'run', limit: '', removeColor: false, skipJobTags: '', templateType: 'job', throwExceptionWhenFail: true, towerCredentialsId: '1b7eb453-b70c-4778-a2a9-fcbf6cf2bec0', towerServer: 'awx', verbose: false
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
startup_timeout: ${startup_timeout}''', importTowerLogs: true, importWorkflowChildLogs: false, inventory: '', jobTags: '', jobTemplate: 'deploy_demo-app_test', jobType: 'run', limit: '', removeColor: false, skipJobTags: '', templateType: 'job', throwExceptionWhenFail: true, towerCredentialsId: '1b7eb453-b70c-4778-a2a9-fcbf6cf2bec0', towerServer: 'awx', verbose: false
            }
        
		}
		}
}
