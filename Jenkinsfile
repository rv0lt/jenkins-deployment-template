pipeline {
    agent {
        kubernetes {
// Use of latests tag is not recomended
            yaml '''
apiVersion: v1
kind: Pod
metadata:
spec:
  containers:
  - name: curl # to make requests
    image: nixery.dev/shell/curl
    resources:
      requests:
        memory: "8Mi"
        cpu: "10m"
      limits:
        memory: "8Mi"
        cpu: "10m"
    command:
    - sleep
    args:
    - infinity
    securityContext:
      readOnlyRootFilesystem: true
  - name: linter
    image: stackrox/kube-linter:latest-alpine
    imagePullPolicy: Always # to ensure we will always use the lastest version of the tool
    resources:
      requests:
        memory: "32Mi"
        cpu: "100m"
      limits:
        memory: "32Mi"
        cpu: "100m"
    command:
    - sleep
    args:
    - infinity 
    securityContext:
      readOnlyRootFilesystem: true 
  - name: trivy # image scanner
    image: aquasec/trivy:latest-amd64
    imagePullPolicy: Always # to ensure we will always use the lastest version of the tool
    resources:
      requests:
        memory: "256Mi"
        cpu: "200m"
      limits:
        memory: "512Mi"
        cpu: "200m"
    command:
    - sleep
    args:
    - infinity
'''
        }
    }
    stages {
        stage('Syntax Scanning') {
            steps {
                container('linter') {
                    sh '/kube-linter lint files/'
                }
            }             
        }

        stage('image Scanning') {
            steps {
                container('trivy') {
                    sh 'for f in files/*.yaml; \
                        do \
                            cat $f | grep "image:" | sed "s/^.*: //" >> images.txt; \
                        done'

                    sh 'while read i; do \
                            trivy image --timeout 30m0s "$i" --ignore-unfixed --exit-code 1 --severity CRITICAL; done < images.txt'
                }
            }
        }
            
        stage('Deploy'){
            steps {
                container('curl'){
                    sh 'for f in files/*.yaml; \
                    do \
                        curl \
                            --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt \
                            --header "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
                            -X POST https://kubernetes.default.svc/api/v1/namespaces/devops-tools/pods \
                            -H "Content-Type: application/yaml" \
                            --data "$(cat $f)"; \
                    done'
                }
            }
        }
    }
}