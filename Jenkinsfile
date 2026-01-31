pipeline {
    agent any

    // tools {
    //     nodejs 'node18'
    // }

    environment {
    //     AWS_REGION        = "us-east-1"
    //     ECR_REPO          = "734468801857.dkr.ecr.us-east-1.amazonaws.com/reddit-app"
    //     IMAGE_TAG         = "${BUILD_NUMBER}"
         SONAR_PROJECT_KEY = "reddit-app"
    //     K8S_NAMESPACE     = "reddit"
    //     DEPLOYMENT_NAME   = "reddit-app"
    //     CONTAINER_NAME    = "reddit-app"
    }
     
    stages {
        


        // stage('Checkout Code') {
        //     steps {
        //         checkout scm
        //     }
        // }

        // stage('SonarQube Scan') {
        //     steps {
        //         withSonarQubeEnv('sonarqube') {
        //             sh '''
        //             sonar-scanner \
        //               -Dsonar.projectKey=$SONAR_PROJECT_KEY \
        //               -Dsonar.sources=src \
        //               -Dsonar.exclusions=node_modules/**,.next/**,coverage/**
        //             '''
        //         }
        //     }
        // }

//         stage('Quality Gate') {
//             steps {
//                 timeout(time: 5, unit: 'MINUTES') {
//                     waitForQualityGate abortPipeline: true
//                 }
//             }
//         }

  stage('OWASP Dependency Check') {
  steps {
    withEnv(["PATH+DC=/usr/local/bin"]) {
      sh '''
        mkdir -p reports
        mkdir -p .dependency-check

        dependency-check \
          --project reddit-app \
          --scan . \
          --format XML \
          --out reports \
          --failOnCVSS 7

        ls -la reports
      '''
    }
  }
}







        // stage('Docker Build (Multi-Stage)') {
        //     steps {
        //         sh '''
        //         docker build -t $ECR_REPO:$IMAGE_TAG .
        //         '''
        //     }
        // }

        // stage('Trivy Image Scan') {
        //     steps {
        //         sh '''
        //         trivy image \
        //           --exit-code 1 \
        //           --severity HIGH,CRITICAL \
        //           --format json \
        //           --output trivy-image-report.json \
        //           $ECR_REPO:$IMAGE_TAG
        //         '''
        //     }
        // }

        // stage('Login to ECR') {
        //     steps {
        //         sh '''
        //         aws ecr get-login-password --region $AWS_REGION \
        //         | docker login --username AWS --password-stdin $ECR_REPO
        //         '''
        //     }
        // }

    //     stage('Push Image to ECR') {
    //         steps {
    //             sh '''
    //             docker push $ECR_REPO:$IMAGE_TAG
    //             '''
    //         }
    //     }

    //     stage('Update Deployment Image') {
    //         steps {
    //             sh '''
    //             sed -i "s|IMAGE_PLACEHOLDER|$ECR_REPO:$IMAGE_TAG|g" k8s/deployment.yaml
    //             '''
    //         }
    //     }

    //     stage('Commit & Push Deployment') {
    //         steps {
    //             sh '''
    //             git config user.email "jenkins@yourdomain.com"
    //             git config user.name "Jenkins CI"

    //             git add k8s/deployment.yaml

    //             git diff --cached --quiet || git commit -m "Update image to $IMAGE_TAG"

    //             git push origin HEAD:${GIT_BRANCH}
    //             '''
    //         }
    //     }

    //     stage('Deploy to EKS') {
    //         steps {
    //             sh '''
    //             kubectl apply -f k8s/deployment.yaml
    //             kubectl rollout status deployment/$DEPLOYMENT_NAME -n $K8S_NAMESPACE
    //             '''
    //         }
    //     }
    // }
    }
    post {
        success {
            echo "reddit-app deployed successfully to namespace reddit"
        }
        failure {
            echo "Pipeline failed"
        }
        
    
}
}
