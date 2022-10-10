// 流水线定义(声明式/脚本式)，CI/CD流程，可以使用片段生成器生成
pipeline{
    // 集群中任意可用代理节点中执行
    agent any

    // 环境信息
    environment {
        name="123"
    }

    // 流水线阶段
    stages {
       stage('环境检查') {
            steps {
                sh 'pwd & ls -alh' // 查看工作目录
                sh 'printenv' // 查看默认配置
                sh 'java -version'
                sh 'git --version'
            }
       }
       // 1. 代码编译
        stage('编译') {
            // 使用自定义代理，需要安装docker pipeline插件,
            // ubuntu需要添加当前用户到docker组,cd /var/run/,sudo chmod 777 docker.sock
            agent {
               docker {
                    image 'maven:3-alpine'
                    // 相当于docker run -v 
                    args '-v /var/jenkins_home/config/maven/.m2:/root/.m2'

               }
            }
            steps {
                sh 'mvn -v'
                // 打包并指定配置文件
                sh 'mvn clean package -s "/var/jenkins_home/config/maven/settings.xml" -Dmaven.test.skip=true'
            }
        }
        // 2. 代码测试
        stage('测试') {
            steps {
                echo "测试"
                echo "$HOME"
            }
        }
        // 3. 打包
        stage('打包') {
            steps {
                echo "打包"
            }
        }
        // 4. 部署
        stage('部署') {
            steps {
                echo "部署"
                sh 'pwd & ls -lah'
            }
        }
    }
}
