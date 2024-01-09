pipeline {
    
    agent any 
    
    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        
        stage('Checkout'){
           steps {
                git credentialsId: 'jenkins_cicd', 
                url: 'https://github.com/makresh-dev/cicd_django_todo_e2e',
                branch: 'main'
           }
        }

        stage('Build Docker'){
            steps{
                script{
                    sh '''
                    echo 'Buid Docker Image'
                    docker build -t mknnyk/django_todo_cicd:${BUILD_NUMBER} .
                    '''
                }
            }
        }

        stage('Push the artifacts'){
           steps{
                script{
                    withCredentials([usernamePassword(credentialsId: 'docker_hub', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh '''
                    echo 'Push to Repo'
                    docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
                    docker push mknnyk/django_todo_cicd:${BUILD_NUMBER}
                    '''
            }
                }
            }
        }
        
        stage('Checkout K8S manifest SCM'){
            steps {
                git credentialsId: 'jenkins_cicd', 
                url: 'https://github.com/makresh-dev/cicd_django_todo_e2e.git',
                branch: 'main'
            }
        }
        
        stage('Update K8S manifest & push to Repo'){
            steps {
                script{
                    withCredentials([usernamePassword(credentialsId: 'jenkins_cicd', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh '''
                        sed -i "s|mknnyk/django_todo_cicd:[0-9]*|mknnyk/django_todo_cicd:${BUILD_NUMBER}|g" cicd_manifest/deploy.yaml
                        cat cicd_manifest/deploy.yaml
                        git add cicd_manifest/deploy.yaml
                        git commit -m "Updated the deploy yaml | Jenkins Pipeline"
                        git push https://$GIT_USERNAME:$GIT_PASSWORD@github.com/makresh-dev/cicd_django_todo_e2e.git HEAD:main
                        '''                        
                    }
                }
            }
        }
    }
}
