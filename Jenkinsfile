pipeline {
  agent { label 'linux' } 

  environment {
    token = credentials('DIGITALOCEAN_TOKEN')
  }
  stages {
    stage('install kubectl') {
      steps {
        sh '''echo $token | curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
           curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
           echo "$(<kubectl.sha256) kubectl" | sha256sum --check
           sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
           kubectl version --client'''
      }
    }


    stage('get nodes') {
      steps {
        sh'echo $token | kubectl get nodes'
      }
    }
    stage('apply ingress') {
      steps {
        sh'kubectl apply -f ingress.yaml'
      }
    }

    stage('apply deployment') {
      steps {
        sh'kubectl apply -f deployment.yaml'
      }
    }
    stage('apply service') {
      steps {
        sh'kubectl apply -f service.yaml'
      }
    }

    stage('install cert-manager') {
      steps {
        sh '''helm repo add jetstack https://charts.jetstack.io
         helm repo update
         kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.5.4/cert-manager.crds.yaml
         helm install \\
         cert-manager jetstack/cert-manager \\
         --namespace cert-manager \\
         --create-namespace \\
         --version v1.5.4 \\
         # --set installCRDs=true
         helm install \\
         cert-manager jetstack/cert-manager \\
         --namespace cert-manager \\
         --create-namespace \\
         --version v1.5.4 \\
         --set prometheus.enabled=false \\  # Example: disabling prometheus using a Helm parameter
         --set webhook.timeoutSeconds=4   # Example: changing the wehbook timeout using a Helm parameter'''
      }
    }
  stage('apply issures') {
    steps {
      sh'kubectl apply -f issures.yaml'
      }
    }
  
  }
}