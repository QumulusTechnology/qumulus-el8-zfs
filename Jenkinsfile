pipeline {
  agent {
    kubernetes {
      yaml """
kind: Pod
spec:
  containers:
  - name: qumulus-centos-jnlp
    image: repo.qumulus.io/jenkins/jenkins-inbound-agent-centos-stream8:latest
    imagePullPolicy: IfNotPresent
    imagePullSecrets:
      - name: "qumulus-repo-docker-credentials"
    resources:
      limits:
        cpu: "2000m"
        memory: "2Gi"
      requests:
        cpu: "2000m"
        memory: "2Gi"
    command:
    - sleep
    args:
    - 9999999
"""
    }
  }
  stages {
    stage('Build RPMS') {
      steps {
        container(name: 'qumulus-centos-jnlp', shell: '/bin/bash') {
          withCredentials([usernamePassword(credentialsId: 'packagecloud-credentials', usernameVariable: 'PACKAGECLOUD_USERNAME', passwordVariable: 'PACKAGECLOUD_TOKEN')]) {
            sh '''
            VERSION=2.1.5
            git clone https://github.com/openzfs/zfs.git -b zfs-$VERSION
            cd zfs
            sh ./autogen.sh
            ./configure
            make -j 4 rpm
            for filename in *.rpm; do
              echo ${filename##*/}
              package_cloud yank dniasoff/zfs/el/8 ${filename##*/}  || true
            done
            package_cloud push --yes dniasoff/zfs/el/8 *.rpm
            '''
          }
        }
      }
    }
  }
}
