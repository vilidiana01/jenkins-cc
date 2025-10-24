pipeline {
    agent any
    environment {
    IMAGE_NAME = "casino_game"
    IMAGE_TAG  = "${env.BUILD_NUMBER}"
    }
    stages {
        stage('Build') {
            steps {
                sh 'rm -rf build'
                sh 'cmake -B build -S .'
                sh 'cmake --build build'
            }
        }
        stage('Test') {
            steps {
                sh './build/casino_game'
                sh './build/test_game'
            }
        }
        stage('Deliver') {
            steps {
                sh 'tar -czf casino_game.tar.gz build/casino_game'
                archiveArtifacts artifacts: 'casino_game.tar.gz', fingerprint: true
            }
        }
        stage('Image') {
          steps {
            // se serve: sh 'docker login registry.example.com -u $USER -p $PASS'
            sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
          }
        }
        stage('Run (smoke)') {
          steps {
            // avvia in background, controlla che parta, poi termina
            sh '''
              set -e
              docker rm -f casino_smoke || true
              docker run -d --name casino_smoke $IMAGE_NAME:$IMAGE_TAG
              # attesa breve e log
              sleep 2
              docker logs --tail=50 casino_smoke || true
              # healthcheck "naïve": processo vivo?
              docker ps --filter "name=casino_smoke" --filter "status=running" --format '{{.Names}}' | grep -q casino_smoke
              docker rm -f casino_smoke
            '''
          }
        }
    }
}
