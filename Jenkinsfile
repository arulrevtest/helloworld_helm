def label = "worker-${UUID.randomUUID().toString()}"

podTemplate(label: label, containers: [
  containerTemplate(name: 'maven', image: 'maven', ttyEnabled: true, command: 'cat'),
  containerTemplate(name: 'helm', image: 'lachlanevenson/k8s-helm:v2.5.1', ttyEnabled: true, command: 'cat'),
  containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'kubectl', image: 'gcr.io/cloud-builders/kubectl', command: 'cat', ttyEnabled: true)
],
volumes: [
  hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
]) {
  node(label) {
    def project = 'arulgkedemo'
    def  appName = 'helloworld'
    def  imageRepo = "arulkumar1967"
    def myRepo = checkout scm
    def gitCommit = myRepo.GIT_COMMIT
    def gitBranch = myRepo.GIT_BRANCH
    def shortGitCommit = "${gitCommit[0..10]}"
    def previousGitCommit = sh(script: "git rev-parse ${gitCommit}~", returnStdout: true)

    stage('Build') {
        container('maven') {
          sh """
            pwd
            echo "GIT_BRANCH=${gitBranch}" >> /etc/environment
            echo "GIT_COMMIT=${gitCommit}" >> /etc/environment
            mvn package
          """
        }
    }
    stage('Build and push image with Container Builder') {
        container('docker') {
          sh "docker build -t ${imageRepo}/${appName}:${env.BUILD_NUMBER} ."
          withDockerRegistry([ credentialsId: "2d3a7c2b-58a2-4740-9195-e9082d59468b", url: "" ]) {
            sh "docker push ${imageRepo}/${appName}:${env.BUILD_NUMBER}"
        }
      }
    }
    stage('Deploy docker image') {
        container(name: 'kubectl') {
            withCredentials([string(credentialsId: '16469a0d-1ac3-4404-9f66-f81893abe4cf', variable: 'K8S_TOKEN')]) {
            withEnv([
                // Ensure that kubectl is using our special robot deployer’s kubeconfig
                "KUBECONFIG=/home/[jenkins_user]/.kube/kubernetes_deployment_config",
                "KUBECTL=kubectl --token $K8S_TOKEN",
            ]) {
                 sh "kubectl get nodes"
            }
         }
        }
        container(name: 'helm') {
            def overrides = "image.repository=${imageRepo}/${appName},image.tag=${env.BUILD_NUMBER}"
            def releaseName = "helloworld"
            def chart_dir = "helloworld"

            sh "helm init"
            sh "helm version"
            sh "helm lint ${chart_dir}"
            sh "helm upgrade --install --wait ${releaseName} ${chart_dir} --set ${overrides} --namespace='dev'"
        }
    }
    /* stage('Deploy') {
        container('kubectl') {
          withCredentials([string(credentialsId: '16469a0d-1ac3-4404-9f66-f81893abe4cf', variable: 'K8S_TOKEN')]) {
            withEnv([
                // Ensure that kubectl is using our special robot deployer’s kubeconfig
                "KUBECONFIG=/home/[jenkins_user]/.kube/kubernetes_deployment_config",
                "KUBECTL=kubectl --token $K8S_TOKEN",
            ]) {
                // Execute the code that is now wrapped with the correct kubectl
                sh("sed -i.bak 's#arulkumar1967/rev_helloworld_gke_sb#${imageTag}#' ./k8s/app/*.yaml")
                sh("$KUBECTL --namespace=dev apply -f k8s/app")
                sh("echo http://`KUBECTL get service -o jsonpath='{.status.loadBalancer.ingress[0].ip}'`")
            }
         }

        }
    }
    */
  }
}
