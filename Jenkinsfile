pipeline {
  agent {
    kubernetes {
      label 'gke-agent'
      defaultContainer 'gke-agent'
      yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    jenkins: gke-agent
spec:
  securityContext:
    fsGroup: 1000
  serviceAccountName: jenkins
  containers:
    - name: jnlp
      image: jenkins/inbound-agent:latest-jdk17
      args: ['\$(JENKINS_SECRET)','\$(JENKINS_NAME)']
      volumeMounts:
        - name: workspace-volume
          mountPath: /home/jenkins/agent
      resources:
        limits:
          memory: 512Mi
    - name: gke-agent
      image: docker.io/lavi324/gke_agent:1.0
      imagePullPolicy: IfNotPresent
      command: ['cat']
      tty: true
      volumeMounts:
        - name: workspace-volume
          mountPath: /home/jenkins/agent
  volumes:
    - name: workspace-volume
      emptyDir: {}
"""
    }
  }

  options {
    skipDefaultCheckout()
  }

  environment {
    GIT_CREDENTIALS_ID    = 'github'
    DOCKER_CREDENTIALS_ID = 'dockerhub'
    USER_EMAIL            = 'lavialduby@gmail.com'
    DOCKER_REPO           = 'lavi324/public1-frontend'
    HELM_REPO             = 'oci://lavi324/public1-frontend-helm-chart'
    CHART_NAME            = 'public1-frontend-helm-chart'
  }

  stages {
    stage('Checkout & Bump Versions') {
      steps {
        container('gke-agent') {
          dir("${WORKSPACE}") {
            // 1. Clone repo into shared volume
            checkout scm

            // 2. Increment both tags
            sh 'chmod +x scripts/increment_version.sh'
            sh './scripts/increment_version.sh'

            // 3. Commit & push from the same directory
            withCredentials([usernamePassword(
              credentialsId: GIT_CREDENTIALS_ID,
              usernameVariable: 'GIT_USERNAME',
              passwordVariable: 'GIT_PASSWORD'
            )]) {
              sh '''
                git config user.name "$GIT_USERNAME"
                git config user.email "$USER_EMAIL"
                git add public1-frontend-helm-chart/templates/frontend-app.yaml \
                         public1-frontend-helm-chart/Chart.yaml
                git commit -m "chore: increment versions"
                git push https://$GIT_USERNAME:$GIT_PASSWORD@github.com/lavi324/Public1.git HEAD:main
              '''
            }
          }
        }
      }
    }

    stage('Build & Push Docker Image') {
      steps {
        container('gke-agent') {
          script {
            def newTag = sh(
              script: '''
                awk -F ':' '/image:/ {print $2}' public1-frontend-helm-chart/templates/frontend-app.yaml \
                  | tr -d ' '
              ''',
              returnStdout: true
            ).trim()
            withCredentials([usernamePassword(
              credentialsId: DOCKER_CREDENTIALS_ID,
              usernameVariable: 'DOCKER_USERNAME',
              passwordVariable: 'DOCKER_PASSWORD'
            )]) {
              sh """
                docker build -t ${DOCKER_REPO}:${newTag} frontend/
                echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                docker push ${DOCKER_REPO}:${newTag}
              """
            }
          }
        }
      }
    }

    stage('Package & Push Helm Chart') {
      steps {
        container('gke-agent') {
          script {
            def newTag = sh(
              script: '''
                awk -F ':' '/image:/ {print $2}' public1-frontend-helm-chart/templates/frontend-app.yaml \
                  | tr -d ' '
              ''',
              returnStdout: true
            ).trim()
            withCredentials([usernamePassword(
              credentialsId: DOCKER_CREDENTIALS_ID,
              usernameVariable: 'DOCKER_USERNAME',
              passwordVariable: 'DOCKER_PASSWORD'
            )]) {
              sh """
                helm package public1-frontend-helm-chart --version ${newTag} --app-version ${newTag}
                echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                helm push ${CHART_NAME}-${newTag}.tgz ${HELM_REPO}
              """
            }
          }
        }
      }
    }
  }
}
