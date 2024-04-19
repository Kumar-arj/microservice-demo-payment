/* groovylint-disable LineLength, NestedBlockDepth */
def label = 'shopagent'
podTemplate(label: label, yaml: '''
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: build
  annotations:
    sidecar.istio.io/inject: "false"
spec:
  containers:
  - name: build
    image: kumararj/eos-jenkins-agent-base:latest
    command:
    - cat
    tty: true
  - name: docker
    image: docker:latest
    securityContext:
      privileged: true
'''
) {
    node(label) {
        stage('CleanWorkspace') {
            cleanWs()
        }
        stage('Checkout SCM') {
            /* groovylint-disable-next-line LineLength */
            git credentialsId: 'git', url: 'https://github.com/Kumar-arj/microservice-demo-payment.git', branch: 'master'
        }

        stage('Docker Build') {
            container('docker') {
                stage('Build Image') {
                    /* groovylint-disable-next-line NestedBlockDepth */
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        /* groovylint-disable-next-line NoDef, VariableTypeRequired */
                        def customImage = docker.build("kumararj/sock-shop-micro-services-payment:${BUILD_NUMBER}")
                        customImage.push()
                        def finalImage = docker.build("kumararj/sock-shop-micro-services-payment:latest")
                        finalImage.push()
                    }
                }
            }
        }


        stage('Helm Chart') {
            container('build') {
                dir('charts') {
                    withCredentials([usernamePassword(credentialsId: 'nexuslogin', usernameVariable: 'username', passwordVariable: 'password')]) {
                        sh '/usr/local/bin/helm package micro-services-payment'
                        sh 'curl -v -u $username:$password --upload-file micro-services-payment-1.0.tgz http://nexus.k4m.in/repository/sock-shop-helm-local/'
                    }
                }
            }
        }
    }
}

