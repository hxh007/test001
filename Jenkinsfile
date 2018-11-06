pipeline {
    // 在任何可用的代理上执行pipeline
    agent none
    parameters {
        // git代码路径
        string(name:'repoUrl', defaultValue:'https://github.com/hxh007/test001.git', description:'git代码路径')
        string(name:'repoBranch', defaultValue:'master', description:'git分支名称')
        choice(name: 'server', choices:'192.168.181.132,22,root,hxh999999', description: '测试服务器列表选择(IP,JettyPort,Name,Passwd)')
        // 测试服务器的dubbo服务端口
        string(name:'dubboPort', defaultValue: '31100', description: '测试服务器的dubbo服务端口')
        // 单元测试代码覆盖率要求，各项目视要求调整参数
        string(name:'lineCoverage', defaultValue: '20', description: '单元测试代码覆盖率要求(%)，小于此值pipeline将会失败！')
        // 若勾选在pipelie完成后会邮件通知测试人员进行验收
        booleanParam(name: 'isCommitQA',description: '是否邮件通知测试人员进行人工验收',defaultValue: false )
    }
    // 常量参数
    environment { 
        // git 服务全系统只读账号cred_id
        CRED_ID = 'github_test'
    }
    options {
        // 保持构建的最大个数
        buildDiscarder(logRotator(numToKeepStr: '10')) 
    }
    // 定期检查代码更新
    triggers {
        pollSCM('H 4 * * 1-5')
    }
    // pipeline运行结果通知给触发者
    post {
        success {
            script {
                wrap($class: 'BuildUser') {
                    mail to: "${BUILD_USER_EMAIL}",
                    subject: "PipeLine '${JOB_NAME}' (${BUILD_NUMBER}) result",
                    body: "${BUILD_USER}'s pineline '${JOB_NAME}' (${BUILD_NUMBER}) run success\n请及时前往${env.BUILD_URL}进行查看"
                }
            }
        }
        failure{
            script { 
                wrap([$class: 'BuildUser']) {
                mail to: "${BUILD_USER_EMAIL }",
                subject: "PineLine '${JOB_NAME}' (${BUILD_NUMBER}) result",
                body: "${BUILD_USER}'s pineline  '${JOB_NAME}' (${BUILD_NUMBER}) run failure\n请及时前往${env.BUILD_URL}进行查看"
                }
            }

        }
        unstable{
            script { 
                wrap([$class: 'BuildUser']) {
                mail to: "${BUILD_USER_EMAIL }",
                subject: "PineLine '${JOB_NAME}' (${BUILD_NUMBER})结果",
                body: "${BUILD_USER}'s pineline '${JOB_NAME}' (${BUILD_NUMBER}) run unstable\n请及时前往${env.BUILD_URL}进行查看"
                }
            }
        }
    }
    // pipeline的各个阶段场景
    stages {
        stage('代码获取') {
            steps {
            //根据param.server分割获取参数,包括IP,jettyPort,username,password
            script {
                def split=params.server.split(",")
                serverIP=split[0]
                jettyPort=split[1]
                serverName=split[2]
                serverPasswd=split[3]
            }
              echo "starting fetchCode from ${params.repoUrl}......"
              // 获取源代码
             git branch: 'master', credentialsId: 'github_test', url: 'https://github.com/hxh007/test001.git'
            }
        }
        // stage('单元测试') {
        //     steps {
        //       echo "starting unitTest......"
        //       //注入jacoco插件配置,clean test执行单元测试代码. All tests should pass.
        //       sh "mvn org.jacoco:jacoco-maven-plugin:prepare-agent -f ${params.pomPath} clean test -Dautoconfig.skip=true -Dmaven.test.skip=false -Dmaven.test.failure.ignore=true"
        //       junit '**/target/surefire-reports/*.xml'
        //       //配置单元测试覆盖率要求，未达到要求pipeline将会fail,code coverage.LineCoverage>20%.
        //       jacoco changeBuildStatus: true, maximumLineCoverage:"${params.lineCoverage}"
        //     }
        // }

        // stage('部署测试环境') { 
        //     steps {
        //         echo "starting deploy to ${serverIP}......"
        //         //编译和打包
        //         sh "mvn  -f ${params.pomPath} clean package -Dautoconfig.skip=true -Dmaven.test.skip=true"
        //         archiveArtifacts warLocation
        //         script {
        //             wrap([$class: 'BuildUser']) {
        //             //发布war包到指定服务器，虚拟机文件目录通过shell脚本初始化建立，所以目录是固定的
        //             sh "sshpass -p ${serverPasswd} scp ${params.warLocation} ${serverName}@${serverIP}:htdocs/war"
        //             //这里增加了一个小功能，在服务器上记录了基本部署信息，方便多人使用一套环境时问题排查，storge in {WORKSPACE}/deploy.log  & remoteServer:htdocs/war
        //             Date date = new Date()
        //             def deploylog="${date.toString()},${BUILD_USER} use pipeline  '${JOB_NAME}(${BUILD_NUMBER})' deploy branch ${params.repoBranch} to server ${serverIP}"
        //             println deploylog
        //             sh "echo ${deploylog} >>${WORKSPACE}/deploy.log"
        //             sh "sshpass -p ${serverPasswd} scp ${WORKSPACE}/deploy.log ${serverName}@${serverIP}:htdocs/war"
        //             //jetty restart，重启jetty
        //             sh "sshpass -p ${serverPasswd} ssh ${serverName}@${serverIP} 'bin/jettyrestart.sh' "
        //             }
        //         }
        //     }
        // }

        stage('通知人工验收'){
            steps{
                script{
                    wrap([$class: 'BuildUser']) {
                    if(params.isCommitQA==false){
                        echo "不需要通知测试人员人工验收"
                    }else{
                        //邮件通知测试人员人工验收
                         mail to: "${QA_EMAIL}",
                         subject: "PineLine '${JOB_NAME}' (${BUILD_NUMBER})人工验收通知",
                         body: "${BUILD_USER}提交的PineLine '${JOB_NAME}' (${BUILD_NUMBER})进入人工验收环节\n请及时前往${env.BUILD_URL}进行测试验收"
                    }

                    }
                }
            }
        }

    }
}
