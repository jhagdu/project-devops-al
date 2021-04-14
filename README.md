<img src="https://www.contrastsecurity.com/hs-fs/hubfs/images/DevOps%20Solutions/devops-old-way.gif?width=1322&name=devops-old-way.gif" height=400 width=600 alt="DevOps CI/CD" /> 

# DevOps Assembly Lines: End To End Automation

Automation is the ultimate need for DevOps practice and 'Automate everything' is the key principle of DevOps. Here is the Jenkins Pipeline to provide an End to End Automation i.e. Build, Test, Deploy, Monitor, Analyze and Auto Scale the Web Infrastructure

### Here the Pipeline has 7 Stages -  

**S.No.** | **Stage Name** | **Stage Description**
------------- | -------------------- | --------------------------------------
**1** | **Build** | Downloads the code from GitHub, Builds a Containerized Application, Publish it to Centralised Repo
**2** | **Deploy: Test** | Deploy that containerized application on Testing Environment
**3** | **Approve and Notify** | Test and Approve the application and Notify the Developers
**4** | **Deploy: Production** | Deploy application to Production Environment if Approved
**5** | **Monitor** | Deploy the Monitoring Environment i.e. Prometheus, Grafana, Metricbeat etc
**6** | **Analysis** | Deploy the Logging analysis Environment i.e. EFK Stack and EMK Stack
**7** | **Final Test and Notify** | Take a Final Test on Application running in production Environment and Notify

## Tools and Technologies Used

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/e/e9/Jenkins_logo.svg/1200px-Jenkins_logo.svg.png" height=100 width=100 alt="Jenkins" /> &emsp; <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/39/Kubernetes_logo_without_workmark.svg/126px-Kubernetes_logo_without_workmark.svg.png" height=100 width=100 alt="Kubernetes" /> &emsp; <img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcR1Wk-MBDhJQ8XwYKDTfbVpkHRuHD8dzXneRDDpRemYvkegSBYN0Ue2T4CzBJ_89nH22JU&usqp=CAU" height=100 width=100 alt="Ansible" /> &emsp; <img src="https://jackmckew.dev/img/Moby-logo.png" height=100 width=100 alt="Docker" /> <br>  

<img src="https://assets.zabbix.com/img/brands/elastic.svg" height=100 width=100 alt="Elasticsearch" /> &emsp; <img src="https://iconape.com/wp-content/png_logo_vector/elastic-kibana.png" height=100 width=100 alt="Kibana" /> &emsp; <img src="https://www.kindpng.com/picc/m/189-1890553_beats-filebeat-logo-hd-png-download.png" height=100 width=100 alt="Filebeat" /> &emsp; <img src="https://coralogix.com/wp-content/uploads/2018/12/metricbeat.png" height=100 width=100 alt="Metricbeat" /> <br>  

<img src="https://cdn.iconscout.com/icon/free/png-512/prometheus-282488.png" height=100 width=100 alt="Prometheus" /> &emsp; <img src="https://pbs.twimg.com/media/EYyhuorWAAEj9-D.png" height=100 width=100 alt="Grafana" /> &emsp; <img src="https://image.flaticon.com/icons/png/512/2111/2111615.png" height=100 width=100 alt="Slack" /> &emsp; <img src="https://pbs.twimg.com/profile_images/1052324554171764736/LQoMp4Xr.jpg" height=100 width=100 alt="AWS" />  

## Prerequisites   
- Jenkins should be installed and configured
- Kubernetes, Ansible, Docker should be installed and configured to work with Jenkins
- Jenkins CI should be configured in Slack

## How to Start 
- Create a Pipeline Job in Jenkins  
- Configure it with Jenkinsfile Code or directly with this Github Repository  
- For automatic triggering of Pipeline, Configure Github Webhooks

# Links

[Click here for Video](https://youtu.be/oZgJ6oz5bVw)

***Feel free to Contact if any issue!!***

<a href="https://www.linkedin.com/in/amanjhagrolia143" target="_blank"> <img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white" /> </a>
