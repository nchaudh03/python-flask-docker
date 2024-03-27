podTemplate(yaml: '''
    apiVersion: v1
    kind: Pod
    spec:
      containers:
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
    stage('Build Python Image') {
      git 'https://github.com/nchaudh03/python-flask-docker'
      container('kaniko') {
        stage('Build a Go project') {
         sh '/kaniko/executor -f `pwd`/Dockerfile -c `pwd` --insecure --skip-tls-verify --cache=true --destination=nchaudh03/python-flask-docker:v1.17'
        }
      }
    }
    stage('Update Flux Repo') {
      environment {
        GITHUB_TOKEN=credentials('PAT_GITHUB')
      }
      git 'https://github.com/nchaudh03/flux_mlops'
      container('kaniko') {
          echo "$GITHUB_TOKEN"
      }
    }

  }
}