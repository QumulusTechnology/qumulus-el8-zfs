pipeline {
  agent {
    kubernetes {
      yaml """
kind: Pod
spec:
  containers:
  - name: qumulus-centos-jnlp
    image: dniasoff/jenkins-inbound-agent-centos-stream8:latest
    imagePullPolicy: IfNotPresent
    resources:
      limits:
        cpu: "4000m"
        memory: "2Gi"
      requests:
        cpu: "4000m"
        memory: "2Gi"
    command:
    - sleep
    args:
    - 9999999
"""
    }
  }
  environment {
      PACKAGECLOUD_TOKEN = credentials('PACKAGECLOUD_TOKEN')
  }
  stages {
    stage('Build RPMS') {
      steps {
        container(name: 'qumulus-centos-jnlp', shell: '/bin/bash') {
          sh '''
          VERSION=2.1.4
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
