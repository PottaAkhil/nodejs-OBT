podTemplate(yaml: '''
    apiVersion: v1
    kind: Pod
    spec:
      containers:
      - name: nodejs
        image: thetips4you/nodeapp:latest
        command:
        - sleep
        args:
        - 999999
      - name: kubectl
        image: bitnami/kubectl
        command:
        - sleep
        args:
        - 9999999
      - name: kaniko
        image: gcr.io/kaniko-project/executor:debug
        command:
        - sleep
        args:
        - 9999999
        volumeMounts:
        - name: kaniko-secret
          mountPath: /kaniko/.docker
      restartPolicy: Never
      volumes:
      - name: kaniko-secret
        secret:
            secretName: dockercred
            items:
            - key: .dockerconfigjson
              path: config.json
''') {
  node(POD_LABEL) {
    stage('Get a nodejs project') {
      git url: 'https://github.com/PottaAkhil/nodejs-demo.git', branch: 'master'
      container('nodejs') {
        stage('Build a nodejs project') {
          sh '''
            echo pwd
          '''
        }
      }
    }

    environment {
        registry = "957288871734.dkr.ecr.ap-southeast-1.amazonaws.com/images"
    }
    
    stage('Build nodejs Image') {
      container('kaniko') {
        stage('Build a Go project') {
          sh '''
            /kaniko/executor --context `pwd` --destination 957288871734.dkr.ecr.ap-southeast-1.amazonaws.com/images/hello-kaniko:1.1      
          '''
        }
      }
    }
    stage('Pushing to ECR') {
      steps{  
         script {
                sh 'aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 957288871734.dkr.ecr.ap-southeast-1.amazonaws.com'
                sh 'docker push 957288871734.dkr.ecr.ap-southeast-1.amazonaws.com/images:latest'
         }
        }
      }  
    stage('Deploy to k8s') {
      container('kubectl') {
        stage('Deploy to K8s') {
          sh '''
            echo pwd && \
            kubectl apply -f  deploymentservice.yaml 
          '''  
      }
    }
  }
}
}
