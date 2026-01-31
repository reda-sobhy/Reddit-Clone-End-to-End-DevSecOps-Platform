pipeline {
    agent any

    

    environment {
        AWS_REGION        = "us-east-1"
        EKS_CLUSTER_NAME = "my-eks"
          ECR_REPO          = "734468801857.dkr.ecr.us-east-1.amazonaws.com/redit"
        IMAGE_TAG         = "${BUILD_NUMBER}"
         SONAR_PROJECT_KEY = "reddit"
        K8S_NAMESPACE     = "reddit"
          GIT_REPO = "github.com/reda-sobhy/Reddit-Clone-End-to-End-DevSecOps-Platform.git"
        
    }
     
    stages {
        


        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                    sonar-scanner \
                      -Dsonar.projectKey=$SONAR_PROJECT_KEY \
                      -Dsonar.sources=src \
                      -Dsonar.exclusions=node_modules/**,.next/**,coverage/**
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

stage('OWASP Dependency Check') {
  steps {
    sh '''
      # Create cache folder inside workspace
      mkdir -p $WORKSPACE/.dependency-check

      # Run OWASP Dependency Check in Docker container
      docker run --rm \
        -v $WORKSPACE:/src \
        -v $WORKSPACE:/report \
        -v /home/abdul/nvd:/root/.dependency-check \
        owasp/dependency-check \
        --project reddit-app \
        --scan /src \
        --format XML \
        --out /report \
        --failOnCVSS 7

      ls -la /home/abdul/nvd
    '''
  }
}






        stage('Docker Build (Multi-Stage)') {
            steps {
                sh '''
                docker build -t $ECR_REPO:$IMAGE_TAG .
                '''
            }
        }

        stage('Trivy Image Scan') {
  steps {
    sh '''
    trivy image \
      --exit-code 0 \
      --severity HIGH,CRITICAL \
      --format json \
      --output trivy-image-report.json \
      $ECR_REPO:$IMAGE_TAG
    '''
    archiveArtifacts artifacts: 'trivy-image-report.json', fingerprint: true
  }
}


        stage('Login to ECR') {
  steps {
    withCredentials([[
      $class: 'AmazonWebServicesCredentialsBinding',
      credentialsId: 'aws-cred'
    ]]) {
      sh '''
        aws ecr get-login-password --region $AWS_REGION \
        | docker login --username AWS --password-stdin $ECR_REPO
      '''
    }
  }
}

stage('Push Image to ECR') {
  steps {
    sh '''
      docker push $ECR_REPO:$IMAGE_TAG
    '''
  }
}

stage('Update Deployment Image') {
  steps {
    sh '''
      sed -i "s|IMAGE_PLACEHOLDER|$ECR_REPO:$IMAGE_TAG|g" kubernetes/deployment.yaml
    '''
  }
}

        stage('Commit & Push Deployment') {
  steps {
    withCredentials([usernamePassword(credentialsId: 'git-cred', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
      sh '''
        git config user.email "jenkins@gmail.com"
        git config user.name "Jenkins CI"

        git add kubernetes/deployment.yaml

        git diff --cached --quiet || git commit -m "Update image to $IMAGE_TAG"

        git push https://$GIT_USER:$GIT_PASS@$GIT_REPO HEAD

      '''
    }
  }
}


       stage('Apply to EKS') {
  steps {
    withCredentials([[
      $class: 'AmazonWebServicesCredentialsBinding',
      credentialsId: 'aws-cred'
    ]]) {
      sh '''
        aws eks update-kubeconfig --region $AWS_REGION --name $EKS_CLUSTER_NAME

        kubectl apply -f kubernetes/namespace.yaml
        kubectl apply -n $K8S_NAMESPACE -f kubernetes/
 
      '''
    }
  }
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
