pipeline {
    agent {
        kubernetes {
            yaml """
            apiVersion: v1
            kind: Pod
            metadata:
                name: kaniko
            spec:
                restartPolicy: Never
                volumes:
                - name: kaniko-secret
                  secret:
                    secretName: kaniko-secret
                containers:
                - name: kaniko
                  image: gcr.io/kaniko-project/executor:debug
                  command:
                    - /busybox/cat
                  tty: true
                  volumeMounts:
                  - name: kaniko-secret
                    mountPath: /kaniko/.docker
            """
        }
    }

    stages {
        stage('Cloning the repo') {
            steps {
                script {
                    // Clone the repository
                    git branch: 'main', url: 'https://github.com/amanravi-squareops/inventory-service'
                }
            }
        }
        stage('kaniko build & push') {
            steps {
                container('kaniko') {
                    script {
                        sh '''
                        ls
                        pwd 
                        /kaniko/executor --dockerfile /Dockerfile \
                        --context=$(pwd) \
                        --destination=amanravi12/inventory-service:${BUILD_NUMBER}
                        '''
                    }
                }
            }
        }

        stage('Update values.yaml') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github-cre', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        git branch: 'main', 
                            url: "https://${USERNAME}:${PASSWORD}@github.com/amanravi-squareops/springboot-helm.git"
                    }
                    sh '''
                    cd inventory-service
                    sed -i "s/tag: .*/tag: ${BUILD_NUMBER}/" values.yaml
                    cat values.yaml
                    git config --global user.email "aman.ravi@squareops.com"
                    git config --global user.name "amanravi-squareops"
                    git add values.yaml
                    git commit -m "Update imageTag in values.yaml"
                    git push origin main
                    '''
                }
            }
        }
    }
}
