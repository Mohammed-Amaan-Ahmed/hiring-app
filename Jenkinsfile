pipeline {
    agent any

    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Prepare Artifact') {
            steps {
                // Try to download the war file, or use 'mvn package' if you’re building it locally
                sh '''
                    echo "Downloading WAR..."
                    # Download WAR from your repo - replace with actual credentials if needed
                    curl --fail --retry 3 -O http://54.157.190.6:8081/repository/hiring-app/junit/hiring/0.1/hiring-0.1.war
                    if [ ! -f hiring-0.1.war ]; then
                      echo "WAR file not found! Aborting...";
                      exit 1
                    fi
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build . -t sabair0509/hiring-app:${BUILD_NUMBER}"
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([string(credentialsId: 'docker-hub', variable: 'hubPwd')]) {
                    sh "docker login -u sabair0509 -p ${hubPwd}"
                    sh "docker push sabair0509/hiring-app:${BUILD_NUMBER}"
                }
            }
        }

        stage('Checkout K8S manifest SCM'){
            steps {
                git branch: 'main', url: 'https://github.com/betawins/Hiring-app-argocd.git'
            }
        } 

        stage('Update K8S manifest & push to Repo'){
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'Github_server', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) { 
                        sh """
                        cat /var/lib/jenkins/workspace/\$JOB_NAME/dev/deployment.yaml
                        sed -i "s/5/${BUILD_NUMBER}/g" /var/lib/jenkins/workspace/\$JOB_NAME/dev/deployment.yaml
                        cat /var/lib/jenkins/workspace/\$JOB_NAME/dev/deployment.yaml
                        git add .
                        git commit -m 'Updated the deploy yaml | Jenkins Pipeline'
                        git remote -v
                        git push https://\$GIT_USERNAME:\$GIT_PASSWORD@github.com/betawins/Hiring-app-argocd.git main
                        """
                    }
                }
            }   
        }
    }
}
