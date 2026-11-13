pipeline {
    agent any

    parameters {
        choice(name: 'TARGET_ENV', choices: ['blue', 'green'], description: 'Select environment to deploy and switch live traffic to')
    }

    environment {
        DOCKER_IMAGE = "kp6969/bg-demo-app"
    }

    stages {
        stage('1. Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/kritak0808/k8s-blue-green.git'
            }
        }

        stage('2. Build & Push Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-creds') {
                        def img = docker.build("${DOCKER_IMAGE}:${params.TARGET_ENV}")
                        img.push()
                    }
                }
            }
        }

        stage('3. Deploy Target Environment') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig-id']) {
                    sh 'kubectl --insecure-skip-tls-verify=true apply -f service.yaml'
                    sh "kubectl --insecure-skip-tls-verify=true apply -f ${params.TARGET_ENV}-deployment.yaml"
                    sh "kubectl --insecure-skip-tls-verify=true rollout restart deployment/myapp-${params.TARGET_ENV}"
                    sh "kubectl --insecure-skip-tls-verify=true rollout status deployment/myapp-${params.TARGET_ENV}"
                }
            }
        }

        stage('4. Switch Live Traffic') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig-id']) {
                    sh """kubectl --insecure-skip-tls-verify=true patch service bg-app-service -p '{"spec":{"selector":{"app":"bg-app","version":"${params.TARGET_ENV}"}}}'"""
                }
            }
        }
    }
}
