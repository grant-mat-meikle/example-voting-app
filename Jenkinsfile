pipeline {
    agent none

    stages {
        stage('vote_build') {
            agent{
                docker{
                    image 'python:2.7.16-slim'
                    args '--user root'
                }
            }
            when{
                changeset "**/vote/**"
            }
            steps {
                echo 'Compile vote app'
                dir('vote'){
                    sh 'pip install -r requirements.txt'
                } 
            }
        }
        stage('vote_test') {
            agent{
                docker{
                    image 'python:2.7.16-slim'
                    args '--user root'
                }
            }
            when{
                changeset "**/vote/**"
            }
            steps {
                echo 'Running unit tests on vote app'
                dir('vote'){
                    sh 'pip install -r requirements.txt'
                    sh 'nosetests -v'
                }
            }
        }
        stage('vote_docker-package') {
            agent any
            when{
                branch 'master'
                changeset "**/vote/**"
            }
            steps {
                echo 'Packaging vote app image with docker'
                script{
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerLogin') {
                        def voteImage = docker.build("xs98zprt/vote:v${env.BUILD_ID}","./vote")
                        /* Push the container to the custom Registry */
                        voteImage.push()
                        voteImage.push("${env.BRANCH_NAME}")
                    }
                }
            }
        }
        stage('worker_build') {
            agent{
                docker{
                    image 'maven:3.6.1-jdk-8-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when{
                changeset "**/worker/**"
            }
            steps {
                echo 'Compile worker app'
                dir('worker'){
                    sh 'mvn compile'
                } 
            }
        }
        stage('worker_test') {
            agent{
                docker{
                    image 'maven:3.6.1-jdk-8-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when{
                changeset "**/worker/**"
            }
            steps {
                echo 'Running unit tests on worker app'
                dir('worker'){
                    sh 'mvn clean test'
                }
            }
        }
        stage('worker_package') {
            agent{
                docker{
                    image 'maven:3.6.1-jdk-8-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when{
                branch 'master'
                changeset "**/worker/**"
            }
            steps {
                echo 'Packaging worker app'
                dir('worker'){
                    sh 'mvn package -DskipTests'
                    archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                }
            }
        }
        stage('worker_docker-package') {
            agent any
            when{
                branch 'master'
                changeset "**/worker/**"
            }
            steps {
                echo 'Packaging worker app image with docker'
                script{
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerLogin') {
                        def workerImage = docker.build("xs98zprt/worker:v${env.BUILD_ID}","./worker")
                        /* Push the container to the custom Registry */
                        workerImage.push()
                        workerImage.push("${env.BRANCH_NAME}")
                    }
                }
            }
        }
        stage('result_build') {
            agent{
                docker{
                    image 'node:8.16.0-alpine'
                }
            }
            when{
                changeset "**/result/**"
            }
            steps {
                echo 'Compile result app'
                dir('result'){
                    sh 'npm install'
                } 
            }
        }
        stage('result_test') {
            agent{
                docker{
                    image 'node:8.16.0-alpine'
                }
            }
            when{
                changeset "**/result/**"
            }
            steps {
                echo 'Running unit tests on result app'
                dir('result'){
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }
        stage('result_docker-package') {
            agent any
            when{
                branch 'master'
                changeset "**/result/**"
            }
            steps {
                echo 'Packaging result app image with docker'
                script{
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerLogin') {
                        def resultImage = docker.build("xs98zprt/result:v${env.BUILD_ID}","./result")
                        /* Push the container to the custom Registry */
                        resultImage.push()
                        resultImage.push("${env.BRANCH_NAME}")
                    }
                }
            }
        }
    }

    post{
        always{
            echo "Pipeline for instavote is complete.."
        }
        failure{
            echo "Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        }
        success{
            echo "Build Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        }
    }
}