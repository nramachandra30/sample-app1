node {
  def project = 'REPLACE_WITH_YOUR_PROJECT_ID'
  def appName = 'gceme'
  def feSvcName = "${appName}-frontend"
  def imageTag = "gcr.io/${project}/${appName}:${env.BRANCH_NAME}.${env.BUILD_NUMBER}"

  checkout scm

  stage 'Build image'
  sh("sudo docker build -t ${imageTag} .")

  stage 'Run Go tests'
  sh("sudo docker run ${imageTag} go test")

  stage 'Push image to registry'
  sh("sudo gcloud docker push ${imageTag}")

  stage "Deploy Application"
  switch (env.BRANCH_NAME) {
    // Roll out to canary environment
    case "canary":
        // Change deployed image in canary to the one we just built
        sh("sudo sed -i.bak 's#gcr.io/cloud-solutions-images/gceme:1.0.0#${imageTag}#' ./k8s/canary/*.yaml")
        break

    // Roll out to production
    case "master":
        // Change deployed image in canary to the one we just built
        sh("sudo sed -i.bak 's#gcr.io/cloud-solutions-images/gceme:1.0.0#${imageTag}#' ./k8s/production/*.yaml")
        break

    // Roll out a dev environment
    default:
        // Create namespace if it doesn't exist
        // Don't use public load balancing for development branches
        echo 'To access your environment run `kubectl proxy`'
        echo "Then access your service via http://localhost:8001/api/v1/proxy/namespaces/${env.BRANCH_NAME}/services/${feSvcName}:80/"
  }
}
