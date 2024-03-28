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
      dir('flux_mlops') {
        git branch: 'main', credentialsId: 'PAT_GITHUB', url: 'https://github.com/nchaudh03/flux_mlops'
      }
      container('kaniko') {
        script {
                    def output = sh(returnStdout: true, script: 'ls ./flux_mlops')
                    echo "Output: ${output}"
                }
        script{ 
          datas = readYaml (file: './flux_mlops/apps/dev_mlops/python-flask-docker/python-flask-docker-values.yaml')
          echo datas.spec.chart.spec.version.toString()
        }
      }
    }

  }
}