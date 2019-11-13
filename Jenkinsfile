podTemplate(label: 'golang-pod',  containers: [
    containerTemplate(
            name: 'golang',
            image: 'creack/golang-dcind:latest',
            ttyEnabled: true,
            command: 'cat',
            envVars: [envVar(key:'BUILD_ID',value:"${BUILD_ID}")]
    ),
    containerTemplate(
            name: 'jnlp',
            image: 'openshift/jenkins-slave-base-centos7:v3.9',
            args: '${computer.jnlpmac} ${computer.name}',
            command: '',
            envVars: [envVar(key: 'GO15VENDOREXPERIMENT', value: '1'),envVar(key:'BUILD_ID',value:"${BUILD_ID}")]
        )
  ]
  ,volumes: [
        /*persistentVolumeClaim(mountPath: '/home/jenkins', claimName: 'jenkins', readOnly: false),*/
        hostPathVolume(hostPath: '/minikube/jenkins/jenkins2', mountPath: '/home/jenkins'),
        hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock'),
        hostPathVolume(hostPath: '/tmp/', mountPath: '/tmp/'),
])
{
    node ('golang-pod') {
        def PROJECT_NAME = "${JOB_NAME}"
        def HUB_BASE_URL = "github.com"
        def HUB_BRANCH_NAME = "pipeline-test"
        def deploymentPath = "Deployment.yaml"
        container('golang') {
           checkout scm
            stage('构建镜像') {
              sh("docker build -t ${HUB_BASE_URL}/${HUB_BRANCH_NAME}/${PROJECT_NAME}:${BUILD_ID} .")
            }
        }
         container('jnlp') {
            stage('部署镜像' ) {
             /*  if (fileExists("${deploymentPath}")) {
                 sh("kubectl apply -f ${deploymentPath} --overwrite=true")
               } */
               sh("kubectl cluster-info")
               sh("kubectl set image deployment/${PROJECT_NAME} ${PROJECT_NAME}=${HUB_BASE_URL}/${HUB_BRANCH_NAME}/${PROJECT_NAME}:${BUILD_ID} -n kube-ops")
               sh("kubectl rollout status deployment/${PROJECT_NAME} -n kube-ops")
            }
         }
    }
}