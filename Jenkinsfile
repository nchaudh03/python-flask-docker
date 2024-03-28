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
         sh '/kaniko/executor -f `pwd`/python-flask-docker/Dockerfile -c `pwd`/python-flask-docker --insecure --skip-tls-verify --cache=true --destination=nchaudh03/python-flask-docker:v1.20'
        }
      }
    }
    /*
  stage('Update Flux Repo') {
      node {
          dir('flux_mlops') {
              // Checkout the Git repository
              git branch: 'main', credentialsId: 'PAT_GITHUB', url: 'https://github.com/nchaudh03/flux_mlops'

              // Update the YAML file
              def yamlFile = './apps/dev_mlops/python-flask-docker/python-flask-docker-values.yaml'
              def datas = readYaml(file: yamlFile)
              echo "Current version: ${datas.spec.chart.spec.version.toString()}"
              datas.spec.chart.spec.version = "v0.1.4"
              echo "Updated version: ${datas.spec.chart.spec.version.toString()}"
              writeYaml file: yamlFile, data: datas, overwrite: true

              withCredentials([usernamePassword(credentialsId: 'PAT_GITHUB', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                // Configure Git user
                sh 'git config --global user.email "nchaudh03@gmail.com"'
                sh 'git config --global user.name $USERNAME'
                sh 'git config --global url.https://$PASSWORD@github.com/.insteadOf https://github.com/'
                sh 'git config --list'
                sh 'git config --get remote.origin.url'

              // Check for changes using git status
              def changes = sh(returnStdout: true, script: 'git status --porcelain').trim()

              // Commit and push changes if there are any

              if (!changes.empty) {
                    sh 'echo $USERNAME'
                    sh 'git add .'
                    sh "git commit -m 'Update version to v0.1.4'"
                    sh 'git push origin main'
                } else {
                    echo "No changes to commit."
                }
            }
          }
      }
  }
*/
  }
}