pipeline {
    agent any

    environment {
        VENV_DIR = 'venv'
        GCP_PROJECT ='project-d45e7cf3-58e8-4f79-bdc'
        GCLOUD_PATH = "/var/jenkins_home/google-cloud-sdk/bin"
        KUBECTL_AUTH_PLUGIN = "/usr/lib/google-cloud-sdk/bin"
    }

    stages{
        stage("Cloning from Github...."){
            steps{
                script{
                    echo 'Cloning from Github...'
                    checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github-token2', url: 'https://github.com/Aakash5937k/Anime-Recommender.git']])
                }
            }
        }

        stage("Making a virtual environment...."){
            steps{
                script{
                    echo 'Making a virtual environment...'
                    sh '''
                    python -m venv ${VENV_DIR}
                    . ${VENV_DIR}/bin/activate
                    pip install --upgrade pip
                    pip install -e .
                    pip install  dvc
                    '''
                }
            }
        }

        stage('DVC Pull'){
            steps{
                withCredentials([file(credentialsId:'gcp-key2' , variable: 'GOOGLE_APPLICATION_CREDENTIALS' )]){
                    script{
                        echo 'DVC Pull....'
                        sh '''
                        . ${VENV_DIR}/bin/activate
                        dvc pull
                        '''
                    }
                }
            }
        }

        stage('Build and Push Image to GCR'){
            steps{
                withCredentials([file(credentialsId:'gcp-key2' , variable: 'GOOGLE_APPLICATION_CREDENTIALS' )]){
                    script{
                        echo 'Build and Push Image to GCR'
                        sh '''
                            set -e
                            export PATH="$PATH:${GCLOUD_PATH}"
                            gcloud auth activate-service-account \
                                --key-file="$GOOGLE_APPLICATION_CREDENTIALS"
                            gcloud config set project "$GCP_PROJECT"
                            gcloud auth configure-docker gcr.io --quiet
                            docker buildx build \
                                --platform linux/amd64 \
                                -t "gcr.io/${GCP_PROJECT}/ml-project:latest" \
                                --push \
                                .
                        '''
                    }
                }
            }
        }
        stage('Deploying to Kubernetes'){
            steps{
                withCredentials([file(credentialsId:'gcp-key2' , variable: 'GOOGLE_APPLICATION_CREDENTIALS' )]){
                    script{
                        echo 'Deploying to Kubernetes'
                        sh '''

                            export PATH="$PATH:${GCLOUD_PATH}"

                            gcloud auth activate-service-account \
                                --key-file="$GOOGLE_APPLICATION_CREDENTIALS"

                            gcloud config set project "$GCP_PROJECT"

                            gcloud container clusters get-credentials ml-app-cluster \
                                --region us-central1 \
                                --project "$GCP_PROJECT"

                            kubectl apply -f deployment.yaml
                        '''
                    }
                }
            }
        }
    }
}