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
      dir('python-flask-docker') {
        git 'https://github.com/nchaudh03/python-flask-docker'
      }
      container('kaniko') {
        stage('Build a Go project') {
         sh '/kaniko/executor -f `pwd`/python-flask-docker/Dockerfile -c `pwd`/python-flask-docker --insecure --skip-tls-verify --cache=true --destination=nchaudh03/python-flask-docker:v1.17'
        }
      }
    }
    stage('Update Flux Repo') {
      dir('flux_mlops') {
        git branch: 'main', credentialsId: 'PAT_GITHUB', url: 'https://github.com/nchaudh03/flux_mlops'
      }
      container('kaniko') {
        script{ 
          def yamlFile = './flux_mlops/apps/dev_mlops/python-flask-docker/python-flask-docker-values.yaml'
          def datas = readYaml(file: yamlFile)
          echo "Current version: ${datas.spec.chart.spec.version.toString()}"
          datas.spec.chart.spec.version = "v1.1.1"
          echo "Updated version: ${datas.spec.chart.spec.version.toString()}"
          writeYaml file: yamlFile, data: datas
          sh "git -C ./flux_mlops commit -am 'Update version to v1.1.1'"
          sh "git -C ./flux_mlops push"
        }
      }
    }

  }
}