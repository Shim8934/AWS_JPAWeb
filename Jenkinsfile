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
        //container('git') {
          // stage('Edit Manifest & Push') {
               git branch: 'main', credentialsId: 'shim8934', url: 'https://github.com/jooseop/goorm-kube1-team4.git'
               withCredentials([usernamePassword(credentialsId: 'shim8934', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'shim8934')]) {
                   sh "echo 1st Line"
                   def pass = [GIT_PASSWORD];

                   sh "echo $pass"

                   sh "echo 2nd Line"
                   def encodedPassword = URLEncoder.encode('[GIT_PASSWORD]','UTF-8')
                   def gitUrl = "https://shim8934:$encodedPassword@github.com/jooseop/goorm-kube1-team4.git"


                   dir ("user-api") {
                    sh("""
                       set +x
                       export GIT_SSH_COMMAND="ssh -oStrictHostKeyChecking=no"
                       git config --global user.email "shim8934@gmail.com"
                       git config --global user.name "ShimKiYoung"

                       git pull origin main:refs/remotes/origin/main

                       sed -i 's/jpasampleshop:.*/jpasampleshop:${BUILD_NUMBER}/' manifest/jpasampleshop/base/jpasampleshop.yaml
                       git add manifest/jpasampleshop/base/jpasampleshop.yaml
                       git commit -m "Update Spring Service Tag Image By CD Automate"
                       git push --set-upstream origin main ${gitUrl}
                    """)
                   }
               } // to use ${GIT_PASSWORD} -> 시스템관리 전역 변수로 깃허브 비밀번호를 GIT_PASSWORD 키값으로 등록 후 사용
         //  } //
       // }
    } // Edit manifest stage End
  }
}