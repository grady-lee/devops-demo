// 流水线定义(声明式/脚本式)，CI/CD流程，可以使用片段生成器生成
pipeline{
    // 集群中任意可用代理节点中执行
    agent any

    // 环境信息
    environment {
        WS=${WORKSPACE}
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
                sh 'docker build -t devops-java:v1 .'
            }
        }
        // 5. 部署
        stage('部署') {
            steps {
                sh 'pwd & ls -lah'
                echo "部署"
                sh 'docker run -d -p 8080:8080 --name devops-java devops-java:v1'
            }
        }
    }
}
