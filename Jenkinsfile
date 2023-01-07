pipeline {
  agent any

  environment {
    NAME = "solar-system"
    VERSION = "${env.BUILD_ID}-${env.GIT_COMMIT}"
    IMAGE_REPO = "eromski"
    GIT_TOKEN = credentials('github-token')
    ARGOCD_TOKEN = credentials('argocd-token')
  }
  
  stages {
    stage('Unit Tests') {
      steps {
        echo 'Implement unit tests if applicable.'
        echo 'This stage is a sample placeholder'
      }
    }

    stage('Build Image') {
      steps {
            sh "docker build -t ${NAME} ."
            sh "docker tag ${NAME}:latest ${IMAGE_REPO}/${NAME}:${VERSION}"
        }
      }

    stage('Push Image') {
      steps {
        withDockerRegistry([credentialsId: "docker_hub", url: ""]) {
          sh 'docker push ${IMAGE_REPO}/${NAME}:${VERSION}'
        }
      }
    }

    stage('Clone/Pull Repo') {
      steps {
        script {
          if (fileExists('for_argocd_practice')) {

            echo 'Cloned repo already exists - Pulling latest changes'

            dir("for_argocd_practice") {
              sh 'git pull'
              sh 'git clean -xffd'
            }

          } else {
            echo 'Repo does not exists - Cloning the repo'
            sh 'git clone -b eroms-feature https://github.com/eromsubebe/for_argocd_practice.git'
            sh 'git clean -xffd'
          }
        }
      }
    }
    
    stage('Update Manifest') {

      steps {
        dir("for_argocd_practice/ArgoCD-Apps/solar-system") {
          sh "git config --global user.email 'ci@ci.com'"
          sh 'sed -i "s#siddharth67.*#${IMAGE_REPO}/${NAME}:${VERSION}#g" deployment.yaml'
          sh 'cat deployment.yaml'
        }
      }
    }

    stage('Commit & Push') {

      steps {
        dir("for_argocd_practice/ArgoCD-Apps/solar-system") {
          sh 'git remote set-url origin https://$GIT_TOKEN@github.com/eromsubebe/for_argocd_practice.git'
          sh 'git clean -xffd'
          sh 'git checkout eroms-feature'
          sh 'git add -A'
          sh 'git diff-index --quiet HEAD || git commit -am "Updated new image version for VERSION - $VERSION"'
          sh 'git push origin eroms-feature'
          sh 'git clean -xffd'
        }
      }
    }

    stage('Raise PR') {
      steps {
          sh 'gh auth login -h github.com  -p https --with-token < /var/lib/jenkins/token.txt'
          sh 'gh auth status'
          sh 'cd for_argocd_practice/ArgoCD-Apps/solar-system'
          sh 'git checkout eroms-feature'
          sh 'gh pr create -a @me --title "Updated Image Version - $VERSION" --body "Planets Updated in Solar System - $VERSION"  -B main'
      }
    }
  }
}