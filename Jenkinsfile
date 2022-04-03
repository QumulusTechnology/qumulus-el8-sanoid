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
        cpu: "1000m"
        memory: "2Gi"
      requests:
        cpu: "1000m"
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
          VERSION=v2.1.0
          VERSION_SHORT=$(echo -n $VERSION | cut -c2- )
          mkdir -p /var/lib/jenkins/rpmbuild/SOURCES
          curl https://codeload.github.com/jimsalterjrs/sanoid/tar.gz/refs/tags/$VERSION -o /var/lib/jenkins/rpmbuild/SOURCES/sanoid-$VERSION_SHORT.tar.gz 
          tar -xf /var/lib/jenkins/rpmbuild/SOURCES/sanoid-$VERSION_SHORT.tar.gz 
          cd sanoid-$VERSION_SHORT
          rpmbuild -bb packages/rhel/sanoid.spec
          for filename in /var/lib/jenkins/rpmbuild/RPMS/noarch/*.rpm; do
            echo ${filename##*/} 
            package_cloud yank dniasoff/sanoid/el/8 ${filename##*/} || true
          done          
          package_cloud push --yes dniasoff/sanoid/el/8 /var/lib/jenkins/rpmbuild/RPMS/noarch/*.rpm    
          '''
        }
      }
    }
  }
}
