#!/usr/bin/env groovy
final NEXUS_URL = '10.9.61.229:8081'

stage('Get code from SCM') {
    node {
        checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'nexus_cred', url: 'https://github.com/craig-br/juice-shop.git']]])
    }
} 
stage('Code Analysis') {
    node {
        def scannerHome = tool 'sonar'
        withSonarQubeEnv('sonar'){
            sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=craig-br_juice-shop -Dsonar.sources=."
        }
    }
}

stage('Deploy Cloud Servers with Tower') {
    node {
        ansibleTower(
            towerServer: 'devsecops_tower',
            templateType: 'workflow',
            jobTemplate: 'DevSecOps EC2 Environment',
            importTowerLogs: true,
            jobTags: '',
            skipJobTags: '',
            limit: '',
            removeColor: false,
            verbose: true,
            credential: '',
        )
    }
}
stage('Deploy') {
    parallel('Deploy App with Tower': { 
        node {
            ansibleTower(
                towerServer: 'devsecops_tower',
                templateType: 'job',
                jobTemplate: 'DevSecOps EC2 Deploy App',
                importTowerLogs: true,
                jobTags: '',
                skipJobTags: '',
                limit: '',
                removeColor: false,
                verbose: true,
                credential: '',
            )
        }
        
        }, 'Configure Load Balancer': {
            node {
                ansibleTower(
                    towerServer: 'devsecops_tower',
                    templateType: 'workflow',
                    jobTemplate: 'DevSecOps Configure Firewall',
                    importTowerLogs: true,
                    jobTags: '',
                    skipJobTags: '',
                    limit: '',
                    removeColor: false,
                    verbose: true,
                    credential: '',
                )
            }
        }
    
    )
}