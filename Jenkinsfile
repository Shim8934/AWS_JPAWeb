podTemplate(yaml: '''
    apiVersion: v1
    kind: Pod
    spec:
      containers:
      - name: gradle
        image: gradle:6.3-jdk14
        command:
        - sleep
        args:
        - 10d
      - name: kaniko
        image: gcr.io/kaniko-project/executor:debug
        imagePullPolicy: Always
        command:
        - /busybox/cat
        tty: true
        volumeMounts:
          - name: docker-config
            mountPath: /kaniko/.docker
      - name: git
        image: alpine/git
        imagePullPolicy: Always
        command:
        - sleep
        args:
        - 10d
      restartPolicy: Never
      volumes:
        - name: docker-config
          projected:
            sources:
            configMap:
              name: docker-config
''') {
  node(POD_LABEL) {
    stage('Get a Gradle project') {
      git url: 'https://github.com/Shim8934/jpasample.git', branch: 'main'
      container('gradle') {
        stage('Build a Gradle project') {
          sh '''
          gradle build -x test
          echo `pwd`
          'printenv'
          '''
        }
      }
    }

    stage('Build Java Image') {
      container('kaniko') {
        stage('Build a ECR Image') {
          sh '''
            /kaniko/executor --context `pwd` --destination=963886026253.dkr.ecr.ap-northeast-2.amazonaws.com/team4/jpasampleshop:${BUILD_NUMBER}
          '''

        }
      }
    }

    stage ('Edit Manifest & Push') {
        container('git') {
           stage('Checkout external proj') {
                   steps {
                       git branch: 'main',
                           credentialsId: 'shim8934',
                           url: 'https://github.com/jooseop/goorm-kube1-team4.git'

                       sh "ls -lat"
                   }
               }

        }
    }
  }
}