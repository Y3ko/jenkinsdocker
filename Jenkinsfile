pipeline {
    agent none // Dış agent tanımı kullanmamak için none kullanıyoruz.

    environment {
        DOCKER_IMAGE = 'y3ko/jenkins:test3'
    }

    stages {
        stage('Build and Push Docker Image') {
            agent {
                kubernetes {
                    // Docker-in-Docker için bir pod şablonu tanımlıyoruz
                    defaultContainer 'jnlp'
                    yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    some-label: docker-builder
spec:
  containers:
  - name: docker
    image: docker:19.03.12-dind
    env:
    - name: DOCKER_HOST
      value: 192.168.1.120:2375
    securityContext:
      privileged: true
    volumeMounts:
    - name: docker-graph-storage
      mountPath: /var/lib/docker
  volumes:
  - name: docker-graph-storage
    emptyDir: {}
"""
                }
            }
            steps {
                container('docker') {
                    script {
                        // Docker imajını derle.
                        sh "docker build -t ${env.DOCKER_IMAGE} ."
                        // Docker Registry'ye giriş yap ve imajı push et.
                        withDockerRegistry([ credentialsId: "dockerhub", url: "" ]) {
                            sh "docker push ${env.DOCKER_IMAGE}"
                        }
                    }
                }
            }
        }
    }
}
