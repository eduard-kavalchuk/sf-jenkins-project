pipeline {
    agent any

    triggers {
        pollSCM ''
    }

    environment {
        BUILD_URL = 'localhost:9889'
    }

    stages {
        stage('Start Nginx container') {
            steps {
                container_create()
            }
        }

        stage('Get a HTTP response') {
            steps {
                script {
                    def status = sh(
                        script: "curl -s -o /dev/null -w '%{http_code}' ${env.BUILD_URL}",
                        returnStdout: true
                    )
                    echo "Received HTTP status: $status"
                    if (status != '200') {
                        def errorMsg = "Unexpected HTTP status when requesting an index.html page: $status"
                        sendTGNotification(errorMsg)
                        error(errorMsg)
                    }
                }
            }
        }

        stage('Fetch index.html page'){
            steps {
                sh "wget -O $WORKSPACE/html/fetched.html ${env.BUILD_URL}"
            }
        }

        stage('Compare md5 hashes'){
            steps {
                script{
                    def hash_original = sh(
                        script: 'md5sum $WORKSPACE/html/index.html | awk \'{print $1;}\'',
                        returnStdout: true
                    )
                    println "md5 hash of original index.html page: $hash_original"
                    def hash_received = sh(
                        script: 'md5sum $WORKSPACE/html/fetched.html | awk \'{print $1;}\'',
                        returnStdout: true
                    )
                    println "md5 hash index.html page received from nginx: $hash_received"
                    if (hash_original != hash_received) {
                        def errorMsg = "md5 hash of a received page differs from hash of original page!"
                        sendTGNotification(errorMsg)
                        error(errorMsg)
                    }
                }
            }
        }
    }
    post {
        always {
            container_delete()
            sendTGNotification("*${env.JOB_NAME}*: Build ${env.BUILD_ID}: Completed successfully.")
        }
    }
}

def container_create() {
    sh "sudo docker run --name nginx-test --mount type=bind,source=$WORKSPACE/html,target=/usr/share/nginx/html,readonly -p 9889:80 -d nginx"
}

def container_delete() {
    println 'Deleting a container...'
    sh 'sudo docker stop nginx-test'
    sh 'sudo docker rm nginx-test'
}

def sendTGNotification(msg) {
    sh("""
        curl -s -X POST https://api.telegram.org/bot""/sendMessage -d chat_id="" -d text=\"${msg}\"
       """)
}
