pipeline {
    agent any

    stages{
        stage("Cloning from Github...."){
            steps{
                script{
                    echo 'Cloning from Github...'
                    checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github-token2', url: 'https://github.com/Aakash5937k/Anime-Recommender.git']])
                }
            }
        }
    }
}