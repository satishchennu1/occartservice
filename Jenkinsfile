pipeline {
  agent {
    label 'maven'
  }
  stages {
    stage('Build App') {
      steps {
        sh "mvn clean package "
      }
    }
    stage('Integration Test') {
      steps {
        sh "mvn verify -s "
      }
    }
    stage('build') {
      openshift.withCluster() {
        openshift.withProject() {
           def bld = openshift.startBuild('cart')
           bld.untilEach {
              return it.object().status.phase == "Running"
             }
           bld.logs('-f')
          }  
        }
     }
    stage('Deploy') {
      openshift.withCluster() {
            openshift.withProject() {
              dc = openshift.selector("dc", "cart")
              dc.rollout().latest()
              timeout(10) {
              dc.rollout().status()
          }
       }
     }
    stage('Component Test') {
      steps {
        script {
          sh "curl -s -X POST http://cart:8080/api/cart/dummy/666/1"
          sh "curl -s http://cart:8080/api/cart/dummy | grep 'Dummy Product'"
        }
      }
    }
  }
}
