
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
    stage('Deploy Artifact') {
            steps {
ansibleTower async: false, credential: '', extraVars: '''artifactid: ${artifact_id}
nexus_url: ${localnexus}
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
startup_timeout: ${startup_timeout}''', importTowerLogs: true, importWorkflowChildLogs: false, inventory: '', jobTags: '', jobTemplate: "${env.deployment_job}", jobType: 'run', limit: '', removeColor: false, skipJobTags: '', templateType: 'job', throwExceptionWhenFail: true, towerCredentialsId: 'b199b399-2469-418d-aae1-08ae76d0aa65', towerServer: 'awxweb', verbose: false
            }
        
		}
		}
}
