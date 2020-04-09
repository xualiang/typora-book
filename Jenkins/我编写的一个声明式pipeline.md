将以下内容写入为Jenkinsfile文件，放置到项目代码工程的根目录下，然后在jenkins中配置一个构建任务即可。

```groovy
pipeline {
  agent {
    kubernetes {
      yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    some-label: some-label-value
spec:
  containers:
  - name: jnlp
    image: 10.10.71.100:5001/jenkins/jnlp-slave:latest
  - name: nodejs
    image: 10.10.71.100:5001/node:alpine
    command:
    - cat
    tty: true
  - name: maven
    image: 10.10.71.100:5001/maven:3.6.3-jdk-8
    command:
    - cat
    tty: true
    volumeMounts:
    - name: docker-sock
      mountPath: /var/run/docker.sock
    - name: docker
      mountPath: /usr/bin/docker
  - name: kubectl
    image: 10.10.71.100:5001/kubectl:1.16.2
    command:
    - cat
    tty: true
  volumes:
  - name: docker-sock
    hostPath:
      path: /var/run/docker.sock
  - name: docker
    hostPath:
      path: /usr/bin/docker
"""
    }
  }
  environment {
    NPM_REGISTRY_URL = 'http://10.10.70.220:8081/nexus/repository/npm-group/'
    DOCKER_REGISTRY_URL = '10.10.71.100:5001'
    //Use Pipeline Utility Steps plugin to read information from pom.xml into env variables
    IMAGE = readMavenPom().getArtifactId()
    VERSION = readMavenPom().getVersion()
  }
  stages {
    /*
    stage('Checkout Code') {
      steps {
        git branch: 'develop', credentialsId: '5c5...37d83', url: 'http://ip/YFCD/cale.git'
      }
    }
    */
    stage('Test Read Pom') {
      steps {
        println readMavenPom().getGroupId()+readMavenPom().getArtifactId()+'---'+readMavenPom().getVersion()
        sh '''
          echo $IMAGE
          echo $VERSION
        '''
        script {
          def pom = readMavenPom file: 'pom.xml'
          echo "group: ${pom.groupId}, artifactId: ${pom.artifactId}, version: ${pom.version}"
        }
      }
    }
    stage('NodeJS Build') {
      steps {
        container('nodejs') {
          sh '''
            cd cale.web/src/main/webview
            npm config set registry $NPM_REGISTRY_URL
            npm install
            npm run-script build
          '''
        }
      }
    }
    stage('Maven Package') {
      steps {
        container('maven') {
          sh 'mvn clean package -Dfile.encoding=UTF-8 -Dmaven.test.skip=true'
        }
      }
    }
    stage('Build Image') {
      steps {
        container('maven') {
          sh 'docker build -t $DOCKER_REGISTRY_URL/$IMAGE/$IMAGE:$VERSION .'
        }
      }
    }
    stage('Push Image') {
      steps {
        container('maven') {
          sh 'docker push $DOCKER_REGISTRY_URL/$IMAGE/$IMAGE:$VERSION'
        }
      }
    }
    stage('Run Kubectl') {
      steps {
        container('kubectl') {
          // need install jenkins plugin: kubernetes-cli
          withKubeConfig([credentialsId: 'ec68-...-71cb0', serverUrl: 'https://kubernetes.default']) {
            sh 'cd install/test && kubectl delete -k . ; kubectl apply -k .'
          }
        }
      }
    }
  }
}

```

