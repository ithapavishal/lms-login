pipeline {
    agent any

    environment {
        dockerImage = "thapavishal/elearning"
    }

    stages {
        stage('Checkout from GitHub') {
            steps {
                git url: 'https://github.com/ithapavishal/lms-login.git', branch: 'jenkins-pipeline'
            }
        }
        stage('Build Django App') {
            agent {
                label 'ubuntu-pipeline-slave-node'
            }
            steps {
                sh 'docker build -t $dockerImage:$BUILD_NUMBER .'
            }
        }

        stage('Push Image') {
            agent {
                label 'ubuntu-pipeline-slave-node'
            }
            steps {
                withDockerRegistry([credentialsId: 'dockerhub-credentials', url: '' ]) {
                    sh '''
                    docker push $dockerImage:$BUILD_NUMBER
                    '''
                }
            }
        }

        stage('Deploy to Development Env') {
            agent {
                label 'ubuntu-pipeline-slave-node'
            }
            // steps {
            //     echo 'Running app on development env'
            //     sh '''
            //     docker stop elearningdev || true
            //     docker rm elearningdev || true
            //     docker run -itd --name elearningdev -p 8000:8000 $dockerImage:$BUILD_NUMBER
            //     '''
            // }
            steps {
                echo 'Deploying to development environment using Ansible'
                sh '''
                ansible-playbook -i inventory deploy.yaml --extra-vars "env=dev image_tag=$dockerImage:$BUILD_NUMBER"
                '''
           
            }
        }
        stage('Deploy to Production Env') {
            agent {
                label 'ubuntu-pipeline-slave-node'
            }
            // steps {
            //     timeout(time: 1, unit: 'DAYS') {
            //         input id: 'confirm', message: 'Approve deployment to production environment?'
            //     }
            //     echo "Running app on prod env"
            //     sh '''
            //     docker stop elearningprod || true
            //     docker rm elearningprod || true
            //     docker run -itd --name elearningprod -p 8001:8000 $dockerImage:$BUILD_NUMBER
            //     '''
            // }

            steps {
                timeout(time: 1, unit: 'DAYS') {
                    input id: 'confirm', message: 'Approve deployment to production environment?'
                }
                echo 'Deploying to production environment using Ansible'
                sh '''
                ansible-playbook -i inventory deploy.yaml --extra-vars "env=prod image_tag=$dockerImage:$BUILD_NUMBER"
                '''

            }
        }
    }
    post {
        always {
            mail to: 'v01.thapa@gmail.com',
            subject: "Job '${JOB_NAME}' (${BUILD_NUMBER}) status",
            body: "Please go to ${BUILD_URL} and verify the build"
        }
        success {
            mail bcc: '', body: """Hi Team,
            Build #$BUILD_NUMBER is successful, please go through the url
            $BUILD_URL
            and verify the details.
            Regards,
            Devops Team""", cc: '', from: '', replyTo: '', subject: 'BUILD SUCCESS NOTIFICATION', to: 'v01.thapa@gmail.com'
        }
        failure {
            mail bcc: '', body: """Hi Team,
            Build #$BUILD_NUMBER is unsuccessful, please go through the url
            $BUILD_URL
            and verify the details.
            Regards,
            Devops Team""", cc: '', from: '', replyTo: '', subject: 'BUILD FAILED NOTIFICATION', to: 'v01.thapa@gmail.com'
        }
    }
}