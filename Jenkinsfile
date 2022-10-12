// 流水线定义(声明式/脚本式)，CI/CD流程，可以使用片段生成器生成
pipeline{
    // 集群中任意可用代理节点中执行
    agent any

    // 环境信息
    environment {
        WS = "${WORKSPACE}"
        // 读取配置在凭据管理器的阿里云用户名和密码  _USR  _PSW
        ALIYUN_SECRTE=credentials("aliyun")
    }

    // 流水线阶段
    stages {
       stage('环境检查') {
            steps {
                // /var/jenkins_home/workspace/devops-demo
                sh 'pwd & ls -alh' // 查看工作目录
                sh 'printenv' // 查看默认配置
                sh 'java -version'
                sh 'git --version'
            }
       }
       // 1. 代码编译
        stage('编译') {
            // 使用自定义代理，需要安装docker pipeline插件,
            //  ubuntu需要添加当前用户到docker组,cd /var/run/,sudo chmod 777 docker.sock
            agent {
               docker {
                    // 用完容器关闭删除，镜像存在
                    image 'maven:3-alpine'
                    // 因此将配置挂载出去，不会影响到到以后的maven操作
                    // 相当于docker run -v xxx:xx  ,也可以使用volume方式
                    args '-v /var/jenkins_home/config/maven/.m2:/root/.m2'
               }
            }
            steps {
                sh 'mvn -v'
                // /var/jenkins_home/workspace/devops-demo@2
                sh 'pwd & ls -alh'
                echo '${WS}'
                // 打包并指定配置文件
                sh 'cd ${WS} && mvn clean package -s "/var/jenkins_home/config/maven/settings.xml" -Dmaven.test.skip=true'
            }
        }
        // 2. 代码测试
        stage('测试') {
            steps {
                // /var/jenkins_home/workspace/devops-demo
                sh 'pwd & ls -alh'
                echo "测试"
            }
        }
        // 3. 打包
        stage('打包') {
            steps {
                sh 'pwd & ls -alh'
                echo "打包"
            }
        }
        // 4. 构建镜像
        stage('构建镜像') {
            steps {
                sh 'pwd & ls -lah'
                echo "构建镜像"
                sh 'docker build -t devops-java .'
            }
        }
        // 5. 推送镜像到阿里云仓库(在Jenkins凭据管理中配置用户名和密码)
        stage('推送镜像') {
            input {
                message "确认推送到远程仓库吗？"
                ok "确认"
                // submitter "alice,bob"  用户
                parameters {
                    string(name: 'APP_VER', defaultValue: 'v1.0', description: '稳定版本号')
                    choice choices: ['bj01', 'sh02', 'tj03'], description: '部署的区域', name: 'DEPOLY_WHERE'
                }
            }
            steps{
                script{
                    def where = ${DEPOLY_WHERE}

                    if(where == "bj01"){
                        sh "echo 部署到bj01区了"
                    }else if(where == "sh02"){
                        sh "echo 部署到sh02区了"
                    }else{
                        sh "echo 部署到tj03区了"

                        // 推送到阿里云仓库
                        // docker login -u ${ALIYUN_SECRTE_USR} -p ${ALIYUN_SECRTE_PSW} registry.cn-hangzhou.aliyuncs.com
                        withCredentials([usernamePassword(credentialsId: 'aliyun',
                                                    passwordVariable: 'ali_pwd',
                                                    usernameVariable: 'ali_user')]) {
                             docker login -u ${ali_user} -p ${ali_pwd} registry.cn-hangzhou.aliyuncs.com
                         }

                        docker tag devops-demo registry.cn-hangzhou.aliyuncs.com/grady/devops-demo:${APP_VER}
                        docker push registry.cn-hangzhou.aliyuncs.com/grady/devops-demo:${APP_VER}


                        //ssh 秘钥文件配置到 jenkins 全局秘钥中
                        /* withCredentials(ssh){
                          //ansible 没有
                          sh "ssh root@xxxx  "
                          //不应该的操作。
                          sh "远程操作其他机器。。。。"

                          //k8s集群
                          //动态切换k8s集群

                        } */
                    }

                }
            }
        }

        // 5. 部署
        stage('部署') {
            steps {
                sh 'pwd & ls -lah'
                echo "部署"
                sh 'docker rm -f devops-java'
                sh 'docker run -d -p 8888:8080 --name devops devops-java'
            }

            //后置执行
            //             post {
            //               failure {
            //                 // One or more steps need to be included within each condition's block.
            //                 echo "炸了.. ."
            //               }
            //
            //               success {
            //                 echo "成了..."
            //               }
            //             }
                    }
        }
        // 6. 报告
        stage('报告'){
            steps {
                echo '发送邮件/短信等，查看模板'
            }
        }

        // 7.部署生产环境
        stage('生产环境'){
            steps{
                // 手动输入版本【参数化构建】,推荐生成器

//                 input {
//                     message "需要部署到生产环境吗?"
//                     ok "是的，赶紧部署"
// //                     submitter "alice,bob"
//                     parameters {
//                         //手动传入的参数
//                         string(name: 'APP_VERSION', defaultValue: 'v1.0', description: '请指定生产版本号')
//                     }
//                 }
                sh "echo 发布版本咯......"
                // 版本的保存。代码的保存。镜像的保存。存到远程仓库
            }
        }
    }
}
