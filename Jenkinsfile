//所有的脚本命令都放在pipeline中
pipeline{
    //指定执行Jenkins的机器，any是任意，后期有多个jenkins可以指定
    agent any
    //全局变量
    environment{
        harborUser='admin'
        harborPasswd='Harbor12345'
        harborAddr='192.168.164.60:80'
        harborRepo='repo'
    }
    //步骤
    stages {
        stage('拉取git仓库代码') {
            steps {
                git branch: 'main', url: 'https://github.com/ibin888/prometheus-demo.git'
            }
        }
        stage('通过maven构建项目') {
            steps {
                sh '/var/jenkins_home/maven/bin/mvn clean package -DskipTests'
            }
        }

        stage('通过docker制作自定义镜像') {
            steps {
                sh '''mv target/*.jar docker/
                docker build -t ${JOB_NAME}:latest docker/'''
            }
        }
        stage('将自定义镜像推送到harbor') {
            steps {
                sh '''docker login -u ${harborUser} -p ${harborPasswd} ${harborAddr}
                    docker tag ${JOB_NAME}:latest ${harborAddr}/${harborRepo}/${JOB_NAME}:latest
                    docker push ${harborAddr}/${harborRepo}/${JOB_NAME}:latest'''
            }
        }
        stage('将yml传到k8s master上') {
            steps {
                sshPublisher(publishers: [sshPublisherDesc(configName: 'k8s', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: 'pipeline.yml')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            }
        }
        stage('在k8s master执行kubectl apply '){
            steps{
                sh '''ssh root@192.168.164.60 kubectl apply -f /usr/local/k8s/pipeline.yml
                        ssh root@192.168.164.60 kubectl rollout restart deploy prometheus-demo -n test'''
            }
        }
    }

}