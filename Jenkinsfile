pipeline {
  agent {
    kubernetes {
      cloud 'k8s-api-cs-software.sunvalley.com.cn'
      nodeSelector 'app=jenkins'
      idleMinutes 5
      yaml """
        apiVersion: v1
        kind: Pod
        metadata:
          labels:
            app: jenkins-jnlp
        spec:
          containers:
          - name: jnlp
            image: harbor.sunvalley.com.cn/library/jenkins-jnlp-slave:3.35-5
          - name: maven
            image: cs-harbor.sunvalley.com.cn/library/maven-base-image:3.0.5
            command:
            - cat
            tty: true
            volumeMounts:
            - mountPath: /data/m2_dev/
              name: m2-dev
          - name: docker-ci
            image: harbor.sunvalley.com.cn/library/docker-ci:v19.03.5
            command:
            - cat
            tty: true
            volumeMounts:
            - mountPath: /var/run/docker.sock
              name: docker-sock
            - mountPath: /data/m2_dev/
              name: m2-dev
          - name: kubectl-tools
            image: harbor.sunvalley.com.cn/library/kubectl-tools:1.16.3
            command:
            - cat
            tty: true
          volumes:
          - name: docker-sock
            hostPath:
              path: /var/run/docker.sock
          - name: m2-dev
            hostPath:
              path: /data/m2_dev/
          serviceAccountName: admin-sample-user
        """
    }
  }
  stages {
    stage('Run maven') {
      steps {
        container('maven') {
          sh 'mvn -B -f pom.xml -s /data/m2_dev/settings.xml clean package -Dmaven.test.skip=true -pl com.sunvalley:mwp-message-center -am -Pdev1 -e -U'
        }
        container('docker-ci') {
          dir(path: "mwp-message-center") {
            script {
              docker.withRegistry('https://cs-harbor.sunvalley.com.cn', 'harbor' ){
                  def customImage = docker.build("cs-harbor.sunvalley.com.cn/internal/${env.JOB_NAME}:${env.GIT_COMMIT}")
                  customImage.push()
              }
            }
          }
        }
        container('kubectl-tools') {
          sh "sed -i 's/<PROJECT_NAME>/${env.JOB_NAME}/' deployment.yaml"
          sh "sed -i 's/<BUILD_TAG>/${env.GIT_COMMIT}/' deployment.yaml"
          sh 'kubectl apply -f deployment.yaml'
        }
      }
    }
  }
}
