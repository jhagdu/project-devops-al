//Function to Run an Ansible Playbook
def runAnsiblePlaybook(path) {
    dir("gitpull/Infrastructure/K8s") {
        sh "sudo ansible-playbook $path"
    }
}

//Declarative Pipeline Code
pipeline {

    //By default, Use any Available Agent
    agent any

    //Specifying Triggers for Pipeline
    triggers {
        githubPush()
    }

    //Declaring Parameters for Pipeline
    parameters {
        booleanParam(name: 'PATCH_PRODUCTION', defaultValue: true, description: 'Does Production environment needs Patch?')
        booleanParam(name: 'PATCH_MONITOR', defaultValue: true, description: 'Does Monitor environment needs Patch?')
        booleanParam(name: 'PATCH_ANALYSIS', defaultValue: true, description: 'Does Analysis environment needs Patch?')
    }

    stages {
        //Downloads the code from GitHub then builds a Containerazied Application
        stage('Build') {
            steps {
                dir("gitpull") {
                    git branch: 'main', url: 'https://github.com/jhagdu/project-devops-al.git' 
                }
                dir("gitpull/Infrastructure/Docker") {
                    sh "sudo docker build -t jhagdu/web:${BUILD_NUMBER} ."
                    sh "sudo docker push jhagdu/web:${BUILD_NUMBER}"
                }
            }
            post {
                success {
                    slackSend(message: "Image jhagdu:/web:${BUILD_NUMBER} built Successfully!", color: "#008000", channel: "#build")
                }
                failure {
                    slackSend(message: "Image built Failed!", color: "#FF0000", channel: "#build")
                }
            }
        }

        //Deploy that containserized application on Testing Environment
        stage('Deploy: Test') {
            steps {
                dir("gitpull/Infrastructure/K8s/testing") {
                   sh "sudo sed -i s/IMAGE_VERSION/${BUILD_NUMBER}/g deploy-test.yaml"
                   sh "sudo kubectl apply -f deploy-test.yaml --kubeconfig /root/.kube/testk8s.kubeconfig"
                }
            }
            post {
                success {
                    slackSend (message: "Application Deployed to Testing Environment Successfully!", color: "#008000", channel: "#deploy")
                    slackSend (message: "Application Ready for Approval to Production Environment!", color: "#FFFF00", channel: "#test-approve")
                }
                failure {
                    slackSend (message: "Application Deployement to Testing Environment Failed!", color: "#FF0000", channel: "#deploy")
                }
            }
        }

        //Test and Approve the application and Notify the Developers
        stage('Approve and Notify') {
            agent {
                kubernetes {
                    label "jenslave-testapp"
                    yaml """
                        apiVersion: v1
                        kind: Pod
                        metadata:
                          namespace: testing
                        spec:
                          containers:
                          - name: testapp
                            image: centos
                            command: ["tail", "-f", "/dev/null"]
                         """
                }
            }
            steps {
                container('testapp') {
                    script {
                        for (i=0; i < 3; i++) {
                            test_rslt = sh(script: "curl -s -w '%{http_code}' -o /dev/null http://test-svc.testing.svc.cluster.local/indexerror.html", returnStdout: true)
                            if (test_rslt != '200') {
                                error("Application Unreachable!")
                            }
                        }
                    }
                }
            }
            post{
                always{
                    slackSend (message: "Application Tested!!", color: "#FFFF00", channel: "#test-approve")
                }
                success{
                    slackSend (message: "Ready for Production, Approved...!!", color: "#008000", channel: "#test-approve")
                }
                failure{
                    slackSend (message: "Testing Failed, Not Ready for Production, Please Review!!", color: "#FF0000", channel: "#test-approve")
                }
            }
        }

        //Deploy application to Production Environment
        stage('Deploy: Production') {
            when {
                expression { params.PATCH_PRODUCTION }
            }
            steps {                
                runAnsiblePlaybook("production/deploy-prod.yaml -e release_img=jhagdu/web:${BUILD_NUMBER}")
            }
            post{
                success{
                    slackSend (message: "Application Deployed on Production Environment!!", color: "#008000", channel: "#deploy")
                }
                failure{
                    slackSend (message: "Production Environment Deploymnet Failed", color: "#FF0000", channel: "#deploy")
                }
            }
        }

        //Deploy the Monitoring Environmnet i.e. Prometheus, Grafana, Metricbeat etc
        stage('Monitor') {
            when {
                expression { params.PATCH_MONITOR }
            }
            steps {          
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    runAnsiblePlaybook("monitor/deploy-monitor.yaml")
                }
            }
            post{
                success{
                    echo "Monitor Environment Deployed!"
                }
                failure{
                    slackSend (message: "Monitor Environment Deploymnet Failed", color: "#FF0000", channel: "#deploy")
                }
            }
        }

        //Deploy the Logging analysis Environment i.e. ELK Stack
        stage('Analysis') {
            when {
                expression { params.PATCH_ANALYSIS }
            }
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    runAnsiblePlaybook("analysis/deploy-log-analysis.yaml")
                }
            }
            post{
                success{
                    echo "Analysis Environment Deployed!"
                }
                failure{
                    slackSend (message: "Analysis Environment Deploymnet Failed", color: "#FF0000", channel: "#deploy")
                }
            }
        }

        //Take a Final Test on Application running in production Environment and Notify
        stage('Final Test and Notify') {
            steps {
                script {
                    for (j=0; j < 7; j++) {
                        appurl = sh(script: "sudo kubectl get svc web-svc -n production -o jsonpath={.status.loadBalancer.ingress[0].hostname}", returnStdout: true)
                        try {
                            final_test_rslt = sh(script: "curl -s -w '%{http_code}' -o /dev/null http://$appurl/index.html", returnStdout: true)
                        }
                        catch (exception) {
                            final_test_rslt = '400'
                        }
                        if (final_test_rslt == '200') {
                            break
                        } else {
                            sh "sleep 30"
                        }
                    }
                    if (final_test_rslt != '200') {
                        error("Application Unreachable!")
                    }
                }
            }
            post{
                always{
                    slackSend (message: "Application Final Test in Production!!", color: "#FFFF00", channel: "#test-approve")
                }
                success{
                    slackSend (message: "Successfully Deployed App on Production Environment!!", color: "#008000", channel: "#deploy")
                }
                failure{
                    slackSend (message: "Test Failed, Rolling Back Updates!!", color: "#FF0000", channel: "#test-approve")
                    sh "sudo kubectl rollout undo deployment web-dep -n production"
                }
            }
        }
    }
    post{
        success{
            script {
                app = sh(script: "sudo kubectl get svc web-svc -n production -o jsonpath={.status.loadBalancer.ingress[0].hostname}", returnStdout: true)
                prom = sh(script: "sudo kubectl get svc prom-svc -n monitor -o jsonpath={.status.loadBalancer.ingress[0].hostname}", returnStdout: true)
                graf = sh(script: "sudo kubectl get svc graf-svc -n monitor -o jsonpath={.status.loadBalancer.ingress[0].hostname}", returnStdout: true)
                kib = sh(script: "sudo kubectl get svc kib-svc -n analysis -o jsonpath={.status.loadBalancer.ingress[0].hostname}", returnStdout: true)
            }
            slackSend (message: "\nFinal Production URLs :-\n\n\tApplication: http://$app/index.html \n\n\tPrometheus: http://$prom:9090\n\n\tGrafana: http://$graf:3000\n\n\nKibana: http://$kib:5601\n", color: "#008000", channel: "#devops")
        }
        failure{
            slackSend (message: "There are Some Issues, Please Review!!", color: "#FF0000", channel: "#test-approve")
        }
    }
}
